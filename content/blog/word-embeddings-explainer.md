---
title: "Word Embeddings Explainer"
date: 2018-04-30T15:22:07-04:00
draft: false
---
<meta name="twitter:card" content="summary">
<meta name="twitter:site" content="@pmbaumgartner">
<meta name="twitter:creator" content="@pmbaumgartner">
<meta name="twitter:title" content="Word Embeddings Explainer">
<meta name="twitter:description" content="What are word embeddings and what can we do with them?">
<meta name="twitter:image" content="https://github.com/pmbaumgartner/personal-hugo-site/raw/master/static/images/wordville.png">

### What are word embeddings?

Imagine if every word had an address you could look up in an address book. Now also imagine if words that shared meaning lived in the same neighborhood. This is a simplified metaphor for word embeddings.

For a visual example, here are simplified word embeddings for common 4- and 5-letter english words.

![Wordville](/images/wordville.png)

I've drawn 3 *neighborhoods* over this embedding to illustrate the semantic groupings.

### What are they good for?

A word embedding model transforms words into numbers so that we can do interesting measurements with them. Below are some applications of word embeddings.

#### Word Similarity

You can ask a word who its neighbors are. With enough words, this similarity query can work like a synonym finder. In the 2D grid representation above, each immediate neighbor is equally distant from some central term, but with the full dimension word embeddings you can get a more nuanced similarity score for each word. 


#### Sentence Similarity

Sentences are made of words, so we can calculate a semantic distance between sentences using embeddings. Here's 3 sample sentences, with words we have addresses for in bold.

1. There's a **tall** **patch** of **grass**.
2. The **root** of the **tree** is in **soil**.
3. Let's **chat** about **pizza** and **cake**!

It's clear to us, as humans, that the first two sentences share meaning and the 3rd sentence is different than them. A common way of calculating the similarity of two pieces of text is to count how many words overlap — however,  none of the sentences above have any meaningful words in common (once we remove common *[stopwords](https://en.wikipedia.org/wiki/Stop_words)* like "the"). When measuring similarity in this way, all three of our example sentences are equally similar, since they all don't share any words. With word embeddings, we can use semantic distance for this case.

##### Measuring semantic similarity with embeddings

With word embeddings there are a few ways to measure the similarity of two sentences.

1. **Average Embeddings** - Find the *average* location (centroid) of the words in both sentences. Then measure the distance between these two *average* locations.
2. **Word Movers Distance** - Find the total cost of moving from all the words in one sentence to all the words in another sentence. 
3. **Soft Cosine Similarity** - Combine the magic of linear algebra and the location of your word embeddings to create a new space to measure similarity. (There's a lot behind this one, but you should know that its fast and it [works well](http://www.redalyc.org/html/615/61532067007/).)

### What's the difference between this simplification and actual word embeddings?

Word embeddings are usually more than 2 dimensions (commonly 50, 100, 200, and 300). For this example, I used the 300D vectors trained on Google News. That high dimensionality is helpful for solving large computational problems, and each dimension captures some unique semantic structure of the words given text the model was trained on.

 To get to the 2D Map of Wordville, I did the following:

1. I used [this](https://norvig.com/ngrams/) helpful list of common english words as a source. I looped through that list in order, picking out words that were 4- or 5-letters long (so their label would fit in the map) and were semantically similar (measured with a word embedding model) to some basic seed words (car, dog, food) until I had 625 words.
2. Projected the vectors of these words down to two dimensions with a technique called [Universal Manifold Approximation and Projection](https://github.com/lmcinnes/umap) (UMAP).
3. Used an optimization algorithm to map each point in 2D to a grid of 2D points, following [this notebook](https://github.com/kylemcdonald/CloudToGrid/blob/master/CloudToGrid.ipynb).

### How can I make my own word embeddings?

The [gensim](https://github.com/RaRe-Technologies/gensim) library in python allows you to create your own word embeddings. It requires your input text to be tokenized.

Here's a basic example -- we will create a model from 10000 copies of 4 tokenized sentences. (Note: You will not get good results with this input data.)

```python
import gensim 

tokenized_sentences = [
    ['I', 'love', 'my', 'cat', '.'],
    ['I', 'love', 'my', 'dog', '!'],
    ['I', 'love', 'my', 'cat', ',', 'sometimes', '.'],
    ['I', 'also', 'like', 'my', 'dog', '!']
]

many_tokenized_sentences = tokenized_sentences * 10000

# create the model
model = gensim.models.Word2Vec(sentences=many_tokenized_sentences)

# find similar words
model.wv.most_similar('cat')

```

One thing to note is that in comparison to other common NLP tasks, you will not want to remove stopwords or punctuation from the vocabulary, since they play an important role in providing context.