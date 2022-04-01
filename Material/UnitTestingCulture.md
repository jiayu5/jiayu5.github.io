# Goto Fail, Heartbleed, and Unit Testing Culture

[TOC]

## 1. Abstract

*This article considers the role unit testing could play, showing how unit tests, and more importantly a unit testing culture, could have identified these particular bugs.* 



## 2. Two serious flaws

1. Apple’s “goto fail” bug and OpenSSL’s “Heartbleed” bug.
2. I wrote these tests to validate my intuition, and to demonstrate to others how unit tests could have detected these defects early and without heroic effort.
3. I hope to establish a compelling case for the adoption of unit testing as part of everyday development.
4. It troubles me that most of these analyses fall back on facile excuses that miss the mark, and promote resigned acceptance of such defects due to the ever-increasing complexity of modern software systems.
5. Bugs will happen, but neither software developers nor the public should be satisfied with that as a response to defects this colossal in scope. 



## 3. Goto fail

> A short-circuit skipping the final step of the SSL/TLS handshake algorithm left users vulnerable to a man-in-the-middle attack, whereby a malicious system relaying traffic between an affected system and another system could present the illusion of a secure connection using false credentials, and subsequently intercept all communications between the other two systems.

```c++
if ((err = SSLHashSHA1.update(&hashCtx, &signedParams)) != 0)
    goto fail;
    goto fail;
```

1. In C, there is no essential problem with or confusion surrounding the use of `goto` in this context. In other words, `goto` should not be considered harmful here.
2. The algorithm gets short-circuited by the extra `goto fail` statement.
3. Coding style requiring the use of curly braces for all `if`statements or enabling unreachable-code compiler warnings could have helped. 



### 3.1 How Could Unit Testing Have Helped?

Focus on the external effects of the code:

- What is the contract fulfilled by the code under test?
- What preconditions are required, and how are they enforced?
- What postconditions are guaranteed?
- What example inputs trigger different behaviors?
- What set of tests will trigger each behavior and validate each guarantee?

**Note:**

1. For higher-level or more complex operations, such close "mirroring" can make for brittle tests and should generally be avoided. 

2. *It's arguably even more important to test that the code doesn't do what it shouldn't do.*



### 3.2 Proof-of-Concept Unit Test

1. This test was written without a testing framework, to demonstrate that an effective test can be written using tools already in-use by a project. 
2. Well-organized test cases using well-organized objects and functions with well-chosen names mean that if a test fails, you can usually diagnose the failure from the information in the test case alone without digging through the full implementation of the test program. 
3. The tests act as a double-check: once with examples, once with logic. 
4. The automated double-check provided by unit tests provides a fast and painstaking (yet painless!) code review, in the sense that the tests will likely catch potential merge errors before a human inspects the merged code. 
5. It helps reveal mistakes made by programmers far into the future.
6. Unit testing introduces pressure to minimize copy/paste, because the copy/pasted code also has to be unit tested. I



### 3.3 Cultural Implications

- This is evidence of a development culture that tolerates duplicated, untested code.
- Not shaming, not condemnation, etc., but accountability and the due diligence that follows it. 



## 4. Heartbleed

> The bug enabled an attacker to send an empty handshake request and declare that it sent up to 64k of data; a vulnerable system would read but not verify the declared size and would respond with whatever contents resided in up to 64k of its memory adjacent to the request buffer. 



### 4.1 How Could Unit Testing Have Helped?

> payload_length:  The length of the payload.
> 
> [...snip...]
> 
> If the payload_length of a received HeartbeatMessage is too large,
> the received HeartbeatMessage MUST be discarded silently.

The protocol spec provides a strong hint that `payload_length` should receive special attention.



### 4.2 Proof of Concept Unit Test

Test cases.



### 4.3 Break It Up, Break It Down

1. A smaller, well-tested change containing only the above functions could have better enabled the author, the reviewer, or an interested onlooker to notice the use of an externally-supplied value to read a block of memory, and to verify that such a value had been handled properly. 
2. This would be in addition to requiring that all code submitted for review be covered by new or existing unit tests as a matter of policy.



### 4.4 If It Ain’t Tested, It Ain’t Fixed

*No bug is considered properly fixed without an automated regression test.*



### 4.5 Linus's Law Revisited

> *Given enough eyes, all bugs are shallow*.

**A corollary to Linus’s Law:**

*Not all eyes that notice bugs in Open Source code belong to saints who will report or repair them in the interest of the public good.*

