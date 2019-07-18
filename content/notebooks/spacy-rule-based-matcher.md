---
title: Spacy Rule Based Matcher Workflow
date: 2017-05-21
---

### Links

[Link to Notebook](https://github.com/pmbaumgartner/binder-notebooks/blob/master/rule-based-matching-with-spacy-matcher.ipynb)

[![Binder](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gh/pmbaumgartner/binder-notebooks/master)

### What's in this notebook?

This is a worked example derived from my blog post on [Making the Most of spaCy's Rule-Based Matcher]({{< relref "/blog/spacy-rule-based-matcher-workflow.md" >}}). It works through developing a matching algorithm to identify reasons people purchase products from a dataset of Amazon product reviews. For example, you can automatically generate a list like this for any product:

```
Customers buy this product ...
... as a replacement for a Salton model.
... as a Christmas gift for my son in law.  
... as a new coffee maker for serving guests of an annual retreat we host in our home.  
... as a spice grinder and are very happy.  
... as a christmas gift for a friend and it works great.
... as a spice grinder and a coffee grinder
... because it was cheap and had good reviews, and I have not been disappointed.
... because of the excellent reviews. 
... for grinding dry spices.
... for grinding coffee
... to grind beans for cold-brewed coffee.  
... to grind spices instead of coffee and it works great.  
... to grind xylitol and truvia sweeteners into a powder for cooking.
```