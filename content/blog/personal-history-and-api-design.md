---
title: "My Personal History with NLP or Side-Effects of Good API Design"
date: 2022-01-16T07:39:47-05:00
draft: false
---

<meta name="twitter:card" content="summary">
<meta name="twitter:site" content="@pmbaumgartner">
<meta name="twitter:creator" content="@pmbaumgartner">
<meta name="twitter:title" content="My Personal History with NLP or Side-Effects of Good API Design">
<meta name="twitter:description" content="On Word Clouds, Don Norman, and Good API Design">
<meta name="twitter:image" content="https://i.ibb.co/X8PXNdL/Peter-Avatar-Final.png">

I'm joining Explosion AI as a Machine Learning Engineer. This is my first  career move in 6 years and I thought I'd take some time to reflect on my personal experience in data science and natural language processing. 

Since I've been in data science, I've been working in professional services/consulting environments. My last job was working mostly with social scientists and researchers to incorporate machine learning into their research projects. Consulting takes the "jack of all trades, master of none" spirit of data science and cranks it up to 11 by having to work across multiple projects. In my career, I've probably been on at least 50 different projects, giving me a lot of different exposure to different domains and also giving me a broad experience for "what works" on projects.

There's a key lesson I've learned about "what works" on most projects - what works is actually the models themselves[^1]. In most cases, the performance of any algorithm or machine learning model is not usually a reason why a project fails[^2]. There are rare exceptions when performance is worse than a random guess, but even if it's slightly better than random, there's almost always value in automating some decision if you can frame the problem correctly, even if this done only to understand how a model makes different decisions than a human would.

Framing the problem correctly _is_ where most projects stumble. There are many obstacles in the way of a correct problem framing:

- The stakeholders (i.e. the client or subject matter experts) do _not_ understand what machine learning can do
	- In one direction, this can just be naive ignorance - it's not their field, so they shouldn't know it
	- In the opposite direction, this can be insane expectations due to hype combined with misunderstanding, leading to inflated expectations about what can happen
- There might be a set of methods that can work for your scenario, but if you don't already know what those are you can't apply them.
	- Also, it's difficult to keep up up with the insane pace of new methods and models.
- You might know a method (or set of methods) that you think might work for a given problem, but once you actually start working with it you discover issues with it.

In sum, the big problem with executing successful projects is solving this data science alignment problem—mapping the problem at hand to the the combination of methods you know.

In the social sciences, the best description of this problem I've seen of this is this paper: [Machine Learning for Social Science: An Agnostic Approach](https://www.annualreviews.org/doi/full/10.1146/annurev-polisci-053119-015921)

> By agnostic, we mean that our general view is that in many instances there is no correct or true model that we target with machine learning methods, and there is no one best method that can be used for all applications of a data set. Instead, we advocate that researchers use the method that optimizes performance for their particular research task. Unlike in many computer science applications, this task will often not be prediction but may instead be discovery, measurement, description, or causal inference. How we evaluate the quality of a model and compare models will be specific to the research question.

In this paper they map machine learning methods to the research tasks of discovery, measurement, prediction, or inference. This is essentially the same process that applies to any project: mapping what methods you have to the common tasks of that domain. However, most domains are not as easily structured as social science research, so there's also a bit of back and forth that is necessary to figure out how a client *thinks about* the tasks that they accomplish so you can start mapping that to methods you know.

A key aspect of this process is that you will use methods (plural). Even with something like text classification, there are many ways you can do that (weak supervision, rules-based systems, supervised learning, unsupervised learning), that depend on the data. It's likely that your final solution will string several of methods together, or use combinations of both in interesting ways. 

## Gaining Intuition through Experience

Over time you accumulate this experience of "what works for what" and solving the alignment problem will be easier. Early in my career, for example, I learned over many failed attempts that topic modeling (LDA) plainly doesn't work for short texts like tweets. But at the time, LDA was dominating the applied NLP literature I was reading. It was very frustrating to not be able to get things to work, especially before I understood this data science alignment problem and the fact that these methods were not completely data-agnostic: the type of data you were inputting them really matters. LDA works well with many, long documents, not with a few hundred short tweets. But how was I supposed to learn that without first just trying it myself?

The best way to gain experience in solving this alignment problem is to be able to try things out. However, there's an initial problem: you're also not an expert in methods yet and you need some way to try things out. You need tools that make it easy to try things out AND give you a conceptual framework for understanding components of your problem.

