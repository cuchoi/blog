---
title: The reason I am using Altair for most of my visualization in Python
date: 2019-04-14
description: Visualizing data in Python.
...

Sadly, in Python, we do not have a ggplot2.

Python's go to visualization library, matplotlib, is **very powerful**[^matplotlib] but has severe limitations. At times its flexibility is a blessing, but it is easy to get frustrated adding a small feature to your graph. Also, matplotlib dual [object oriented and state-based interface](https://matplotlib.org/tutorials/introductory/lifecycle.html) is confusing. I still don't completely grasp it even though I have been using matplotlib for years. Lastly, it makes only static graphs.

## Altair and the grammar of graphics

Enter Altair. Altair is a wrapper for [Vega-Lite](https://vega.github.io/vega-lite/), a JavaScript high-level visualization library. One of Vega-Lite[^refer-vega-lite] most important features is that its API is based in the grammar of graphics.

_Grammar of graphics_ may sound like an abstract feature, but it is the main difference between Altair and other Python visualization libraries. Altair matches the way we _reason_ about visualizing data.

Altair only needs three main parameters:

- **Marks**. Do you want the data represented by points? lines? bars? circles?
- **Channels**. Which variable should be mapped to the x-axis? to the y-axis? to the color of the mark? to the size of the mark?
- **Encoding**. Is the variable a date? a number? a category?

Based on these Altair will pick sensible defaults to display your data.


My favorite example of Altair’s sensibility is how it chooses colors. If you tell Altair to color a **quantitative variable** then it will use a continuous color scale (light blue, blue, dark blue). If you tell Altair to color a **categorical variable**[^nominal] then it will use a different color for each category (red, yellow, blue).

Let's see a concrete example:

I made up 6 countries and population numbers. The data looks like this:

```python
import pandas as pd
import altair as alt

data = pd.DataFrame({'population': [1, 100, 200, 300, 400, 500],
                     'country_id': [1, 2, 3, 4, 5, 6],
                     'income':     [1000, 50, 200, 300, 200, 150]})
```

| country_id | population  |  income  |
|:---------|:---------------|:--------|
| 1        |   1            | 1000    |
| 2        |   100          | 50      |
| 3        |   200          | 200     |
| 4        |   300          | 300     |
| 5        |   400          | 200     |
| 6        |   500          | 150     |

```python
"""As we mentioned before, we need to define 3 parameters:
 1. Mark: We do this by using "mark_circle".
 2. Channel: We only define an x-axis and we map it to the population.
 3. Encodings: We define both variables as quantitative by using :Q after the column name"""

categorical_chart = alt.Chart(data).mark_circle(size=200).encode(
                        x='population:Q',
                        y='income:Q',
                        color='country_id:Q')
```
![We defined the **country\_id** as a quantitative variable, but it is really a categorical one.]( {{
url_for("static", filename="img/altair_color_cont_1d.svg")}} "Altair Quantitative Color"){}

```python
# We changed color='country_id:Q' to color='country_id:N' to indicate it is a nominal variable
categorical_chart = alt.Chart(data).mark_circle(size=200).encode(
                        x='population:Q',
                        y='income:Q',
                        color='country_id:N')
```

![This makes more sense. Each country should be represented by its own distinctive color.]( {{
url_for("static", filename="img/altair_color_cat_1d.svg")}} "Altair Categorical Color"){}

We only changed the encoding of the variable _country_id_. Instead of using Q (Quantitative) we use N (Nominal). That's enough for Altair to know that it shouldn't use a continuous color scale.

Another beauty of Altair than usually you easily build-up from your existing graph. For example, let's say that now we want to add income to our graph. We simply tell Altair to map the y-axis to income:

```python
categorical_chart = alt.Chart(data).mark_circle(size=200).encode(
                        x='population:Q',
                        y='income:Q',
                        color='country_id:N')
```
![]( {{
url_for("static", filename="img/altair_color_cat_2d.svg")}} "Two dimensional Altair plot"){}

Want to add tooltips? One line is all you need:

```python
categorical_chart = alt.Chart(data).mark_circle(size=200).encode(
                        x='population:Q',
                        y='income:Q',
                        color='country_id:N',
                        tooltip=['country_id', 'population', 'income']))
```




## Is that all?

At first, I was skeptic of using a wrapper as my main visualization tool. There are many ports and wrappers for ggplot2 that haven't been widely adopted by the Python community. It is hard to create one that is feature complete and up to date. But Altair is different:


- **Altair's API is comprehensive**. Thanks to [Jake Vanderplas (JVP) great design](https://twitter.com/jakevdp/status/1006929120628916224) everything you can do in Vega-Lite you can do it in Python. JVP wrote an schema API that allows to use Python objects that map to the Vega-Lite JSON schema.

- **Intuitive and pythonic interface**. Like every library, it needs some time to get used to it. But one key advantage is that everything is set up that way for a _reason_. It just makes sense. This means that you will quickly understand its inner workings and become more and more productive.

- **Interactivity**. Vega-Lite interactivity is very powerful. You can [add tooltips with one line of code](https://altair-viz.github.io/gallery/scatter_tooltips.html). You can [link the selection of a graph with another visualization](https://altair-viz.github.io/gallery/selection_histogram.html).

- **Flexibility**. Altair marks can be thought as building blocks. For example, I made the graph below using a combination of circle marks, line marks, and text marks. The code ends up being readable and easy to modify (something that would be hard to say for a similar implementation in matplotlib).

![Combination of line, circle, and text marks. The output can easily be made interactive.]( {{
url_for("static", filename="img/dot_comparison.svg")}} "Dot comparison chart"){}



## Altair main disadvantages

* No 3d plotting. Personally, I have little use of 3d visualization in my day to day but if you need to plot multidimensional data then Altair is not for you.

* Altair is no D3.js. As many high-level visualization frameworks, Altair is not 100% customizable and at some point you will find a chart you can't do with it.

* Not great statistical support. I still rely on Seaborn for quick visualization that needs to fit a linear regressions


----

If this got you excited (or at least curious) I highly recommend [Altair's documentation](https://altair-viz.github.io/). It is a concise and clear place to start. And don't forget to check out the [example gallery](https://altair-viz.github.io/gallery/index.html).

[^matplotlib]:
    matplotlib recently came into the spotlight again for being [attributed the first black hole image](https://twitter.com/matplotlib/status/1116477991763218432).

[^refer-vega-lite]:
    In the rest of the article, I will only mention Altair, but Vega-Lite deserves as much (or more) credit.

[^nominal]:
    Vega-Lite has two types of categorical data: [nominal and ordinal](https://altair-viz.github.io/user_guide/encoding.html#encoding-data-types). Nominal is a discrete unordered category, for example, the continents which are Europe, Asia, Africa, America, and Oceania (for me America is a continent, not the USA). Ordinal is a discrete ordered quantity, for example, an Amazon review from 1 to 5.
