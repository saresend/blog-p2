+++
title = 'Modeling scaled CI'
date = 2024-01-31T15:39:32-05:00
draft = true
+++
# Intro

Prior to going to grad school, I worked on a team that managed continuous test infrastructure. Coming from a background in product development doing mobile iOS, the move to infrastructure was refreshing. The core issues were not nearly as nebulous, and were far more central to everything we did - the success of the team needed only a few metrics to explain. The problem we were aiming to solve was the following: given thousands of code changes going in all at once, and millions of tests that were part of a test suite, how do you efficiently verify that all these changes are safe. For smaller codebases, the answer is fairly straightforward - just run every test on every change. However, this quickly becomes far too expensive - we were forced to sample smaller and smaller subsets of our test suite, and were tasked with doing so in a way that minimzes the number of breaking changes we allow through our CI pipeline. 

At this scale, the problem becomes pretty interesting - there's no longer a clear optimum, and it becomes a matter of juggling tradeoffs. The organic growth of the project meant that we were left with a lot of expensive and hard to remove design choices, and as is the nature of large projects, changes were small and incremental. One sentiment that endlessly came up during discussions was the feeling that we were hamstrung by the size and scale of the existing system - there was no "rewriting it from scratch". Since leaving the question has stuck with me - if we had imagined this future from the beginning, how might we design a better CI system from the ground up? 

# The Current Method

To talk about how we can start testing and modeling a large codebase, its important to start by answering an even more basic question - how is our codebase even structured? How should we discover tests, or associate which tests belong to certain modules in the code? To answer these, most large projects come with some sort of build system. There's buck (or buck2!), bazel, among others. The central abstraction which drives all of these systems is that we view our codebase as a graph - modules of code are vertices, and dependencies between them are the edges. This perspective makes it far simpler to answer questions about our codebase such as what modules depend on `foo`, or which tests are related to `bar`. In fact, most of this is just a breadth-first search throughout our "code graph", and from there we can discover quite effectively the relation between different modules of code. 

When considering the questions "what tests should I run to test some change", a natural idea would be to start with considering the modules changed by the code change. From there, we can continue down to all modules that depend on it, and so forth. In fact, as far as I know this is observation is the primary optimization used - rather than running every single test, just run the tests for modules which depend on the changes made.  

However, even this starts to break down - changes that affect large chunks of the codebase don't benefit very much from that observation, and if your code graph is fairly dense, everything can start depending on everything else pretty quick. Where can we go from here? 

# A model for CI

To begin tackling this, we need to start with some structures on which we can simulate the scenario described above. Let's take the role of the system maintainer - as far as we're concerned, we only care that all tests remain green, and that testing changes is a cheap as we can make it. Of course there's some tension between the two objectives - presumably, we can run fewer tests on a change, in exchange for a higher risk that it breaks something, and vice versa. 

Let's capture these two objectives, the first being correctness $C(x)$. Ultimately any CI system reports a binary result - either a change is accepted or it's rejected. So, $C(x) \in \{0, 1\}$, where 1 means that we correctly report the ground truth, and 0 otherwise. The other piece is price - a certain method has a cost to it, related to how many tests it needs to run. We assume this is linear - in particular, let $P(x) = \sum_i P_i$, where $P_i$ is the price of the ith test run by a given method. 

Another aspect we must capture in any adequate model is the structure of our codebase. From the perspective of the maintainer, we can view code changes as somewhat random changes that have some likelihood of breaking some subset of tests. A complication to capture is that not all tests are independent - often larger tests like integration tests will have overlapping coverage with other smaller tests like unit tests. So, we need to encode that dependence somehow. 

One possible model is a graph $G = (V, E)$, where each vertex is a random variable. In particular, each vertex can be described as an indicator random variable, where it is 1 only if the change under test in fact breaks the test, and 0 otherwise. The edges indicate the following relation - if one test fails, a related test may also be more likely to fail, and similarly for success. Note that this relation is not transitive - 2 completely independent tests may both share some coverage with some larger test, without themselves testing the same functionality. 

We must also augment each vertex with the cost it takes to observe its result. In our model, let each vertex $v_i \in V$ be defined to be a 2-tuple $(P_i, x_i)$, where $P_i$ is the price to observe this random variable, and $x_i$ is its result. For simplicity each vertex $v_i$ is modeled a bernoulli random variable of some probability $p$. 

Let's model each edge with some value $w_i \in [0, 1]$, that indicates the "strength" of the relation. The idea is rather simple - if an overlapping tests succeeds, then there's a better chance its neighboring tests succeed because they test overlapping functionality. A rather crude method for computing this is the following -  for a given edge $(u, v)$ with weight $w_i$, if we get a realization of $u$, we either shift the probability of $v_i$ down or up by $w_i$, depending on whether $u$ is a success or failure. Mathematically, we have: $P(v_i = 1 | u_i = 0) = \max(0, p - w_{uv})$ and $P(v_i = 1 | u_i = 1) = \min(1, p + w_{uv})$.

## A sanity check 

With this model in place, let's look at how our "run the world" method performs. However, its hard to evaluate unless we have some other strategies to compare it against. Indeed, let's consider the class of strategies defined by some parameter p, with which we evaluate each test indepependently with probability p. Our run the world strategy is exactly the strategy when we take $p = 1$. 

## Finding the pareto frontier

# Departing from determinism




