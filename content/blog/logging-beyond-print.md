---
title: "Two Logging Options Better than Print Statements"
date: 2022-01-12T21:02:41-05:00
draft: false
---

<meta name="twitter:card" content="summary">
<meta name="twitter:site" content="@pmbaumgartner">
<meta name="twitter:creator" content="@pmbaumgartner">
<meta name="twitter:title" content="Two Logging Options Better than Print Statements">
<meta name="twitter:description" content="Three brief examples of loguru and tqdm">
<meta name="twitter:image" content="https://i.ibb.co/LYz2DyQ/1537215809678.jpg">

This is a short blog post on two things I've found helpful when I'm *not* using notebooks and running python code that I want to log things in.

## Loguru

Here's an example of how I use `loguru`. Typically I don't include all the boilerplate at the top, but for illustration of some of the functionality I'm including it.

In this snippet I'm changing the default logger format by adding the time elapsed since program start. The program itself loads data and a spaCy model, and I want to know how long both of those steps take.

```python
import sys

import spacy
from loguru import logger

from utils import load_data

# loguru boilerplate
# don't really need this, but adds time elapsed
# can convert to environment variable
logformat = (
    "<blue>{elapsed.seconds:0>4}s</blue> | "
    "<green>{time:YYYY-MM-DD HH:mm:ss.SSS}</green> | "
    "<level>{level: <8}</level> | "
    "<cyan>{name}</cyan>:<cyan>{function}</cyan>:<cyan>{line}</cyan> - <level>{message}</level>"
)

logger.remove()  # remove default logger
logger.add(sys.stderr, format=logformat)
logger.add("default.log", format=logformat)  # log to a file as well
logger = logger.opt(colors=True)

if __name__ == "__main__":
    df = load_data("data.parquet", columns=["count"])
    logger.info("Data Loaded")

    logger.info("Loading Model")
    nlp = spacy.load("en_core_web_lg", disable=["ner"])
    logger.info(f"loaded model. Pipeline: {nlp.pipe_names}")
```

```
0000s | 2022-01-12 09:12:22.212 | INFO     | utils:load_data:7 - Loading Data
0001s | 2022-01-12 09:12:22.770 | INFO     | __main__:<module>:21 - Data Loaded
0001s | 2022-01-12 09:12:22.771 | INFO     | __main__:<module>:23 - Loading Model
0005s | 2022-01-12 09:12:27.229 | INFO     | __main__:<module>:25 - loaded model. Pipeline: ['tagger', 'parser']
```
*(colors not rendered)*

We can use this to eyeball how long certain parts of our program take to run. In addition, it let's us know the namespace, function, and line number of where something gets logged. For example, in the first logger call I'm calling a function `load_data` in the `utils` file (not included here), which calls the logger on line 7 - hence `utils:load_data:7`.

There's lots more to customize with `loguru`. For example, you can change the default format [using an environment variable](https://loguru.readthedocs.io/en/stable/overview.html#personalizable-defaults-through-environment-variables) so you don't have to include that boilerplate in every file. However, the killer feature for me is that I can just use `from loguru import logger` and be able to log things *without* including boilerplate.

## Tqdm
### Subsidiary Data
Sometimes I will iterate over something and there's a subsidiary *thing* that I want to count. For example, if I'm tokenizing documents with spaCy, sometimes I'll count the vocab size and total tokens.

```python
import spacy
from tqdm import tqdm

nlp = spacy.load("en_core_web_lg")

docs = ["this is a document", "here is another document", "please tokenize me"]

# collections setup
tokenized_docs = []
vocab = set()

# tqdm setup
vocab_size = 0
n_tokens = 0
pbar = tqdm(nlp.pipe(docs), desc="Tokenizing Docs", total=len(docs))

for doc in pbar:
    tokens = [t.text for t in doc]
    vocab |= set(tokens)
    vocab_size = len(vocab)
    n_tokens += len(tokens)
    tokenized_docs.append(tokens)
    pbar.set_postfix({"vocab_size": vocab_size, "tokens": n_tokens})
```

```
Tokenizing Docs: 100%|██████████| 3/3 [00:00<00:00, 294.98it/s, vocab_size=9, tokens=11]
```

### Multi-Step Processes
You can also use `tqdm` when you have a logically coherent set of steps for a process, and the steps might not take the same amount of time. Here's an example from some [plotter](https://twitter.com/search?q=from%3A%40pmbaumgartner%20%23plottertwitter&src=typed_query&f=live) code I wrote that deduplicates overlapping lines and merges them if they're connected. There are three stages that take different amounts of time, but I use this to update the progress bar with what stage the process is at and capture the total time elapsed when it's done.

```python
with tqdm(
    total=3, desc="Line Cleanup", postfix={"Stage": "Making LineString"}
) as pbar:
    lines = geo.MultiLineString(lines)
    pbar.update()
    pbar.set_postfix({"Stage": "Self-Intersecting"})
    lines = lines.intersection(lines)
    pbar.update()
    pbar.set_postfix({"Stage": "Merging Lines"})
    lines = ops.linemerge(lines)
    pbar.update()
```

```
Line Cleanup: 100%|██████████| 3/3 [00:02<00:00, 1.11it/s, Stage=Merging Lines]
```

## Summary

Check out `loguru` and `tqdm` for lightweight ways of logging things that provide supplemental info about your program.