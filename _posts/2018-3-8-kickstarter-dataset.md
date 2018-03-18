---
layout: post
title: Kickstarter Dataset
---

[Kickstarter](https://www.kickstarter.com/){:target="_blank"} is a global crowdfunding platform focused on creativity. Users post their projects and ideas there for people to see and possibly offer to buy at an early adopter rate if they like it. 


There is a wide range of categories of projects that are available. Some of them are successful but some are not. A project is successful if they sign up enough buyers or "backers" within a certain time frame. However, if they can't reach a certain funding goal before the deadline they must refund the "backers" money and cancel the project. 

In this post we will do some basic exploration of Kickstarter projects in 2018. We want to see what project categories are available out there and find out how many of them are successful versus how many are not. 


## Let's get started

First we need to import all the required libraries.
```python
import numpy as np 
import pandas as pd 
from IPython.core.display import display, HTML
```

Next we will load the dataset from Kaggle: [https://www.kaggle.com/kemical/kickstarter-projects](https://www.kaggle.com/kemical/kickstarter-projects){:target="_blank"} that contains more than 378,000 projects in 2018.

```python
df = pd.DataFrame(pd.read_csv("../input/ks-projects-201801.csv"))
```

We will break down these projects by categories and draw bar graphs for the number of successful and unsuccessful projects. 


```python
all = df['main_category'].value_counts()
successful = df['main_category'][df['state'] == 'successful'].value_counts()
df2 = pd.DataFrame({'all': all, 'successful': successful})
df2.plot.bar()
```

![png]({{"../images/notebook_2_1.png"}})


Apparently Film & Video is the most popular category with more than 60,000 projects followed by Music with a little over 50,000 projects and Publishing at 40,000 projects. On the other hand, looks like Dance is the least popular category with around 3,000 projects followed by Journalism at around 5,000 projects and Crafts at around 8,000 projects.

I notice the ratio of successful and unsuccessful projects vary widely between categories. Looks like some of the more popular projects have lower success rate than the less popular ones. Let's find out what the success rates really are for each category.

```python
success_rate = successful / all.astype(float)
df3 = pd.DataFrame({'success_rate': success_rate})
output = df3.to_html(formatters={'success_rate': '{:,.2%}'.format})

display(HTML(output))
```

<table border="1" class="table table-bordered table-sm">
  <thead class="thead-light">
    <tr>
      <th></th>
      <th>success_rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Art</th>
      <td>40.88%</td>
    </tr>
    <tr>
      <th>Comics</th>
      <td>54.00%</td>
    </tr>
    <tr>
      <th>Crafts</th>
      <td>24.01%</td>
    </tr>
    <tr>
      <th>Dance</th>
      <td>62.05%</td>
    </tr>
    <tr>
      <th>Design</th>
      <td>35.08%</td>
    </tr>
    <tr>
      <th>Fashion</th>
      <td>24.51%</td>
    </tr>
    <tr>
      <th>Film &amp; Video</th>
      <td>37.15%</td>
    </tr>
    <tr>
      <th>Food</th>
      <td>24.73%</td>
    </tr>
    <tr>
      <th>Games</th>
      <td>35.53%</td>
    </tr>
    <tr>
      <th>Journalism</th>
      <td>21.28%</td>
    </tr>
    <tr>
      <th>Music</th>
      <td>46.61%</td>
    </tr>
    <tr>
      <th>Photography</th>
      <td>30.66%</td>
    </tr>
    <tr>
      <th>Publishing</th>
      <td>30.85%</td>
    </tr>
    <tr>
      <th>Technology</th>
      <td>19.75%</td>
    </tr>
    <tr>
      <th>Theater</th>
      <td>59.87%</td>
    </tr>
  </tbody>
</table>

Surprisingly Dance (the least popular category by number) is the one with the highest success rate!

Though looks like there is really no correlation between success rate and popularity of a category. Take Crafts for example. It's neither popular nor does it have a high success rate. 

I wonder if the success rate is at all influenced by the target that the project is aiming. 

Perhaps projects with higher success rate had lower goal and hence easier for backers to reach. Which means doesn't matter which category is a project in, they are more likely to be successful when they aim for lower money. 

We'll see if this is true or not. 

```python
is_USA = df['currency'] == 'USD'
was_successful = df['state'] == 'successful'
was_failed = df['state'] == 'failed'

df4 = df[['main_category', 'usd pledged', 'usd_pledged_real', 'usd_goal_real']]

print(df4[is_USA & was_successful].groupby('main_category').median())
print('\n')
print(df4[is_USA & was_failed].groupby('main_category').median())
```

To eliminate the noise, we will drop all non U.S. projects because they have different exchange rates which most likely will skew our numbers distribution. 

Further, for each category, we will first explore the median values of successful projects followed by the median values of the unsuccessful ones.

I am not 100% certain what is the difference between `usd pledged` and `usd_pledged_real` in the dataset. I assume the latter is the final number that the project owners get. 

With that get out of the way, we only deal with only two numbers here `usd_goal_real` and `usd_pledged_real` which represent the initial goal the project is raising versus the actual money it gets from the backers at the end.

```
                   usd pledged  usd_pledged_real  usd_goal_real
    main_category                                              
    Art               2312.500          2797.250         2000.0
    Comics            3023.065          4184.000         3000.0
    Crafts            1274.000          1963.500         1000.0
    Dance             3066.500          3545.000         3000.0
    Design           10186.000         15435.000         8000.0
    Fashion           4654.500          6586.500         5000.0
    Film & Video      5032.000          5500.010         5000.0
    Food              6902.000          9538.000         7500.0
    Games             7869.005         11196.665         5000.0
    Journalism        3102.000          3975.000         3000.0
    Music             3385.500          4075.000         3500.0
    Photography       3132.000          3784.000         3000.0
    Publishing        3079.000          4096.000         3000.0
    Technology       14777.380         24652.500        10800.0
    Theater           2960.000          3185.000         3000.0
```

Let’s first look at successful Thechnology projects with a whooping $10,800 median goal value. Interestingly, from the previous analysis, Technology has the least success rate!

Sure enough successful Dance projects have relatively low median value. This is kind of expected since it has the highest success rate among other categories.

On a different note, looks like Crafts has a pretty low median value despite having a low success rate.

Let's take a look at the unsuccessful projects next.
```    
                   usd pledged  usd_pledged_real  usd_goal_real
    main_category                                              
    Art                   55.0              80.0         4000.0
    Comics               178.0             250.0         5000.0
    Crafts                25.0              38.0         3000.0
    Dance                100.0             124.0         5000.0
    Design               339.5             477.0        13000.0
    Fashion               50.0              63.0         5500.0
    Film & Video         101.0             125.0        10000.0
    Food                  75.0             101.0        12000.0
    Games                196.0             275.0        10000.0
    Journalism            14.0              25.0         5000.0
    Music                 56.0              75.0         5000.0
    Photography           51.0              65.0         5000.0
    Publishing            50.0              65.0         5000.0
    Technology            72.0             110.0        20000.0
    Theater              141.0             164.0         5000.0
```

Technology still has the highest median goal. In fact, the unsuccessful Technology projects try to raise double than its successful counterpart! 

Another interesting finding here is that the unsuccessful projects tend to try to raise a lot of money yet only get very little out of it. Looks like the unsuccessful projects might not have been very carefully thought out.

# tl;dr

So a conclusion I can draw from exploring the Kaggle dataset here is that being in a popular category doesn't automatically make a project more successful. 

Though it is still not very clear what makes a project successful, but we found that the successful projects tend to raise less money than its unsuccessful counterpart. And this is universally true for all categories.

Finally we also found that unsuccessful projects tend to get significantly lower funding than its initial goal, which might tell that those projects are not well thought out.