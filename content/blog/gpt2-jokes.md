---
title: "Fine-Tuning GPT-2 Small for Generative Text"
date: 2019-03-23T11:16:39-04:00
draft: false
---

> Why did the chicken cross the road? Because it had no legs.

These are the types of *hilarious* jokes the GPT-2 model can generate for you. 

After reading a few blog posts [here](https://www.gwern.net/RNN-metadata#finetuning-the-gpt-2-small-transformer-for-english-poetry-generation) and [here](https://svilentodorov.xyz/blog/gpt-finetune) I thought I would write up the process I used to fine tune gpt-2.

For this example, we'll use a dataset of jokes pulled from the `/r/jokes` subreddit to fine tune the GPT-2 small model to generate new jokes. 

You'll need a computer with a GPU and nvidia-docker for this.

We'll start by cloning the code to download and train the GPT-2 Small model. Fortunately, others have done the hard work of adding code to train on top of the `gpt-2 small` model that OpenAI released. 

```bash
git clone https://github.com/nshepperd/gpt-2
cd gpt-2
sh download_model.sh 117M
```

We're going to use docker from here on out, just because it's easier to manage the code and dependencies. The repository comes with a dockerfile, let's build the image:

```bash
docker build --tag gpt-2 -f Dockerfile.gpu .
```

Great! Let's get to a shell using our image:

```bash
docker run --runtime=nvidia -it \
  -v $(pwd):/gpt-2 \
  -e NVIDIA_VISIBLE_DEVICES=0 \
  gpt-2 bash
```

At this point, you can play with the base `gpt-2 small` model and generate some text. Let's try it out with this prompt:

> "A pair of jumper cables walks into a bar" 


```
$ python src/interactive_conditional_samples.py --top_k 40 --temperature 0.9
[...some tensorflow logging]
Model prompt >>> A pair of jumper cables walks into a bar
======================================== SAMPLE 1 ========================================
 and a few minutes later, they are all turned on. A man in jeans and a black dress, comes out of the bar and says he got a job and wanted to know if he could tell me what he is wearing, how much he is wearing and whether he's wearing something different. He's wearing a red skirt, a blue T-shirt and black heels, on a black tie, like a red cape with two red crosses, the same as on the white one.
```

The original punchline was: 

> The bartender sighs and says; "I'll serve you, but don't start anything!"

That's the level of sense we can expect out of this thing without fine tuning.

## Fine-Tuning on a Specific Corpus

We're going to fine-tune it a set of jokes. What this is going to do is train the model to pick up both the structure of the joke (setup, punchline) as well as how language is used (both vocabulary and structure).

We'll download a set of jokes from [this repository](https://github.com/taivop/joke-dataset). Note that there is some NSFW and racist, sexist, and plain unfunny content in some of these jokes, look out for this both in the training data and in our model output. 

There are a few different structures for jokes in our dataset. For short jokes, the setup will be the title of the post, and the punchline will be the body. For longer jokes that have a bit of setup, typically the title of the post is also the first sentence of the joke. Fine-tuning the model will pick up on the structure and language of the jokes, so what we'll do is separate the setup (post title) and punchline (post body) with a pipe (`|`). Additionally, `gpt-2` uses a special token `<|endoftext|>`, to signify the end of a piece of text, so we'll be formatting data with those structural constraints.

First, let's download the data:

```bash
curl -O https://raw.githubusercontent.com/taivop/joke-dataset/master/reddit_jokes.json
```

The following python script will get the data in the format we need:

```python
import json
from pathlib import Path

jokes_raw = json.loads(Path("reddit_jokes.json").read_text())

jokes_parsed = "<|endoftext|>".join("{0}|{1}".format(j['title'], j['body']) for j in jokes_raw)

Path("input-text.txt").write_text(jokes_parsed)
```

(side note: `pathlib` is amazing, you should be using it if you aren't already)

Now we must encode our text in the input format for gpt-2. This is something that normally would happen during training anyway, but we can speed it up by preprocessing.

The `requirements.txt` needs to be updated with the `tqdm` dependency, so you will need to install it in the container before running the encoding script.

```bash
pip install tqdm
PYTHONPATH=src ./encode.py input-text.txt input-text.txt.npz
```

And with that, let's get training! This script will train until you `Ctrl-C` out of it. The flags that we're passing will print out a sample as well as checkpoint the model every 250 epochs.

```bash
PYTHONPATH=src ./train.py --dataset input-text.txt.npz --sample_every=250 --save_every=250
```

Choosing the right amount of training for these types of things is difficult. Personally, I enjoy the surrealism, absurdity, and nonsense of jokes from models with fewer training epochs.

### Training Examples

Below I'll post some jokes from the sample output that gets generated every 250 epochs:

250:


```
What do you call it when the baby is on your chin?|Lion's Pie
Did you hear about the guy who lost a job for eating his lunch?|He was just going to eat it.
I tried to take my dog to the bathroom in the hospital...|My dog was a dachshund.
What did the priest say after an elephant died?|"What is it for?"
```

500 epochs:
```
I bought my car last week, and it wasn't working really well.|I was going to give it a better start, but the engine just wasn't good enough.
```

750:

```
Two dogs are walking through the woods|When they meet up the first dog yells "Hey is this dog our friend?". The second dog thinks it's funny and says "Is this dog that he walks around alone".

And the two dogs quickly stop walking.

"I don't think so, your mom always walks by his yard."
Why did the chicken cross the road?|To get a job at the hen.
What do you call a man with dyslexia?|A dyslexic.
A man walks into a bar and asks for a drink|Bartender looks at him and says, "This is not a drinking game."
```

1000:
```
Puns aren't jokes.|...they're punchlines.
```

1250:

```
The bartender says.

"How much is $0.5 for an idiot?" The mathematician says "Not that much, I just went bowling.
```


1500:

```
Why did the chicken cross the road?|Because it had no legs.
What do you get when you cross a sheep with a goat?|I'm a sheep with no legs. Just look at that sheep!
I've been in some really terrible relationships and wanted to share.|So I made some tea and got out.`
```

### After Training

When you've finished training, you'll need to copy the checkpointed model weights to the model directory like so:

```cp -r /gpt-2/checkpoint/run1/* /gpt-2/models/117M/```

Now, you can use your fine-tuned joke model to generate a bunch of jokes for a certain prompt. Let's figure out more ways the chicken could cross the road...


```
python src/interactive_conditional_samples.py --top_k 40 --temperature 0.7 --nsamples 10
[... wait for startup...]
Model prompt >>> Why did the chicken cross the road?|
```

Some answers:

```
> So I can get my steak
> To the left... I'll see myself out
> Because I thought the chicken crossed the road.
> I don't know, but his face is too close to the road.
> Because it was a chicken.
> The chicken crossed the road in the morning, and the morning is fine, the morning is fine.
````

What happens when `A guy walks into a bar...`?

```
Model prompt >>> A guy walks into a bar...|
> A guy walks into a bar and orders a drink. The bartender asks him what the occasion is. The guy says that it's the new year and all the old ladies are going to get married. The bartender says no worries, they'll be there in time for the anniversary. The guy says that he's really excited and he's a bit nervous. The bartender is a bit more hesitant and asks him what the occasion is. The guy says "I'm a new guy, I was going to give a speech about my experience with alcohol." So the bartender gives him a little push and the guy says "You know what, I'm drunk." Then the bartender says "Sorry, but you don't have the guts to talk like that."
```

What about `What's the difference between`?

```
Model prompt >>> What's the difference between
" a woman that's going to have a baby and a guy that's never gonna have a baby?|One's a girl that's going to have a baby and the other is a guy that's never gonna have a baby.
```

## Wrap-up
My personal thought is that we've probably overfit on the data. The examples I'm not selecting are pretty reflective of the typical jokes on `/r/jokes` (lots of dogs, "whats the difference?" and racial stereotyping). For my sense of humor, favoring a little more generality and abstractness, I'd prefer text from the models after 500-1000 epochs of training.

