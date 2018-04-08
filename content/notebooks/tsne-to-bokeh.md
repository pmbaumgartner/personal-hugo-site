---
Title: Dimension Reduction with TSNE and Bokeh Scatterplot
Date: 2017-03-25
---

[Link to Notebook](https://github.com/pmbaumgartner/binder-notebooks/blob/master/dimension-reduction-tsne-to-bokeh-scatterplot.ipynb)

[![Binder](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gh/pmbaumgartner/binder-notebooks/master)

### What's in this notebook?


This is a workflow I use often in data exploration. TSNE gives a good representation of high-dimensional data, and Bokeh is helpful in creating a simple interactive plots with contextual info given by colors and tooltips. 

This workflow has been extremely helpful for:

- text analytics/NLP tasks if text data is passed through a `TfidfVectorizer` or similar from `scikit-learn`
- understanding `word2vec` or `doc2vec` vectors by passing them to TSNE
- getting an idea of *separability* in doing prediction / classification by passing the outcome variable to bokeh

This example uses the [Australian atheletes data set](http://math.furman.edu/~dcs/courses/math47/R/library/DAAG/html/ais.html), which contains 11 numeric variables. This workflow is even more helpful on larger datsets with higher dimensionality.