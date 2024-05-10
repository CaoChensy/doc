> 词袋模型

```python
#词袋模型
from sklearn.feature_extraction.text import CountVectorizer
vectorizer = CountVectorizer()
corpus = [
    'This is a very good class',
    'students are very very very good',
    'This is the third sentence',
    'Is this the last doc',
    'PS teacher Mei is very very handsome'
]

X = vectorizer.fit_transform(corpus)

vectorizer.get_feature_names()

X.toarray()

vec = CountVectorizer(ngram_range=(1,3))
X_ngram = vec.fit_transform(corpus)
vec.get_feature_names()

X_ngram.toarray()
```

> TF-IDF

`TF-IDF`是`Term Frequency - Inverse Document Frequency`的缩写，
即“词频-逆文本频率”。它由两部分组成，`TF`和`IDF`。

- `TF`: 词频，
- `IDF`: 反应一个词在所有文本中出现的频率，如果一个词在很多的文本中出现，
那么它的`IDF`值应该低，而反过来如果一个词在比较少的文本中出现，
那么它的IDF值应该高。比如一些专业的名词如“Machine Learning”。
这样的词IDF值应该高。一个极端的情况，如果一个词在所有的文本中都出现，
那么它的IDF值应该为0。

一个词`x`的`IDF`的基本公式如下：
$$
IDF(x)=log\frac{N+1}{N(x)+1} + 1
$$
其中，N代表语料库中文本的总数，而N(x)代表语料库中包含词x的文本总数。

TF-IDF值：$TF−IDF(x)=TF(x)∗IDF(x)$
，其中$TF(x)$指词$x$在当前文本中的词频。

```python
from sklearn.feature_extraction.text import TfidfVectorizer
tfidf_vec = TfidfVectorizer()
tfidf_X = tfidf_vec.fit_transform(corpus)
tfidf_vec.get_feature_names()

tfidf_X.toarray()
```

