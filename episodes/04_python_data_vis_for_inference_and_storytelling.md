---
title: 'Data Visualization with Python for Statistical Inference and Storytelling'
teaching: 10
exercises: 2
---

:::::::::::::::::::::::::::::::::::::: questions 

- How can you create scatter plots, bubble charts, and correlograms with Python?
- When are these graphs useful for inferring information from data?
- How are these visualizations valuable for humanists?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Create scatter plots, bubble charts and correlograms in Python, using the Seaborn library
- Understand the process of creating these graphs to infer information from a dataset
- Comprehend how these graphs can be used to infer insights from humanities data and for data storytelling

::::::::::::::::::::::::::::::::::::::::::::::::

In the previous episodes, we explored ten types of graphs and their use cases, as well as the concepts of correlation 
and regression in the context of inferential statistics. Now, it’s time to put this knowledge into practice! 
In this episode, we’ll work with the *Income and Happiness Correlation dataset* from Kaggle (see the Setup episode), 
which consists of 111 data points. We’ll visualize this dataset and learn how to perform inferential statistical 
analysis on it.

::::::::::::::::::::::::::::::::::::::: discussion

### Note

Before visualizing any dataset, it’s important to answer the following questions:

1. What kind of data is stored in the dataset?
2. What are the dimensions of the dataset?
3. Why do I want to visualize the dataset? What information do I hope to gain through visualization? 
And which type of graph best represents the information I'm looking for?

Let’s answer these questions for our dataset by writing some code.

:::::::::::::::::::::::::::::::::::::::::::::::::::


## 4.1. Exploring the Dataset

The dataset we're working with is stored in a CSV (comma-separated values) file on GitHub. Let's load it into 
our notebook and store it in a pandas DataFrame called `happy_df`: 

```
import pandas as pd

# path to the dataset: 
url= "https://raw.githubusercontent.com/HERMES-DKZ/data_challenges_data_carpentries/main/\
data_carpentries/statistical_inferece_data_visualization/data_statistical_inference_data_visualization/income_happiness_correlation.csv"

# loading the dataset and storing it in a pandas DataFrame:
happy_df= pd.read_csv(url)

# displaying the first five rows of the DataFrame: 
happy_df.head()

```

![](fig/output_1.png)

Take a moment to examine the first five rows of the DataFrame. What types of values do you see in each column? 
Which columns contain numerical values, and which contain categorical values? What information does the dataset 
include, and what information might be missing?

Run the following line of code to gain more information about the structure of `happy_df`: 

```
# displaying information about the DataFrame:
happy_df.info()
```

![](fig/output_2.png)

::::::::::::::::::::::::::::::::::::: challenge 

## Question

Now that you know the dataset better, you can answer the question: what information can be *inferred* from 
this dataset? Which column contains values that could be dependent on other features, and thereby correlates 
with them? Would it be possible to predict the values of this column, given the values of one or more other 
columns in the dataset?

:::::::::::::::::::::::: solution 

## Answer
 
Intuitively, one might assume that the `happyScore` column contains values that could be predicted based on 
the other features. So, it may be possible to estimate the average happiness score of a country’s population 
if we know its region, GDP, income inequality, and average income. In other words, there might be a correlation 
between `happyScore` and the other features.

But how can we determine with greater confidence that such a correlation exists, and identify which features 
are more strongly correlated with `happyScore` than others? Data visualization can help us answer these questions.

:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::

## 4.2. Drawing Heatmaps

One of the best and easiest ways to visualize correlations is through correlographic heatmaps. However, heatmaps 
can only show how changes in one *numerical value* are correlated with changes in another *numerical value*. 
Therefore, to create a heatmap of all numerical features that could be correlated with `happyScore`, we need to 
exclude the columns in `happy_df` that contain non-numerical values:

```
# selecting only the columns whose values are not of type 'object' and storing them in a new DataFrame:
numerical_df= happy_df.select_dtypes(exclude=['object'])

# displaying the first five rows of the new DataFrame:
numerical_df.head()
```

![](fig/output_3.png)

Now, let's use the Python library [Seaborn](https://seaborn.pydata.org/) to create a heatmap of all the values 
in `numerical_df`:

