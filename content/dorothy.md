title: dorothy-treasure-hunt-game-clojure
date: 2024-09-04
tags: clojure, seven languages, the joy of clojure, beginner project, gpt4
description: Clojure in one week was a very tall order. GPT-4 sort of allowed me to cheat. And I'm glad it's over.

# Can Clojure really be learned in a week? 

If I could sum up my feeling for ``Clojure`` this last week in one word, it is **deception**. The error messages, the syntax, the style, the books, STM promise of concurrency: everything has a layer of deception. Everything is "not quite what it seems". Occasionally I found it liberating to be duped, as it exposed my weaknesses and hardened me as a programmer. But for the most part I'm tired, annoyed, and just want to get away as fast as I could. 

I thought I had some advantages coming to Clojure as a newbie. I've been teaching ``racket`` to my daughter with [Realm of Racket](https://nostarch.com/realmofracket.htm), so prefix notation and parentheses are a breeze. I've been studiously following [the book](https://pragprog.com/titles/btlang/seven-languages-in-seven-weeks/), so functional programming, tail recursion, or concurrency aren't foreign concepts. I have always cared about distributed computing as well, so ``agents`` felt like ``actors``. Easy right? Well, day 1 and 2 were fine. On Day 3, my hair was on fire. 

In picking ``Clojure`` as one of the seven languages, the book bit off more than it can chew. Over five pages it flew through STM (software transactional memory), ``refs``, ``agents`` and ``atoms``. I struggled to understand STM, or when to use ``refs`` and when to use ``atoms``. I supplemented my learning with [The Joy of Clojure](https://joyofclojure.github.io/), which was... a good reference but very dense, and not exactly joyful.

In the end I built my capstone project and called it a week. Dorothy found her Wizard, and I found my escape out of Clojure land. 

<video autoplay muted playsinline controls width="640">
  <source src="dorothy.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

## Clojure: the cons

There wasn't a one-big-thing I dislike about Clojure, it's more like a culmination of small annoyance that iritates. First, there are too many macros leading to too many shortcut syntaxes to learn. Right from the start, we have ``def``, ``defn``, then ``#()``. Here are three different ways you can ``add``

```clojure
(def add (fn [a b] (+ a b)))

(defn add [a b] (+ a b))

(def add #(+ %1 %2))
```

``%`` is your implicit argument. ``#()`` is your anonymous function. Now, one of this is the "right way" per community convention. (I'd say the ``defn`` in this example). But in other cases I'm not sure which is "right". It makes parsing codes extra hard. And it's not just functions: I could write ``\D`` or ``'D`` to mean the character D. As far as I could tell, ``' `` doesn't work to represent a space, I need to write ``\space``.  Well - why bother with the ``'D`` notation then??

The issue was exacerbated by my reliance on GPT4. It would rewrite my codes with sleak macros that I didn't know about. Here is an example. 

```clojure
    (if (can-move? target-position) 
        (swap! game-state
              (fn [state]
                (-> state
                    (assoc :dorothy target-position)
                    (update :visited conj target-position))))
        )
```

I learned that ``->`` is the ``thread-first`` macro, used to remove nesting and make codes more readable. Well, it would be more readable if I was introduced to it in the first place, not stumble upon it like a random Latin word in the middle of a French text. It's hard enough to learn French, it's extra annoying when new words kept on popping up while you're practicing. What's worse, the ``->`` is only used once through my project. Without repetitions, I just could not build confidence or affinity for Clojure. 

Related to this issue is kebab case. It doesn't help that ``VSCode`` sucks at dealing with kebab cases and indentation for Clojure. I cannot tell if I should write
```clojure
  (let [A a
        B (if dog? woof meow)
        C (some-horribly-long-block arg1 arg2 
            (more-nesting arg3)
          )
       ] 
    (do
        ...
    )
  )
```
or if I should keep things inline, or some mixture of both, like
```clojure
  (let [A a
        B (if dog? woof meow)
        C (some-horribly-long-block arg1 arg2 (more-nesting arg3))] 
    (do ...)
  )
```

GPT4 recommends this sort of formatting
```clojure
(defn display-grid []
  (clear-screen)
  (let [[cx cy] (:dorothy @game-state)
        size (:grid-size @game-state)  ; if size is 2, this will create a 5x5 grid (2 to each side of current-position)
        positions (for [y (range (- cy size) (+ cy size 1))
                        x (range (- cx size) (+ cx size 1))]
                    [x y])]
    (doseq [row (partition (inc (* 2 size)) positions)] ; partition positions into rows
      (println (apply str (map (fn [[x y]] (generate-board x y)) row))))))
```
which is fine but took some time to verify that the brackets close up, for example, so that everything is still in the scope of ``let``. I constantly had to squint, on top of thinking about prefix and broadcasting. It's a shame that stylistic issues like these stop me from getting to the Joy of Clojure. But style matters. 

Then there's interop with Java. I have an irrational dislike of Java. It was my first language, and at the time I hated how verbose Java is, how ugly Swing looked, how confusing the error messages were, how everyone around me said "oh you should write it in C". I am sure Java is good for something, it's just never *my* thing that needed Java. So Clojure with the Java interop greatly annoys me. Java crept up in the error messages, and OMG those things are incomprehensible. ``lein repl`` didn't work so I didn't have a functioning REPL, and had to rely on honest debugging. Event GPT4 at one point gave up. 

> 2. Possible Source of the Error:
>
> The error message Unable to resolve symbol: position in this context suggests that position is not recognized in some context, but based on the functions you’ve provided, it doesn’t seem like a problem directly within these function definitions.

Then there's ``ref, agent, atom, var``. Seven Languages was too brief, The Joy of Clojure helped, but it kept on mingling anti-examples with examples, making it FAR from joyful to read. Take for example, Chapter 10.2.2 titled ``Commutative change with commute``. It started out like this

> Figure 10.8 showed that using ``alter`` can cause a transaction to retry if a ref it depends
on is modified and committed while it’s running. But there may be circumstances
where the value of a ref in a given transaction isn’t important to its completion seman-
tics. For example, the ``num-moves`` ref is a simple counter, and surely its value at any
given time is irrelevant for determining how it should be incremented. To handle
these loose dependency circumstances, Clojure offers an operation named ``commute``
that takes a function to apply to a ref.


Great, so you would think that update ``num-moves`` with ``commute`` will be our example here. Nope. Turns out that ``num-moves`` is not compatible with ``commute``, because the function that runs ``commute`` will be ran twice. ``place-piece`` is fine since it's idempotent, but the update of ``num-moves``, which just increases the value of ``moves``, is not idempotent. Then why present this example in the first place??!! 

Then there is STM. The introduction of both Seven Languages and The Joy made me think that it's some magic bullet to resolve multithreading issues without locks (amazing!!). I had to read until the end of the chapter, and tinker with toy programs, to understand that there's no magic. In fact, there's quite some amount of opacity around thread resolution; I cannot be sure when I read codes dry which threads would be asked to retry when. Chapter 10.2.4 had a good example to stress-test refs, and I understood the role of ``:max-history``, but still have not grasped the role of ``:min-history``. (Why can't the min always be 0?) 

## Clojure: pros

Stepping back, all my issues with Clojure are about the cosmetics rather than Clojure itself. There are things about Clojure I do like: the local scope of ``let``, the concept of a transaction block, shared memory, quoting, lazy evaluation. I have yet to be enlightened by ``macros``, but I trust the gurus of the world that they' great. I would reach out to Clojure if I inherit a large Java code base and want to write in a more functional style. 

## GPT-4 and programming at a higher level is bad for beginners, good for veterans

GPT-4 is good at Clojure. Not everything it wrote is bug-free, but I found that it enabled me to switch into a "meta" mode. Instead of worrying about syntax issues, I can scrutinize the codes that it write, think about corner cases or redesigns and restructuring. I often used the first version that GPT-4 write as the first draft, then build on it. It took a lot of discipline to *not* query GPT-4 in subsequent rewrite, and I tried to use the API as much as possible. 

But I would in general **recommend against** this use of GPT-4 for learning languages. ``Clojure`` is the *only* language out of Seven Languages where I relied a lot on GPT-4 to draft, and it comes at a steep cost: I'm not as proficient at Clojure as I am at, say, Erlang or Prolog. But most importantly, relying on GPT-4 to rewrite codes made me **tired and annoyed**: I didn't walk away feeling that the time was well spent. I sometimes barely advanced my Clojure knowledge even though I read a lot of Clojure codes, and if I did, I would have promptly forgotten how to write ``doseq`` the next morning. It took the joy out of programming and replaced it with a productivity treadmill, a feeling that I just wanted to "get it to work". 

It's probably a bit like learning to solve word math problems without a good grasp of the basic operations, and ne had to rely on a calculator as the crutch. If one already knew how to add, the calculator is a productivity tool. If one doesn't know how to add, one feels like an imposter. 



