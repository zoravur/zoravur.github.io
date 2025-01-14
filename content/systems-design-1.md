+++
title = 'Napkin Math 1: Fermi Estimation'
date = 2025-01-13T19:40:15-05:00
draft = false
tags = ["systems design"]
+++

Napkin math - also known as back-of-the-envelope calculation - is a powerful technique for quickly estimating answers to complex problems using simplifying assumptions and rules of thumb. This approach is particularly valuable in systems design, where quick approximations often need to be made about performance, capacity, and resource requirements. In programming specifically, these calculations help establish upper and lower bounds, typically aiming to be within an order of magnitude of the actual answer.

One common technique used in napkin math is Fermi estimation, which I'll demonstrate below. Given the breadth of this topic, I'll be breaking this discussion into digestible parts, with this serving as the first in what I hope will be a series of posts on systems design.

**Question 1: How many text messages are sent in the United States in a single day?**

Think about how you would go about solving a question like this. Most people don't think about this number a lot, so it's hard to build an 
intuition for it. However, if you're tempted to estimate it using some known reference points, it suddenly isn't so difficult, at least to get
an approximate answer. You might think about how many texts you send in a day, and how many people there are in the United States. You might do 
a little better by estimating the average number of texts instead of using your own, because you might have some idea whether or not you text
more or fewer people than average. A good estimate for this might be between 20-40 texts a day. Multiplying this by the current population of 
the United States, you get around 340 million * 30 = 10.2 billion texts per day. 

We can get a precise estimate by taking the number of yearly texts in 2021 (2 trillion), and divide it by the number of days in a year (365). This 
yields about 6 billion texts a year, in 2021. So, our answer is not terribly precise, but not bad for an answer that took thirty seconds and is 
within a factor of 2. 

A problem where this technique, of multiplying known or estimated reference points, is known as a Fermi problem. Let's look at a slightly more 
complicated example.

**Question 2: How many pizzas are eaten in New York City on a typical Friday night?**

This was the first problem that I attempted with Claude. I found practicing with an LLM made this process much easier and more intuitive than
it was in the beginning. Here's my original attempt:

> 1. There are around 10 million people in NYC.
> 2. I probably order pizza around once every 2-3 months. But I also don't get out much. So maybe the average person eats pizza once a month, 
and when they do, they eat maybe 3 slices (on average).
> 3. So maybe the average person a whole pizza every 4 months
> 4. That's 3 pizzas per year.
> 5. That's 30 million pizzas a year.
> 6. Pizza is a party food, so let's say that roughly a third of all pizzas are eaten on a friday night.
> 7. So that drops it back down to 10 million on fridays. Divided by the number of fridays (52 ~= 50), we get around 200000 pizzas on a Friday 
night.
>
> This seems high to me...

Oftentimes, it helps to validate the same problem using multiple techniques. Claude's response:

> Let me help analyze your reasoning and suggest another approach:
> 
> Your method is logical, but you're right to be skeptical of the final number. Let's try another way:
> 
> 1. NYC Population: ~10 million
> 2. On a Friday night, what percentage of people eat pizza?
>    - It's a popular Friday dinner option
>    - But competes with other takeout, restaurants, home cooking
>    - Let's say 10% of people eat pizza on a given Friday
>    - So that's 1 million people eating pizza
> 
> 3. Average pizza serves 2-3 people on Friday night
>    - Some eat alone (whole pizza)
>    - Some families/groups share (1/3 or 1/4 pizza per person)
>    - Let's say average is 2.5 people per pizza
> 
> 4. Calculation:
>    1 million people ÷ 2.5 people per pizza = 400,000 pizzas
> 
> Interestingly, we got to a similar order of magnitude through a different method! This suggests we might be closer than you think. Consider:
> - NYC has thousands of pizza places
> - Many are extremely busy on Friday nights
> - A busy pizzeria might make 100-200 pizzas on a Friday
> - With several thousand pizzerias, these numbers become plausible
> 
> Would you like to try estimating this a third way to further validate our thinking?

Claude makes a calculation error here -- it assumes that it takes between 3 and 4 people to finish a pizza, and then assumes an average of 
2.5 people per pizza, which is incorrect. Interestingly, if we plug in 3.5, we get something closer to our original 200 000 number. In this way, 
we can begin verifying our estimate by arriving at a similar result using different assumptions.

Let's move on to a different example, before I start getting hungry.

**Question 3: How many people are on airplanes over the United States right now?**

My answer follows:

