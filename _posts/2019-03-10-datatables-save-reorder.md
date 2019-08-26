---
layout: post
title: "Datatables: Saving Reordered Data into the Database"
categories: Code
author: "Muhammad Taha"
---

I've been working on improving a DBA that I built almost an year ago. And one requirement that came up was to be able to reorder the data entereed into the table. I've been using 
[Datatables](https://datatables.net/), which I should mention is a very useful tool and is customizable to a great extent. You can style it, extend it, control the rendering or make it work however you like.
So, i looked up its extension catalog for something that could help me reorder the data and I found an [extension](https://datatables.net/extensions/rowreorder/) that does exactly what I needed.

You can find the initialisation for it at the source website so I'm not going to get into that. Once however, you've enabled it you'll see that you can now drag the rows however you want. 
But upon refreshing the table, you'll notice that the data rows have gone back to their previous state. This happened because we never saved the reordered information anywhere. Also, if your row jumps back to its original place immediately after reordering it then check if you have 
`ordering:` set to `false` because that's probably what's causing that behaviour and setting it to `true` should fix it up.
Anyway, what you need to do is to figure out a way to save that data to the existing table. We could use the `id` of the row and replace it everytime a row is re-ordered but the issue with this approach is that you don't want to be messing with the `id` column ever.
So, lets create a new column named `serial` in our database which copies the `id` of the newly created data so we could use the `serial` column for manipulation rather than the `id`. For that, lets create a new trigger in mysql with this query:
{% highlight sql %}
CREATE TRIGGER `this_trigger_name` BEFORE INSERT ON `table_name`
      FOR EACH ROW SET NEW.`serial_column` = (
            SELECT AUTO_INCREMENT 
            FROM information_schema.TABLES 
            WHERE TABLE_SCHEMA = DATABASE() 
            AND TABLE_NAME = 'design'
      )
{% endhighlight %}
This will copy the data from `id` to `serial` on every new data creation. Now, to get the re-ordered data from the table: 
1. Create a variable to hold the datatables row data like so: `var rowData = oTable.row( diff[i].node ).data();`
2. Get the original `id` of the row you're reordering like so: `rowData['id']` and the new `serial` like this: `diff[i].newData`
3. Send both this data variables to a php script via AJAX (jQuery)
4. Keep all of this in a for loop

Of course, you'll be putting this in a function and you'll want to call that function everytime a row is reordered. we can use events for that. Lets use the [row-reordered](https://datatables.net/extensions/rowreorder/examples/initialisation/events.html) event like so:
{% highlight js %}
yourDataTable.on('row-reordered', function ( e, diff, edit) {
    for (var i = 0; i < diff.length; i++) {
        var rowData = yourDataTable.row( diff[i].node ).data();
        $.post("/link-to-script.php", {
            oldData: rowData['id'],
            newData: diff[i].newData
        }, function(data, status) {
            //empty
        });
    }
});
{% endhighlight %}
There's a big problem with this, however. You'll be calling the server everytime you displace the row. So for example, if you displace the row 10 places down, the AJAX call will be placed 10 times. 
And this seems wasteful, not to mention it'll needlessly tax your server. This happens even though the `row-reordered` event docs contrast it to `row-reorder` event as only being fired when you have completed the reordering process. I don't know maybe I don't understand the documentation correctly or whatever but anyways here's the modified js code along with the steps:
1. Create a global variable (I know, but I couldn't find any other way) to hold the reorder information. Call it `serialsDict`.
2. Add this code that utilizes the `draw` event (which is fired every time the row is done reordering among other times):
{% highlight js %}
yourDataTable.on('row-reorder', function ( e, diff, edit) {
    for (var i = 0; i < diff.length; i++) {
        var rowData = yourDataTable.row( diff[i].node ).data();
        serialsDict.push({
            oldData: rowData['id'],
            newData: diff[i].newData
        });
    }
});

yourDataTable.on('draw', function () {
    if (serialsDict.length) {
        $.post("/link-to-script.php", {
        serialsDict: serialsDict
        }, function(data, status) {
            serialsDict = [];
        })
    } ;
});
{% endhighlight %}
We're using the if statement to combat unnecessary AJAX calls because the `draw` event is pretty common and we don't want to call the order update script unnecessarily.

Now, on to the PHP script to handle the AJAX call:
{% highlight js %}
$object = new CRUD($pdo, 'design', 'id');
            foreach ($_POST['serialsDict'] as $rowId) {
                $object->update(['id' => $rowId['oldData'],
                'serial' => $rowId['newData']]);
          }
{% endhighlight %}
The PHP code is using custom ORM that provides basic SQL injection protection and a few OOP-styled database calls but essentially I'm running this sql script behind this update function:
{% highlight js %}
$query = 'UPDATE `serial` = 'newData' WHERE `id` = 'oldData''
{% endhighlight %}

And there you have it! I'm pretty sure the code can be made better but this solution works well and isn't very ugly either so I guess it suits me.

Hope this helps :)