**Benifits of Open Sourced code:**

*Having access to the Open Source code enabled me to submit my unit/regression test to the central OpenSSL source repository.*

**Why didn’t the teams responsible for the code write or insist upon such tests years ago at the time the bugs were introduced?**

1. The development teams at many companies are focused on high-level business goals, lack any direct incentive to improve code quality, and perceive an investment in code quality to be at odds with shipping on-time. 
2. Many Open Source projects are volunteer-driven, and the central developers are short on either the time or the skills required to enforce the policy that each code change be accompanied by thorough, well-crafted unit tests. 



## 5. The Costs and Benefits of a Unit Testing Culture



### 5.1 Startup Costs

1. There will be a learning curve. It will cause an initial slow-down in development as people grow accustomed to the practice.
2. This is a one-time cost. The learning curve is steepest for teams that don't have any unit testing practices at all.
3. Poorly-written tests can actually be worse than no tests at all, leaving the impression that testing is a waste of time. 



### 5.2 Training

1. As the code base grows and more developers join the team, the value becomes increasingly clear;
2. If developers are not motivated to research available materials and improve their skills, or just don't have any idea how to begin, this may imply the need to invest in internal training programs or to contract outside help to provide training. 

> [Code coverage](http://en.wikipedia.org/wiki/Code_coverage) is a measurement of how much code is actually executed by a test or test suite. It can help identify areas of a system that have not been sufficiently tested.



### 5.3 Painting Yourself Into a Corner

1. You eventually learn to step back, reevaluate the goal of the code and the test, and rewrite one, the other, or both. In the meanwhile, it may become necessary at times to replace an overly-restrictive test rather than to spend the effort salvaging it.
2. While it’s always nice to get testing experience as early as possible on a project, sometimes you just need to write throw-away, prototype code; in that case thorough unit testing is probably overkill. 
3. It is up to the team to gauge the acceptable limits of such debt, and at which point it must be paid to avoid an even more expensive rewrite once maintenance and new feature development grow too cumbersome.

> **Dependency injection** is the design principle whereby a piece of code does not contain direct references to its dependencies, but contains abstract interface references instead. This can improve the isolation of a piece of code to make it easier to test, by replacing concrete dependencies with stubs, mocks, or other [Test Doubles](https://martinfowler.com/bliki/TestDouble.html).



### 5.4 Who Tests the Tests?

1. As one possible measure to avoid buggy tests in the future, the team responsible for such a bug could endeavor to take a closer look at the test code submitted as part of future code reviews, to provide it with the same priority and care as "production" code.

2. Writing multiple test cases that check that the code doesn't do what it shouldn't do (instead of just checking the happy path where all inputs are valid) may reveal bugs in other test cases. 



### 5.5 Tests Are For Chumps

The company effectively could manage the complexity by only hiring the "smartest" programmers who could rapidly get up to speed in that environment.



### 5.6 The Google Web Server Story

> *The Google Web Server Team took a hard line: No code was accepted without an accompanying unit test.*

1. An influx of new Google developers eventually helped accelerate the cultural shift towards unit testing adoption, both because these new developers were open to the idea, and because testing eventually proved effective in helping these new folks get up to speed and avoid making mistakes.
2. The barrier that was stopping people from making changes as rapidly as possible was the same that slows change on most mature codebases: a quite reasonable fear that changes will introduce bugs.
3. [Fear is the mind-killer.](http://en.wikipedia.org/wiki/Bene_Gesserit#Litany_against_fear) It stops new team members from changing things because they don't understand the system, and it stops experienced people changing things because they understand it all too well.
4. New team members found themselves becoming productive far more quickly because the tests allowed them to gain a deeper perspective on a system one unit at a time, and to begin contributing changes with the confidence that the existing tests would likely detect any unexpected side-effects. 
5. Adding more new developers actually allowed the team to move faster and do more, avoiding the scenario described by [Brooks's Law](http://en.wikipedia.org/wiki/Brooks'_law) in which "adding manpower to a late software project makes it later".
6. As people experienced what it meant to cast aside the fear of change, they came to see this side-effect as easily outweighing those lines of code, in terms of its impact on their happiness, on their team's happiness, and on the bottom-line of productive output.



### 5.7 Tight Feedback Loops

*Over time, unit testing discipline allowed the Google Web Server Team to move faster and do more. Unit tests are just as much about improving productivity as catching bugs.*

1. Unit tests codify the intent of a specific low-level unit of code. They are focused, and they are fast.
2. Without unit tests, we have to use more of our brains to remember weird corner cases and strange side-effects, giving us less time and energy to do the thing we're better at than the computer: advancing solutions to new problems rather than juggling the weight of all the problems that have already been solved.
3. You don't need to start up some heavyweight server if you can just run a unit test instead. 



### 5.8 Improved Code Quality

1. Having to write code that uses your own code can lead to improved designs that are more readable, maintainable, and debuggable.
2. All of the smaller pieces that make up the larger system become not just more reliable, but easier to understand. 



### 5.9 Executable Documentation

1. *Unit test names can act as a specification of the code's behavior; the tests themselves act as code samples for each behavior case.* 
2. Set the same quality bar for test code as production code. 



### 5.10 Accelerated Understanding

1. If you're new to a team, breaking many tests as you begin to make changes to the system can help you become productive far more quickly.
2. If you've been on the team for a long time, existing tests will answer many questions that new contributors may have.



### 5.11 Faster Bug Hunting

The developer adds a new test to reproduce the bug, verifying that the defect is well-understood before attempting to fix it. This new test verifies the fix for the bug, and the existing tests provide a high degree of confidence that the fix is free of unintentional side-effects. 



### 5.12 Are You Experienced?

There is no greater argument in favor of unit testing than the actual experience of unit testing. You Cannot Measure Productivity, but you can feel it. 



### 5.13 Get Your Hands Dirty

*Immediate gratification is what really hooked most of us who swear by our unit tests. No rational arguments, no data, no charts or dollar amounts needed.*



## 6. Other Useful Tools and Practices

*Simplified description*

1. Static Analysis/Compiler Warnings

1. Modern Languages

1. Open-Sourcing

1. Style Guides/Coding Standards

1. Code Review

1. Integration/System Tests

1. Documentation

1. Fuzz Testing

1. Continuous Integration

1. Crashing and Core Dumps

1. Release Engineering

1. Site Reliability Engineering and Production Monitoring

1. Costs

1. All Part of a Balanced Breakfast



## 7. Google's Retrofitted Testing Culture, or: Déjà Vu All Over Again

*The author introduces the overall picture of Google development environment.*

1. **Resistance** to unit testing at Google was largely a matter of developers undereducated in unit testing struggling to write new code using old tools that were straining heavily under the load of Google's ever-growing operation. 

2. The [Testing Grouplet](http://mike-bland.com/2011/09/27/testing-grouplet.html) was a team of Google developers who worked together in their 20% time (time provided by Google to allow developers to work on Google-related projects of their choosing aside from their main projects) to address the challenges in promoting unit testing adoption throughout Google.

3. Testing on the Toilet.

4. Test Certified.
5. Test Mercenaries.
6. Testing Fixits.
7. Style Guides/Coding Standards.
8. Code Review.
9. Common Infrastructure to Hide Low-Level Details.
10. Test Automation Platform Continuous Integration Service.
11. TAP Goes to Eleven.
12. Build Monitoring Orbs.
13. Noogler Indoctrination.



## 8. How to Change a Culture

1. Be the Change You Wish to See

2. Start Small with the Existing Code

3. The Small/Medium/Large Test Pyramid

4. Set Up Continuous Integration

5. Maximize Visibility

6. Partners-In-Crime

7. Educate

8. Delegate, Delegate, Delegate!

9. Be the Walrus

10. Embrace the Power of Teamwork

11. Make Yourself Obsolete

12. Run a Fixit

13. Eschew Authoritah

14. Trust Yourself

15. Maintain Focus

16. Develop Fingertip Feel

17. Let a Strategy Emerge

18. Find a Mentor

19. Work With Nature, Not Against It

20. One Team at a Time

21. Measure, Enforce, Strive

22. Make a Stand

23. Be Redundant and Repeat Yourself

24. Man on the Moon

25. Persevere

26. Follow Through

27. Reward and Recognize

28. Make It Fun



## 9. Final Thoughts

*Why isn’t unit testing a part of every development culture?*

1. Some programmers and teams just don't know about unit testing and what it can do for them, lack experience with it, or need help to get started.
2. Teams often claim that they “don’t have time to test”, or that their code is “too hard to test”. 
3. Hand washing alone does not a doctor make, nor a patient save, but we would not trust a doctor who didn’t wash his or her hands. As software developers, we should assume a similar [duty of care](http://en.wikipedia.org/wiki/Duty_of_care) on behalf of our users. No one should trust software developed without unit tests.
