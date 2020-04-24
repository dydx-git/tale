---
layout: post
title: "A Refresher: NLP - Text Preprocessing and Wrangling"
categories: Code
author: "Muhammad Taha"
---

This post aims to show some basic text preprocessing and wrangling techniques

## What is preprocessing and wrangling in NLP?
### Data Preprocessing
Most real-world data especially natural textual data is very inconsistent, unstructured and noisy. By preproceesing, we remove certain aspects of data that we don't need.
Like for example, if we're scrapping the web for text analysis we might want to remove html tags, css etc. 

### Data Wrangling
Data wrangling converts preprocessed data in a form that is compatible with the model currently in development. It changes the data to be more easily consumable for the ML models.

## Basic Techniques

### Removing Tags
As mentioned before, html tags and other data specific to web rendering do not add much value our text analysis. Beautiful soup is a popular library that we can use to remove these like so:
```python
def strip_html_tags(text):
    soup = BeautifulSoup(text, "html.parser")
    [s.extract() for s in soup(['iframe', 'script'])]
    stripped_text = soup.get_text()
    stripped_text = re.sub(r'[\r|\n|\r\n]+', '\n', stripped_text)
    return stripped_text
```

### Removing accented texts
Some words particularly belonging languages like French, Spanish etc. contain accented letters like â, ñ, ü which for the most part adds confusion and uncertainty so we might
want to remove these like so:
```python
def remove_accented_chars(text):
    text = unicodedata.normalize('NFKD', text).encode('ascii', 'ignore').
    decode('utf-8', 'ignore')
    return text
```

### Expanding Contractions
Contractions like *don't, *can't are commonly used in informal speech but they're not so great when processing text. We can fix these simply by utilizing PyContractions like so:
```python
def expand_contractions(text):
    # GoogleNews-vectors-negative300.bin should be on disk and in correct path
    cont = Contractions('GoogleNews-vectors-negative300.bin')
    return cont.expand_texts(text)
```

### Removing Special Characters
We can remove emojis and other special unicode characters is they do not serve our purpose like so:
```python
def remove_special_characters(text, remove_digits=False):
    pattern = r'[^a-zA-Z0-9\s]' if not remove_digits else r'[^a-zA-Z\s]'
    text = re.sub(pattern, ", text)
    return text
```

### Spelling Correction
Wrong spellings and typo are always unwelcomed. Here's a simple method to correct them:
```python
from textblob import Word
def correct_spell(text):
    words = nltk.word_tokenize(text)
    # you can also try Word.spellcheck() for spell suggestions
    corrected_words = [Word(word).correct() for word in words]
    return ' '.join(corrected_words)
```

### Lemmatizing Text
Words are often affixed in languages to provide better understanding and grammar. However, to a machine learning model, *wisely* and *wise* may seem like two different words so we can get the lemma which is a lexicographically correct root word. Here's how:
```python
def lemmatize_text(text):
    text = nlp(text)
    text = ' '.join([
        word.lemma_ if word.lemma_ != '-PRON-' else word.text for word in text
    ])
    return text
```

### Removing Stopwords
And finally, words like *of*, *the*, *an* etc can occupy the majority of word count in a document so we can remove them like so:
```python
stopword_list = nltk.corpus.stopwords.words('english')
# We want the negation to stay in the document
stopword_list.remove('no')
stopword_list.remove('not')
def remove_stopwords(text, is_lower_case=False):
    words = nltk.word_tokenize(text)
    words = [word.strip() for word in words]
    if is_lower_case:
        filtered_words = [word for word in words if word not in stopword_list]
    else:
        filtered_words = [
            word for word in words if word.lower() not in stopword_list
        ]
    return ' '.join(filtered_words)
```

There are possibly more ways to wrangle text but these should be plentiful for the basic ones.
