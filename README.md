# AI PRODUCTIVITY ANALYSIS

## Team Members
- Aleks Rzesny
- Hadya Osman
- Betel Yared

# 1. Introduction

Artificial intelligence is often seen as a tool that makes work faster and more efficient. In many cases, this is true – tasks can be completed in less time and teams are able to produce more output. However, this raises a more important question: does producing more actually mean creating more value?

This project explores how AI affects not just productivity, but also the quality of work and overall profitability within business operations. Instead of assuming that higher AI usage automatically leads to better results, we take a closer look at what happens behind the scenes. In particular, we examine whether increased reliance on AI introduces hidden costs, such as rework or errors, that may cancel out the benefits of speed.

Following the project framework, we treat AI not simply as a tool for efficiency, but as something that reshapes the entire workflow. By analyzing how AI interacts with factors like rework, employee experience, and profit, we aim to understand the underlying mechanisms that drive outcomes. By exploring the different pricing models, as well as the impact of AI based on task type and complexity, we strive to gain a deeper understanding of key value-driving factors.

Ultimately, the goal of this project is to determine when AI truly adds value, when it starts to create problems, and how businesses can use it more effectively to make better decisions.

# 2. Methods

## Approach
Our project analyzes AI productivity as a business system rather than only a technical efficiency problem. Instead of simply asking whether AI makes work "faster", we ask whether AI creates real value after accounting for characteristics such as quality, rework, and profitability.

We follow the project framework of:
- Task Characteristics: task type, task complexity
- Input: AI usage percentage or whether the task was AI-assisted, and pricing model
- Process: hours spent, billable hours, rework hours, extra non-billable hours, errors
- Output: outcome score
- Value: cost, revenue and profit

In the notebook, we start by conducting exploratory data analysis to see the basic patterns. We compare how AI usage relates to things like profit, quality of work (outcome score), rework hours, billable hours, task type, and employee seniority. Doing this helps to get an inititial sense of where AI seems helpful of harmful. After that, we go a step further by combining these factors into “value” and “loss” measures. Instead of looking at each variable separately, we try to capture the bigger picture, where AI is actually creating value for the business and where it’s adding hidden costs, like extra rework or unpaid time.

## Features Used

We focused on features that allow us to connect AI usage to business value, quality, and operational cost.

The main features used were:

- ai_usage_pct: to measure how much AI was used in a task
- ai_assisted: indicates whether a task was AI-assisted or not
- cost
- revenue
- profit
- pricing_model: pricing model used for a given task (hourly, fixed or value-based)
- outcome_score: represents the quality or success of the final output
- hours_spent: shows how much time was needed to complete the task
- billable_hours: shows how much time could actually be charged to the client
- rework_hours: captures how much correction or revision work was needed
- task_type: helps compare where AI works better or worse across different kinds of work
- task_complexity_score: measures how difficult a given task is on a 1–5 scale
- seniority: helps test whether experience affects how well employees use AI

We also created an additional feature called `extra_hours`, calculated as:
extra_hours = hours_spent - billable_hours
This helps us capture hidden costs. For instance, if a task takes more time than the company can bill to the client, that extra time reduces the value of the task even if the output was completed.

## Design Choices

Before starting the analysis, we cleaned the dataset to make sure our results would be reliable and meaningful. For example, we removed rows with negative `billable_hours`, since billable hours represent time that can be charged to a client, and negative values do not make sense in a real business setting.

We also removed columns such as `task_id`, `client`, `project_id`, `created_by`, `updated_at`, and `jira_ticket`. These are identifiers or metadata that do not directly help explain how AI affects performance, quality, or profit. Keeping them would add noise without contributing to the analysis.

Next, we standardized categorical variables. In the raw data, the same category sometimes appeared with different spellings (e.g., `SEO `, `contennt`, `desgn`). We corrected these to consistent labels like `seo`, `content`, and `design`. This was crucial because grouping and comparisons would otherwise treat these as different categories which could lead to incorrect conclusions.

For missing numerical values, we chose to use iterative imputation (predicting missing values based on relationships in the data) instead of simply dropping rows. Dropping data would reduce the size of our dataset and potentially remove useful patterns. Instead, imputation allows us to estimate missing values based on relationships between variables. To make sure this did not distort the data, we compared the distributions before and after imputation.

