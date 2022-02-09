# Hightouch’s Code Review Process

Code reviews are one of the most critical steps in software development. They’re an opportunity to learn (for both the reviewer and PR author), and can catch issues before they get deployed to production. By taking code reviews seriously, we can collectively write better code than any single person would by themselves!

It’s common and expected for complex PRs to go through multiple rounds of review. If you open a PR in the morning, it might not get merged for a couple days while you iterate on improvements. This is ok and expected!

At Hightouch, we strive to give **thorough** and **empathetic** code reviews. The reviews are *thorough* because they provide useful comments (versus always approving with [lgtm](https://alisterbscott.com/wp-content/uploads/2018/07/2dsag2.jpg)). They’re *empathetic* because we understand that comments are made for the benefit of the overall codebase, and aren’t a criticism of the PR author.

Becoming a skilled and trusted PR reviewer is one of biggest impacts you can have on the team.

# Philosophy: Approval == Sharing Responsibility 🤝

When you approve a PR, **you’re taking on shared responsibility for the PR**. If bugs get reported, that’s as much on the reviewer as the original author. This has two benefits: the quality of the codebase improves, and the reviewer learns more.

### Benefit: Codebase quality

In a codebase as complex as ours, no single person ever has the full picture on a change. We all have expertise in different areas of the codebase, and different prior experience in building similar systems. We should leverage this knowledge to its max potential.

### Benefit: Learning by reviewing

*This is an extremely underrated benefit!!*

Becoming a great engineer takes a long time because the feedback loop is so slow — you can only write so much code, and it takes awhile to see how it behaves in production over time. By taking shared ownership over PRs you can shortcut this feedback loop, and maximize your learning from the code **others** write!

Early on in my [Kevin] career, I started reviewing all of my mentor’s code. It was intimidating at first, but I learned so much by forcing myself to understand all of his decisions. Plus, he was a faster coder than me, so the feedback loop was even faster than just getting review comments on my own code.

Some ways you can learn by reviewing are:

1. Deeply understand **why** the author chose to write the code the way they did. What tradeoffs did they make?
    1. Do you disagree or not understand a decision? That’s a great opportunity to start a conversation and learn.
2. Learn from review comments that other people make. If you missed something in your review, you might miss it when in your own code too.
3. Learn from what happens after the code ships. Did it cause regressions? Were there bugs in the implementation? Did it cause scalability issues down the line? Did a future PR introduce a regression to the code from this PR?
    1. Like (2), if you missed something in your review, your own code might have similar issues.

# Being a great reviewer

*Everyone has their own style. This guide is very opinionated, but don’t be afraid to develop your own approach. Also, it’s meant as a guide, you can use your judgement on what’s appropriate given the PR. For example, I usually don’t run PRs locally.*

Hopefully you’re now convinced that it’s worthwhile to become a great code reviewer. The first thing to know is that reviewing code is **hard**. It’s a skill that takes time to develop, and each review takes focused time to do well, just like coding.

## Catch issues by reviewing *actively* 🧐

You’ll never catch issues in PRs if you just skim the diff and nod along to all of the changes. We’re all good engineers so any issues are probably subtle, and require concentrated effort to find.

Reviewing actively means that you actively try to understand what the code is doing, and what assumptions it’s making. You’re like Sherlock Holmes, making sure to explore every nook and cranny of the PR.

### The #1 rule: Make sure you understand what the code is trying to do

A good technique is to start by understanding what the code is trying to do, and then consider if the PR takes the best approach. You can do this evaluation at multiple levels — like a tree, each level feeds into the next. Here’s an example of common levels of abstraction:

1. Highest level — What **problem** is the PR solving as a whole?
    1. You might question if this is the right problem to solve from a product perspective. Note that it’s ok if you don’t have opinions on this, but you still need to understand what the PR is trying to do, or else you can’t evaluate if it’s doing it correctly!
2. Medium — What **technical approach** is the PR taking?
3. Low — What’s the **implementation** of each code block? (e.g. function)

For example, when reading through [this PR](https://github.com/hightouchio/hightouch/pull/2552), you could break it down into:

1. Problem: Metabase sends an email to the user everytime we refresh access tokens, so we want to refresh tokens less often.
    1. This seems like a worthwhile problem to solve.
2. Technical approach: Cache access tokens in our database. Only refresh them if the cached version is going to expire. Store the cache in a new column in the database.
    1. Caching information per-source in our database is a new pattern, and will require a lot of new code to implement. I can’t think of anything simpler, though, and we should be able to handle edge cases like invalidating the cache if the user changes their credentials.
3. Implementation: There’s lots of implementation decisions implicit in the PR. [One discussion](https://github.com/hightouchio/hightouch/pull/2552#discussion_r800912543) was about whether the caching interface should be exposed via a method on the Source object, or as a helper function that’s imported.

### **Tips for reviewing actively**

- **Read and understand every line in the PR**. If you do just this, you’ll be in a great spot.
    - PRs often need multiple reads in order to figure out what’s going on. As you read, leave temporary comments that you can come back to on places that you want to dig into (e.g. particularly important or error prone pieces of code).
    - Understand the **data model**. This is usually the key design choice that affects all the code in the PR.
- Review slowly. Reading actively takes awhile. It’s expected for a large PR review to take multiple hours.
- Question assumptions. Often, bugs are hiding outside the diff. For example, the PR might make bad assumptions about a library/API. Or it might break code that wasn’t changed.
- Pay attention to what’s confusing. If you find it confusing while reading, future maintainers will find it confusing. Areas that require a lot of thinking for you to understand are a sign that the code could be refactored, or that it deserves a comment.
- Pay attention to common sources of errors. As you read more code, you’ll develop instincts for error prone code (e.g. shared state, string manipulation, pagination).
- Run the code locally and QA it manually. You can also add logging/use a debugger to understand the control flow better.

## Types of PR comments

In general, PR feedback usually falls into one of the following categories:

1. **Correctness** - Does the code do what it says it does?
    1. Are there edge cases where it breaks?
        1. Are there race conditions? Unhandled error conditions? Does it work at scale?
    2. Does it introduce regressions in other parts of the codebase?
        1. Does it introduce any backwards incompatibility that will complicate deployment?
2. **Maintainability** - Assuming it works, what’s the longterm cost of the feature?
    1. Does the design align with our future needs? Or will we need to rewrite it soon?
    2. Does the code protect against accidental regressions in the future?
        1. Is it future proof? I.e. Are people prone to accidentally breaking it in the future?
        2. Does it have appropriate tests to catch regressions?
        3. Is it easy for future readers to understand? Does it have comments and good code structure/flow?
    3. Does it conform to our standard style? (Our style is tribal knowledge right now, but eventually we’ll document it...)
    4. Does it reinvent the wheel? Are there pieces of the PR that are already implemented elsewhere that we can reuse? (Or should this PR introduce a shared helper?)

A good PR review catches issues with both (1) and (2). A trap with code review is leaving comments on just “low hanging fruit”, like style nits, but missing more serious issues.

## Leaving good comments

Good PR comments...

- Provide a reason for **why** you think a change is warranted.
- If possible, provide suggestions on how the code could be changed.
- Acknowledge that your suggestion may be wrong. Comments are a start of a conversation, and it’s ok for the author to push back.

It’s ok to leave “nit” comments, just prefix them with “Nit - <comment>“. This way the author knows that the comment is minor and doesn’t spend a bunch of mental energy on it.

Don’t just leave negative comments either! I try to comment on at least one thing I liked in each PR (e.g. a particularly clean implementation for a function).

# The end goal is still to ship 🚀

At the end of the day, the code review process should still be **pragmatic**. PRs shouldn’t get stuck in code review while debating unnecessary details. Everyone should use their judgement on what comments are worth leaving/addressing.

One of the key skills of a senior engineer is that they know when some technical decisions should be blocking, and when it’s ok to accumulate some **tech debt**. No one will care about tech debt if the company fails 😄

Some rules of thumb (but as always, use your judgement):

- It’s ok for a feature to get shipped more slowly so that the author can improve their code. In the moment, it might seem like it’s not worth it to make the PR perfect, but taking a little more time and communicating underlying principles will pay longterm dividends to the author, since in the future they can just take the suggested approach from the beginning!
- It’s ok for a PR to get merged with unaddressed comments. Make sure that the reviewer thinks this is appropriate before merging though.
    - As an example, here’s an [exchange](https://github.com/hightouchio/hightouch/pull/2393#discussion_r789376904) between Kevin and Daishan where Kevin left a comment, but we shipped without making the change first, and then came back to it in [another PR](https://github.com/hightouchio/hightouch/pull/2577).
    - Sometimes the risk with a PR isn’t technical, and we want to just get the feature out ASAP in front of users to decide whether it’s worth investing more in.
