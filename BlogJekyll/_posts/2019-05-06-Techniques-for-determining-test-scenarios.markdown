---
layout: post
title:  "Techniques for determining test scenarios"
date:   2019-05-06 00:00:00 +0100

tags: Testing
---

Usually, I think of test scenarios while examining specifications and writing code. Even though I am not a test first TDD practitioner and usually write and executing tests after writing some logic, I do think about and test scenarios throughout my coding  process.

The techniques I want to describe in this post have been helpful to me in writing better quality code. They help me catch missing requirements earlier and my eventual tests (manual or automated) are more complete. Knowing these techniques also helps me collaborate better with the QA team during development and/or testing.

The techniques may describe graphing out paths and making decision tables but I almost never write down the described graphs and tables. If you find yourself having trouble identifying or explaining scenarios, the described graph or table might help you as a means of visual aid. 

The techniques are described identifying all possible test scenarios for 100% coverage. Analyzing and testing all scenarios (manual or automatic) is usually not possible due to time constraints. You should always try to focus on the parts of the software that are most likely to fail due to complexity and the parts that have the highest cost to the business when they do fail. Your test set should be maintainable and provide good enough coverage so you have enough confidence in your test set. When in doubt collaborate with your team members or QA team to determine the value of test scenarios and test coverage.

Testing techniques are divided into white and black box techniques. White box techniques look at the actual statements of the source code to determine test cases. Black box techniques use specifications to determine if outputs are correct when providing inputs. 

**Table of contents:**
* Table of Contents
{:toc}

### Equivalence class and boundary partitioning (black box technique)

Equivalence class partitioning is a technique for partitioning possible inputs into possible valid and invalid scenarios. 

For string input for a certain field you could identify:

Valid input
-  Input length has a minimum length of 0 (empty string) and a maximum length of 50.
-  It contains only alphanumeric characters and no diacritics.

Invalid input
-  Null value.String larger than 50 characters.
-  String with diacritics or other special characters (@# etcetera).

The equivalence class technique states that you should create a test scenario for each of the valid and each of the invalid scenarios. If a method has multiple parameters and you are testing invalid scenarios you should never use more than one invalid input at a time to avoid defect masking.  This way you can relate expected errors back to the single invalid input or unexpected errors to one of the valid inputs. 

Boundary partitioning is add on to equivalence class partitioning. You basically look at the equivalence classes and use the boundary value (for example string has a length of 50), boundary value -1 (string length 49) and boundary value +1 (string length 51) as additional test cases. This technique is identifying the so-called edge cases of input. You should look at all equivalence classes (valid and invalid) to determine if you can set up test cases with the boundary, boundary+1 and boundary-1, a value between 2 boundaries and see if the outputs are the expected values.

### Statement and decision-based testing (white box techniques)

Statement and decision-based testing use a [control-flow graph](https://en.wikipedia.org/wiki/Control-flow_graph). Basically, each node in the graph is a control statement where a decision is made about which path to execute next (so if, switch, while, for statements ). The branches (edges) of the graph are the possible code paths executed between the control statements. 

Statement-based testing creates test scenarios for each node (not all the edges). So you should have passed through each control statement (condition is true) at least once for coverage for this technique. When looking at the source code if you pass through each control statement and execute the code within the control statements, you should end up covering all lines in the source code.  

Decision-based testing looks to cover the edges of the control-flow graph. To cover all the edges of the graph you should create tests for each conditional state for the control statements. So for an if statement you should test with conditional false and true. For loops, you should iterate at least through one loop and use a test case that ignores the loop. When using a switch/case statements all possible cases should be covered. 

For some good examples of these two techniques, look at the short YouTube/Udacity links in the resource section of this post.

### Conditional-based testing (white box technique)

Control statements (for example if, switch, while, for statements ) usually have some kind of conditional logic. If the conditional logic has multiple inputs you can apply the rules for equivalence and boundary testing (see the previous paragraph in this post) to determine all possible inputs to the conditional statement to see if the expected path is executed.

### State transition testing (black box technique)

Sometimes your software uses a [finite state machine](https://en.wikipedia.org/wiki/Finite-state_machine) (maybe some kind of status table or status enum is used to describe the possible states of an object). The linked Wikipedia article for the finite state machine contains multiple techniques for identifying the states, path between states and the conditions per path or state. 

Each path between a state usually identifies a "happy flow" test scenario. Besides obvious valid state transitions, you should try to identify invalid paths and if they are possible to execute in the system. 

Also, look at the conditions per path. See if you can use the rules for equivalence and boundary testing (see the previous paragraph in this post) to determine if the guards work as expected.

### Resources

- [ISTQB: Foundation Level 2018 Syllabus (chapter 4 contains a list of testing techniques)](https://www.istqb.org/downloads/send/51-ctfl2018/208-ctfl-2018-syllabus.html)
- Wikipedia:
  - [Equivalence class partitioning](https://en.wikipedia.org/wiki/Equivalence_partitioning)
  - [Boundary value analysis](https://en.wikipedia.org/wiki/Boundary-value_analysis)
  - [Control-flow graph](https://en.wikipedia.org/wiki/Control-flow_graph)
  - [Finite state machine](https://en.wikipedia.org/wiki/Finite-state_machine)
- [StackExchange: How do we calculate Statement coverage, Branch coverage, Path coverage and Condition coverage in White box testing?](https://sqa.stackexchange.com/questions/20226/how-do-we-calculate-statement-coverage-branch-coverage-path-coverage-and-cond)
- YouTube/Udacity: 
  - [Statement Coverage - Georgia Tech - Software Development Process](https://www.youtube.com/watch?v=9PSrhH2gtkU)
  - [Control Flow Graphs - Georgia Tech - Software Development Process](https://www.youtube.com/watch?v=0lVA7TPpxUE)
  - [Branch Coverage - Georgia Tech - Software Development Process](https://www.youtube.com/watch?v=JkJFxPy08rk)
  - [Condition Coverage - Georgia Tech - Software Development Process](https://www.youtube.com/watch?v=ZnPmJd5aqyw)
  - [Branch and Condition Coverage - Georgia Tech - Software Development Process](https://www.youtube.com/watch?v=kd1_3CwYr60)
- [Software Testing Class: How to Design Test Cases Using State Transition Testing Technique?](https://www.softwaretestingclass.com/design-test-cases-using-state-transition-testing-technique/)
- [Software Testing Material: State Transition Test Case Design Technique](https://www.softwaretestingmaterial.com/state-transition-test-design-technique/)

