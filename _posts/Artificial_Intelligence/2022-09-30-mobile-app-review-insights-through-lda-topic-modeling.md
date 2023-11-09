---
title: "Mobile App Review Insights through LDA Topic Modeling"
categories:
  - Artificial Intelligence
tags:
  - DataMining
  - MachineLearning
  - Data Analysis
  - LDA Topic Modeling
toc: true
toc_sticky: true
toc_label: "Mobile App Review Insights"
toc_icon: "book"
---

## Abstract
In this project, I analyze customer needs through text mining of
healthcare app reviews and based on this, I propose a design strategy
for healthcare apps. I have collected 34,230 reviews from 10 healthcare
apps in the Google Play Store. I performed LDA topic modeling to analyze customer needs in-depth through.

## Dataset
### 1. User reviews
> Collected 34,320 reviews from 10 health care apps in the Google Play Store

> Data Collection Method: Crawiling the reviews on Google Play store

### 2. Preprocessing
The dataset used for preprocessing purposes is as follows:

1. List for word substitution

Since LDA topic modeling provides results based on the most frequent vocabulary, unifying words with the same meaning into a single word is an effective way to perform semantic analysis on text. For example, 'iPhone' and 'galaxy s8' are both the same word as 'smartphone'. A human can determine that the words all have the same meaning, but the computer recognizes them as all different words. This may cause keywords to be missed because the number of occurrences of a word with a specific meaning is counted less as it is used as a different word even though it is a frequent vocabulary. Therefore, in text mining techniques where the frequency of occurrence of words is important, such as LDA topic modeling, prior word replacement is one of the ways to increase the effectiveness of data analysis.

2. List for stopword

A stopword is a word that appears frequently in text mining, but it is a predicate or investigation that is far from the user's reactions or opinions. They have nothing to do with user experience. Therefore, it is necessary to organize these stopwords well in the preprocessing stage.

## LDA topic modeling concepts

