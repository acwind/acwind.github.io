---
title: "The Human Cost of 10x: How AI Is Physically Breaking Senior Engineers"
categories:
 - 科技思考
tags: ["AI", "Productivity", "Engineering"]
key: the-human-cost-of-10x-how-ai-is-physically-breaking-senior-engineers
---

这篇文章探讨了 AI 带来的爆发性生产力背后，资深工程师正在面临的物理性极限与由于上下文切换、工作量膨胀导致的心理与生理压力。

原文出处: https://techtrenches.dev/p/the-human-cost-of-10x-how-ai-is-physically

# The Human Cost of 10x: How AI Is Physically Breaking Senior Engineers

Your brain processes 10 bits per second. AI just increased your review queue by 98%. The math doesn’t work.

*By Denis Stetskov, Apr 07, 2026*

Last Tuesday, I stood up from my desk at 7 PM and felt a vacuum in the front of my skull. Not a headache. Not fatigue. A physical emptiness, like the frontal lobe had been running at redline all day and finally shut down. I stood there for ten seconds trying to remember what I was going to do next. Nothing came.

In the past year, the volume of information passing through my brain on any given Tuesday has become what used to take a week. Code review is the worst of it, but the real killer is the context switches. AI-generated PRs, client architecture decisions, three Slack threads about deployment issues, a candidate’s CV that needs review, an air defense alarm outside the window, then back to reviewing code that a machine wrote in seconds and I need hours to validate. Each of these demands a different mental model. Each one burns working memory. By 4 PM I’m making decisions I wouldn’t trust from a junior. By 7 PM my brain is physically empty.

"The industry calls this “10x productivity.” I call it what it is: a system that generates output at machine speed and forces humans to process it at biological speed."

## Workload Creep

In February 2026, UC Berkeley researchers published findings from eight months embedded inside a 200-person tech company. Over 40 in-depth interviews. Their conclusion: AI doesn’t reduce work. It intensifies it.

They found three mechanisms of “workload creep.” Task expansion: everyone’s scope inflates because AI makes it possible to do more. Blurred boundaries: AI prompting happens during lunch, commute, evenings. Implicit pressure: when colleagues visibly do more with AI, expectations rise for everyone.

The Upwork Research Institute quantified it: 77% of employees using AI say it has added to their workload. Not reduced. Added. 71% report burnout.

The finding that keeps me up at night: workers who report the highest AI productivity gains are the most burned out. 88% burnout rate among the “most productive” AI users. They’re twice as likely to quit.

The people who look best on your dashboard are the ones closest to walking out the door.

## Your Brain Runs at 10 Bits Per Second

In 2025, Zheng and Meister published in Neuron that the human brain processes conscious, analytical thought at approximately 10 bits per second. Your sensory systems gather data at roughly 1 billion bits per second. But the bottleneck for code review, the part where you actually think, is 10 bits per second.

Working memory holds roughly 4 chunks of information at a time. The SmartBear/Cisco study established numbers everyone ignores: defect detection drops from 87% for PRs under 100 lines to 28% for PRs over 1,000 lines. Quality collapses after 60 minutes.

Now look at what AI did to the review queue.

GitHub’s Octoverse 2025 shows 43.2 million pull requests merged per month. Up 23% year-over-year. Lines of code per developer grew from 4,450 to 7,839 in eight months. A 76% increase.

Faros AI analyzed 10,000+ developers and found AI users merge 98% more pull requests with AI assistance. Every single one lands on a senior engineer’s desk.

As MIT reported: "juniors produce far more code with AI tools, but the sheer volume is saturating senior developers’ capacity to review. One OCaml maintainer rejected a 13,000-line AI-generated PR outright. Nobody had the bandwidth."

I wrote about the [supervision tax](https://techtrenches.dev/p/your-claudemd-pi-is-a-wish-list-not).