To grasp a measurable impact of AI use on performance, we conducted Gradient Boosting with SHAP analysis. This allowed us to inspect the key factors influencing profit, outcome score, errors and hours spent. 

To better understand how AI actually affects the business, we created two combined measures to capture the bigger picture:

- AI Value Index: shows when AI is helping by improving profit, quality, or saving time  
- AI Loss Index: shows when AI is hurting performance by increasing rework, extra time, or lowering quality  

Instead of trying to interpret each variable separately, these basically serve to see the overall patterns more clearly.

We also grouped AI usage into low, medium, and high levels to make it easier to see if there’s a point where using more AI stops being helpful and starts causing problems, like more rework or lower quality. We can tie this back to our main goal of not just figuring out if AI helps but also understanding when and why it does. 

## Environment
This project was developed in a Jupyter Notebook using Python. Full list of required packages and their versions available in `requirements.txt`.
Main tools and libraries used:
- Python 3.12
- pandas: data loading, cleaning, and grouping
- numpy: numerical operations
- matplotlib: visualizations
- seaborn: statistical plots and heatmaps
- scikit-learn: imputation, scaling, train-test splitting, and machine learning

The main notebook is:
`main.ipynb`

The main dataset is:
`ai_productivity_dataset_final.csv`

# 3. Experimental Design

We designed a set of experiments to see how AI usage affects productivity, quality, and hidden costs. Instead of focusing on just one metric, we looked at factors like rework, time, and quality individually and also combined them to understand overall value and loss. All of the experiments were done in Python using Pandas for data handling and Matplotlib for visualization.

## Data Preparation

Before running the experiments, we cleaned and prepared the data to make sure everything was consistent across all metrics. This included:

- Handling missing values
- Converting categorical variables(like task type and AI usage) into a usable format
- Creating new variables to capture hidden costs

One key derived metric was **extra (non-billable) hours** which captures hidden labor:
```python
df_clean['extra_hours'] = df_clean['hours_spent'] - df_clean['billable_hours']
```
We also categorized AI usage into discrete bins to compare behavior across levels:
```python
df_clean['ai_bin'] = pd.cut(
    df_clean['ai_usage_pct'],
    bins=[0, 0.3, 0.6, 1],
    labels=['Low', 'Medium', 'High'],
    include_lowest=True
)
```
## Experiment 1: AI Usage and Profit

- **Purpose:** To see whether using more AI actually leads to higher profit or if increased usage has little or even negative impact.
  
- **Method:** We analyzed the relationship between **ai_bin** and **profit** using aggregation and visualization.
```python
df_clean.groupby('ai_bin')['profit'].mean().plot(kind='bar')
```
- **Baseline:** Tasks with low Ai usage were used as a baseline for comparison to help us understand whether increasing usage improves results relative to minimal usage.

- **Evaluation Metric:** we used **profit** as the main metric since it directly reflects business value which allowed us to determine whether the AI is actually creating value and not just speeding up work.

## Experiment 2: AI Usage vs Rework

- **Purpose:** To test whether higher usage leads to more rework (which could indicate lower quality or more errors).

- **Method:** We grouped the dataset by AI usage bins and computed the mean rework hours:
```python
df_clean.groupby('ai_bin')['rework_hours'].mean().plot(kind='bar')
```
- **Baseline:** Tasks with low AI usage.

- **Evaluation Metric:**  We used **rework_hours**, since it captures how much additional effort is needed to fix or revise work(important because rework represents any hidden costs).

## Experiment 3: AI Usage vs Quality

- **Purpose:** To evaluate whether AI improves/harms the quality of outputs.

- **Method:** We computed average outcome scores across AI usage levels:

```python
df_clean.groupby('ai_bin')['outcome_score'].mean().plot(kind='bar')
```
- **Baseline:**  Tasks with little or no AI usage.

- **Evaluation Metric:** We used **outcome_score** (which reflects the quality/success of the completed task) to assess whether higher usage corresponds to better or worse outcomes.

