# **American Cereals: A Data-Driven Scoring Model**

## Table of Contents
* [Background & Business Goal](#background--business-goal)
* [Data Overview](#data-overview)
* [Data Preparation & Cleaning](#data-preparation--cleaning)
* [The Custom Health Score & Validation](#the-custom-health-score--validation)
* [Key Insights & Market Gap Analysis](#key-insights--market-gap-analysis)
* [Strategic Recommendations](#strategic-recommendations)

## **Background & Business Goal**
The breakfast cereal market is heavily saturated and driven by marketing tactics that often obscure actual nutritional value. Supermarket shelves are filled with products that claim to be healthy but hide high amounts of sugar and sodium. 

**Project Goal:** To engineer a custom "Health Score" metric based purely on nutritional data, and compare it against actual consumer ratings. By cross-referencing objective health data with subjective consumer popularity, this project aims to uncover deceptive shelf-placement strategies, identify underrated healthy options, and provide actionable insights for both health-conscious consumers and retailers.

## **Data Overview**
* **Dataset:** 77 commercial breakfast cereals.
* **Features:** Macro and micronutrients (calories, protein, fat, sodium, fiber, carbohydrates, sugars, potassium, vitamins) along with manufacturer details and Consumer Reports ratings.
* **Limitations:** The dataset lacks pricing data and real-time sales volume. The consumer rating is an aggregated metric and lacks demographic breakdowns (e.g., age or gender of the raters).

## **Data Preparation & Cleaning**
Python was utilized for its powerful data manipulation capabilities using Pandas, alongside Matplotlib and Seaborn for visualization.

### Importing Libraries
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
```

### Cleaning & Standardization
During the initial audit, I identified anomalous `-1` values in the dataset representing missing data for certain nutrients. 
* Replaced `-1` values in `sugars` and `sodium` with the dataset medians to prevent skewed calculations.
* Replaced remaining `NaN` values across nutritional columns with `0`.

```python
# Replacing anomalies with medians
cereals['sugars'] = cereals['sugars'].replace(-1, cereals['sugars'].median())
cereals['sodium'] = cereals['sodium'].replace(-1, cereals['sodium'].median())

# Filling NAs
columnas_nutricion = ['calories', 'protein', 'fat', 'sodium', 'fiber', 'carbo', 'sugars', 'potass', 'vitamins']
cereals[columnas_nutricion] = cereals[columnas_nutricion].fillna(0)
```

## **The Custom Health Score & Validation**
To quantify the actual nutritional value of each cereal, I engineered a proprietary metric that rewards beneficial nutrients and penalizes unhealthy additives.

**The Formula:** `Health_Score = (Protein + Fiber + Vitamins) / (Sugars + (Sodium / 100) + 1)`

```python
# Engineering the Health_Score column
cereals['Health_Score'] = (cereals['protein'] + cereals['fiber'] + cereals['vitamins']) / (cereals['sugars'] + (cereals['sodium'] / 100) + 1)
```

### Validation vs. Popularity
To validate the metric, I ran a regression analysis against the official Consumer Reports rating to see if healthier cereals were generally rated higher.

```python
# Correlation between custom score and official rating
correlation = cereals['Health_Score'].corr(cereals['rating'])
print(f"Correlation between Health Score and Rating: {correlation:.2f}")

# Visualizing the relationship
plt.figure(figsize=(8, 5))
sns.regplot(data=cereals, x='Health_Score', y='rating', color='green')
plt.title('Validation: Does our Health Score match Consumer Ratings?')
plt.show()
```

<p align="center">
  <img src="Charts/Health Score vs Consumer Ratings.png" width="500" alt="Market Gap Analysis: Health Score vs. Consumer Rating"/>
</p>

## **Key Insights & Market Gap Analysis**

While there is a positive correlation between health and consumer rating, the outliers reveal massive market gaps. By defining "High" and "Low" based on the medians, we can segment the market into behavioral quadrants.

```python
# 1. Define 'High' and 'Low' based on the median
med_rating = cereals['rating'].median()
med_health = cereals['Health_Score'].median()

# 2. Find the "Posers" and "Hidden Gems"
posers = cereals[(cereals['rating'] > med_rating) & (cereals['Health_Score'] < med_health)]
hidden_gems = cereals[(cereals['rating'] < med_rating) & (cereals['Health_Score'] > med_health)]

print(f"We found {len(posers)} 'Posers' and {len(hidden_gems)} 'Hidden Gems'.")
```

### The Quadrant Analysis
By mapping Health Score against Consumer Ratings and using the medians as crosshairs, the market was segmented into four distinct categories:
* **The Winners:** Healthy & Popular (High Score, High Rating)
* **The Posers:** Tasty but Junk (Low Score, High Rating)
* **Hidden Gems:** Healthy but Underrated (High Score, Low Rating)
* **The Losers:** Unhealthy & Disliked (Low Score, Low Rating)

```python
plt.figure(figsize=(12, 8))
sns.set_theme(style="white")
sns.scatterplot(data=cereals, x='Health_Score', y='rating', s=100, alpha=0.7, color='teal', edgecolor='w')

plt.axvline(med_health, color='red', linestyle='--', alpha=0.5, label='Median Health')
plt.axhline(med_rating, color='blue', linestyle='--', alpha=0.5, label='Median Rating')

plt.text(med_health + 0.1, 80, "THE WINNERS\n(Healthy & Popular)", color='green', fontweight='bold', ha='left', va='center')
plt.text(0.1, 80, "THE POSERS\n(Tasty but Junk)", color='orange', fontweight='bold', ha='right', va='center')
plt.text(0.1, 20, "THE LOSERS\n(Unhealthy & Disliked)", color='red', fontweight='bold', ha='right', va='center')
plt.text(med_health + 0.1, 20, "HIDDEN GEMS\n(Healthy but Underrated)", color='blue', fontweight='bold', ha='left', va='center')

plt.title('Market Gap Analysis: Health Score vs. Consumer Rating', fontsize=16, pad=20)
plt.xlabel('Custom Health Score (Engineered Metric)', fontsize=12)
plt.ylabel('Consumer Reports Rating', fontsize=12)
plt.grid(True, alpha=0.2)
plt.show()
```

<p align="center">
  <img src="Charts/Market Gap Analysis.png" width="700" alt="Market Gap Analysis: Health Score vs. Consumer Rating"/>
</p>

## **Strategic Recommendations**

<ul>
  <li>
    <strong>Retail Repositioning for "Hidden Gems":</strong> Supermarkets should identify the "Hidden Gems" (cereals with high health scores but low consumer ratings) and move them to eye-level shelving or include them in "Healthy Start" promotional displays to boost awareness of genuinely nutritious products.
  </li>
  <li>
    <strong>Consumer Awareness Campaigns:</strong> Health and wellness apps or consumer advocacy groups can use this scoring model to expose "The Posers" cereals that enjoy high popularity and "health halo" marketing but objectively fail the nutritional test. 
  </li>
  <li>
    <strong>Product Reformulation:</strong> Manufacturers whose flagship products fall into "The Posers" or "The Losers" quadrants have a clear data-backed mandate to reformulate. Gradually reducing sugar and sodium while fortifying with fiber and vitamins can shift their products into the "Winners" quadrant without sacrificing their established consumer base.
  </li>
</ul>

---
