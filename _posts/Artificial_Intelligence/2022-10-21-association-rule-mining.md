---
title: "Association Rule Mining"
categories:
  - Artificial Intelligence
tags:
  - Data Mining
  - Machine Learning
  - Data Analysis
toc: true
toc_sticky: true
toc_label: "Association Rule Mining"
toc_icon: "news"
---


## What is Association Rule Mining?
Association rule mining is a rule-based machine-learning method for finding interesting relationships between items that were not previously found in large databases. By using association rule analysis, it is possible to know which products are additionally purchased by customers who have purchased a specific product. Through this, it is possible to know what efficient product display is and which products should be bundled, and through this, a marketing strategy can be established.

## How to analyze association rules

In this project, I will use the Apriori algorithm, which is a representative method for finding association rules with minimum support and minimum confidence.

The Apriori algorithm consists of two steps.

1.  Create a frequent item set
2.  Create association rule

A Frequent Item Set refers to a set of items with a specific number of occurrences or more for each item.

When generating association rules, all subsets except the empty set of Frequent Item Sets are considered, and among them, association rules exceeding the minimum confidence are found.


## Considerations

3.1 Selection of useful association rules

Not all association rules found from association rule analysis will be useful. This is because there are useful rules, rules that are too obvious, and rules that are difficult to relate to.

For example, the rule \"men who go to the supermarket on Saturdays tend to buy beer with their baby diapers\" is useful information because it can be used directly in a marketing strategy. On the other hand, the information that \"customers who buy iPhones tend to buy iPhones\" is not a worthwhile rule because it is somewhat obvious. Lastly, it will be difficult to find a correlation with the rule, \"A lot of fans are sold in places that sell groceries.\" In conclusion, it is up to the analyst to decide which rule is useful among the rules obtained through the analysis of the association rules.


3.2 calculation problem

As the number of items increases, the computational amount increases exponentially, and it takes an enormous amount of time to find association rules.

## Code 

Mount a drive:
``` python
from google.colab import drive

drive.mount('/content/gdrive')
```
    Mounted at /content/gdrive


Import packages:
``` python
import matplotlib.pyplot as plt
import matplotlib.colors as mcl
import pandas as pd
import numpy as np

from matplotlib.colors import LinearSegmentedColormap
from mlxtend.preprocessing import TransactionEncoder
from mlxtend.frequent_patterns import apriori, association_rules
```

Load data:

> This dataset has 7500 transactions over the course of a week at a French retail store

``` python
store_df = pd.read_csv('/content/gdrive/MyDrive/final-project/store_data.csv', header=None)
store_df.head()

```

  <div id="df-249d1b51-b6e0-43d0-abe0-0595fa22166a">
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
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>10</th>
      <th>11</th>
      <th>12</th>
      <th>13</th>
      <th>14</th>
      <th>15</th>
      <th>16</th>
      <th>17</th>
      <th>18</th>
      <th>19</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>shrimp</td>
      <td>almonds</td>
      <td>avocado</td>
      <td>vegetables mix</td>
      <td>green grapes</td>
      <td>whole weat flour</td>
      <td>yams</td>
      <td>cottage cheese</td>
      <td>energy drink</td>
      <td>tomato juice</td>
      <td>low fat yogurt</td>
      <td>green tea</td>
      <td>honey</td>
      <td>salad</td>
      <td>mineral water</td>
      <td>salmon</td>
      <td>antioxydant juice</td>
      <td>frozen smoothie</td>
      <td>spinach</td>
      <td>olive oil</td>
    </tr>
    <tr>
      <th>1</th>
      <td>burgers</td>
      <td>meatballs</td>
      <td>eggs</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>chutney</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>turkey</td>
      <td>avocado</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>mineral water</td>
      <td>milk</td>
      <td>energy bar</td>
      <td>whole wheat rice</td>
      <td>green tea</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-249d1b51-b6e0-43d0-abe0-0595fa22166a')"
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
          document.querySelector('#df-249d1b51-b6e0-43d0-abe0-0595fa22166a button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-249d1b51-b6e0-43d0-abe0-0595fa22166a');
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