## Experiment 4: AI-Assisted vs Non-AI-Assisted Tasks

-**Purpose:** To compare performance between AI-assisted and non-AI-assisted tasks

-**Method:** We grouped tasks based on whether they were AI-assisted and computed averages:

```python
sq_metrics = ['hours_spent', 'outcome_score', 'rework_hours']
sq_df = df_clean.groupby('ai_assisted')[sq_metrics].mean()
```

We then calculated percentage changes relative to the non-AI baseline:

```python
sq_change = {
    'time_change': (sq_df.loc[False, 'hours_spent'] - sq_df.loc[True, 'hours_spent']) / sq_df.loc[False, 'hours_spent'],
    'quality_change': (sq_df.loc[True, 'outcome_score'] - sq_df.loc[False, 'outcome_score']) / sq_df.loc[False, 'outcome_score'],
    'rework_change': (sq_df.loc[True, 'rework_hours'] - sq_df.loc[False, 'rework_hours']) / sq_df.loc[False, 'rework_hours']
}
```

-**Baseline:** Non-AI-assisted tasks

-**Evaluation Metrics:** **hours_spent**(speed); **outcome_score**(quality), **rework_hours**(correction cost). Through these, we were able to compare ai-assisted tasks to non-ai-assisted tasks which helped focus on time, quality and rework.

## Experiment 5: Combined Value Index

-**Purpose:** To capture multiple sources of value-capture in a single metric.

-**Method:** We first computed three value-related metrics:

- Profit
- Outcome score
- Hours spent
  
```python
metrics = ['profit', 'outcome_score', 'hours_spent']
impact_df = df_clean.groupby(['task_type', 'ai_assisted'])[metrics].mean().unstack()
```
We then calculated relative changes between ai-assisted and non-ai tasks an to ensure comparability, all metrics were normalized using MinMax scaling:

```python
scaler = MinMaxScaler()
normalized_lifts = pd.DataFrame(
    scaler.fit_transform(lifts), 
    index=lifts.index, 
    columns=lift_cols
)
```
Finally, we computed a combined value score using:
```python
impact_df['AI_Value_Index'] = normalized_lifts.mean(axis=1)
```
-**Baseline:** non-AI-assisted tasks

-**Evaluation Metrics:** **Profit increase**, **Quality change**(outcome score) and **Time savings**(Hours spent), helping us bring together different value-drivers into one score so they’re easier to understand.

## Experiment 6: Combined Loss Index

-**Purpose:** To capture multiple sources of inefficiency in a single metric.

-**Method:** We first computed three loss-related metrics:

- Rework increase
- Extra (non-billable) hours increase
- Quality decline
```python
loss_metrics = ['rework_hours', 'extra_hours', 'outcome_score']
loss_df = df_clean.groupby(['task_type', 'ai_assisted'])[loss_metrics].mean().unstack()
```
We then calculated relative changes between ai-assisted and non-ai tasks an to ensure comparability, all metrics were normalized using MinMax scaling:

```python
from sklearn.preprocessing import MinMaxScaler

scaler = MinMaxScaler()
normalized_losses = pd.DataFrame(
    scaler.fit_transform(losses),
    index=losses.index,
    columns=losses.columns
)
```
Finally, we computed a combined loss score using:
```python
loss_df['AI_Loss_Index'] = normalized_losses.mean(axis=1)
```
-**Baseline:** non-AI-assisted tasks

-**Evaluation Metrics:** **Rework increase**, **Hidden cost**(extra hours) and **Quality decline**, helping us bring together different inefficiencies into one score so they’re easier to understand.

## Experiment 7: Profit Stability (Risk Analysis)
- **Purpose:** To check whether higher AI usage makes profit more stable or more unpredictable.
- **Method:** We measured the variability of profit across different AI usage levels using standard deviation:
```python
profit_risk = df_clean.groupby('ai_bin')['profit'].std()
profit_risk.plot(kind='bar')
```
- **Baseline:** Low AI usage
- **Evaluation Metric:** profit standard deviation, since it captures how consistent (or risky) outcomes are.

