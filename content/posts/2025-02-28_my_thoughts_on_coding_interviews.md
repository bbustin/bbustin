+++
title = "My thoughts on coding interviews"

[taxonomies]
tags = ["Mentoring", "Interviewing", "Career", "CodingInterview"]
+++

Let me start off by saying I am just plain bad at coding interviews. Some people
are fast on their feet and others need more time to think. I need more time to think.

Coding interviews are a way to eliminate lots of candidates when a ton have applied.
They are; however, in no way representative of actual day to day work.
<!--more-->

# What is a coding interview

In case you have not participated in a coding interview, let me explain what
a coding interview is. Coding interviews are typically around 45 minutes long.
They are, most often, conducted over a video conference.

The first 15 minutes or so is usually an introduction and a chance to ask a few
questions. Then you usually need to open up a web-based coding tool. It is
usually like a text editor, though some more advanced ones have syntax highlighting
and allow for running the code.

The interviewer then pastes a coding riddle, asks you to read it, and then
you start. It is incumbent upon you to ask for clarification. You also
need to verbalize your though process. If you get particularly stuck,
the interviewer will often give you a clue or a nudge in the right direction.
They want you to do well too, as they have likely gone through this process themselves
and know how tough it can be.

Once around 15 minutes have passed, the interviewer will stop you and paste another
coding riddle in. There are only 15 minutes left in the interview and the question
must be solved in that time. Once again, you need to confirm your understanding of
the problem with the interviewer and express your thought process out loud.

The idea behind the coding interview is to assess how you think and to
quantify your coding proficiency.

# My thoughts

I do not feel the coding interview is an effective way to assess coding proficiency.
The process is so divorced from the day to day of a Software Engineer that the results
are not representative. What do I mean?

## Kinds of Problems

Software Engineers have to solve new problems all the time. Problem solving is an
extremely important skill to have. Most problems a Software Engineer will be faced
with on any given day will be related to the problem domains of the projects they are
working on. They are somewhat scope-limited.