Generate frequent item set


``` python
records = []
for i in range(len(store_df)):
    records.append([str(store_df.values[i,j]) \
                    for j in range(len(store_df.columns)) if not pd.isna(store_df.values[i,j])])
```

Creates a dataframe for performing association rule analysis using mlxtend library.

This dataframe will be 1 or 0 if each row contains an item or not.

```python
te = TransactionEncoder()
te_ary = te.fit(records).transform(records, sparse=True)
te_df = pd.DataFrame.sparse.from_spmatrix(te_ary, columns=te.columns_)
te_df.head()
```



  <div id="df-4eb9d259-8865-428b-b2df-bc82dccd2de2">
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
      <th>asparagus</th>
      <th>almonds</th>
      <th>antioxydant juice</th>
      <th>asparagus</th>
      <th>avocado</th>
      <th>babies food</th>
      <th>bacon</th>
      <th>barbecue sauce</th>
      <th>black tea</th>
      <th>blueberries</th>
      <th>...</th>
      <th>turkey</th>
      <th>vegetables mix</th>
      <th>water spray</th>
      <th>white wine</th>
      <th>whole weat flour</th>
      <th>whole wheat pasta</th>
      <th>whole wheat rice</th>
      <th>yams</th>
      <th>yogurt cake</th>
      <th>zucchini</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 120 columns</p>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-4eb9d259-8865-428b-b2df-bc82dccd2de2')"
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
          document.querySelector('#df-4eb9d259-8865-428b-b2df-bc82dccd2de2 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-4eb9d259-8865-428b-b2df-bc82dccd2de2');
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
  


The minimum support was 0.005 and the maximum number of item sets was
set to three.

``` python
frequent_itemset = apriori(te_df,
                           min_support=0.005,
                           max_len=3,
                           use_colnames=True
                          )
frequent_itemset['length'] = frequent_itemset['itemsets'].map(lambda x: len(x))
frequent_itemset.sort_values('support',ascending=False,inplace=True)
```


``` python
frequent_itemset
```


  <div id="df-e7948339-6243-4ad6-b295-ecdba9c81d92">
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
      <th>support</th>
      <th>itemsets</th>
      <th>length</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>60</th>
      <td>0.238368</td>
      <td>(mineral water)</td>
      <td>1</td>
    </tr>
    <tr>
      <th>27</th>
      <td>0.179709</td>
      <td>(eggs)</td>
      <td>1</td>
    </tr>
    <tr>
      <th>83</th>
      <td>0.174110</td>
      <td>(spaghetti)</td>
      <td>1</td>
    </tr>
    <tr>
      <th>33</th>
      <td>0.170911</td>
      <td>(french fries)</td>
      <td>1</td>
    </tr>
    <tr>
      <th>20</th>
      <td>0.163845</td>
      <td>(chocolate)</td>
      <td>1</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>646</th>
      <td>0.005066</td>
      <td>(tomatoes, mineral water, eggs)</td>
      <td>3</td>
    </tr>
    <tr>
      <th>648</th>
      <td>0.005066</td>
      <td>(spaghetti, eggs, olive oil)</td>
      <td>3</td>
    </tr>
    <tr>
      <th>674</th>
      <td>0.005066</td>
      <td>(soup, mineral water, frozen vegetables)</td>
      <td>3</td>
    </tr>
    <tr>
      <th>680</th>
      <td>0.005066</td>
      <td>(grated cheese, mineral water, ground beef)</td>
      <td>3</td>
    </tr>
    <tr>
      <th>724</th>
      <td>0.005066</td>
      <td>(pancakes, spaghetti, olive oil)</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
<p>725 rows × 3 columns</p>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-e7948339-6243-4ad6-b295-ecdba9c81d92')"
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
          document.querySelector('#df-e7948339-6243-4ad6-b295-ecdba9c81d92 button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-e7948339-6243-4ad6-b295-ecdba9c81d92');
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