## Experiment 8: Pricing model analysis
- **Purpose:** Compare effect on pricing model viability when AI integration is introduced
- **Method:** We analyzed the financial and operative performance metrics (revenue, cost, profit, and revenue per hour) based on the pricing model and whether the task was AI-assisted or not. 

Find revenue per hour:
```python
df_clean['rev_per_hour'] = df_clean['revenue'] / df_clean['hours_spent']
```
Calculate the pricing impact:
```python
pricing_impact = df_clean.groupby(['pricing_model', 'ai_assisted'])[['revenue', 'cost', 'profit', 'hours_spent', 'rev_per_hour']].mean().reset_index()
```
Visualize the impact:
```python
sns.barplot(ax=axes[0][0], data=pricing_impact, x='pricing_model', y='revenue', hue='ai_assisted', palette='magma')
sns.barplot(ax=axes[0][1], data=pricing_impact, x='pricing_model', y='cost', hue='ai_assisted', palette='magma')
sns.barplot(ax=axes[1][0], data=pricing_impact, x='pricing_model', y='profit', hue='ai_assisted', palette='magma')
sns.barplot(ax=axes[1][1], data=pricing_impact, x='pricing_model', y='rev_per_hour', hue='ai_assisted', palette='magma')
```

## Experiment 8: Impact of AI Usage on Features by Task Complexity and Task Type
- **Purpose:** Find in which areas AI integration brings most value and where AI erodes it
- **Method:** Create a feature (profit, outcome score, errors, rework hours, hours spent) matrix and visualize it as a heatmap.

For example, to create the profit matrix we pivot data to show mean profit by AI usage bin and task complexity:

```python
profit_matrix = df_clean.pivot_table(
    index='task_complexity_score', 
    columns='ai_bin', 
    values='profit', 
    aggfunc='mean'
)
```

Then we create a heatmap to visualize the results:

```python
plt.figure(figsize=(12, 8))
sns.heatmap(profit_matrix, annot=True, fmt=".0f", cmap='RdYlGn', center=0)
plt.title('Profit by Task Complexity and AI Usage')
plt.xlabel('AI Usage Intensity')
plt.ylabel('Task Complexity')
```

The same procedure is repeated for other variables and for task type.

# 4. Results
From our experiments, we found that increasing AI usage does not simply improve performance. While AI can increase profit and sometimes efficiency, it also introduces trade-offs, 
especially in terms of rework and hidden costs. Overall, the results show that AI improves output in some areas, but too much reliance on it can reduce quality and increase correction effort.

## 1. AI Usage vs Profit

Profit generally increases as AI usage increases and we can see that the trend becomes clearer when looking at the smoothed profit curve, which shows a steady upward pattern as AI usage rises.
However, this increase is not perfectly stable and there are fluctuations in profit at different usage levels.

What we can conclude:
- AI helps improve productivity and output value but gains are not perfectly consistent

Raw line plot: Profit vs AI Usage
<img width="592" height="442" alt="image" src="https://github.com/user-attachments/assets/62d38a4e-c08d-4707-a4ca-19140873f1fe" />
We can see in this figure the relationship between AI usage and profit at the individual task level. The data is highly scattered, meaning profit varies a lot even at the same level of AI usage. While there is a slight positive correlation, the high variability suggests that AI does not guarantee consistent outcomes on its own. Some tasks benefit a lot, while others do not, which supports our idea that AI’s impact depends on how it is used rather than just how much it is used.

Smoothed line plot: Smoothed Profit Trend vs AI Usage
<img width="695" height="547" alt="image" src="https://github.com/user-attachments/assets/708546e3-3cb0-4682-b8c6-d18b0069138f" />
After smoothing the data, a clearer pattern appears. Profit generally increases as AI usage rises, especially between 40% and 60% usage, where the trend shows the most consistent growth. There appears to be a noticeable improvement starting around 40–45% AI usage, suggesting this may be a threshold where AI begins to significantly improve productivity. We can also see that below 40% is mostly a decline 
in profit. However, beyond this range (around 75%+ usage), the growth in profit becomes less stable, with small fluctuations and periods of decline. This supports our theory that while AI can increase output, the benefits are not strictly linear and may level off or introduce trade-offs at higher levels of usage, such as increased rework or hidden costs.