```
import matplotlib.pyplot as plt
import seaborn as sns

# creating a matrix that contains the correlation of every feature in the DataFrame with every other feature:
corr= numerical_df.corr(method='pearson')

# defining the size of the graph: 
plt.figure(figsize=(9, 7))

# generating a heatmap of the corr matrix, using the seaborn library:
sns.heatmap(corr, annot=True, fmt=".2f", cmap='coolwarm', cbar=True)

# giving the graph a title:
plt.title('correlation heatmap')

# diyplaying the graph: 
plt.show()
```

![](fig/output_4.png)

In the code above, we’ve set `cmap='coolwarm'`. This means we want the heatmap to distinguish between negative 
and positive correlations using red and blue colors. More saturated shades of blue or red indicate stronger 
negative or positive correlation values

We've set `annot=True` in our code, which means we want the correlation coefficients to be displayed on the heatmap. 
The correlation coefficients are calculated using the [Pearson method](https://www.youtube.com/watch?v=k7IctLRiZmo) 
in `corr= numerical_df.corr(method='pearson')`. 

"The Pearson correlation measures the strength of the linear relationship between two variables. It has a value 
between -1 to +1, with a value of -1 meaning a total negative linear correlation, 0 being no correlation, and +1 
meaning a total positive correlation." ([ScienceDirect](https://www.sciencedirect.com/topics/computer-science/pearson-correlation#:~:text=content%20were%20calculated.-,The%20Pearson%20correlation%20measures%20the%20strength%20of%20the%20linear%20relationship,meaning%20a%20total%20positive%20correlation))

#### How to read and interpret the heatmap:

- Darker red colors, accompanied by values closer to +1, indicate stronger positive correlations. 
This means that as one value *increases* at a certain rate, the other *increases* at a similar rate.
- Darker blue colors, accompanied by values closer to -1, indicate stronger negative correlations. This means 
that as one value *increases* at a certain rate, the other *decreases* at a similar rate.

::::::::::::::::::::::::::::::::::::: challenge 

## Question

What patterns does the heatmap above reveal?

:::::::::::::::::::::::: solution 

## Answer
 
- Each feature is most strongly correlated with itself, with a correlation coefficient of +1.
- Values derived from the same feature demonstrate a high correlation. For example, the correlation 
coefficient between avg_satisfaction and adjusted_satisfaction is +0.98, because both stem from the 
satisfaction degree. The same is true about avg_income and median_income with the correlation coefficient being +1.

:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::

To create a more precise graph without redundant information, let's retain only one column from the 
DataFrame that contains data on satisfaction or income, and remove the others:

```
# dropping a list of columns from numerical_df and storing the result in a new DataFrame:
reduced_numerical_df= numerical_df.drop(['adjusted_satisfaction', 'std_satisfaction', 'median_income'], axis=1)

reduced_numerical_df.head()
```

![](fig/output_5.png)

Let's create the heatmap again, this time using `reduced_numerical_df` insted of `numerical_df`:

```
corr= reduced_numerical_df.corr(method='pearson')
plt.figure(figsize=(5.5, 4))
sns.heatmap(corr, annot=True, fmt=".2f", cmap='coolwarm', cbar=True)
plt.title('reduced correlation heatmap')
plt.show()
```

![](fig/output_6.png)

::::::::::::::::::::::::::::::::::::::: discussion

### Insights

This heatmap is more meaninful than the previous one and reveals more insights:

- `avg_satisfaction` and `avg_income` have a correlation greater that +0.5, indicating that higher income 
is correlated with greater life satisfaction.
- There is a strong positive correlation between `avg_satisfaction` and `happyScore`.
- `avg_satisfaction` shows a positive correlation with `GDP`.
- There is a positive correlation between `avg_income` and `happyScore`.
- There is a positive correlation between `avg_income` and `GDP`.
- The slight negative correlation between `avg_income` and `income_inequality` is also interesting: as the average 
income in a country increases, income inequality tends to decrease.
- There is a negative correlation between `GDP` and `income_inequality`: higher GDP in a country is associated 
with lower income inequality.

:::::::::::::::::::::::::::::::::::::::::::::::::::

Let's take a closer look at the correlations we've observed between the `happyScore` and the other features by 
drawing different graphs. 

## 4.3. Drawing Scatter Plots

**WE ARE HERE**


::::::::::::::::::::::::::::::::::::: keypoints 

- Use `.md` files for episodes when you want static content
- Use `.Rmd` files for episodes when you need to generate output
- Run `sandpaper::check_lesson()` to identify any issues with your lesson
- Run `sandpaper::build_lesson()` to preview your lesson locally

::::::::::::::::::::::::::::::::::::::::::::::::

