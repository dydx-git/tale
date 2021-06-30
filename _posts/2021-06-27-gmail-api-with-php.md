---
layout: post
title: "Using Gmail API with PHP 7: Send or read emails, get attachments and more"
categories: Tutorial
author: "Muhammad Taha"
---

Google's documentation for Gmail API Client Library for PHP is very basic. It has a quickstart guide which is somewhat useful but it gets you started with code which works on CLI during connection.
There's obviously a better way to do this so here you go:

# Connecting to Gmail
Start with installing the Gmail API through Composer:
`composer require google/apiclient:^2.0`

Then create a new PHP file say, `GmailConnector.php`:
```php
<?php

require __DIR__ . '/vendor/autoload.php';

class GmailConnector
{
    public bool $isConnected;
    protected string $credentials;

    public function __construct()
    {
        $this->credentials = __DIR__ . '/credentials.json';
        $this->isConnected = false;
        $this->client = $this->createClient();
    }

    public function getClient()
    {
        return $this->client;
    }

    public function getCredentials()
    {
        return $this->credentials;
    }

    public function getUnauthenticatedData()
    {
        $authUrl = $this->client->createAuthUrl();

        echo "<a href='{$authUrl}'>Click here to link your account</a>";
    }

    private function credentialsInBrowser()
    {
        return isset($_GET['code']) ? true : false;
    }

    private function createClient()
    {
        $client = new Google_Client();
        $client->setApplicationName('Gmail API PHP Quickstart');
        $client->setScopes(Google_Service_Gmail::MAIL_GOOGLE_COM);
        $client->setAuthConfig($this->credentials);
        $client->setAccessType('offline');
        $client->setPrompt('select_account consent');

        // Load previously authorized token from a file, if it exists.
        // The file token.json stores the user's access and refresh tokens, and is
        // created automatically when the authorization flow completes for the first
        // time.
        $tokenPath = __DIR__ . "/tokens/token.json";
        if (file_exists($tokenPath)) {
            $accessToken = json_decode(file_get_contents($tokenPath), true);
            $client->setAccessToken($accessToken);
        }

        // If there is no previous token or it's expired.
        if ($client->isAccessTokenExpired()) {
            // Refresh the token if possible, else fetch a new one.
            if ($client->getRefreshToken()) {
                $client->fetchAccessTokenWithRefreshToken($client->getRefreshToken());
            } elseif ($this->credentialsInBrowser()) {
                $authCode = $_GET['code'];

                // Exchange authorization code for an access token.
                $accessToken = $client->fetchAccessTokenWithAuthCode($authCode);
                $client->setAccessToken($accessToken);

                // Check to see if there was an error.
                if (array_key_exists('error', $accessToken)) {
                    throw new Exception(join(', ', $accessToken));
                }
            } else {
                $this->isConnected = false;

                return $client;
            }
            // Save the token to a file.
            if (!file_exists(dirname($tokenPath))) {
                mkdir(dirname($tokenPath), 0700, true);
            }
            file_put_contents($tokenPath, json_encode($client->getAccessToken()));
        }
        // echo 'not expired';

        $this->isConnected = true;

        return $client;
    }
}
```
In order to get this file to work, you need your Google Cloud credentials for the Gmail API. 
* Follow [this guide to create a project](https://developers.google.com/workspace/guides/create-project) in Google Cloud for your Gmail.
* Then follow [this to create and download your credentials](https://developers.google.com/workspace/guides/create-credentials#web). 
* Now make it accessible by modifying `$this->credentials` variable in the constructor.

# Authenticating through OAuth 2.0
Create another file in the same directory and name it something like `GmailAuthenticator.php`:
```php
<?php

use enums\Company;

require __DIR__ . '/vendor/autoload.php';

class GmailAuthenticator
{
    private GmailConnector $connector;

    public function __construct()
    {
        $this->connector = new GmailConnector();

        if (!$this->connector->isConnected) {
            $this->connector->getUnauthenticatedData();
        }
    }

    public function getClientHelper()
    {
        $gmail = new GmailClientHelper($this->connector->getClient());
        return $gmail;
    }
}
```

Next, we need the `GmailClientHelper.php` file:
```php
<?php

use Soundasleep\Html2Text;

class GmailClientHelper
{
    protected Google_Client $client;
    protected Google_Service_Gmail $service;

    public function __construct(Google_Client $client)
    {
        $this->client = $client;
        $this->service = new Google_Service_Gmail($this->client);
        $this->ner = new NamedEntityRecognizer();
        $this->currentAuthenticatedUserEmail = $this->readUserEmailInfo()['emailAddress'];
    }

    public function readUserEmailInfo()
    {
        $user = 'me';
        return $this->service->users->getProfile($user);
    }

    public function readUnreadEmails(string $timeRangeFromNow = '-1 week')
    {
        $baseTime = new DateTime('+1 day');
        $endTime = new DateTime('+1 day');
        $endTime = $endTime->modify($timeRangeFromNow)->format('Y/n/j');
        $messageIdsThreads = $this->listMessages(
            'me',
            "is:unread is:inbox after:{$endTime} before:{$baseTime->format('Y/n/j')}"
        );

        $messageIds = array_keys($messageIdsThreads);

        foreach ($messageIds as $messageId) {
            // code...
        }
        return $this->getMultipleMessagesDetails($messageIds);
    }

    public function readAllInAMonth(string $monthFromNow = '-1', bool $getThreads = false)
    {
        $dateRange = $this->getStartAndEndDateForMonth($monthFromNow);
        //listIds is a child method
        $messageIdsThreads = $this->listIds(
            'me',
            "is:inbox after:{$dateRange['baseTime']->format('Y/n/j')} AND before:{$dateRange['endTime']->format('Y/n/j')}"
        );

        return $messageIdsThreads;
    }

    protected function getStartAndEndDateForMonth(string $monthFromNow = '-1')
    {
        $now = new DateTime();
        if ($now->format('d') > 5) {
            $monthFromNow = 'this';
        }
        $baseTime = new DateTime("first day of {$monthFromNow} month");
        $endTime = new DateTime("last day of {$monthFromNow} month");

        return ['baseTime' => $baseTime, 'endTime' => $endTime];
    }

    public function getMultipleMessagesDetails(array $messageIds)
    {
        $emails = [];
        foreach ($messageIds as $id) {
            $emails[] = $this->getMessageDetails($id, true);
        }

        return $emails;
    }

    public function getMessageDetails(string $id, bool $convertHtmlToText = false)
    {
        $msg = $this->getMessage($id);
        $headers = $msg->getPayload()->getHeaders();
        $currentSender = $this->getCurrentSender($headers)['email'];
        libxml_use_internal_errors(true);
        $msgBody = $convertHtmlToText ? Html2Text::convert($this->msg_body($msg)) : $this->msg_body($msg);
        $entitiesToRemove = $this->ner->recognize($msgBody);

        foreach ($entitiesToRemove as $entity) {
            $msgBody = str_ireplace($entity, '', $msgBody);
        }
        $emails['id'] = $id;
        $emails['body'] = $msgBody;
        $emails['email'] = $currentSender;
        $emails['date'] = $this->getHeader($headers, 'Date');
        $subject = $this->getHeader($headers, 'Subject');
        $emails['designName'][] = $this->speller->check(ucwords($subject));
        $attachment = $this->getAttachments($id, $msg->getPayload()->parts);
        $attachment = count($attachment) > 0 ? $attachment[0] : $attachment;
        if (array_key_exists('filename', $attachment)) {
            $emails['designName'][] = ucwords($this->speller->removeExtensions($attachment['filename']));
        }

        return $emails;
    }

    public function isEmailSentThisWeek(string $emailToCheck)
    {
        $baseTime = new DateTime('+1 day');
        $endTime = new DateTime('+1 day');
        $endTime = $endTime->modify('-1 week')->format('Y/n/j');

        $emailIdsFromClient = $this->listMessages(
            'me',
            "is:sent {$emailToCheck} after:{$endTime} before:{$baseTime->format('Y/n/j')}"
        );

        return count($emailIdsFromClient) > 0 ? true : false;
    }

    private function createMessage(array $sender, array $to, $subject, $messageText, $attachment = null)
    {
        $message = new Google_Service_Gmail_Message();

        $boundary = uniqid(rand(), true);

        $senderEmail = $sender['email'];
        $senderName = $sender['name'];

        if ($senderEmail !== $this->currentAuthenticatedUserEmail) {
            echo 'user sending email is not the same as the one signed in';
            return false;
        }

        $recipientEmail = $to['email'];
        $recipientName = $to['name'];

        $rawMessageString = "From: {$senderName} <{$senderEmail}>\r\n";

        if (is_array($recipientEmail)) {
            $rawMessageString .= "To: {$recipientName} <{$recipientEmail[0]}>\r\n";
            array_shift($recipientEmail);
            foreach ($recipientEmail as $email) {
                $rawMessageString .= "CC: {$recipientName} <{$email}>\r\n";
            }
        } else {
            $rawMessageString .= "To: {$recipientName} <{$recipientEmail}>\r\n";
        }

        $rawMessageString .= 'Subject: =?utf-8?B?' . base64_encode($subject) . "?=\r\n";
        $rawMessageString .= "MIME-Version: 1.0\r\n";

        if ($attachment) {
            $fileName = $attachment['name'];
            $mimeType = 'application/octet-stream';
            $rawMessageString .= 'Content-type: Multipart/Mixed; boundary="' . $boundary . '"' . "\r\n";
            $rawMessageString .= "\r\n--{$boundary}\r\n";
            $rawMessageString .= 'Content-Type: ' . $mimeType . '; name="' . $fileName . '";' . "\r\n";
            $rawMessageString .= 'Content-ID: <' . $senderEmail . '>' . "\r\n";
            $rawMessageString .= 'Content-Description: ' . $fileName . ';' . "\r\n";
            $rawMessageString .= 'Content-Disposition: attachment; filename="' . $fileName . '";\r\n';
            $rawMessageString .= 'Content-Transfer-Encoding: base64' . "\r\n\r\n";
            $rawMessageString .= $attachment['content'] . "\r\n";
            $rawMessageString .= "--{$boundary}\r\n";
        }

        $rawMessageString .= "Content-Type: text/html; charset=utf-8\r\n";
        $rawMessageString .= 'Content-Transfer-Encoding: quoted-printable' . "\r\n\r\n";
        $rawMessageString .= "{$messageText}\r\n";

        $rawMessage = strtr(base64_encode($rawMessageString), ['+' => '-', '/' => '_']);

        $message->setRaw($rawMessage);
        return $message;
    }

    public function sendEmail(array $from, array $to, string $subject, string $messageText, $attachment = null)
    {
        $message = $this->createMessage($from, $to, $subject, $messageText, $attachment);

        try {
            $message = $this->service->users_messages->send('me', $message);
            echo 'Message sent';
            return $message;
        } catch (Exception $e) {
            echo 'An error occurred: ' . $e->getMessage();
        }
    }

    public function readLabels()
    {
        $user = 'me';
        $results = $this->service->users_labels->listUsersLabels($user);

        if (0 != count($results->getLabels())) {
            echo "Labels:\n";
            foreach ($results->getLabels() as $label) {
                printf("- %s\n", $label->getName());
            }
        }
    }

    protected function listMessages(string $userId, string $query, bool $listThreads = false)
    {
        $pageToken = null;
        $messages = [];
        $opt_param = [];
        do {
            try {
                if ($pageToken) {
                    $opt_param['pageToken'] = $pageToken;
                }
                if ($query) {
                    $opt_param['q'] = $query;
                }
                $opt_param['maxResults'] = 1000;
                $response = $listThreads ?
                    $this->service->users_threads->listUsersThreads($userId, $opt_param) :
                    $this->service->users_messages->listUsersMessages($userId, $opt_param);

                if ($listThreads) {
                    if ($response->getThreads()) {
                        $messages = $response->getThreads();
                        $pageToken = $response->getNextPageToken();
                    }

                    continue;
                }

                if ($response->getMessages()) {
                    $messages = $response->getMessages();
                    $pageToken = $response->getNextPageToken();
                }
            } catch (Exception $e) {
                echo 'An error occurred: ' . $e->getMessage();
            }
        } while ($pageToken);

        return $listThreads ? array_column($messages, 'id') : array_column($messages, 'threadId', 'id');
    }

    private function getCurrentSender($header)
    {
        $currentSender = $this->getHeader($header, 'From');
        if (preg_match_all('/<(.*?)>/', $currentSender, $matches)) {
            return [
                'name' => strtok($currentSender, '<'),
                'email' => reset($matches[1])
            ];
        }

        return ['name' => null, 'email' => null];
    }

    private function getHeader($headers, $name)
    {
        foreach ($headers as $header) {
            if ($header['name'] == $name) {
                return $header['value'];
            }
        }
    }

    private function getMessage($messageId, string $userId = null)
    {
        try {
            $userId = $userId ?? 'me';

            return $this->service->users_messages->get($userId, $messageId, ['fields' => 'payload']);
            // echo 'Message with ID: '.$message->getId().' retrieved.';
        } catch (Exception $e) {
            echo 'An error occurred: ' . $e->getMessage();
        }
    }

    public function getThread(string $threadId, string $userId = null)
    {
        try {
            $userId = $userId ?? 'me';

            return $this->service->users_threads->get($userId, $threadId);
            // echo 'Message with ID: '.$message->getId().' retrieved.';
        } catch (Exception $e) {
            echo 'An error occurred: ' . $e->getMessage();
        }
    }

    private function decodeBody($encoded)
    {
        $sanitizedData = strtr($encoded, '-_', '+/');

        return base64_decode($sanitizedData);
    }

    private function msg_body_recursive($part)
    {
        if ('text/html' == $part->mimeType) {
            return ['html' => $this->decodeBody($part->body->data)];
        }
        if ('text/plain' == $part->mimeType) {
            return ['plain' => $this->decodeBody($part->body->data)];
        }
        if ($part->parts) {
            $return = [];
            foreach ($part->parts as $sub_part) {
                $result = $this->msg_body_recursive($sub_part);
                $return = array_merge($return, $result);
                if (array_key_exists('html', $return)) {
                    break;
                }
            }

            return $return;
        }

        return [];
    }

    private function getAttachments($messageId, $parts)
    {
        $attachments = [];
        foreach ($parts as $part) {
            if (!empty($part->body->attachmentId)) {
                // $attachment = $this->service->users_messages_attachments->get('me', $messageId, $part->body->attachmentId);
                $attachments[] = [
                    'filename' => $part->filename,
                    'mimeType' => $part->mimeType,
                    // 'data' => strtr($attachment->data, '-_', '+/'),
                ];
            } elseif (!empty($part->parts)) {
                $attachments = array_merge($attachments, $this->getAttachments($messageId, $part->parts));
            }
        }

        return $attachments;
    }

    private function msg_body($msg)
    {
        $body = $this->msg_body_recursive($msg->payload);

        return array_key_exists('html', $body) ? $body['html'] : $body['plain'];
    }
}
```
> Also, note that you might need to install the following dependency: https://github.com/soundasleep/html2text

The `GmailClientHelper` class has quite a few helper methods that let you read labels, list unread emails, get attachments, send emails etc. So explore the methods a little bit as there's no documentation but the method names are descriptive enough. You can use these classes by creating a new file `test.php` like so:
```php
<?php 

$connector = new GmailAuthenticator();
$gmail = $connector->getClientHelper();
var_dump($gmail->readLabels());
```

This should print out the labels that you have on your Gmail account after you've signed in.
And there you have it! Using Gmail API with PHP.
