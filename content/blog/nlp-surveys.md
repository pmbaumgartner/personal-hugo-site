---
title: "Notes on NLP for Survey Design"
date: 2021-12-13T21:06:35-05:00
draft: false
---

<meta name="twitter:card" content="summary">
<meta name="twitter:site" content="@pmbaumgartner">
<meta name="twitter:creator" content="@pmbaumgartner">
<meta name="twitter:title" content="Notes on NLP for Survey Design">
<meta name="twitter:description" content="How can NLP make survey design easier? How can survey design make NLP easier?">
<meta name="twitter:image" content="https://i.ibb.co/yg70SRL/Frame-1.png">

I've worked on several projects where we're applying some natural language processing techniques on responses to open-ended survey items. Typically this means putting them into categories—either following a pre-existing coding scheme or by creating a new one with unsupervised learning. Through these experiences, I've developed a few principles for working on these types of problems.

### Put Yourself Out Of Business
There's not a project I've been on where my recommendation wasn't "If you want better data, turn this open-ended question into a close-ended question that asks about the specific thing you care about." 

One example of how this shows up is that a question often asks for *strengths and weaknesses* of some topic. Usually this is followed someone mentioning they need to do "sentiment analysis" to pull apart the strengths and weaknesses from a single long response. One option in this scenario is to split this apart into two questions, one about strengths and one about weaknesses. This way, you can extract the specific strengths and weaknesses and not have to identify which part of the response is a strength of a weakness. The best thing to do is to convert this into a close ended response that operates like a [rubric](https://www.betterevaluation.org/en/evaluation-options/rubrics) with a [scaled response](https://www.qualtrics.com/blog/three-tips-for-effectively-using-scale-point-questions/).

### Question Development is Critical

I was on a project where one version of the survey asked a question about a process was "disrupted", and another version asked about how a process was "affected". The goal was to compare responses from those who answered toe "disruptions" version of the question to those who answered the "affect" version. The challenge was that there are several things that might "affect" the process but not be "disruptions", and use of the word "disruption" can anchor folks into 

In this case, combining NLP/ML with cognitive interviewing and question design is a powerful tool—questions that are difficult to structure with supervised or unsupervised methods (i.e. via labeling or clustering) are could be that way because of a wide variety of interpretations of the question itself. 

In addition to question writing for respondent understanding, there's also writing an open-ended question that is easy to analyze requires fine-tuning the specificity of what you ask for and providing a "template" for how a respondent should answer. Asking for something too broad (e.g. "What are your thoughts about dogs?") means that both the semantic aspects of a response are going to have high variability, but also the grammatical aspects of the response are going to have high variability. Usually you care most about the semantic aspects, but often models pick up on grammatical artifacts, so you want to reduce the grammatical variation as much as possible.

The best way I've seen to do this is to word the question with an implicit response length. For example, on an [employee survey](https://osf.io/fueyj/) I worked on our open-ended question was "What is the most important change [COMPANY] could make to improve your experience working here?" While this doesn't limit a respondent from providing multiple changes or writing a litany of gripes, it does nudge the respondent to think of _one_ most important thing. Another way to help frame the response is to structure the question as a fill-in-the-blank, e.g. "The most important change [COMPANY] could make to improve my experience is \_\_\_\_\_". 

### Think Hard about Question Blocks
Often times a series of questions will appear in a question _block_: a group of questions that cover the same topic. If these questions are open-ended, respondents can "meld" their responses together across questions. Remember, you've got a human on the other side whose goal is to get through the survey as quickly as possible while providing reasonable information.

If you start the block by asking a broad question, one thing respondents may do is respond to a later question with some variation of "see previous". For example, if your first question asks "how are you feeling today?", the respondent answers with some variation of "good, I have a feeling of satisfaction", and a second question asks "What positive feelings are you having today?", saying "see above" makes sense. In this case, you might need to provide instructions. Here's a few ideas: Instruct the user to copy/paste their response from above, instruct them to use "see above" with a specific question id like "See Previous: Q26", or instruct them to read the whole question block first to understand the topic, then answer the most specific questions first.

The other complicating factor is the order of the questions. This is because of [anchoring](https://en.wikipedia.org/wiki/Anchoring_(cognitive_bias)) and [priming](https://en.wikipedia.org/wiki/Priming_(psychology)). The order in which a respondent proceeds through the questions means that the interpretation and response to each question is influenced by prior questions. In addition, priming means that you can solicit certain responses by the question text alone—commonly by including examples in the question text. If you include examples in your question text, it might make a better close-ended question, lest all your open-ended responses just include the examples you included in the question stem.

**Summary & Recommendations**

- If there are consistent patterns in open-ended responses you care about, convert those to close-ended questions
- Consult with the other experts on your team when developing questions, being sure to deliver your recommendations on what makes a good question for analysis
- You need to understand the relationship of any open-ended questions to the other questions in the survey, especially within the same block. The respondent will make their own inferences about how they should answer questions—be explicit in how you would like them to respond if they're providing redundant information.