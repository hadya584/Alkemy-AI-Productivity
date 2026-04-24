# AI PRODUCTIVITY ANALYSIS

## Team Members
- Aleks___
- Hadya Osman
- Betel Yared

# 1. Introduction

Artificial intelligence is often seen as a tool that makes work faster and more efficient. In many cases, this is true—tasks can be completed in less time and teams are able to produce more output. However, this raises a more important question: does producing more actually mean creating more value?

This project explores how AI affects not just productivity, but also the quality of work and overall profitability within business operations. Instead of assuming that higher AI usage automatically leads to better results, we take a closer look at what happens behind the scenes. In particular, we examine whether increased reliance on AI introduces hidden costs, such as rework or errors, that may cancel out the benefits of speed.

Following the project framework, we treat AI not simply as a tool for efficiency, but as something that reshapes the entire workflow. By analyzing how AI interacts with factors like rework, employee experience, and profit, we aim to understand the underlying mechanisms that drive outcomes.

Ultimately, the goal of this project is to determine when AI truly adds value, when it starts to create problems, and how businesses can use it more effectively to make better decisions.

# 2. Methods

## Approach
Our project analyzes AI productivity as a business system rather than only a technical efficiency problem. Instead of simply asking whether AI makes work "faster", we ask whether AI creates real value after accounting for quality, rework, and profitability.

We follow the project framework of:
- Input:AI usage percentage and whether the task was AI-assisted
- Process:hours spent, billable hours, rework hours, and extra non-billable hours
- Output:outcome score and task completion quality
- Value: profit and financial impact

In the notebook, we start by exploring the data to see basic patterns. We compare how AI usage relates to things like profit, quality of work (outcome score), rework hours, billable hours, task type, and employee seniority. Doing this helps to get an inititial sense of where AI seems helpful of harmful. After that, we go a step further by combining these factors into “value” and “loss” measures. Instead of looking at each variable separately, we try to capture the bigger picture, where AI is actually creating value for the business and where it’s adding hidden costs, like extra rework or unpaid time.

## Features Used

We focused on features that allow us to connect AI usage to business value, quality, and operational cost.

The main features used were:

- ai_usage_pct: to measure how much AI was used in a task
- ai_assisted: indicates whether a task was AI-assisted or not
- profit: measures the financial outcome of each task
- outcome_score: represents the quality or success of the final output
- hours_spent: shows how much time was needed to complete the task
- billable_hours: shows how much time could actually be charged to the client
- rework_hours: captures how much correction or revision work was needed
- task_type: helps compare where AI works better or worse across different kinds of work
- seniority: helps test whether experience affects how well employees use AI

We also created an additional feature called `extra_hours`, calculated as:
extra_hours = hours_spent - billable_hours
This helps us capture hidden costs. For instance, if a task takes more time than the company can bill to the client, that extra time reduces the value of the task even if the output was completed.

## Design Choices

Before starting the analysis, we cleaned the dataset to make sure our results would be reliable and meaningful. For example, we removed rows with negative `billable_hours`, since billable hours represent time that can be charged to a client, and negative values do not make sense in a real business setting.

We also removed columns such as `task_id`, `client`, `project_id`, `created_by`, `updated_at`, and `jira_ticket`. These are identifiers or metadata that do not directly help explain how AI affects performance, quality, or profit. Keeping them would add noise without contributing to the analysis.

Next, we standardized categorical variables. In the raw data, the same category sometimes appeared with different spellings (e.g., `seo `, `contennt`, `desgn`). We corrected these to consistent labels like `seo`, `content`, and `design`. This was crucial because grouping and comparisons would otherwise treat these as different categories which could lead to incorrect conclusions.

For missing numerical values, we chose to use iterative imputation (predicting missing values based on relationships in the data) instead of simply dropping rows. Dropping data would reduce the size of our dataset and potentially remove useful patterns. Instead, imputation allows us to estimate missing values based on relationships between variables. To make sure this did not distort the data, we compared the distributions before and after imputation.

We also created a new variable called `extra_hours` to capture hidden costs. We defined it as:
`extra_hours = hours_spent - billable_hours`

For example, if a task takes more time than what can be billed to the client, that extra time reduces profitability even if the task is completed successfully.

To better understand how AI actually affects the business, we created two combined measures to capture the bigger picture:

- AI Value Index: shows when AI is helping by improving profit, quality, or saving time  
- AI Loss Index:shows when AI is hurting performance by increasing rework, extra time, or lowering quality  

Instead of trying to interpret each variable separately, these basically serve to see the overall patterns more clearly.

We also grouped AI usage into low, ledium, and high levels to make it easier to see if there’s a point where using more AI stops being helpful and starts causing problems, like more rework or lower quality. We can tie this back to our main goal of not just figuring out if AI helps but also understanding when and why it does. 

## Environment
