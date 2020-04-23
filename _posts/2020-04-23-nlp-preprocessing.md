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
