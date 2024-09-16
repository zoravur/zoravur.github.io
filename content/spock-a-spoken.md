+++
title = "Spock: A Spoken Programming Language"
date = 2024-08-13T00:00:00.000Z
draft = false
tags = ["projects"]
+++

In December 2023, I came up with the idea of a programming language you would be able to speak out loud, into your phone or computer, hands-free. I thought it was an interesting idea for a couple of reasons:

- It would allow you to program on the go without a laptop, while still approaching threshold efficiency (130wpm output[^1]). Also, reducing the time from thought → action seemed like a powerful way to increase productivity.
- It would be an interesting experiment to see how voice programming would change software: Would being able to speak algorithms increase fluency with them? Would programming languages naturally become more terse and expressive, mirroring natural language? Would voice-written software libraries become spaghetti, or would the necessity of fitting the entire problem domain in your brain at once lead to more elegant solutions?
- There are many voice assistants which run code on your behalf, but none that are Turing complete. A language that allows you to communicate any task that a computer can conceivably do would be the ultimate in power and flexibility.

At the time, I couldn't really pursue the idea due to other projects, nor had I really done any work in programming languages since second year, so I was forced to give up on the idea. Nonetheless, I briefly reviewed my course notes and kept the idea on the back burner.

In the past week, I've decided to pick the idea back up, and due to the beautifully written [Crafting Interpreters](https://craftinginterpreters.com/), along with the real-time speech-to-text software [whisper_streaming](https://github.com/ufal/whisper_streaming), I've been able to make surprisingly swift progress toward making the language a reality. Here is some example code (euler1.spk):

```text
define sum of multiples as lambda takes limit and divisor does begin
    define m as call quotient with limit and divisor
    define total as zero
    while m begin 
        set total as call sum with total and call product with m and divisor
        set m as call difference with m and one
    finish
    return total
finish
define total as call sum of multiples with 999 and 3
set total as call sum with total and call sum of multiples with 999 and 5
set total as call difference with total and call sum of multiples with 999 and 15
print total
```

As you may have been able to guess by reading (either the code or the file name!), this code prints 233168, and is the solution to [Project Euler Problem 1](https://projecteuler.net/problem=1). (As there are no non-scalar data types, I stuck to more mathematical problems while building up the language.)

Of note, this language consists of only numbers and easily pronounceable English words, and numbers are interchangeable with their word equivalents. Symbols can be multiple words, due to a carefully constructed grammar. There is no punctuation of any kind, and in fact, all punctuation and indentation is stripped during preprocessing. This leaves only a list of words, which are ideally 1-1 mapped to sounds.

The language features themselves are similar to Lox, except this language is (to start) even simpler. Classes are missing, and regular functions are replaced with lambdas—given that lambdas are just functions that you can pass around, it seems perfectly reasonable to just replace functions with lambdas. Recursion is possible, although that's only due to an unintuitive bug (arguably)—I'm referring to the fact that the language is dynamically scoped, instead of lexically scoped.

Classes and types are missing, and the interpreter itself is written in Python—which means all I'm doing is translating Spock to a highly restricted subset of Python. This makes for a very slow and cumbersome language, for now.

I hope to implement it as a bytecode VM as described in part 3 of Crafting Interpreters, written primarily in C, but also with a Foreign Function Interface to Python, that allows me to steal a lot of the useful code that already exists in Python libraries, for all different types of automation and machine learning.

Other features I want to include are: garbage collection, a standard library, and static typing. I also want to add facilities to the interpreter to improve the developer experience (being forced to one shot your code without misspeaking even once is too difficult to be useful).

### Working with Voice

Working with whisper_streaming and programming with voice has been interesting so far. It's not really natural for me yet, and I'm still agonizingly slow compared to how quickly I can write code with a keyboard and screen in front of me. Nonetheless, with very straightforward code, or when I've pre-written the code and I'm just saying it into Whisper, I can definitely say it much faster than I could ever type it.

If it were this simple though, then I'd be well on my way to making this idea a reality. One of the big hurdles, however, is *working around the limitations of Whisper.*

Whisper is an audio-to-text model, designed mainly for the purposes of transcribing media. As such, it automatically performs a number of tricks and hallucinations that allow it to generate an unnaturally polished transcription of most of the media in its training set. The problem, however, is that Spock follows a different grammar to regular English, and Whisper alters the text stream in subtle but important ways. Here are a few hurdles that I've encountered:

- Mixing up similar sounding words: n, and, end
- Whisper removing repeated words: an earlier version of the grammar denoted argument lists using the words "left" and "right" for left and right parentheses. Some programs would have multiple "right" parentheses next to each other, which Whisper would then deduplicate, presumably because it's assuming that the speaker is stuttering, even when they are not.
- The infamous ["Thank you for watching!" bug](https://www.reddit.com/r/ChatGPT/comments/15mmraf/why_does_it_keep_transcribing_thank_you_for/), and random text in other languages: on silence, whisper transcribes a bunch of garbage (quite frequently), presumably because the subtitles it was trained on would contain some text that isn't matching the audio that it should. I imagine something similar to threshold probabilities would remedy this, but this is not easily configurable within Whisper.

I've addressed the first two by modifying the grammar slightly, (notice, "begin" is normally paired with "end", but here it is paired with "finish", and instead of left and right, we have lists joined with the word "and". The logical AND function is consequently called conj), but the garbage transcription is something I'm still dealing with.

One idea that I had for improving the annoying situations where the transcription fails is fine-tuning—by providing labeled examples of programs, I could greatly improve Whisper's ability to detect the words that are common in the language. Hopefully, this would deal with the garbage transcription as well, as some of these examples may contain long pauses in which I'm thinking about the next line of code to write.

I have more to say about the fine-tuning problem (it would fix the grammar of the language, possible methods of generating data/programs), but I think I'll continue this part of the discussion in a future post.

### The Far Future

With a large repository of language data, one may see spoken programming languages diverge from how English is spoken (though cross-pollination would always exist). Perhaps, like natural language, language constructs could be assigned to shorter and shorter sounds depending on how frequently they are used. One could maybe group syntax based on the pauses between words, allowing one to generate mathematical expressions by saying things in exactly the right way. Maybe one day, we'll all be talking to computers—[like in Star Trek](https://www.youtube.com/watch?v=Drr6_zikuZQ): perfectly logical and precise, not flaky, unreliable, and too-emotional LLMs.

## Notes

1. The average speaking rate is 100-130 wpm according to Google, and I know people personally that type 130+ wpm, so it seems that speaking is comparable to typing in terms of max speed. Typing may win on an increased character set, but a fast speaker can easily go 160 wpm, so my not-so-scientific guess is that they're pretty comparable. Both, of course, are ultimately limited by the speed of thought.