## 2. Rework Increases with AI Usage

Rework hours increase as AI usage goes from low to high. Tasks with higher AI usage require more corrections, suggesting that while AI speeds up initial work, it may introduce more mistakes that need to be fixed later.

So:
- AI introduces hidden costs through rework
- Faster output does not always mean better output


Bar chart: Rework Hours by AI Usage Level
<img width="536" height="435" alt="image" src="https://github.com/user-attachments/assets/f940ab01-271a-41f7-9795-5747f28713f0" />
In the graph, we can see an upward trend in rework hours as AI usage increases from low to high. Tasks with high AI usage have the highest average rework while low AI usage has the least which confirms that although AI may help complete tasks faster initially, it leads to more corrections afterward. The visual pattern directly supports the idea that AI introduces additional hidden costs in the form of rework.

## 3. Hidden Costs (Extra Hours)

Extra(non-billable) hours are highest at medium AI usage and slightly decrease at high usage but they remain significant overall. This shows that even when AI is used, 
there is still additional effort that is not captured in billable time.

We can deduce:
- AI does not eliminate inefficiencies
- There are hidden labor costs even when productivity seems higher

Bar chart: Extra Hours by AI Usage Level
<img width="523" height="435" alt="image" src="https://github.com/user-attachments/assets/04ccf199-ac67-4ccc-afdb-68b138545d2e" />
In this graph, extra(non-billable) hours are highest at medium AI usage and then drop a bit at high usage, but they are still higher than at low usage. This means that even when usage is high, there is still extra time being spent that isn’t billed. The graph supports the idea that AI does not remove inefficiencies and that hidden labor costs persist across all usage levels.

## 4. Quality Slightly Decreases

We can see that the outcome scores slightly decrease as AI usage increases. While the difference isn't huge, there is a slight drop noticable when moving from low to high AI usage.

We can deduce:

- AI does not improve quality
- On average, it slightly reduces it

Bar chart: Outcome Score by AI Usage Level
<img width="531" height="435" alt="image" src="https://github.com/user-attachments/assets/68796174-b3de-43a8-a27a-119901e92a02" />
The graph shows a small but consistent decrease in outcome score as AI usage moves from low to high. While the drop is not dramatic, the downward pattern suggests that higher AI usage is associated with slightly lower quality results. This supports the conclusion that AI does not significantly improve quality and may even reduce it slightly(especially when heavily relied on).

## 5. AI vs Non-AI Comparison

When directly comparing AI-assisted tasks to non-AI tasks, we can see that:

a) AI-assisted tasks are faster
b) They have slightly lower quality scores
c) They require less rework

And so we can say that AI is not always faster in practice and that AI-assisted work may require more corrections.

Bar chart: AI Impact: Speed vs Quality
<img width="565" height="374" alt="image" src="https://github.com/user-attachments/assets/85f19060-e1f7-410d-aa45-79a8820dcc60" />

## 6. AI Value and Loss per Task Indices

Looking at the AI Value Index (profit + outcome score + hours spent) we can see that AI assistance does add value, but only to certain tasks. Those that benefit most from AI usage are release and design tasks.

Bar chart: AI Value Index
<img width="850" height="509" alt="image" src="https://github.com/user-attachments/assets/6facec4b-ae0f-400c-a125-f17de2e52b23" />

On the other hand, the AI Loss Index (Rework + Hidden cost + Quality decline) paints the reverse image. Tasks that lose the most value from AI assistance are report and dev.

Bar chart: AI Loss Index
<img width="850" height="509" alt="image" src="https://github.com/user-attachments/assets/c49cbaa7-6d1e-4f86-b160-dd6bb1736459" />

## 7. Pricing models

In order to determine which pricing models perform the best, depending on whether AI was used or not, we compare them in terms of key value metrics: revenue, cost, profit, and revenue per hour. 

In terms of revenue, costs and profit, the value based model seems to perform most similarly whether using or not using AI. For the fixed and hourly priced models, using AI drives down both costs and revenue, which results in greater profits for AI assisted fixed model tasks, however lower profits for AI assisted hourly model tasks. Revenue per hour metrics highlight a unique risk for the hourly model – it is the only pricing structure where AI integration leads to a decrease in hourly earnings.

