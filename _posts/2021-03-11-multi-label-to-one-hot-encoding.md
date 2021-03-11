---
layout: post
title: "Converting Multi-label Data to One Hot Encoding"
categories: Code
author: "Muhammad Taha"
---

I recently needed to train a model using multi-label data but of course for any machine learning task, data is of the utmost importance. So, I wanted to transform a column with multiple labels into one hot encoded data.
Following are the dependencies: 
* Python 3
* Numpy
* Scikit-learn

Here's the code to do that:
{% highlight python %}
import pandas as pd
import numpy as np
from sklearn.preprocessing import MultiLabelBinarizer


df = pd.read_csv('doc-tc/abstracts_data.csv')
df = df.dropna()
df['label'] = df['label'].apply(lambda x: set(str(x).strip().split(' ')))
one_hot = MultiLabelBinarizer()
y = pd.DataFrame(one_hot.fit_transform(df['label']), columns=one_hot.classes_)
df = df.join(y)
df.drop('label', inplace=True, axis=1)
df.to_csv('doc-tc/encoded_data.csv', index=False)
{% endhighlight %}