Topic Modeling is a text mining methodology that finds key topics in text-based document data. In particular, Latent Dirichlet Allocation (LDA) is the most representative algorithm for topic modeling. Specifically, LDA topic modeling analyzes which topics in a document and at what ratio by analyzing a large amount of document data through a probability-based modeling technique [(Blei et al., 2003)](https://www.jmlr.org/papers/volume3/blei03a/blei03a.pdf?ref=https://githubhelp.com). In addition, since it provides information on what keywords are configured for each topic, it has an effective advantage in deriving insights through keyword combinations. Recently, research has been actively conducted in various fields, such as automatically classifying similar topics on SNS through LDA topic modeling or deriving customer needs by analyzing airline online reviews [(Lu et al., 2013](https://ieeexplore.ieee.org/abstract/document/6454769), [Kwon et al., 2021)](https://www.mdpi.com/2078-2489/12/2/78).

## LDA topic modeling visualization

In this project, considering the review rating is out of 5, I classified 4-5 as positive reviews and 1-2 reviews as negative reviews. LDA topic modeling will be performed and visualized for each review group that received positive/negative ratings, as shown in Figures 1 and 2 below.
![image](https://github.com/sungbinlee/sungbinlee.github.io/assets/52542229/34a6a52f-af6b-4cb2-b4aa-64e3f470788a)
![image](https://github.com/sungbinlee/sungbinlee.github.io/assets/52542229/14fface6-c3d1-4e85-97f5-831288fe09df)

## Code

### 1. Google drive mount

``` python
from google.colab import drive

drive.mount('/content/gdrive')
```
    Drive already mounted at /content/gdrive; to attempt to forcibly remount, call drive.mount("/content/gdrive", force_remount=True).

### 2. Package install and import
``` python
!pip install pyLDAvis==2.1.2
```
``` python
import numpy as np
import pandas as pd
import warnings # ignore warning msg
warnings.filterwarnings(action='ignore')
# Use NLTK
import nltk
import pickle
import re
nltk.download('all')

from tqdm import tqdm # work process visualization
import re # Regular expression package for string
from gensim import corpora # word frequency counting package
import gensim #LDA
import pyLDAvis
import pyLDAvis.gensim
from collections import Counter
```

### 3. Load dataset
``` python
dataset_raw = pd.read_excel('/content/gdrive/MyDrive/NLP-final-project/dataset_raw.xlsx')
dataset_raw.head()
```

<div id="df-9fa29d9c-50d5-47f5-bf6e-ecfe26e017a8">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
    .dataframe tbody tr th {
        vertical-align: top;
    }
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>app</th>
      <th>review</th>
      <th>rating</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>FitCoach: Fitness Coach &amp; Diet</td>
      <td>Its a nice app but so many features don't actu...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1</th>
      <td>FitCoach: Fitness Coach &amp; Diet</td>
      <td>Deceptive. Not at all as advertised. Annoying ...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>FitCoach: Fitness Coach &amp; Diet</td>
      <td>Not the app shown in the Facebook ads. I signe...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>FitCoach: Fitness Coach &amp; Diet</td>
      <td>Updated review. The app is good, the range of ...</td>
      <td>4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>FitCoach: Fitness Coach &amp; Diet</td>
      <td>Was alright in the beginning. You can't change...</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-9fa29d9c-50d5-47f5-bf6e-ecfe26e017a8')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">
        
  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>
      
  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

  <script>
        const buttonEl =
          document.querySelector('#df-9fa29d9c-50d5-47f5-bf6e-ecfe26e017a8 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-9fa29d9c-50d5-47f5-bf6e-ecfe26e017a8');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
  </div>
 </div>

### 4. Data exploration
``` python
dataset_raw.info
```
      <bound method DataFrame.info of                                   app  \
    0      FitCoach: Fitness Coach & Diet   
    1      FitCoach: Fitness Coach & Diet   
    2      FitCoach: Fitness Coach & Diet   
    3      FitCoach: Fitness Coach & Diet   
    4      FitCoach: Fitness Coach & Diet   
    ...                               ...   
    34315    8fit Workouts & Meal Planner   
    34316    8fit Workouts & Meal Planner   
    34317    8fit Workouts & Meal Planner   
    34318    8fit Workouts & Meal Planner   
    34319    8fit Workouts & Meal Planner   
    
                                                      review  rating  
    0      Its a nice app but so many features don't actu...       3  
    1      Deceptive. Not at all as advertised. Annoying ...       1  
    2      Not the app shown in the Facebook ads. I signe...       1  
    3      Updated review. The app is good, the range of ...       4  
    4      Was alright in the beginning. You can't change...       2  
    ...                                                  ...     ...  
    34315        It's the perfect app for a perfect workout.       5  
    34316      good app. i like the meal plans and workouts.       5  
    34317  Easy and practical to use. Love the variety of...       5  
    34318                            It helps ME and MY body       5  
    34319           Good app so far with the first exercise.       5  
    
    [34320 rows x 3 columns]>
### 5. Data preprocessing

#### 1. Check for missing 
``` python
dataset_raw.isnull().sum()
```
    app       0
    review    2
    rating    0
    dtype: int64
#### 2. Remove missing values
``` python
# axis = 0: remove missing value's row
dataset = dataset_raw.dropna(axis = 0)
dataset.isnull().sum()
```
    app       0
    review    0
    rating    0
    dtype: int64
    
#### 3. Load dictionary for preprocessing
``` python
stopword_list = pd.read_excel('/content/gdrive/MyDrive/NLP-final-project/stopword_list.xlsx')
stopword_list.tail()
```
<div id="df-49e016fe-1c4d-41ff-8d62-002ef1c5e5ca">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
    .dataframe tbody tr th {
        vertical-align: top;
    }
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>stopword</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>147</th>
      <td>really</td>
    </tr>
    <tr>
      <th>148</th>
      <td>great</td>
    </tr>
    <tr>
      <th>149</th>
      <td>nice</td>
    </tr>
    <tr>
      <th>150</th>
      <td>like</td>
    </tr>
    <tr>
      <th>151</th>
      <td>love</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-49e016fe-1c4d-41ff-8d62-002ef1c5e5ca')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">
        
  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>
      
  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

  <script>
        const buttonEl =
          document.querySelector('#df-49e016fe-1c4d-41ff-8d62-002ef1c5e5ca button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-49e016fe-1c4d-41ff-8d62-002ef1c5e5ca');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
  </div>
  </div>
  
``` python
replace_list = pd.read_excel('/content/gdrive/MyDrive/NLP-final-project/replace_list.xlsx')
replace_list.head()
```
<div id="df-c7638691-0cae-4b9c-8236-471b5ba190f4">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
    .dataframe tbody tr th {
        vertical-align: top;
    }
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>before_replacement</th>
      <th>after_replacement</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>cell phone</td>
      <td>phone</td>
    </tr>
    <tr>
      <th>1</th>
      <td>smartphone</td>
      <td>phone</td>
    </tr>
    <tr>
      <th>2</th>
      <td>iphone</td>
      <td>phone</td>
    </tr>
    <tr>
      <th>3</th>
      <td>galaxy</td>
      <td>phone</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ipad</td>
      <td>phone</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-c7638691-0cae-4b9c-8236-471b5ba190f4')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">
        
  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>
      
  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

  <script>
        const buttonEl =
          document.querySelector('#df-c7638691-0cae-4b9c-8236-471b5ba190f4 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-c7638691-0cae-4b9c-8236-471b5ba190f4');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
  </div>
  </div>
  
#### 4. Word substitution
```python
def replace_word(review):
    for i in range(len(replace_list['before_replacement'])):
        try:
            # Perform data replacement only when there is a word to be replaced
            if replace_list['before_replacement'][i] in review:
                review = review.replace(replace_list['before_replacement'][i], replace_list['after_replacement'][i])
        except Exception as e:
            print(f"Error: {e}")
    return review
```
```python
dataset['review_prep'] = ''
review_replaced_list = []
for review in tqdm(dataset['review']):
    review_replaced = replace_word(str(review)).lower() #lower case
    review_replaced_list.append(review_replaced)
dataset['review_prep'] = review_replaced_list
dataset.head()
```
<div id="df-b0ffb952-4235-479c-995a-782e83b66d09">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
    .dataframe tbody tr th {
        vertical-align: top;
    }
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>app</th>
      <th>review</th>
      <th>rating</th>
      <th>review_prep</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>FitCoach: Fitness Coach &amp; Diet</td>
      <td>Its a nice app but so many features don't actu...</td>
      <td>3</td>
      <td>its a nice application but so many features do...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>FitCoach: Fitness Coach &amp; Diet</td>
      <td>Deceptive. Not at all as advertised. Annoying ...</td>
      <td>1</td>
      <td>deceptive. not at all as advertised. annoying ...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>FitCoach: Fitness Coach &amp; Diet</td>
      <td>Not the app shown in the Facebook ads. I signe...</td>
      <td>1</td>
      <td>not the application shown in the facebook ads....</td>
    </tr>
    <tr>
      <th>3</th>
      <td>FitCoach: Fitness Coach &amp; Diet</td>
      <td>Updated review. The app is good, the range of ...</td>
      <td>4</td>
      <td>updated review. the application is good, the r...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>FitCoach: Fitness Coach &amp; Diet</td>
      <td>Was alright in the beginning. You can't change...</td>
      <td>2</td>
      <td>was alright in the beginning. you can't change...</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-b0ffb952-4235-479c-995a-782e83b66d09')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">
        
  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>
      
  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

  <script>
        const buttonEl =
          document.querySelector('#df-b0ffb952-4235-479c-995a-782e83b66d09 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-b0ffb952-4235-479c-995a-782e83b66d09');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
  </div>
  </div>

#### 5. Remove non-English text.
``` python
review_removed = list(map(lambda review: re.sub('[^a-zA-Z ]', '', review), dataset['review_prep']))
dataset['review_prep'] = review_removed
```
#### 6. Separation of data based on rating
The Google Play Store has a rating of 5 out of 5. Therefore, in thisproject, 4-5 were classified as positive reviews, and 1-2 were classifiedas negative reviews. This is to distinguish between positive and negative reviews related to the experience of using the service.
``` python
# Positive review (4, 5 out of 5)
review_pos = dataset[(4 == dataset['rating']) | (dataset['rating'] == 5)]['review_prep']
# Negative review (1, 2 out of 5)
review_neg = dataset[(1 == dataset['rating']) | (dataset['rating'] == 2)]['review_prep']
```
``` python
review_pos
```
    3        updated review the application is good the ran...
    13       great way to stay on track with lots of variet...
    18       despite the negative reviews i read i find the...
    20       update after posting review my next workout di...
    21       i loved this application while using it it kic...
                                   ...                        
    34315    its the perfect application for a perfect workout
    34316    good application i like the meal plans and wor...
    34317    easy and practical to use love the variety of ...
    34318                              it helps me and my body
    34319      good application so far with the first exercise
    Name: review_prep, Length: 21017, dtype: object

#### 7. Tokenization
Nouns are a key morpheme to understand the context in a sentence andhave the advantage of being able to easily identify frequent words, only nouns are extracted from the review.
``` python
review_tokenized_pos = list(map(lambda review: nltk.word_tokenize(review), review_pos))
review_tokenized_neg = list(map(lambda review: nltk.word_tokenize(review), review_neg))
```

#### 8. Remove stopwords
``` python
def remove_stopword(tokens):
    review_removed_stopword = []
    for token in tokens:
        # When the number of characters in the token is 2 or more
        if 1 < len(token):
            # Include as review data for analytics only if the token is not a stopword
            if token not in list(stopword_list['stopword']):
                review_removed_stopword.append(token)
    return review_removed_stopword
```
``` python
review_removed_stopword_pos = list(map(lambda tokens : remove_stopword(tokens), review_tokenized_pos))
review_removed_stopword_neg = list(map(lambda tokens : remove_stopword(tokens), review_tokenized_neg))
```

9. Select a specific range of reviews
In general, the longer the review, the more likely it will contain user feedback, such as user experience or technical issues. However, reviews that are rather long may have difficulties in identifying topics or extracting features using combinations of words in the review [(Vasa et al.,2012)](https://dl.acm.org/doi/abs/10.1145/2414536.2414577?casa_token=OQqZbuik4RwAAAAA:_FOT4Fl0bj1reHDV7BNObsGIM1XR6Z7o3TZyBQeIoIX6aEGcUMKhUSMOQv0ytkc9lu3_x7ZI6jDbTVc). Therefore, in this project, only reviews with 3 or more and 15 or less nouns extracted from each review were used for analysis.
``` python
MIN_TOKEN_NUMBER = 3 # Min
MAX_TOKEN_NUMBER = 15 # Max
```
``` python
def select_review(review_removed_stopword):
    review_prep = []
    for tokens in review_removed_stopword:
        if MIN_TOKEN_NUMBER <= len(tokens) <= MAX_TOKEN_NUMBER:
            review_prep.append(tokens)
    return review_prep
```
``` python
review_prep_pos = select_review(review_removed_stopword_pos)
review_prep_neg = select_review(review_removed_stopword_neg)
```

10. Check the preprocessing result

``` python
review_num_pos = len(review_prep_pos)
review_num_neg = len(review_prep_neg)
review_num_tot = review_num_pos + review_num_neg

print(f"Total: {review_num_tot}")
print(f"Positive Reviews: {review_num_pos}({(review_num_pos/review_num_tot)*100:.2f}%)")
print(f"Negative Reviews: {review_num_neg}({(review_num_neg/review_num_tot)*100:.2f}%)")
```
    Total: 13167
    Positive Reviews: 10306(78.27%)
    Negative Reviews: 2861(21.73%)

### 6. LDA Topic Modeling
#### 1. Hyperparameter tuning
``` python
NUM_TOPICS = 10
# passes:  The same concept as the epoch, determining the number of times to train the model with the entire corpus
PASSES = 15
```
#### 2. Model training
``` python
def lda_modeling(review_prep):
    # Word encoding and frequency counting
    dictionary = corpora.Dictionary(review_prep)
    corpus = [dictionary.doc2bow(review) for review in review_prep]
    # LDA model training
    model = gensim.models.ldamodel.LdaModel(corpus,
                                            num_topics = NUM_TOPICS,
                                            id2word = dictionary,
                                            passes = PASSES)
    return model, corpus, dictionary
```
#### 3. Word composition output function by topic
``` python
def print_topic_prop(topics, RATING):
    topic_values = []
    for topic in topics:
        topic_value = topic[1]
        topic_values.append(topic_value)
    topic_prop = pd.DataFrame({"topic_num" : list(range(1, NUM_TOPICS + 1)), "word_prop": topic_values})
    topic_prop.to_excel('/content/gdrive/MyDrive/NLP-final-project/result/topic_prop_' + RATING +  '.xlsx')
    display(topic_prop)
```

#### 4. Visualization function
``` python
def lda_visualize(model, corpus, dictionary, RATING):
    pyLDAvis.enable_notebook()
    result_visualized = pyLDAvis.gensim.prepare(model, corpus, dictionary)
    pyLDAvis.display(result_visualized)
    # Save result
    RESULT_FILE = '/content/gdrive/MyDrive/NLP-final-project/result/lda_result_' + RATING + '.html'
    pyLDAvis.save_html(result_visualized, RESULT_FILE)
```

#### 5. Modeling positive review topics
Using the previously defined model training, topic-specific word composition output function, and visualization function, I will train and visualize the topic modeling model for each positive review and negative review. Here, a total of 10 constituent words (=NUM_WORDS) per topic were set.
``` python
model, corpus, dictionary = lda_modeling(review_prep_pos)
NUM_WORDS = 10
```
``` python
RATING = 'pos'
topics = model.print_topics(num_words = NUM_WORDS)
print_topic_prop(topics, RATING)
```
<div id="df-3afad14d-c2e7-4c92-8b0e-d1f7fbc8249c">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
    .dataframe tbody tr th {
        vertical-align: top;
    }
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>topic_num</th>
      <th>word_prop</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>0.094*"workout" + 0.066*"best" + 0.026*"fitnes...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>0.122*"use" + 0.114*"easy" + 0.028*"simple" + ...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>0.044*"weight" + 0.039*"add" + 0.020*"training...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>0.056*"easy" + 0.046*"workouts" + 0.024*"amazi...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>0.045*"workouts" + 0.028*"meal" + 0.025*"time"...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>0.045*"workouts" + 0.043*"free" + 0.029*"worko...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>0.022*"workout" + 0.022*"exercises" + 0.020*"g...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>0.019*"get" + 0.013*"recipes" + 0.013*"music" ...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>0.021*"fit" + 0.020*"update" + 0.016*"wish" + ...</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>0.051*"track" + 0.038*"keep" + 0.023*"keeps" +...</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-3afad14d-c2e7-4c92-8b0e-d1f7fbc8249c')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">
        
  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>
      
  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

  <script>
        const buttonEl =
          document.querySelector('#df-3afad14d-c2e7-4c92-8b0e-d1f7fbc8249c button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-3afad14d-c2e7-4c92-8b0e-d1f7fbc8249c');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
  </div>
  </div>

``` python
lda_visualize(model, corpus, dictionary, RATING)
```

#### 6. Modeling Negative review topics
``` python
model, corpus, dictionary = lda_modeling(review_prep_neg)
NUM_WORDS = 10
```
``` python
RATING = 'neg'
topics = model.print_topics(num_words = NUM_WORDS)
print_topic_prop(topics, RATING)
```
<div id="df-5943bad4-fdb9-4a30-8e06-9345b10d23ae">
    <div class="colab-df-container">
      <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
    .dataframe tbody tr th {
        vertical-align: top;
    }
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>topic_num</th>
      <th>word_prop</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>0.020*"year" + 0.013*"lost" + 0.011*"app" + 0....</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>0.026*"sign" + 0.026*"google" + 0.015*"keeps" ...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>0.028*"charged" + 0.027*"subscription" + 0.022...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>0.020*"steps" + 0.016*"working" + 0.016*"data"...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>0.036*"steps" + 0.015*"track" + 0.014*"count" ...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>0.023*"use" + 0.017*"account" + 0.016*"dont" +...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>0.039*"pay" + 0.022*"free" + 0.019*"money" + 0...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>0.018*"keeps" + 0.013*"plan" + 0.013*"used" + ...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>0.024*"update" + 0.019*"use" + 0.018*"time" + ...</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>0.035*"free" + 0.026*"download" + 0.022*"get" ...</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-5943bad4-fdb9-4a30-8e06-9345b10d23ae')"
              title="Convert this dataframe to an interactive table."
              style="display:none;">
        
  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M0 0h24v24H0V0z" fill="none"/>
    <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
  </svg>
      </button>
      
  <style>
    .colab-df-container {
      display:flex;
      flex-wrap:wrap;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>
  <script>
        const buttonEl =
          document.querySelector('#df-5943bad4-fdb9-4a30-8e06-9345b10d23ae button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-5943bad4-fdb9-4a30-8e06-9345b10d23ae');
          const dataTable =
            await google.colab.kernel.invokeFunction('convertToInteractive',
                                                     [key], {});
          if (!dataTable) return;

          const docLinkHtml = 'Like what you see? Visit the ' +
            '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
            + ' to learn more about interactive tables.';
          element.innerHTML = '';
          dataTable['output_type'] = 'display_data';
          await google.colab.output.renderOutput(dataTable, element);
          const docLink = document.createElement('div');
          docLink.innerHTML = docLinkHtml;
          element.appendChild(docLink);
        }
      </script>
  </div>
  </div>

### 6. How to interpret the results?
LDA topic modeling provides information on which keywords are composed of each topic and in what ratio. In other words, the user should understand the specific content of the topic through keywords. For example, a topic consisting of keywords such as 'workouts', 'progress', 'easy', and 'plans' will most likely be related to the 'exercise record' feature. As such, it is important for the LDA topic modeling technique to identify which keywords are in the topic and in what ratio. Considering these characteristics, I will discuss how to effectively interpret the data visualized through pyLDAvis.

#### 1. Relevance

Relevance() can be adjusted through the sliding bar on the upper right in the figure below. Relevance is a hyperparameter that balances the frequency of occurrence of a word in a topic with the frequency of its occurrence in the entire document. That is, when there is a word with a high frequency of appearance in a specific topic, whether the word has a high frequency of appearance because it is a keyword that distinguishes the topic from other topics, or simply because it is a word widely used in various document data. It is a parameter that helps to clearly distinguish whether or not was high.

![image](https://github.com/sungbinlee/sungbinlee.github.io/assets/52542229/05893edb-3b0e-4df7-9207-44b24ccfbd72)
> Figure 1. Review topic modeling visualization results of positive evaluation.

The Relevance value is a value between 0 and 1, and the closer it is to 0, the less the number of occurrences in the entire document is, but the focus is on whether the topic is a word that can differentiate it from other topics. On the other hand, the closer the Relevance value is to 1, the more likely it is to be a keyword that appears frequently in the entire document data rather than a keyword constituting a specific topic. For example, in healthcare app review data, the word 'exercise' is very likely to appear in multiple reviews. Therefore, it is difficult to clearly distinguish one topic from another by simply using the word 'exercise'. In these cases, setting the Relevance close to zero can penalize the importance of the word 'exercise', which appears in many reviews. This shows that the word 'exercise' is a widely used word throughout the document, rather than only appearing a lot in that topic. According to a study by [Sievert & Shirley (2014)](https://colab.research.google.com/drive/1JRUe2PGIj7VTBozJEr3fo1rGiPflmCtp#scrollTo=XyABH526UrUu:~:text=The%20Relevance%20value,domain%2C%20dataset%2C%20etc.), a Relevance value of 0.6 is known to be the most effective. However, this value is not always correct. This is because the optimal Relevance value may differ depending on each research domain, dataset, etc.

#### 2. Topics and keywords
All circles on the left in the figure below are each topic. The distance between circles means how similar topics are to each other. A larger circle means that the topic has more words (=tokens). If you hover your mouse over the circle, the ratio of the words constituting the topic to the total document data is displayed on the right side of the current topic's keywords. It also provides the ratio of the words that the topic constitutes to the words of the entire document data. In this way, by identifying which words are composed of each topic and at what ratio, the topic of each topic can be inferred, and furthermore, which topic is composed of the entire document data and at what ratio (= importance).

![image](https://github.com/sungbinlee/sungbinlee.github.io/assets/52542229/285f018f-58db-45ea-9f88-ee382c9fc279)
> Figure 2. Review topic modeling visualization results of Negative evaluation

### 7. insights
Analyze user needs based on the visualization results.

#### 1. Positive review analysis
First, the results of topic modeling of positive reviews are as follows.

![image](https://github.com/sungbinlee/sungbinlee.github.io/assets/52542229/18caa210-c7fd-466c-a2c6-1ba2839bbd06)
* Exercise action description function
  
You can see that the words 'useful', 'accurate', 'easy', 'simple', 'steps', and 'follow' appear frequently in topic 2. It can be interpreted that content that explains the movement of exercise step by step received positive reviews. Therefore, when planning a health care app service, I can consider video-based exercise action lecture content.

* Meal record feature
  
In topic 5, words such as 'meal', 'track', 'schedule', and 'plans' appeared frequently. Through this, it can be interpreted that the meal record function, such as taking a picture of the meal and saving it, had a positive effect on controlling the diet. Computer Vision technology allows you to analyze what food you ate and how much you ate through food photos. This technology is expected to provide a positive user experience in terms of convenience by reducing the hassle of having to record dietary information one by one.

* exercise record feature
  
In topic 10, words such as 'workouts', 'progress', 'motivated', 'steps', 'track', and 'helps' appeared frequently. This can be interpreted as having a positive effect on providing interest and motivation for exercise by making an exercise plan through exercise log recording and checking exercise quantity through exercise amount measurement. As such, adding an exercise log recording feature to the health caring app service that can help users record the amount of exercise and type of exercises, is expected to help promote regular exercise and use of the app.

#### 2. Negative review analysis
![image](https://github.com/sungbinlee/sungbinlee.github.io/assets/52542229/da112c35-343a-45de-bbf4-67c3262fecf3)
* Automatic paid subscription complaint
  
You can see that words such as 'charged', 'refund', 'subscription', 'trial', 'cancel', and 'free' appear frequently in topic 3. This is a number of complaints caused by the operating method of some healthcare apps provide free services for the first 1 to 3 months and then switch to paid subscription services without the user's additional consent after the trial period. There are a lot of reviews in these service policies, where many users request immediate cancellation of subscription and refund. Therefore, in the healthcare app operation method, it is necessary to switch to providing payment and paid service only when the user is additionally asked for payment before switching to a paid subscription service and agrees to this.

* Exercise tracking accuracy issue
  
You can see that the words 'steps', 'track', 'count', 'accurate', 'gps', 'distance', 'mile', 'work', and 'wearable' appear frequently in topic 5. There is a lot of negative feedback related to accuracy issues in exercise tracking, such as step count. For example, the app says that you have taken 10,000 steps, but your smartwatch only counts 5,000 steps. Users may underestimate the reliability of the service as a whole because of the low accuracy of these workout tracking. Therefore, when designing a healthcare service, it is necessary to continuously improve the accuracy of exercise tracking not only in the app but also in the wearable device environment.
