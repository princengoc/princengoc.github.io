title: key-clash-erlang-learning
date: 2024-08-21
tags: erlang, seven languages, books are the best, otp, pid, learn you some erlang, key clash, typing game, beginner project, gen server
description: Maybe my daughter would love to type if she can send annoying messages to her parents? Also, Erlang cured my fear of PID.

# Key Clash! A typing battle game built with Erlang.

My initial plan for [Seven Languages in Seven Weeks](https://pragprog.com/titles/btlang/seven-languages-in-seven-weeks/) was to have a capstone weekend project for each language that showcase its essence. Coming up with such a project for each language turned out to be quite hard. It has to be small enough that a complete beginner can do it over a weekend, **and** interesting enough that I would want to do it specifically **in that language** (and not, say, constantly wishing that I'm writing a web app in Javascript). For ``prolog`` I was happy with the [Zebra Puzzle](https://princengoc.github.io/zebra_puzzle.html). But I yearned for a more complex project. 

Tada! Entered ``erlang`` and the ``Key Clash`` game. 

<video autoplay muted playsinline controls width="640">
  <source src="output.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>


Key Clash is like a real-time, two-player Tetris battle but for typing. Two players race through a document on separate computers, typing as fast and accurately as possible. As they progress, they build up Super Power, which can be unleashed to send a Challenge line to their opponent, making their job harder.

The project was an excellent fit for a beginner. Simple enough that I could finish a first version over a weekend. Interesting enough that I did some code refactor (gasp!) for a version 2 before running out of steam. And just complex enough that I found the excellent book [Learn You Some Erlang](https://learnyousomeerlang.com/) very helpful. 

Here's a player review. 

```quote
Q: Do you like the game? 
A: Yes

Q: What do you like about it? 
A: When I get to do a super power and type silly messages. 

Q: Any features that you wish for? 
A: I feel like the score is a bit hard to see. Would be nice if it was bigger or more noticable. 

Q: Who do you want to play this game with? 
A: Grandpa. 

Q: What would you say to Grandpa when you have the super power? 
A: silllly grandpa siiiiilly grandpa
```

All in all, we had fun. If you play it and have suggestions, let me know! I hope that other Erlang learners might find it interesting to fork. Here's [the code](https://github.com/princengoc/seven_languages/tree/main/erlang) with a laundry list of improvement ideas.

### Erlang: the pros

I have always wondered if one can use Prolog for "normal" programming (not just solving constrained problems over finite fields). Erlang was a satisfying answer. Learning Erlang after Prolog was great. I had no troubles with the syntax, concepts, mindset. Without Prolog, I would recommend [Learn You Some Erlang](https://learnyousomeerlang.com/).

I love Erlang one-process-one-thread and let-it-crash philosophy. I like functional languages in general. I like how front-and-center the process ID is. Up until a week ago, my top three use cases for PIDs were the following. 

1. Locate the offending process that needs to be ``kill`` (90% of use cases). 
2. Nervously monitor some process that is gradually chewing up resources (more % at work, less at home)
3. Follow some complex recipe online that says ``look for the PID of such-and-such service`` while pulling my hair out, wondering if it's faster to handcopy that 10-page long document instead of forcing a ten-year-old printer to work on Linux. (This happens every month, about). 

In short, I had an aversion for PIDs. Erlang cured that magnificently. I'm proud to have built something that can send messages across two computers. Wohoo!

### Erlang: the cons

I hate the lack of an easy Erlang UI package. Initially I wanted to have a typing game that listens to keystrokes and colors the matching letters in real time. But [everyone](https://stackoverflow.com/questions/42750491/read-a-character-input-from-erlang-without-requiring-the-return-key-pressed-from) [told](https://stackoverflow.com/questions/37892072/listening-for-key-input-in-wxerlang) [me](https://chatgpt.com/share/f3e950de-3bcb-42ef-90a5-6caa48cd84da) that ``Erlang + KeyListener = Horror `` (and I confirmed this after half a day of wrangling with ``wxerlang`` and cooked vs raw terminals). Which seems like a real, **real** shame. Gaming is the most exciting application to showcase concurrency imho, and few games can survive without KeyListeners. 

After the initial messy implementation, I rewrote using OTP ``gen_server``. It's a tad cleaner, but not hugely so. I had imagined some ``ruby`` level of syntatic sugar and beauty, but nope. Probably this means I'm still an Erlang newbie and the codes could do with a lot more refactor, or that I need to add more features to appreciate the full power of the Erlang OTP. 

Finally, debugging. I still find that quite confusing. We have an Arch and a Nix computer. With ``net_adm:ping``, somehow Nix must ping Arch first, otherwise Arch cannot ping Nix, and I still cannot figure out why. Yes ``epmd`` is running on both, yes they are listening to 4369. Documentation is a bit sparse, and GPT4 (generally very good with these sorts of bugs) couldn't help. Oh well. 

### Conclusion

Erlang was bult for telecom purposes, and I think it really excels at that. I would reach for Erlang as my first choice if I want to write the backend for simple network tools, manage workers, queues, chat applications, and the likes. 