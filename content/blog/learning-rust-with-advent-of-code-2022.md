---
title: "Learning Rust With Advent of Code 2022"
date: 2022-12-27T06:44:27-05:00
draft: false
---

<meta property="og:title" content="Learning Rust With Advent of Code 2022" />
<meta property="og:description" content="What I learned attempting AoC 2022 in Rust (with Copilot)" />
<meta property="og:type" content="website" />
<meta property="og:url" content="https://peterbaumgartner.com/blog/learning-rust-with-advent-of-code-2022/" />
<meta property="og:image" content="https://i.postimg.cc/CKCfxpsW/image.png" />

This year I decided to participate in [Advent of Code](https://adventofcode.com/) (AoC) and use it as an opportunity to learn Rust. Since I was learning a new language, I also decided to try and use GitHub Copilot within VSCode. 

AoC is a series of 25 daily puzzles that typically need to be solved through programming. Each daily puzzle consists of two parts. This year I fully solved 18 of the 25 daily puzzles and solved part one of an additional 2 puzzles, - so I submitted 39 of a possible 50 answers. 

## Using Advent of Code for Learning

I found AoC a good motivator to write code every day. I'm an early riser, so almost every day I was up at 5:30am doing my morning routine of  making [a nice cup of coffee](https://youtu.be/GrQcVU-eapc) with my Aeropress and then working on that day's problem. In the first 10ish days, I was able to solve it within a few hours and continue my day as normal. Once the puzzles got more difficult, I usually started in the morning and then waited until after my workday to finish it. If I had spent a few hours on a problem and didn't have a clear path to a solution, I would review other solutions online to find what I was missing.

I was on three "private" leaderboards that provided a mixed bag of motivation. One leaderboard I was on was for [Chris Biscardi's youtube channel](https://www.youtube.com/@chrisbiscardi) (more on that later) and was filled with some really productive programmers and puzzle solvers, so I'm currently in 64/188. Another was a leaderboard [Joel Grus](https://www.youtube.com/@JoelGrus) shared in the [NormConf](https://normconf.com) slack - where I'm currently 10/15. Finally, I was in another private leaderboard from someone I know from LinkedIn - where I'm currently 3rd (!!) of 44 (though only 13 participants have no solutions). A former colleague of mine who is also learning Rust participated in this last leaderboard, and it was nice to have someone near my skill level to discuss solutions with. Generally I think the leaderboards can be motivating if you're able to assess your own ability and set your expectations relative to the skill of others on the leaderboard. In this case, I knew I wasn't going to be someone who is up at midnight solving the puzzle instantly, and that I wouldn't be able to solve all the puzzles, so I'm satisfied with where I'm at on all these private leaderboards.

Sometimes the motivation to solve the puzzle quickly was in conflict with my goal to learn. After the first two weeks of puzzles, I was familiar enough with the language that wasn't making basic mistakes and I had a good workflow for each day's problem. I also knew which third-party crates to reach for for certain types of problems, so that I wasn't having to reimplement a lot of things from scratch. My learning and puzzle solving was also assisted with Copilot, which impacted both my speed of solving and what I was learning.

## Learning with Copilot

Copilot was helpful, but it was also another thing that I had to learn how to use effectively. Once I had figured out how to interact with it in a productive way, I found it most helpful as a tool that was there to help me write better structured programs, while the specific implementations of functions within those programs that it suggested might imperfect. While I used it primarily to generate boilerplate code for fairly simple things, that was often code I could write myself and there are probably situations I should have done so.

Copilot is a bit tricky to use because can be subtly wrong and create a bug that goes unnoticed or it can be extremely wrong and suggest something you know is incorrect. I found myself programming _around_ this behavior and trying to use this to my advantage. One thing I noticed is that when it struggles and provides an obviously wrong answer, it's actually a good hint that I need to break some logic up into smaller pieces. A good example of this was day 23. I asked it to generate one big function for rotating through cardinal directions for a given coordinate pair and performing a check on neighboring positions in a grid. It got so confused after my prompt[^1], I realized I needed to pull out the behavior that generated the neighboring positions as well as the logic to cycle through the directions into separate functions. Even after that, it wasn't able to generate a correct function, but it generated something logically similar enough to what I wanted that I used it as a template and made my own corrections.

Parsing the input was the best use case for getting a correct Copilot suggested implementation on the first try. Specifically, it was really helpful for writing regex-based parsers. I would prompt it with an explanation of the input followed by roughly a dozen examples. If there were edge cases (negative numbers, singular values in a list) in the input it would give me an incomplete solution with a too-specific regex. I had to go back and add "don't forget `this thing you messed up the first time`" to the prompt and then it would generate a correct regex.

Frequently Copilot got in the way and I felt the suggestion time was too aggressive or I would want less output than the full autosuggestion -- for example, the function signature was correct but the logic was incorrect. When you're writing the first part of a solution, this is definitely annoying to deal with because it's suggesting things that obviously don't make sense (it has very little context), but is somehow confident enough to suggest. 

My general principle for using Copilot became treating what it suggested as a template and not 100% correct, using what it had generated and making appropriate corrections. This was an especially useful approach for things like display functions, which often involved iterating over a grid -- a problem where the general structure of the solution is clear enough for Copilot to get right, and specific implementation details I can modify on my own.

When using something like Copilot to learn a language, I think it's best to be careful and review what it's suggesting to you line-by-line. Sometimes it generated code with details that I fail to notice, thus implementing something I didn't understand but then unconsciously ignored. For example, this parser for Day 23 uses `move` in the inner `flat_map`, which I didn't notice until I was reviewing after the problem was solved[^2].

```rust
fn parse(input: &str) -> Vec<IVec2> {
    input
        .lines()
        .enumerate()
        .flat_map(|(y, line)| {
            line.chars().enumerate().filter_map(move |(x, c)| match c {
                '#' => Some(IVec2::new(x as i32, y as i32)),
                _ => None,
            })
        })
        .collect()
}
```


In summary: I liked using Copilot, but I wish I had finer grained control over turning it on and off as well as what it would output. For example, having something like "suggest only the function signature" would be extremely useful. I also found it helpful if I could tell I was giving it a "[draw the rest of the owl](https://knowyourmeme.com/memes/how-to-draw-an-owl)" problem that wasn't sufficiently decomposed into smaller problems. 


## Learning & Using Rust

I've spent most of my programming career with python and I was excited to learn a bit more about Rust after seeing a [few](https://www.pola.rs/) [awesome](https://github.com/pydantic/pydantic-core) [uses](https://github.com/huggingface/tokenizers) of [PyO3](https://pyo3.rs/v0.17.3/). I've had a few use cases where python has been a bit too slow, and I've previously explored [Julia](https://www.peterbaumgartner.com/blog/incorporating-julia-into-python-programs/) and [Cython](https://www.peterbaumgartner.com/blog/intro-to-just-enough-cython-to-be-useful/) as solutions for this. 

My prior experience with Rust starts in 2018, when I attempted to go through [_the book_](https://doc.rust-lang.org/book/). I remember struggling quite a bit implementing the exercises at the time and not finishing my read through. I didn't really have enough computer science knowledge to understand what was going on and why all the concepts Rust uses are necessary. I attempted to pick things back up again earlier this year (2022) by reading through [Rust in Action](https://www.manning.com/books/rust-in-action) and completing exercises on [Exercism](https://exercism.org/tracks/rust).

For Advent of Code, I didn't want to spend a lot of time constructing my own tooling, so I used an existing [template](https://github.com/fspoettel/advent-of-code-rust) that provides scaffolding to create a code file and a file for your puzzle input every day. It also creates a test within the daily file which will use the primary `part_{one,two}` function, the example input, and the answer given the example input.

Coming from python, figuring out an exploratory programming workflow in Rust was _painful_. In python I'll typically use an [IPython](https://ipython.org/) REPL which I'm very fluent with and has some nice utilities for more interactive programming. This is nice for something like AoC because you can incrementally program your way towards a solution, keeping around whatever current state you have on your way to figuring out the next step. I'm used to interacting with actual instances and data of the things I'm working with first, and *then* creating some abstractions capture their behavior (rather than thinking about what abstractions make sense first).

After struggling a bit to figure out what my Rust workflow would be, I eventually settled on a process like this:
1. Fill in the details for the test that's provided for each part - this is usually the example input and the answer given that input.
2. Run the failing test incrementally whenever you figure out a component of the program and do lots of printing. For example, after I figured out a parser for each day, I print the output of the parser and visually inspect the output.
3. When things inevitably aren't working, do some combination of the following:
	1. Use the debugger in VSCode. I used this a handful of times on problems that had some complex state I wanted to track a piece of. I would set a breakpoint at some step of a complex function and check where variables weren't what I expected.
	2. Do some print debugging with `println!`. Immediately forget that I didn't implement `Debug` on my `structs` yet. Derive `Debug` on them. For puzzles that have a visual component, write a function (or `Display` implementation) that would display the current state.
	3. Sometimes when inspecting the inputs to a function, the `dbg!` macro is also useful. The debug macro prints the thing you pass it and returns that thing as well, so you can just wrap things with it. It's a really neat tool!

My struggle with an exploratory programming workflow made me realize that I'm a bit of a _sloppy_ programmer when exploring a new problem space and python has rotted my brain a little bit in this regard. Python doesn't force me to organize my thoughts or come up with abstractions _at all_, it's totally possible to solve a problem with a million different variables comprised of native types, relying on some intermediate state that's difficult to replicate and understand, and just treating my code as if I'm building one giant calculator.

Rust forces me to organize the things you work with in a helpful way. The downside is that I think new users of the language have to have a good understanding of two things: what are common "operations" performed on things and how they're done in Rust. These "operations" are often invisible when using another language because you do them so frequently or the language is doing some magic for you. 

One example is integers in python: you almost never have to think about whether they're signed or unsigned. If you want to add two integers together, you can just do it. You also don't have to deal with overflow, a concept I'm assuming most python users have never even thought about. Numbers work like they do in the "real world", not like how they _actually_ need to work on a computer.

Another example is printing. Most things you can just call `print` on in python and you'll get something useful. Even when you don't get something useful, at least your program doesn't crash! 

After getting used to this, I found it helpful that Rust _did_ force me to think about these minor but common operations more. Checking for overflow was really useful in AoC when you had a grid implemented as positions with unsigned integers and it was impossible for something to be in a negative coordinate on a grid. In this case it reminded me I need to write the logic for handling the case in which I wanted something to move "outside" of that grid.

I think the problem for beginners is that they are going to encounter these very basic problems and they have to know that there are standard ways to solve this class of problems within the language. One example here would be something like parsing a string into a custom type. Not only does a user have to know that it's probably more idiomatic to implement the `From` trait instead of writing a `string_to_thing` function. But they also have to think about whether that process can fail, and _then_ also know the thing to do in that case is implement `TryFrom`. Oh, and by the way, if you're dealing with going to/from strings there's actually [specific traits](https://doc.rust-lang.org/rust-by-example/conversion/string.html) to implement that they would have to know about! And, even when they figure all that out, they've got to realize when they call that function they're going to use something like `thing = string.parse::<Thing>().unwrap()` instead of something obvious like `thing = string_to_thing(string)`.  And on top of that, someone is going to review their code and note they probably shouldn't use `unwrap` but instead find a way to handle the case when the parsing fails, and that there's some good crates for that, but they'll have to learn about how to use _those_ crates first. And this is all just because they tried to convert a string[^3] to a thing.

This experience makes me appreciate both languages more. Python is amazing because it allows you to not think about a whole bunch of things that are _usually_ fine. It doesn't bog you down with having to learn a bunch of computer science concepts in order to be productive. That's super important -- when I took my first (and only) CS class in college, we used C++. I loved solving problems in that class, but the professor, curriculum, and tooling were so awful that the thought of doing that as a profession never entered my mind. I have a feeling that if we had done the course in python, and not spent half the semester learning about _pointers_, things might have clicked with me a little better. 

But, I don't think the productivity gained from python is always a good trade off. I recently got bit by a self-imposed bug where I wanted to filter a list of numbers to get their actual value. What I wanted was something like `Option<T>` in Rust, but all I had was the ability to define a value as `None` or `int`. I used a list comprehension of `[value for value in numbers if value]`, which also ended up filtering out `0`s, which I really cared about! I've since made it a habit of never using falsy behavior and using `is not None` instead. This is a type of bug that Rust just wouldn't allow me to write. 

I like that Rust forces me to think about certain problems before I have them. I find this particularly helpful because when I'm solving a problem in a new domain, I don't want to be thinking about how to solve problems created by my own lack of precise behavior definitions or whatever initial representations I come up with. I like to think that Rust helps prevent me from _underspecifying_ what my code does.

### My improvement plan for Rust
By day 12 I felt proficient enough in Rust to write solutions that would compile and run without too much battling. At that point, the bottleneck for me solving problems was my own problem solving ability and weaknesses in some CS concepts (e.g. graphs, recursion, data structures, algorithms). But I also know I was not writing idiomatic code, I wasn't always clear on what to do when code didn't work, and because I was using Copilot there were definite gaps in my understanding of why certain generated code worked.

Given that, my priority as I continue to learn Rust will be concepts within the language itself and understanding idiomatic code. Here's what I plan on doing next:
- Complete the [rustlings](https://github.com/rust-lang/rustlings) exercises. I like writing code to better understand something, and I think these exercises highlight unique aspects of the language, so I'm going to focus on them.
- Review the top Rust repos for AoC 2022, identify common tools, concepts, and idioms they use, and better understand how they work.
- Practice reading Rust documentation. I still struggle a bit to read the method signatures and understand exactly what's going on in a lot of documentation. 
- Remove Copilot and focus more on what rust-analyzer can do
- Every time I encounter an error in future code, build a [MRE](https://en.wikipedia.org/wiki/Minimal_reproducible_example) that replicates that issue and then write a solution

## Resources

I used a few different resources as I was working my way through learning. Here they are in order of usefulness
- [Chris Biscardi's youtube videos](https://www.youtube.com/@chrisbiscardi)
	- Chris solves all the problems in a consistent way and his videos are always under 60 minutes. He uses `nom` to parse the input and then thinks about a simple way to structure the problem. For certain problems, he also makes use of other crates in the rust ecosystem when appropriate.
- [GitHub Search for "aoc2022" in Rust](https://github.com/search?o=desc&q=aoc2022+language%3ARust&s=stars&type=Repositories&l=Rust)
	- This was nice to see other ways of approaching the same problem, and of course some weirdly esoteric or efficient solutions.
- [r/adventofcode](https://www.reddit.com/r/adventofcode)
	- Honestly, the memes are a welcome relief. I had fun predicting what the memes would be if I had solved the problem already, and if I hadn't, they were a nice way to lighten my mood if I was frustrated on a day's problem. 
	- I don't look too deep into individual solutions on the daily megathreads because I find browsing all the comments extremely painful - some people post their solutions, some link to a gist or pastebin, others just talk about how hard it was. It's hard to find interesting things in there - which is why I preferred searching GitHub.
- Mastodon
	- After an impressive job by Copilot to generate some code for me, I [posted it to mastodon](https://fosstodon.org/@pbaumgartner/109445384514055645). Some folks replied noting some issues in that code. I asked those that replied what would make that function better and got some helpful replies. 
	- They also suggested using `nom`, which Chris Biscardi also uses in all of his solutions, but I didn't feel like learning about [parser combinators](https://en.wikipedia.org/wiki/Parser_combinator) and how to use a new library on top learning a new programming language. It's on my todo list for next year.
- [fasterthanli.me Advent of Code 2022 Series](https://fasterthanli.me/series/advent-of-code-2022)
	- Most of the time the solutions here felt like they used more intermediate or advanced Rust concepts, so it was nice to review for alternatives but not something I'd read to understand a straightforward way to solve a problem.


## Conclusion

Solving Advent of Code problems in Rust, assisted by Copilot, was a nice way to get some initial familiarity with the language. AoC is a nice structure for providing contained problems to solve and the leaderboards can be motivating if used right. Copilot can be a helpful programming tool when you treat it's output as a template that will need tweaking and review it's output for things you don't yet understand. I had an enjoyable time learning and using Rust and I'm already looking forward to the next time I get to use it!


[^1]:  The prompt I gave was the following, where bold parts indicate additional prompting I provided to attempt to correct for wrong implementations it was giving me: "For an elves location, check whether there is another elf in the neighboring positions. For example, if the direction is North, check whether there is an elf in the N, NE, or NW position. relative to that elf's location. If there is an elf in that position, check the next direction. If there aren't any neighbors in that direction, return the position they would move to if the headed in that cardinal direction. If there are no available moves, because that elf is surrounded, return None. **Remember for each direction we need to check that direction and the two neighboring diagonals. Also remember that if we cycle through all directions and this elf can't go anywhere, return None.**"
[^2]: I tried to prompt Copilot for an explanation here, adding a comment before the move that said "we need a `move` here" and it finished with "because we're using the `y` variable from the outer scope." 
[^3]: This is of course assuming this hypothetical beginner has also figured out how strings work in Rust ([the highest upvoted Rust question on Stack Overflow.](https://stackoverflow.com/questions/24158114/what-are-the-differences-between-rusts-string-and-str))