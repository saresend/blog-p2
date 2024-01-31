+++
title = 'Models for scaling CI'
date = 2024-01-31T15:39:32-05:00
draft = true
+++

## Introduction 

Prior to going to grad school, I worked on meta's continuous test infrastructure. The problem was essentially this - given thousands of code changes going in all at once, and millions of tests that were part of a test suite, how do you efficiently verify that all these changes are safe. The obvious answer of just running every test on every changes quickly becomes far too expensive - we need to sample smaller and smaller subsets of our test suite, in such a way that minimizes how much test breakage we allow into our codebase. 

At this scale, the problem becomes pretty interesting - there's no longer a clear optimum, and it becomes a matter of juggling tradeoffs. Of course, this effort was massive, and changes were very incremental. However since leaving the question has stuck with me - if we had imagined this future from the beginning, how might we design a better CI system from the ground up? 





