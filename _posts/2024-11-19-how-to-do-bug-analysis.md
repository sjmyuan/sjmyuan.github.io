---
title: How to do bug analysis?
excerpt: What is the purpose of Bug analysis? Some people think it is to classify
  Bugs, others think it is to prevent Bugs from reoccurring, and some think it is
  to improve project delivery quality. Being able to fix all Bugs doesn’t bring us
  benefits because each Bug means waste and loss. What truly benefits us is preventing
  Bugs from happening, getting things right the first time, which means ensuring high-quality
  delivery. So, how can we improve delivery quality through Bug analysis? I will discuss
  it in this blog.
tags:
- Agile
- Team
- Leadership
header:
  overlay_image: https://images.shangjiaming.top/2024-05-04_155540-small.jpg
  overlay_filter: 0.5
date: 2024-11-19 22:07 +0800
---
# A bad question

Recently, while analyzing bugs with the team, I often hear the phrase, **"If we had to do it again, how could we avoid this problem?"** Initially, I didn't think there was anything wrong with this question. It asks us to imagine, if we went back to that scenario, what actions we would take to avoid the problem, and thus find solutions to prevent the problem from recurring in the future. However, after analyzing several bugs, I found that this question is not a good one.

First of all, it leads the team to inefficient solutions. Inefficient solutions are those that cure the symptoms but not the disease. They rely on increased motivation to avoid problems, such as careful inspection, enhanced communication, repeated confirmation, etc. Different people have varying degrees of effectiveness when implementing these solutions, and may become overwhelmed as their energy wanes. The root cause is that under the context of this question, we adopt the wrong approach to problem-solving. Bugs should be considered as known-unknown problems, for which we can find appropriate solutions through analysis. But under the context of this question, we subconsciously assume that bugs are known-known problems, and believe that we can find solutions through categorization. This makes it easy for us to set a category specifically for that bug, resulting in an inefficient solution that only solves that particular bug.

Secondly, it stifles the team's vitality. This question makes us subconsciously assume that the problem could have been completely avoided and that we must have done something wrong to cause the problem. However, according to [the classification of problems](https://blog.shangjiaming.com/how-to-make-a-competent-technical-analysis/), no matter how strong our capabilities or how excellent our team practices are, there will always be some problems that we only know after they occur. Therefore, strictly speaking, we cannot be sure whether the current problem can be completely avoided. We need to continually try to understand and solve the problem. Overemphasizing that the problem must not happen again will make the team overly cautious, which is not conducive to effectively solving the problem.

Lastly, it increases the team's pressure. This question makes us feel that we should immediately provide a solution, turning bug analysis from non-urgent to urgent, forcing immediate response, and thus increasing the team's pressure.

# How to conduct Bug analysis?

## What is the purpose of Bug analysis?

Some people think it is to classify Bugs, others think it is to prevent Bugs from reoccurring, and some think it is to improve project delivery quality. I lean towards the last opinion. Classifying Bugs seems to have no other use than making reports. Each Bug is unique, and if we only want to prevent Bugs from reoccurring, we just need to fix the Bugs without further analysis. However, being able to fix all Bugs doesn’t bring us benefits because each Bug means waste and loss. What truly benefits us is preventing Bugs from happening, getting things right the first time, which means ensuring high-quality delivery.

So, how can we improve delivery quality through Bug analysis? I believe we can improve delivery quality by deriving efficient solutions from Bug analysis.

## What is an efficient solution?

An efficient solution is one that addresses problems by enhancing capabilities or optimizing processes. Here, capabilities refer to the knowledge, skills, tools, etc., that team members have to solve problems. Processes refer to the ways in which team members collaborate, which can be rules such as deployment processes, branch management strategies, or practices such as Agile, Scrum, XP, etc. They can also correspond to the ability and prompts in the [Fogg Behavior Model](https://behaviormodel.org/).

Assume our project status is as follows:

![](https://images.shangjiaming.top/20240604-095257.png)

Inefficient solutions, although they can solve current problems, might overlook other issues.

![](https://images.shangjiaming.top/20240604-095313.png)

Enhancing team members' capabilities can prevent more problems.

![](https://images.shangjiaming.top/20240604-095332.png)

If we cannot quickly enhance team members’ capabilities, we can also avoid more problems by optimizing processes to effectively organize team members' capabilities.

![](https://images.shangjiaming.top/20240604-095349.png)

## How to obtain efficient solutions?

To obtain efficient solutions, we need to conduct root cause analysis of Bugs. However, not all root cause analyses can yield efficient solutions; it depends on the quality of the root cause analysis. One indicator of high-quality root cause analysis is whether the root causes we identify are related to capabilities or processes because only root causes in these areas can lead to efficient solutions.

The key to root cause analysis lies in its structured method of identifying root causes. We should not skip this analysis process because a Bug seems insignificant or the solution is simple. If, for cost reasons, we cannot have all team members participate in the root cause analysis of each Bug, we can [let AI assist us](https://blog.shangjiaming.com/rca-by-chatgpt/).

After obtaining efficient solutions, we can use [Pre-mortem](https://en.wikipedia.org/wiki/Pre-mortem) to further examine the solutions. Here, we can modify the initial question to, **If the problem happens again after we implement this solution, what might be the reasons?** Any reasons related to capabilities or processes that we can think of can be added to the current solution.

# Summary

The most frequently heard words in Bug analysis is **do my best**: doing my best to check code, communicate requirements, improve quality, complete features, etc. So, what does **do my best** mean? How do we measure it? Remember in [The Prime Directive of Retrospective](https://retrospectivewiki.org/index.php?title=The_Prime_Directive) it says: **We understand and truly believe that everyone did the best job they could**. Instead of thinking about how to measure **do my best**, we should choose to believe that every team member has done their best. We should focus Bug analysis on capabilities and processes rather than personal motivations like being proactive, cautious, etc.
