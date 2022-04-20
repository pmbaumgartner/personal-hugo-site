---
title: "What Should Data Scientists Learn?"
date: 2022-04-20T13:59:50-04:00
draft: false
---

This week my pal Vicki tweeted this, which I disagreed with:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">The funny part is that every single response to this is correct. Should you learn Python? Yeah. Should you learn K8s? Yeah. Should you focus on SQL? Yeah. <a href="https://t.co/i9KHigqLXL">https://t.co/i9KHigqLXL</a> <a href="https://t.co/ZCOvI9PIVI">pic.twitter.com/ZCOvI9PIVI</a></p>&mdash; Vicki (@vboykis) <a href="https://twitter.com/vboykis/status/1516747665861758979?ref_src=twsrc%5Etfw">April 20, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 

I responded that beginners shouldn't learn K8S, and Vicki said they might not have a choice depending on their job, I summed up my primary objection in a reply:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Telling beginners to learn everything to appease their employer and failing to draw boundaries doesn&#39;t seem like healthy advice to me.</p>&mdash; Peter Baumgartner (@pmbaumgartner) <a href="https://twitter.com/pmbaumgartner/status/1516796134383095808?ref_src=twsrc%5Etfw">April 20, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 

There's more detail and clarifications of Vicki's position in the replies, and you should go read that, but I thought I'd use this discussion as a jumping off point to answer the question **what _should_ a new data scientist learn?**

### First, I am a hypocrite

My career has been in professional services / consulting organizations, so I've worked on many projects requiring many different skills. My advice to new employees is always "say yes to most opportunities," which is contrary to my objection about telling beginners to learn everything. It's even a point for Vicki's argument that you might not have a choice depending on your job—in this case, a beginner might not have a choice of what they learn because of a client's tech stack.

The reason I give this advice is that when you're new to a field you don't really know what you'll enjoy or what you're good at. A good solution to this is to should try and do is gain awareness of as much of the field as possible so you get a taste of the different skills and experiences you might have. Your experience doing any type of work is much more valuable than any specific advice you receive or any blog posts of the "types of data scientists" or "all data scientists should learn X" genre.

You shouldn't go blindly into any opportunity though. There are two things you need to think about when taking this approach: your search strategy and your exit strategy.

### The search strategy

Your search strategy is where those "types of data scientist" articles are going to come in handy. You should find experiences that give you an opportunity to work as each "type" of data scientist. I was fortunate to work in an environment where different types of projects were abundant, but others might not be. In this case, try to do some side projects doing something very different than what you do at your day job, or find ways to connect what you do in your day-to-day work with skills from another type of work (more on this later).

Here's my personal example: when I started in data science, I was very interested in web development because I wanted to get what I was building into the hands of people that would use it. It took me a long time to get comfortable with a web development skill set, probably 2-3 years. I failed my way through a [Flask book](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world) several times, and it wasn't until I started to learn Django that things started to click for me. But once I had this skill, it opened up a bunch of opportunities for me to decide where to go next. Now that I knew how to make web applications, I could focus on data visualization to include in my apps. Or I might want to focus on more DevOps skills like deploying the apps to production and updating the models. Either way, because I stretched a little bit to learn something outside of the "core" data science skill set, I now had connections to new areas of skills to explore.

### The exit strategy

I'm a [SAS Certified Professional](https://www.sas.com/en_us/certification.html#programming) as well as a [Tableau Certified Associate](https://www.tableau.com/learn/certification) and I have my [AWS Certified Cloud Practitioner](https://aws.amazon.com/certification/certified-cloud-practitioner/) certification. I don't advertise this anywhere because I don't want to do anything with either of those tools. However, I think it's a good thing I got those certifications because they forced me to learn those tools and get my first job.

In the same way, you might learn things as a beginner that you'll never use later in your career and that's okay. What you need to do is reflect on that experience, think about what you liked and didn't like about it, and use that information to find out what you might like. For me, I discovered I didn't enjoy programming in SAS, but this wasn't until after I discovered programming didn't have to be painful or cumbersome when I learned python. I learned to avoid Tableau projects because the licensing model and tech stack didn't work out for the clients I was working with and it introduced a lot of [tech debt](https://www.peterbaumgartner.com/blog/tableau-as-sneaky-technical-data-debt/) on my projects. As for AWS, I was curious about learning more infrastructure skills and we had a bunch of projects I thought I was going to be doing more infrastructure work for, but they never panned out.

As you develop, you'll need to _shed_ your old skills and make sure you're not stuck doing something you don't want to be doing. You might have newer employees that would want to learn these skills. You might be looking at new jobs that focus more on the areas you like doing. There are probably other solutions as well - the important thing is that you realize that you need a way to *stop* doing the things you don't enjoy and focus more on the things you do enjoy, and make a plan to get there.

### Generalizing the approach: what should you learn next?

There's a question I have asked myself whenever I'm trying to figure out what to learn next. I think of my current role and skills, then ask "What happens before I do my job?" and "What happens after I do my job?". It's a middle-out way of identifying new things to learn and increase the likelihood that it's a skill that you can immediately use because of it's adjacency to your current skill set.

Let's say you focus mostly on developing models at your job. Well, what happens before the data gets to your model? It's gotta come from somewhere: that's data engineering! What happens after your model outputs predictions? Well, it might go into an application where users need to interact with it: that's web development. Or you might want to present the model to stakeholders: now you need data visualization and some communication skills. Again, the added benefit here is that you can easily expand your current skillset to learn these things because they're adjacent to what you already do.

### Is there a bad place to start?

Let's bring it back to the original question: what should a beginning data scientist learn? In some sense you're initializing your search strategy with a specific skill. Is there a bad skill to learn first? Well, this was my main beef with Vicki's original tweet[^1]. In my opinion, something like K8S is a bad initialization for this search, because if you take my middle-out skills approach it's a very long way from K8S back to anything resembling data science. I'm not saying that data scientists should never learn K8S[^2], but you'd want to learn it after _initializing_ your search at a core data science skill.

How should you pick that *initialization* skill? I actually think the way to think about that is by inverting the problem: pick any skill that isn't far away from "core" data science tasks. What does this mean? Well, for programming languages you can learn python, R, Julia, and even SAS. You can be a data engineer, a data scientist, a machine learning engineer, a data visualization expert, or whatever other flavor of data scientist exists at the time. As long as you keep applying this middle-out approach and reflecting on your experiences, you'll continually approach your own idea of a "data scientist" and a skill set that works for you.


  [^1]: She's since clarified https://twitter.com/vboykis/status/1516793897065848839
  [^2]: For me learning docker has been really helpful! https://twitter.com/pmbaumgartner/status/1492179636087934985