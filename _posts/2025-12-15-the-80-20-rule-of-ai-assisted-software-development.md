---
title: "The 80/20 Rule of AI-Assisted Software Development"
excerpt: "AI coding assistants can help us complete 80% of basic functions in 20% of the time. We need to spend 80% of our time carefully polishing the remaining 20% of core functions."
tags:
- AI
- GitHub Copilot
- Coding Assistant
header:
  overlay_image: https://images.shangjiaming.top/2025-08-13_162352-small.jpg
  overlay_filter: 0.5
date: 2025-12-15 21:42 +0800
---

## Introduction

Recently, I developed a [Minimalist Clock Web Version](https://github.com/sjmyuan/minimalist-clock) based on GitHub Copilot. Despite the abundance of similar applications already available in the market, I still chose to create one myself. The primary purpose of this decision was to optimize prompts through practice and accumulate experience in AI-assisted software development.

![](https://images.shangjiaming.top/20251214-045528.png)

[Click here to view the demo](https://clock.shangjiaming.com/)

During the development process, I deeply realized two 80/20 rules:
1. **AI coding assistants** can help us complete 80% of basic functions in 20% of the time.
2. We need to spend 80% of our time carefully polishing the remaining 20% of core functions.

Next, I will share my development experience and insights from this project.

## AI Builds the Foundation: 20% Time Completes 80% Basic Functions - 2.5 hours

In the early stages of the project, I quickly built the basic functions of the application through the following steps:

1. **Requirement Validation**  
   I first spent 10 minutes using [Brainstorming Prompts](https://github.com/sjmyuan/prompts/blob/main/brainstoming.md) to validate my ideas and further clarify requirements. Although this prompt still has room for optimization, it helped me better understand the overall direction of the project.

2. **Requirement Breakdown**  
   Then, I spent 10 minutes with the help of [Requirement Breakdown Prompts](https://github.com/sjmyuan/prompts/blob/main/break-down-requirements-to-epics.md), defining user roles, identifying epic stories, breaking down user stories, and formulating acceptance criteria.

   ![](https://images.shangjiaming.top/20251214-055044.png)
   
   [Click to view the full requirement document](https://github.com/sjmyuan/minimalist-clock/blob/main/docs/requirements.md)

3. **Architecture Design**  
   Subsequently, I spent another 10 minutes using [Architecture Design Prompts](https://github.com/sjmyuan/prompts/blob/main/architect-assitant.md) to complete the system architecture design. This includes determining the technology stack, designing data models, formulating interface protocols, planning deployment models, and designing the file structure of the codebase.

   ![](https://images.shangjiaming.top/20251214-055019.png)


   [Click to view the full architecture design](https://github.com/sjmyuan/minimalist-clock/blob/main/docs/architecture.md)

4. **Function Development**  
   Finally, I invested 2 hours in actual development. During this process, I kept the requirement document open and used my own [Coding Assistant](https://github.com/sjmyuan/minimalist-clock/blob/main/.github/agents/coding-assistant.md) to implement each epic one by one. Upon completing an epic, I would start a new session. Eventually, the basic functions of the application were up and running.

## Manual Fine Polishing: 80% Time Completes 20% Core Functions - 7.5 hours

Although the AI coding assistant can quickly complete most of the basic functions, what truly determines the quality of the application is that 20% of the core functions. In this project, the card folding effect is the core highlight of the application and also my main motivation for initially developing this application. However, the realization of this function was full of challenges.

1. **Complex Requirement Description**  
   A simple requirement description cannot make Copilot accurately understand my intentions. For this reason, I had to detail the animation effects and page layout of each time period to barely achieve the preliminary effect.

2. **Bug Fixes and Code Optimization**  
   Even so, the implemented code still had many bugs, and the code structure was not reasonable enough. To fix these problems, I had to provide repair or refactoring ideas step by step, guiding Copilot to gradually optimize the code. If I didn't have a clear idea myself, even switching to other models wouldn't yield ideal results.

3. **Unresolved Issues**  
   Currently, the card folding effect still has some known and unknown bugs. Some issues are ones I haven't discovered yet, while others are ones I've noticed but don't know how to solve.
## Conclusion

Through this development practice, I deeply recognize the great potential of AI coding assistants in software development, as well as its limitations.

1. **Reduce Trial and Error Costs**  
   AI coding assistants can help us quickly complete 80% of the basic functions. This efficient way of development allows us to quickly verify before further investment, thereby significantly reducing trial and error costs. Only after verification passes do we need to invest a lot of energy into fine-tuning the application manually.

2. **Details Determine Success or Failure**  
   What truly makes an application stand out among many similar products is that 20% of core functions. These functions often require developers to invest a lot of time and effort in optimization and polishing. However, during this process, the help that AI coding assistants can provide is relatively limited. Therefore, we should not overly rely on AI to implement complex or innovative functions.

In summary, the AI coding assistant is a powerful tool, but it cannot completely replace human creativity and experience. We need to use it reasonably while focusing on those parts that truly reflect value.