Grouped bar charts: Impact of AI integration across pricing models
<img width="1784" height="1766" alt="image" src="https://github.com/user-attachments/assets/4e6f81a6-d13a-470f-a0bb-3fc63020558d" />

## 8. Impact of AI Usage on Features by Task Complexity

High AI Usage is associated with greater profits for all tasks complexity levels save for the simplest tasks (complexity 1.0), where Mid or Low usage outperform.

Heatmap: Profit by Task Complexity and AI Usage
<img width="927" height="704" alt="image" src="https://github.com/user-attachments/assets/09ce6c48-ce12-47eb-8e1c-edd4343a5834" />

Highest outcome scores are associated with mid AI usage for all but the simplest tasks (complexity 1.0), where low and high AI usage achieves better results. At the highest complexity level, low AI usage also slightly outperforms mid usage.

Heatmap: Outcome Score by Task Complexity and AI Usage
<img width="940" height="707" alt="image" src="https://github.com/user-attachments/assets/5328a4ab-13c6-474b-a54a-185548e35249" />

## 9. Impact of AI Usage on Features by Task Type

High AI usage achieves greatest profits for release, report and dev tasks. Mid AI usage performs best at ad, article and design tasks. Low AI usage is never best.

Heatmap: Profit by Task Type and AI Usage
<img width="927" height="704" alt="image" src="https://github.com/user-attachments/assets/a9ba89e4-c473-4d7f-ba59-2b91578fa6ff" />

High AI usage is generally associated with lowest outcome scores across all task types (especially poorly in articles and report tasks), except for design tasks where it performs best. Generally, low or mid AI usage performs best with similar scores across the board, except for ticket tasks where low AI usage outperforms.

Heatmap: Outcome Score by Task Type and AI Usage
<img width="940" height="707" alt="image" src="https://github.com/user-attachments/assets/ead263e2-5d1f-4a33-a552-0444a7c53630" />

Across various task types and complexities, high AI usage consistently drives greater profitability and significant time savings, yet these gains are often undermined by a sharp increase in rework and errors. While AI excels at accelerating technical tasks like development and reports, it frequently results in lower outcome scores and quality issues, particularly in creative or simple assignments where mid-to-low usage often performs better. Ultimately, the data suggests a quality-efficiency trade-off where the speed of high AI integration is counterbalanced by the necessity for human correction, especially as task complexity increases.

# 5. Conclusion

The analysis shows that AI usage is now a primary driver of performance – improving profit, speed, and outcomes overall, especially in release and design work where it adds the most value. However, these gains come with trade-offs: higher AI use reduces time and costs but increases rework, variability, and sometimes lowers quality, particularly in complex, report-writing, and development tasks where inefficiencies are most concentrated. Pricing models respond differently – value-based pricing remains stable, fixed pricing benefits from cost reductions, while hourly pricing performs worst because efficiency gains reduce revenue, making it increasingly unviable. Overall, the evidence supports shifting away from hourly billing, applying AI selectively rather than maximally, embedding quality control into workflows, focusing on high-return task areas, and closely tracking rework since it can quietly erode the apparent gains from AI adoption.

While this study identifies clear patterns in how AI usage affects productivity, quality, and profitability, several questions remain unresolved. First, the analysis is observational, meaning we cannot establish causal relationships – higher AI usage may correlate with outcomes like increased profit or rework, but we cannot confirm that AI is the direct cause. Second, the dataset does not capture important contextual factors such as the specific type of AI tools used, user skill level, or the nature of human–AI interaction, all of which could significantly influence results. Additionally, the outcome_score variable is a simplified proxy for quality and may not fully reflect real-world performance or client satisfaction. Future work should address these issues in order to provide a more detailed and comprehensive analysis. Future work could focus on controlled experiments or longitudinal data to better isolate the effects of AI usage and understand how factors such as user experience, prompt quality, and type of AI tool influence outcomes. Additionally, collecting more detailed data on workflow processes and human–AI interaction would help clarify when AI enhances performance and when it introduces inefficiencies.