> The US doesn't really seem like a flyover country, just given the geography, except for the case when you're traveling from countries below canada up to canada or vice versa. Let's discount this, and focus on flights into and out of the US.
> 
> The average US citizen flies about once every 6 months, I'd say, with frequent flyers dominating. So let's say that 2/365 * 330 million = 500000 people are in planes over the united states.
> 
> Let's try to gut check this. 
> 
> I'd estimate the carrying capacity of a plane taking the number of rows (around 40) and multiply it by the number of seats in a row (let's say 6 on average). Then, we get around 240 passengers per plane. Dividing 500000 by this figure yields 2000 planes over the US, which seems reasonable?
> 
> There are probably at least 100 airports, and an average of 20 planes at an airport at any time doesn't seem that bad. So maybe our original answer isn't too bad. I'm locking in 500k.

Claude's response:

> Let me help verify this with another approach:
> 
> 1. Major airlines in the US typically operate around 15,000-20,000 flights per day
> 2. Average flight duration is about 2 hours
> 3. There are roughly 24 hours in a day
> 4. So at any given moment: (15,000 flights × 2 hours) ÷ 24 hours = about 1,250 flights in the air
> 5. With your estimate of 240 passengers per plane: 1,250 × 240 = 300,000 people
> 
> Our two estimates (500,000 vs 300,000) are remarkably close - within the same order of magnitude and differing by less than a factor of 2! This suggests we're probably in the right ballpark.
> 
> Your gut check using airports was clever too. The fact that three different ways of thinking about it (passenger-based, flight-based, and airport-based) all lead to similar numbers gives us confidence in the estimate.
> 
> Would you like to try another problem? This time I can give you one with even fewer moving parts to minimize compounding errors.

Once again, within a factor of 2! We're getting pretty lucky, but that's also because I didn't include the one I got wrong. Here's one I skipped
over from earlier. I encourage you to try this one yourself, for a challenge.

**How many books are in all US public libraries combined?**

> There are 330 million people in the US.
> Let's say 5% of them visit libraries regularly.
> When I go to a library, I'd say there's usually around an average of 50 people there. There's more at some times (during summer, during exam week, in student libraries), and less in others, but let's say 50 is a good average.
> Let's say the average person visits a library for about 2 hours when they do go. There are shorter visits, but also many students who grind out a lot of work.
> Let's say the library is open for 8 hours a day on average.
> Then let's say the average daily visitors to a library is about 50 * 8 / 2 = 200.
> So, there must be 0.05 * 330 = 15.5 million library users.
> I'd say the average person that does visit a library does so about once a month. So we can divide this figure by 30 to get ~500,000 daily library visitors, throughout the entire US.
> 500000 / 200 yields about 25000 libraries in the entire US.
> This seems high to me. But compared to the number of pizzerias in new york (6680), it doesn't seem like that high a number.

> The average library, is probably 2 floors on average, with big, public libraries and university libraries being much much larger. This counteracts the smaller libraries.
> Let's say that the average floor of a library is about four times the area of the house that I'm currently in, which kind of makes sense given I have a pretty good feel for the size of this place. I know that the square footage of this is 3600, so multiplying that by 4, I get 13400 sq ft.
> 13400 sq ft * 2 =~ 25000 sq ft. A bookshelf takes up about 10 sq ft, and would contain, by my estimation, about 400 books.
> We can get more precise here. Let's say that the average library bookshelf is double sided, has 8 shelves, and is about 8 feet long, and about 2 feet wide. this yields an area of about 16 sq ft.
> This yields a total shelf space of 8 * 8 = 64 feet, which is 64 12 ~= 664 inches. *
> The average book is, I'd say, 1.2 inches thick, including the cover. This is just from lived experience.
> So 664 / 1.2 ~= 110 books per book shelf. Double sided, that's 220. So fewer than I originally thought.
> 25000 sq ft... The lower floor is usually more open, with upper floors having more books. You need to accommodate for reading areas. Let's say that 20% of floor space is used by actual bookshelves. This also makes the math easier (5000 sq ft of just bookshelves).

> 5000 / 16 ~= 512 / 16 * 10 = 320 bookshelves 110 books per shelf * * 25000 libraries *= 35200 * 25000 = 35 25 * 10 ^ 5 = 87,500,000 books.

Claude suggests that it should be closer to 1 billion. I'd be interested in hearing what you think. If you'd like to discuss it, I've enable 
discussions on the blog repo at https://github.com/zoravur/zoravur.github.io/discussions. 

Like Mr. Miyagi and Daniel-san, Fermi estimation is the fence-painting you need to practice in order to become better at napkin math, and 
eventually systems design. Part 2 will apply the technique of fermi estimation to computer related problems.

<!---
# References
# 1. [Fermi paradox](https://en.wikipedia.org/wiki/Fermi_paradox)
# 2. [Advanced Napkin Math: Estimating System Performance from First Principles](https://www.youtube.com/watch?v=IxkSlnrRFqc)
-->
