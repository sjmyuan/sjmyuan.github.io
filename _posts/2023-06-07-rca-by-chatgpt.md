---
title: How to perform a root cause analysis using ChatGPT?
excerpt: In the article Let's try having ChatGPT ask us questions, we explored how
  to guide ChatGPT to interact with us in the form of questions. This article applies
  this idea to root cause analysis and uses the advanced GPT-4 model. After entering
  the prompts designed in this article, ChatGPT will first clarify the problem, then
  trace back to the root cause using the 5 Whys method, and provide corresponding
  solutions.
tags:
- AI
header:
  overlay_image: https://images.shangjiaming.top/olav-ahrens-rotne-4Ennrbj1svk-unsplash.jpeg
  overlay_filter: 0.5
  caption: 'Photo credit: [**Unsplash**](https://unsplash.com)'
date: 2023-06-07 23:49 +0800
---
In the article [Let's try having ChatGPT ask us questions](https://blog.shangjiaming.com/let-chatgpt-ask-questions/), we explored how to guide ChatGPT to interact with us in the form of questions. This article applies this idea to root cause analysis and uses the advanced GPT-4 model. After entering the prompts designed in this article, ChatGPT will first clarify the problem, then trace back to the root cause using the 5 Whys method, and provide corresponding solutions.

According to the user's ability level in root cause analysis, ChatGPT can provide corresponding assistance:

1. **Beginners**: ChatGPT can act as a coach to guide users through the entire analysis process and help them fully understand the problem. 

2. **Proficient**: ChatGPT can act as an assistant to ask more valuable questions and dig deeper into the root causes.

3. **Expert**: ChatGPT can provide initial analysis results for users to continue optimizing the analysis method and improve efficiency.

This article is based on a premise: ChatGPT can not only give high-quality answers, but also ask valuable questions. If this premise is true, then using ChatGPT for root cause analysis will bring us more thinking and value.

# What is root cause analysis?

[Root Cause Analysis (RCA)](https://en.wikipedia.org/wiki/Root_cause_analysis) is a structured problem solving approach aimed at deeply digging into the root causes of problems rather than just focusing on the surface symptoms. It summarizes a series of methodologies, the main difference between which lies in the techniques for finding the root cause.

The main root cause analysis methods are as follows. They have different application scenarios and advantages:

1. [**5 Whys method**](https://en.wikipedia.org/wiki/Five_whys): It is a method of gradually digging deeper into the root cause of a problem by repeatedly asking "why". This method is simple and easy to use. It is suitable for quickly locating the root cause of a problem, especially when the problem is relatively simple and direct.

2. [**Fishbone Diagram**](https://en.wikipedia.org/wiki/Ishikawa_diagram): Also known as a cause-and-effect diagram. It is a method of analyzing the root cause from six aspects: manpower, machine, material, method, environment and measurement (6M) through brainstorming. The fishbone diagram helps facilitate cross-departmental cooperation and communication. By sorting out the factors in each aspect, it helps to comprehensively and deeply analyze the problem. 

3. [**Failure Modes and Effects Analysis (FMEA)**](https://en.wikipedia.org/wiki/Failure_mode_and_effects_analysis): This is a systematic method for evaluating potential failure modes and their effects on system performance. FMEA evaluates the severity, occurrence probability and detection capability of failures to determine the priority of failure modes. FMEA is widely used in product design, process improvement and reliability engineering to help reduce risks and improve product quality.  

4. [**Fault Tree Analysis (FTA)**](https://en.wikipedia.org/wiki/Fault_tree_analysis): This is a systematic method for analyzing the potential causes of failures in complex systems. FTA constructs fault trees to represent the logical relationship of failures and identifies key events that can lead to system failure. FTA is mainly used in safety-critical fields such as nuclear industry, aviation and chemical industry to improve system safety performance by identifying and managing key risk factors.

This article uses the 5 Whys method to design prompts for ChatGPT.

# Why do root cause analysis?

## Identify the root cause of the problem

Root cause analysis can help us find the root cause of a problem and find lasting solutions to prevent the problem from recurring and reduce waste. For example, when dealing with out of memory problems, increasing memory alone is not enough. We must find the root cause of memory growth in order to completely solve the problem.  

## Determine the correct causal relationship

Root cause analysis helps us find the real causal relationships. For example, when a child has trouble playing with building blocks and suddenly says "I don't want to play anymore" and throws the blocks aside. Without root cause analysis, we may simply think the child doesn't like the blocks. But through root cause analysis, we find that the child actually gave up because he couldn't get help.

## Promote effective decision making

Root cause analysis can help us make better decisions. For example, if we want customers to adopt continuous deployment, we first need to solve the problem: "I can't convince customers to adopt continuous deployment". Without root cause analysis, we may force customers to adopt it from our own perspective. Through root cause analysis, we can understand whether customers are suitable for continuous deployment and what benefits continuous deployment can bring to customers, so as to decide which strategy to adopt to convince customers.

# How to do root cause analysis?  

Root cause analysis usually includes the following steps:

1. **Define the problem**: Clearly define the problem to be solved and ensure that the problem description is clear and accurate.  

2. **Collect data**: Collect all information related to the problem, including the time, place, personnel, process, etc. of the event.  

3. **Analyze the problem**: Use root cause analysis methods to deeply dig into the root causes of the problem. This may involve multiple causes and require distinguishing primary from secondary and correlation. 

4. **Determine solutions**: Based on the analysis results, develop targeted solutions to ensure that the solutions can eliminate the root causes of the problem.  

5. **Implement improvements**: Implement solutions, monitor improvement effectiveness, and make adjustments as needed.

# What are the challenges of root cause analysis?  

1. **Identify the root cause**: There are no unified criteria for determining the root cause, which largely depends on the experience and subjective judgment of the participants. For example, for fever, ordinary people may think the root cause is a cold, while doctors may think the root cause is bacterial infection, flu, COVID-19, etc.

2. **Identify the correct causal relationship**: Causal relationships may be wrong and need to be verified in reverse to determine the correct causal relationship. For example: "Why didn't you give your wife a gift on your wedding anniversary? Because she didn't want a gift." After reverse verification, this causal relationship may not hold. "Because your wife didn't want a gift, you don't need to give a gift on your wedding anniversary." If the analysis continues according to this causal relationship, the solution may lead to a tragedy.

3. **Focus on processes and capabilities rather than individuals**: When human error exists, it is easy to point the problem to finding responsibility rather than focusing on solving the problem, affecting the effectiveness of root cause analysis. For example, "Configuration errors caused online accidents." We should focus on how to avoid similar problems from happening again by modifying processes, improving staff capabilities, etc., rather than pursuing responsibility and requiring everyone to double check when modifying configurations.

# Why use ChatGPT for root cause analysis?  

1. **Easy to identify the root cause**: Compared with ordinary root cause analysis, it can quickly simulate multiple times before everyone discusses to gain a deeper understanding of the problem using [Self-Consistency](https://learnprompting.org/docs/intermediate/self_consistency) techniques and identify deeper causes. It can also simulate multiple times after the discussion to verify causal relationships and find new root causes.  

2. **Easy to verify causal relationships**: It can organize logical trees for us after the analysis to facilitate our verification of causal relationships.  

3. **Easy to focus the analysis process on processes and capabilities**: We can constrain its questions at any time to control the direction of its questions.  

# How to use ChatGPT for root cause analysis?  

## Roles 

[Role Prompting](https://learnprompting.org/docs/basics/roles) is used here to have ChatGPT think as a root cause analysis expert.

```
As a root cause analysis expert, your task is to follow the following activity diagram to help me define the problem, collect data, find the root cause, and suggest actions.
```   

## Instructions  

[Instruction Prompting](https://learnprompting.org/docs/basics/instructions) is used here to define the interaction process between ChatGPT and the user. For simplicity, [PlantUML](https://plantuml.com/) is used, which can be understood by ChatGPT.

![](https://images.shangjiaming.top/root-cause-analysis-prompt-instructions.png)

As you can see, the following loop is used multiple times in the flowchart to allow ChatGPT to actively ask questions so that we do not need to actively ask "Do you have any more questions?".

![](https://images.shangjiaming.top/ask-questions-loop.png)

## Examples  

[Few Shot Prompting](https://learnprompting.org/docs/basics/few_shot) and [Chain of Thought Prompting](https://learnprompting.org/docs/intermediate/chain_of_thought) are used to provide ChatGPT with an example of interacting with a user for better understanding of the above instructions.

![](https://images.shangjiaming.top/20230602-101654.png)  

[Click the link to view the complete example](https://github.com/sjmyuan/prompts/blob/main/root-cause-analysis.md#example-of-conversation-following-the-instructions)  

## Multiple languages  

To support multiple languages, ChatGPT is allowed to interact with the user in the corresponding language when a non-English language is detected.

``` 
If you identify that the problem is composed in a non-English language, kindly utilize the same language for our subsequent communication.
```

## Confirmation  

To confirm ChatGPT's understanding of the instructions, it is asked to describe its understanding step by step, and then a problem will be provided for analysis.

```
If you understand and agree with the above instructions, describe your understanding step by step, then I will provide a problem for you to analyze.
```  

## Complete prompt  

![](https://images.shangjiaming.top/20230602-102158.png)  

[Click the link to view the complete prompt](https://github.com/sjmyuan/prompts/blob/main/root-cause-analysis.md)  

## GPT-3.5  

GPT-3.5 cannot fully follow our instructions and examples to perform root cause analysis. It is difficult for it to continuously ask questions and confirm the analysis results. We can decompose root cause analysis into two independent parts: [Problem Definition](https://github.com/sjmyuan/prompts/blob/main/problem-definer.md) and [Root Cause Analysis](https://github.com/sjmyuan/prompts/blob/main/problem-5whys-questioner.md). After the problem is clearly defined, root cause analysis can be performed with the optimized problem. I have created robots [ProblemDefiner](https://poe.com/ProblemDefiner) and [5WhysQuestioner](https://poe.com/5WhysQuestioner) on [Poe](https://www.poe.com) for these two tasks.  

![](https://images.shangjiaming.top/20230603-032506.png)  

![](https://images.shangjiaming.top/20230603-032434.png)  

# Examples  

## Deleting code by mistake  

The problem we want to analyze is: A line of code in Java Project was accidentally deleted during the resolution of merge conflicts.  

This is a summary of the entire analysis process by ChatGPT. For details, please see [the dialogue record for solving the problem of deleting code by mistake](https://github.com/sjmyuan/prompts/blob/main/examples/rca/missing-one-line-of-code-en.md).  

![](https://images.shangjiaming.top/missing-one-line-of-code-en.png)  

## Unable to use Trunk-Based Development  

The problem we want to analyze is: We can't change our branching policy from GitHub Flow to Trunk-Based Development.  

This is a summary of the entire analysis process by ChatGPT. For details, please see [the dialogue record for solving the problem of unable to use Trunk-Based Development](https://github.com/sjmyuan/prompts/blob/main/examples/rca/can-not-do-trunk-base-development-en.md).
![](https://images.shangjiaming.top/can-not-do-trunk-base-development-en.png)  

# Summary  

ChatGPT's questions sometimes lack depth. Although ChatGPT can ask "why" many times, the ending condition completely depends on ChatGPT's own understanding of the root cause. Sometimes it can ask deeper questions, such as asking "Why is there no automated testing?" in the problem of deleting code by mistake, but sometimes it cannot ask deeper questions, such as not continuing to ask "Why do pull requests have a large amount of code changes?". For this problem, we can either provide feedback when ChatGPT waits for our confirmation or perform multiple analyses on the same problem and consider ChatGPT's questions comprehensively.  

ChatGPT is more logical but lacks divergent thinking. This requires us to provide as much information as possible in answering questions so that ChatGPT can ask more questions, triggering deeper thinking.  

Overall, ChatGPT can play a major role in helping us engage in deep thinking and analysis. We need to make more attempts to improve the quality of answers, optimize the interaction process, optimize the principles for identifying root causes, etc., so that ChatGPT can consistently ask deeper questions.