Coding interviews often present problems like implementing the
[Bubble Sort algorithm](https://en.wikipedia.org/wiki/Bubble_sort). In real life
you will never need to do this. We use libraries to sort. The
algorithms are already implemented. It does not make sense to roll your own
when there can be lots of performance tricks or edge cases to look out for.
It is simply good business sense to use a properly-vetted, popular library. If you
are a bubble sort expert and can implement it from scratch, then I commend you.
That is pretty cool. If you have no clue how to do that, join the club.

Then there are puzzles. The prompt will tell you about a desired behavior,
and then give you example input and the desired output. There is nearly
always some kind of edge case to beware of and the example input will
not reveal it.

Here is an example:
```python
# Given a string with alphanumeric characters and parenthesis, remove the
# smallest amount of parenthesis so they close properly

# Example:
ex_str = "(ase23(fd)343(23)"
result = "ase23(fd)343(23)"
```

This is close to a real problem I was given. So what do we know?

* We can only remove parenthesis.
* If we go through the string from left to right, it should never have a closing parenthesis if we haven't yet seen an opening one.
* Opening and closing parenthesis need to stay in balance. As we go through the string we could possibly remove closing parenthesis when there are too many.
* When we get to the end of the string, there may be too many opening parenthesis. We may need to remove some, but we need to be careful how we do that.

Great. Now create the code to solve this problem. Don't take too long because there is another question
to solve next.

Some problems can be closer to the real-world, like one that might be related to scheduling such as:
```python
# Given two lists of tuples, return a list where all overlapping tuples are merged.
# The first number in each tuple will always be less than the second. The tuples in
# the list will be in ascending order by first value. In any one list, none of the
# tuples will overlap.
#
# Example:
list_one = [(1,3), (5,9), (10,12)]
list_two = [(3,5), (7,8), (13,14)]

result = [(1,9), (10,12), (13,14)]
```

It requires you to merge the tuples together and realize that (1,3), (5,9), (3,5), (7,8) will
all merge to (1,9). Time is almost up, code something!

## Environment

The environment and circumstances coding interviews are performed under do not reflect
day to day realities. These differences can have a large impact on performance.

### Tools

When you have been coding for  along time, you get used to the tools you regularly use.
These tools may, in effect, become a development crutch. That does not change that they
are regularly used because they increase efficiency and help prevent errors. Ultimately,
the end goal is to create solid code that solves the problems at hand for your stakeholders.

Integrated Development Environments (IDEs) help perform code completion. If your IDE is
automatically filling in the name of a function you're trying to call, but now you are
asked to code in a text editor, it stands to reason you may flub some of the names.

IDEs also catch common mistakes. It makes the cost of making these kind of mistakes really low
because it underlines the mistake, often offers to fix it for you, and does not allow you to
commit the mistake to your repository unless you ignore yet another warning. In a rushed coding
interview, it is easy to make simple mistakes. These mistakes may make it appear that you do not
know your stuff when you actually do.

Here is an example of a mistake I made in a coding interview a few years ago.

```java
public class Fruit {
    private String name;

    public Fruit(String name) {
        if (name == null) {
            throw new IllegalArgumentException("'name' can not be null");
        }
        this.name = name;
    }

    public String getName() {
        return this.name;
    }

    @Override
    public boolean equals(Object other) {
        if (other == null) {
            return false;
        }

        if (!(other instanceof Fruit otherFruit)) {
            return false;
        }

        if (otherFruit.getName().equals(this.getName())) {
            return true;
        }

        return false;
    }
}

public class JavaExample {
    public static void main(String args[]) {
        Fruit apple1 = new Fruit("apple");
        Fruit apple2 = new Fruit("apple");

        // mistake - this compares whether they are the same object,
        // not whether they are equivalent to each other
        System.out.println(
            "Incorrect code - Both fruit objects are equivalent: " +
            (apple1 == apple2)
        );

        // correct code - this actually makes sure they are equivalent,
        // even if they are not the same object in memory
        System.out.println(
            "Correct code - Both fruit objects are equivalent: " +
            apple1.equals(apple2)
        );
    }
}
```

Here is the output:
```
Incorrect code - Both fruit objects are equivalent: false
Correct code - Both fruit objects are equivalent: true
```

That was a very embarrassing mistake and I am ashamed to even recount it here.
The mistake could easily lead to unexpected program behavior. It *is* a serious
mistake. My IDE would have underlined `apple1 == apple2` and pointed out the mistake.
If I did not see that, it would also have presented a warning I would need to read through
before allowing a code commit.

In this case, my interviewer asked if I really know Java. That made me feel pretty
low. I made a mistake that would not have occurred had I not been coding under
duress.

I decided to use Python during coding interviews from then on. Java is a much
more verbose language. Even though I use Java for work, it always seemed like an
onerous language without all the tooling to make it somewhat palatable. Python takes
less time and fewer lines of code. This means there are less lines to mess up on.
Pure Python runs slower, but this is just for a coding interview. Python is
a language that is ergonomic to write in without tons of tooling. You will
not have great tooling during a coding interview.

### Thoughts out loud

Interviewers will try to gain insight into several aspects of your thought process.
They want to understand how you think and what your approach is to problem solving.
This might include trying to answer the following questions:

* Do you think logically?
* Do you try to understand the problem before jumping in and coding?
* Do you ask the interviewer about the problem rather than making assumptions?
* Can you anticipate edge cases in advance?
* How do you work through getting stuck?
* Are you open to changing direction based on hints from the interviewer?

The goal to understand how you will approach and deal with problems makes sense.
In order to support that, you need to express your thoughts out loud. This
feels very unnatural to me. It leads me to lose my train of thought and I
find it incredibly distracting.

Most Software Engineers I've met do not regularly express their thoughts as
they are writing code. They clarify requirements with stakeholders
before beginning to code. When working on larger features, they usually first
come up with a plan for how they are planning to implement it to share with their
team and solicit feedback.

This kind of normal communication is very different.

* It is usually at a higher level.
* There is time to think about the problem first, truly understand it, and formulate ideas before
having to communicate them. You do not have to "think on your feet" while the clock ticks down.
* Not all of the communication is spoken. It might be a combination of diagrams, prose,
and meetings.

### Being watched

The interviewer is watching you through your camera, listening to you through your microphone, and
seeing your keystrokes. Knowing that someone is watching every moment feels strange. It makes the cost
of making a mistake feel greater. I tend to self-censor.

Let's say I'm on the right track. I keep trying to think ahead to make sure it is still a good solution
because I do not want to make a mistake in front of the interviewer. Thinking ahead leads me to either lose
my train of thought or come up with a possible reason the current code I am working on is not correct.
Oftentimes that gets me right back at the starting point. I end up psyching myself out.

This may be [performance anxiety](https://health.clevelandclinic.org/performance-anxiety-stage-fright).
I'm not sure. It is not something that would happen if an interviewer were not there watching every moment.
If I was not being watched, I'd develop the way I normally do.

* Get a good sense of the problem to solve.
* Seek any clarification I might need.
* Start working on some code.
* Come up with various test cases to run through the code to catch mistakes and edge
cases.
* Examine the code for simplicity. Can it be made simpler? Is someone else likely to
understand the code at first glance?
* Examine the code for maintainability.
* If a performance-sensitive part of a system, benchmark the code and try to optimize performance.
* Make sure all test cases still pass.
* Put in the code for a peer to code review. Then incorporate input.

There is not time to accomplish all of the above steps during a coding interview. The biggest
difference is starting to code earlier on and using test-driven development. The omnipresent
interviewer watching the process leads me to get stuck trying to come up with the entire approach
to the problem first rather than just getting started and finding out problems experimentally.

### Time

45 minutes. That is how much time you have to solve two coding riddles. In reality it is
closer to 30 minutes as the first 15 minutes are oftentimes the interviewer trying
to get to know you a little. This leaves 15 minutes for each problem.

That is not much time. If you've seen a similar problem before and pretty much know its
hidden pitfalls, you're golden. Maybe you've spent hours on [LeetCode](https://leetcode.com/),
[HackerRank](https://www.hackerrank.com/), or
[GeeksforGeeks](https://www.geeksforgeeks.org/explore?page=1&sortBy=submissions) solving
questions and happen to have seen something similar. If not, there is a good chance
you will get hung up on something. There is simply not enough time.

### Ability to research and consult with colleagues

Software engineers have resources at their disposal. It is not a problem if you
go out onto the internet to research a problem. Your colleagues would also likely
be glad to answer a question here or there. When working, you do not have to have all
the knowledge in your head.

During a coding interview you can not perform any research. These questions have
all already been solved. You may happen upon the exact question and its answer.
That negates the whole purpose of the coding interview as you have now effectively used
someone else's work.

In the real world we are always using someone else's work to a certain degree. There
are many layers of the tech stack we are developing on top of. Someone else did
that work. The language we are coding in compiles down to machine code or bytecode.
Someone else did that work. The libraries included with the programming language as
well as additional libraries we use are all someone else's work.

We build our projects on top of the shoulders of giants. There is pure genius in the
work we are building on top of and incorporating into our projects. I would love to
understand all of it. It is, frankly, not possible to know all of it. It would
take several lifetimes to truly explore it all. The best we can do is become
experts in our specific niches while staying curious and actively searching
out opportunities to learn new things.

# The blessing and curse of being self-taught

One of my superpowers is being able to learn quickly on my own. It has allowed me
to be really successful. It stems from natural curiosity. This is a huge blessing.

The huge downside is that the coding interviews seem to often go back to
Computer Science Fundamentals. These are the fundamental building blocks of
computing taught in universities. I am not going to downplay CS fundamentals.
Quite the contrary, they are what everything builds upon. In day to day
work you *do* need to be aware of them, but you do not need to be an expert
because you have the tools in front of you to research any problem.

I understand, for example, that using a map where you can look up a
value by key is going to be significantly faster than iterating through
a list of values to find the one you're looking for. This concept is
just intuitive to me. I've never designed my own map data structure though.

Through extensive experience, I've been able to build up a very good
intuitive sense. It helps to guide my development. I often take a
["trust, but verify"](https://en.wikipedia.org/wiki/Trust,_but_verify)
approach with my intuition. This can be in the form of benchmarking code or using
[test-driven development](https://en.wikipedia.org/wiki/Test-driven_development)
techniques. This helps to check my assumptions and further hone my intuition.

Not going the traditional Computer Science route has not been a problem
for my career, but it is very much a problem when it comes to coding
interviews. They often are testing familiarity with Computer Science concepts
rather than problem solving and coding ability. Many of the coding riddles seem
to be designed around knowing one specific data structure or algorithm.
If you happen to know it you're golden. Otherwise you're out of luck.

It seems to me like the process is set up in a way that disadvantages
self-taught professionals.

# My advice

If you are currently going through coding interviews, hang in there. Take
time to study and practice possibly leveraging [LeetCode](https://leetcode.com/),
[HackerRank](https://www.hackerrank.com/), or
[GeeksforGeeks](https://www.geeksforgeeks.org/explore?page=1&sortBy=submissions).

Has this advice helped me so far? Not at all. But I will follow it and hang
in there unless someone has a better idea.