The code below is the code that extracts the association rule. Only
association rules with a confidence level of 0.005 or higher are
extracted.


``` python
association_rules_df = association_rules(frequent_itemset,
                                         metric='confidence',
                                         min_threshold=0.005,
                                        )
all_confidences = []
collective_strengths = []
cosine_similarities = []
for _,row in association_rules_df.iterrows():
    all_confidence_if = list(row['antecedents'])[0]
    all_confidence_then = list(row['consequents'])[0]
    if row['antecedent support'] <= row['consequent support']:
        all_confidence_if = list(row['consequents'])[0]
        all_confidence_then = list(row['antecedents'])[0]
    all_confidence = {all_confidence_if+' => '+all_confidence_then : \
                      row['support']/max(row['antecedent support'], row['consequent support'])}
    all_confidences.append(all_confidence)

    violation = row['antecedent support'] + row['consequent support'] - 2*row['support']
    ex_violation = 1-row['antecedent support']*row['consequent support'] - \
                    (1-row['antecedent support'])*(1-row['consequent support'])
    collective_strength = (1-violation)/(1-ex_violation)*(ex_violation/violation)
    collective_strengths.append(collective_strength)

    cosine_similarity = row['support']/np.sqrt(row['antecedent support']*row['consequent support'])
    cosine_similarities.append(cosine_similarity)

association_rules_df['all-confidence'] = all_confidences
association_rules_df['collective strength'] = collective_strengths
association_rules_df['cosine similarity'] = cosine_similarities
```