At this point, I have to come clean with the most embarrassing part of my NLP career: the first time I used spaCy I was trying to make a word cloud. I had an idea I wanted to experiment with: could I build word clouds that summarized a document, but were filtered by parts of speech - one for nouns, one for verbs, one for adjectives. At this time, my intro to NLP was going through the classic [NLTK book](https://www.nltk.org/book/), which is good but very detailed and focused on computational linguistics—not for building word clouds.

NLTK does allow you to get parts of speech from tokens, but it outputs granular tags by default. More specifically, it gives you tags from the Penn Treebank when I needed the [Universal tagset](https://aclanthology.org/L12-1115/). But I was just starting my adventure into NLP: how was I supposed to know there were even multiple tagsets and what they were called? I knew what nouns were and that's what I wanted. My solution to this was to bookmark [this](https://stackoverflow.com/a/26980638) Stack Overflow question and refer to it often (copied below):

```python
import nltk
from nltk.tag import pos_tag, map_tag

text = nltk.word_tokenize("And now for something completely different")
posTagged = pos_tag(text)
simplifiedTags = [(word, map_tag('en-ptb', 'universal', tag)) for word, tag in posTagged]
```

I had started exploring spaCy a bit at this time and decided to give this problem a try with that. The parts of speech returned in spaCy with the `pos_` attribute are the universal ones. Even more, I didn't have to run through a tokenization, tagging, and the mapping process. Rather I processed a string and asked for an attribute on each token, like this:

```python
import spacy

# I'm in a time machine to spacy 1.0. 
# The "en" model link doesn't exist anymore 
nlp = spacy.load("en")
doc = nlp("And now for something completely different")
pos = [(token.text, token.pos_) for token in doc]
```

There's a difference between spaCy and NLTK for this process. At a surface level, I think spaCy is a little bit simpler and easier to understand. Why is that, though? The NLTK version isn't _that_ complicated: you tokenize some text, get the parts of speech for each token, and apply the `map_tag` function to each token. It's also the same number of lines of code.

It's not just simplicity, though I thought so at the time. At this stage of my career I was getting into design and understanding how people created things that were nice to use and easy to understand. I read Don Norman's "Design of Everyday Things," and a fairly clear message I got from that book was that things should just be "more simple". Get rid of extra features, think about how your thing will be used, and should be obvious *how* something should be used.

But there's not really much *extra* in the NLTK version of how to solve this problem: it's basically just the mapping to universal POS tags. Why did that version feel so much less understandable to me?

Don Norman has another book called "Living With Complexity"[^3]. Reading through the book quickly disabused me of the misconception that well designed things are just "more simple". Here's Norman with his basic definition of what makes something complex:

> What makes something simple or complex? It’s not the number of dials or controls or how many features it has: It is whether the person using the device has a good conceptual model of how it operates.

Norman's point in this book is that perceived complexity arises from a mismatch between the system of the thing *as designed* or *as it exists* and the conceptual model of the person using that system. How do we reduce complexity? Norman explains (emphasis mine):

> Complexity can be tamed, but it requires considerable effort to do it well. Decreasing the number of buttons and displays is not the solution. The solution is to **understand the total system, to  design it in a way that allows all the pieces fit nicely together, so that initial learning as well as usage are both optimal.**


The spaCy API for this task is aligned with this idea. It makes it easy to understand what you need to solve this problem: first you need a model, that model processes the text in various ways including tokenizing it and assigning a part of speech to each token. The tokens are saved with the document, so now you can do arbitrary things with them (like put them in a list if they're nouns). This *initial learning* gets you closer to understanding the *total system* because it's clear what objects exist and how you interact with them.

By comparison, the NLTK API doesn't help me gain an understanding of *how* I'm solving this problem. It feels like I am calling a handful of functions I'm supposed to know about, structuring my data in a way that it needs to be structured for those functions, and outputs my result. It works, but I've got to put more work into understanding why it's doing what it's doing. And as soon as I need to do something else with this text (let's say pull out punctuation tokens), I feel like I have to do a whole search of the docs/the book and write a custom function. 

The difference here is that a good API, when you understand it, will help you *think* about solving problems in that domain. The API within spaCy is cohesive, one thing flows to the next, and it allows a user to form a mental model of what is going on. Meanwhile, NLTK has a collection of functions that I can use to solve my problem, but doesn't allow me to build a mental model of how text is processed.

The advantage of using a tool like spaCy with a cohesive API is that it simultaneously helps you form a conceptual model of how problems are solved. As a part of this process, it also allows you to easily experiment with new ideas because you can _think about_ problems as they fit into this conceptual model. The advantage is that you can solve problems you know how to solve faster, and you are _also_ at a better place when you have problems you _don't_ immediately know how to solve because you at least have a way to organize things.

## Tools for Doing and Thinking: Helping with the Data Science Alignment Problem

"Industrial-strength NLP" is a phrase you'll see if you go back and look at an [archived version of spacy.io](https://web.archive.org/web/20160905151957/http://spacy.io/). When I started using it, spaCy had a ton of functionality in order to get things done—which, when I started using it was exactly what I needed to do. However, as I matured as a data scientist, I started to think more about this alignment problem rather than just getting stuff done. Fortunately, as spaCy matured it expanded from a tool to help do natural language processing to a tool to help think about *ways* I could do natural language processing.

For example, if you go look at the [spaCy Usage](https://spacy.io/usage/) API docs, you'll notice the guides are organized around NLP tasks. Organizing the *things* you can do in this way helps us with our original alignment problem: the ability to organize and "bucket" the types of tasks you can do is essential because you need to map them to the problem you're trying to solve.

Beyond that, the solutions to some problems might involve several tasks strung together. Or you might need to write some logic for a task that is unique to your problem. Also not a problem - [pipelines with custom components](https://spacy.io/usage/processing-pipelines#custom-components). The existing pipeline components serve as great anchors for *how* to do things and also provide helpful inputs to any custom code. Even if you want to write something totally custom, that's alright too - you can use spaCy as a bare-bones framework for writing your own pipeline. 

## Wrap-Up

The hard problem in data science isn't training the best model, but rather a collection of difficult sub-problems:
- it's understanding what the problem is
- having a set of tools you know how to use
- having an organizational framework (conceptual model) that those tools can fit into
- being able to customize your solution when necessary

In my own data science adventure, discovering each of these sub-problems has been a critical part of my journey. It helps to have well-designed tools, like spaCy, to help with all four steps—whether it's your first day as a data scientist or you're a seasoned expert.

[^1]: I for one am excited for the [end of modelitis](https://github.com/HazyResearch/data-centric-ai/blob/main/end-of-modelitis.md). 

[^2]: At least in research projects I've worked on. I understand this is not true if you're working on a product and ML is a core component of your product.

[^3]: It's even better than Design of Everyday Things. Most of us are designing complex systems, not everyday things, and approaching design after understanding complexity is a much better approach (to me)