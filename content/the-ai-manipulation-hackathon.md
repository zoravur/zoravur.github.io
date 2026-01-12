+++
title = 'The AI Manipulation Hackathon'
date = 2026-01-12T09:20:37-05:00
draft = true
+++

Yesterday, I attended 
[Apart Research's AI Manipulation Hackathon](https://apartresearch.com/sprints/ai-manipulation-hackathon-2026-01-09-to-2026-01-11), 
hosted by Trajectory Labs. 

The event ran over three days, and the goal of the hackathon was to develop tools to measure, detect, and defend against AI 
manipulation. I came in with no real agenda as a last minute sign-up, but thanks to my team, I was able to get up to speed and learn
just a little bit about AI manipulation, and safety in general.

## Day 1

The first day, I met up with the only 2 other people doing the hackathon from Trajectory, Christopher and Giles, who would become my
teammates.

Christopher introduced me to the process of reviewing ML literature via Google Scholar, while Giles helped guide the formulation 
of experiments. We went through many ideation whiteboards. This was the first:

![](/board1.webp)

On the left, we were brainstorming what incentives an LLM might have to manipulate -- asking them to take on the role of a 
manipulative occupation, such as a spy, or ethical scenarios where manipulation is essential, such as trolley problems, or monetary
incentives, such as in a business negotiation.

Giles proposed a simple experiment where we take a dataset of prompts which do and do not invoke manipulation in models, and classify
whether the resulting response is "manipulative" or not, whatever that meant. Once we had a way of measuring whether a 
model's response was manipulative, we'd be able to train a linear probe, which is equivalent to finding a vector that represents 
the tendency of models to manipulate. Theoretically, we'd be able to suppress manipulation, increase its probability, or remove
it entirely via abliteration.

On the right hand side, Giles gave me a quick recap on whether language models are deterministic (they are until you sample from them),
and then we discussed whether we should determine manipulative tendencies by looking at the chain of thought or by using the actual
response.

On the bottom right, we drew a little diagram of an idea that eventually became the backbone of our project. Consider an LLM-to-LLM 
interaction. LLM A (representing a malicious actor, either a human directly or a human controlling an LLM) tries to convince LLM B to 
take some action it should not take, such as redirecting a trolley towards innocent civilians. We could run through a variety of 
scenarios to determine how susceptible LLM B is to manipulation, and similarly, we could determine LLM A's ability / tendency to 
engage in manipulation.

After reviewing some papers on manipulation, we came across [a paper by T. Pham](https://arxiv.org/abs/2510.12826), which introduced
two games, _CheapTalk_ and _PeerEvaluation_, that could test LLM's abilities to manipulate and be manipulated. This game was very 
similar to the description above, and we based our scenarios heavily on _CheapTalk_.


Some questions we thought about were exploring were the effect of model size, and whether we could identify a vector which represents
a model's tendency to manipulate. 

## Our project.