``` python
association_rules_df.head()
```

  <div id="df-4facfdc5-a5bc-43a5-9802-45f464566fac">
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
      <th>antecedents</th>
      <th>consequents</th>
      <th>antecedent support</th>
      <th>consequent support</th>
      <th>support</th>
      <th>confidence</th>
      <th>lift</th>
      <th>leverage</th>
      <th>conviction</th>
      <th>all-confidence</th>
      <th>collective strength</th>
      <th>cosine similarity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>(spaghetti)</td>
      <td>(mineral water)</td>
      <td>0.174110</td>
      <td>0.238368</td>
      <td>0.059725</td>
      <td>0.343032</td>
      <td>1.439085</td>
      <td>0.018223</td>
      <td>1.159314</td>
      <td>{'mineral water =&gt; spaghetti': 0.2505592841163...</td>
      <td>1.185493</td>
      <td>0.293172</td>
    </tr>
    <tr>
      <th>1</th>
      <td>(mineral water)</td>
      <td>(spaghetti)</td>
      <td>0.238368</td>
      <td>0.174110</td>
      <td>0.059725</td>
      <td>0.250559</td>
      <td>1.439085</td>
      <td>0.018223</td>
      <td>1.102008</td>
      <td>{'mineral water =&gt; spaghetti': 0.2505592841163...</td>
      <td>1.185493</td>
      <td>0.293172</td>
    </tr>
    <tr>
      <th>2</th>
      <td>(chocolate)</td>
      <td>(mineral water)</td>
      <td>0.163845</td>
      <td>0.238368</td>
      <td>0.052660</td>
      <td>0.321400</td>
      <td>1.348332</td>
      <td>0.013604</td>
      <td>1.122357</td>
      <td>{'mineral water =&gt; chocolate': 0.220917225950783}</td>
      <td>1.135588</td>
      <td>0.266463</td>
    </tr>
    <tr>
      <th>3</th>
      <td>(mineral water)</td>
      <td>(chocolate)</td>
      <td>0.238368</td>
      <td>0.163845</td>
      <td>0.052660</td>
      <td>0.220917</td>
      <td>1.348332</td>
      <td>0.013604</td>
      <td>1.073256</td>
      <td>{'mineral water =&gt; chocolate': 0.220917225950783}</td>
      <td>1.135588</td>
      <td>0.266463</td>
    </tr>
    <tr>
      <th>4</th>
      <td>(mineral water)</td>
      <td>(eggs)</td>
      <td>0.238368</td>
      <td>0.179709</td>
      <td>0.050927</td>
      <td>0.213647</td>
      <td>1.188845</td>
      <td>0.008090</td>
      <td>1.043158</td>
      <td>{'mineral water =&gt; eggs': 0.21364653243847875}</td>
      <td>1.076638</td>
      <td>0.246056</td>
    </tr>
  </tbody>
</table>
</div>
      <button class="colab-df-convert" onclick="convertToInteractive('df-4facfdc5-a5bc-43a5-9802-45f464566fac')"
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
          document.querySelector('#df-4facfdc5-a5bc-43a5-9802-45f464566fac button.colab-df-convert');
        buttonEl.style.display =
          google.colab.kernel.accessAllowed ? 'block' : 'none';

        async function convertToInteractive(key) {
          const element = document.querySelector('#df-4facfdc5-a5bc-43a5-9802-45f464566fac');
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
  

### Results

``` python
max_i = 4
for i, row in association_rules_df.iterrows():
    print("Rule: " + list(row['antecedents'])[0] + " => " + list(row['consequents'])[0])

    print("Support: " + str(round(row['support'],2)))

    print("Confidence: " + str(round(row['confidence'],2)))
    print("Lift: " + str(round(row['lift'],2)))
    print("=====================================")
    if i==max_i:
        break
```

    Rule: spaghetti => mineral water
    Support: 0.06
    Confidence: 0.34
    Lift: 1.44
    =====================================
    Rule: mineral water => spaghetti
    Support: 0.06
    Confidence: 0.25
    Lift: 1.44
    =====================================
    Rule: chocolate => mineral water
    Support: 0.05
    Confidence: 0.32
    Lift: 1.35
    =====================================
    Rule: mineral water => chocolate
    Support: 0.05
    Confidence: 0.22
    Lift: 1.35
    =====================================
    Rule: mineral water => eggs
    Support: 0.05
    Confidence: 0.21
    Lift: 1.19
    =====================================


Looking at the first line, the rule that customers who buy \'mineral
water\' buy \'spaghetti\' has a support of 0.06, a confidence of 0.25,
and a lift of 1.44. Since lift is greater than 1, it can be interpreted
that \'mineral water\' and spaghetti have a kind of positive
correlation.

The code below is a way to visualize association rule analysis results.

``` python
support = association_rules_df['support']
confidence = association_rules_df['confidence']

h = 347
s = 1
v = 1
colors = [
    mcl.hsv_to_rgb((h/360, 0.2, v)),
    mcl.hsv_to_rgb((h/360, 0.55, v)),
    mcl.hsv_to_rgb((h/360, 1, v))
]
cmap = LinearSegmentedColormap.from_list('my_cmap',colors,gamma=2)

measures = ['lift', 'leverage', 'conviction',
            'all-confidence', 'collective strength', 'cosine similarity']

fig = plt.figure(figsize=(15,10))
fig.set_facecolor('white')
for i, measure in enumerate(measures):
    ax = fig.add_subplot(320+i+1)
    if measure != 'all-confidence':
        scatter = ax.scatter(support,confidence,c=association_rules_df[measure],cmap=cmap)
    else:
        scatter = ax.scatter(support,confidence,c=association_rules_df['all-confidence'].map(lambda x: [v for k,v in x.items()][0]),cmap=cmap)
    ax.set_xlabel('support')
    ax.set_ylabel('confidence')
    ax.set_title(measure)

    fig.colorbar(scatter,ax=ax)
fig.subplots_adjust(wspace=0.2, hspace=0.5)
plt.show()
```


![f4d068a13fbc7cd159ae4c16eab1615c431dbf38](https://github.com/sungbinlee/sungbinlee.github.io/assets/52542229/c869dc48-1aa5-4b56-8378-4c9d8619d018)


