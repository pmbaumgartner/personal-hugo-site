---
Title: Dimension Reduction with UMAP and Plotly Scatterplot
Date: 2019-07-17
---

[Link to Notebook](https://github.com/pmbaumgartner/binder-notebooks/blob/master/dimension-reduction-umap-to-plotly-scatterplot.ipynb)

[![Binder](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gh/pmbaumgartner/binder-notebooks/master)

### What's in this notebook?

This is an updated version of my TSNE to Bokeh Scatterplot workflow. I found that UMAP is faster and is able to handle larger datasets where TSNE would previously fail, so I've switched over to UMAP as my dimensionality reduction default. I've been building a lot of dashboards and visualizations in the plotly ecosystem, so I've also switched from Bokeh to Plotly.

This workflow has been extremely helpful for:

- text analytics/NLP tasks if text data is passed through a `TfidfVectorizer` or similar from `scikit-learn`
- understanding `word2vec` or `doc2vec` vectors by passing them to TSNE
- getting an idea of *separability* in doing prediction / classification by passing the outcome variable to bokeh

This example uses the [Australian atheletes data set](http://math.furman.edu/~dcs/courses/math47/R/library/DAAG/html/ais.html), which contains 11 numeric variables. This workflow is even more helpful on larger datsets with higher dimensionality.