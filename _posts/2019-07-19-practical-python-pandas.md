---
full_title: "Entry 0x05 - Pandas Data Analysis Tutorial: Ransomware Tracker"
date: "2019-07-19"
layout: post
coverImage: "screen-shot-2019-06-19-at-8.24.04-pm.png"
tags:
- Beginner
- Data-Analysis
- Data-Cleaning
- Data-Visualization
- Python 
- Pandas
- Ransomware
- Ransomware-Tracker
- Security
- Tutorial
---

In this tutorial, we will learn how to use [Pandas](https://pandas.pydata.org/) - a _must-have_ Python module for Data Analysis and Data Visualization with a real-world example from the Cyber Security domain.

> Note: Ransomware Tracker is no longer operational since **08 December 2019**. It is still recommended that readers leverage the concepts and Jupyter Notebook available in this tutorial.

### Introduction

[Ransomware Tracker](https://ransomwaretracker.abuse.ch/) by [abuse.ch](https://www.abuse.ch/) is a website which tracks and monitors hosts and URLs associated with known Ransomware.

The website maintains a _tracker_ which is frequently updated with threat intelligence associated with known Ransomware families. The screenshot below shows an interactive table on the Ransomware Tracker website populated with Ransomware threat intelligence.

![Screen Shot 2019-06-19 at 8.24.04 PM.png](/assets/images/screen-shot-2019-06-19-at-8.24.04-pm.png)

The most interesting feature of Ransomware Tracker is the availability of a [feed](https://ransomwaretracker.abuse.ch/feeds/csv/) in the CSV (Comma Separated Values) format which allows us to easily capture and utilize this intelligence.

The screenshot below shows the Ransomware Tracker data in its raw CSV format accessible via the URL - `https://ransomwaretracker.abuse.ch/feeds/csv/`

![Screen Shot 2019-06-19 at 8.30.22 PM.png](/assets/images/screen-shot-2019-06-19-at-8.30.22-pm.png)

Our objective is to read, parse, and generate insights from this Ransomware Tracker data using Python with Pandas.

### Getting Started

For the purpose of this tutorial, we will use a [Jupyter Notebook](https://jupyter.org/) to write Python code and produce output. [Here](https://www.dataquest.io/blog/jupyter-notebook-tutorial/) is a complete, easy to understand introduction to Jupyter Notebooks and how to get started.

The first step is to fetch the data. 

As mentioned earlier, our data resides online as a CSV document. Pandas provides us with the [read_csv](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_csv.html) function to read CSV data and store it into a [DataFrame](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html) structure.

```
import pandas as pd
url = "https://ransomwaretracker.abuse.ch/feeds/csv/"
df = pd.read_csv(url, skiprows=8, encoding="latin-1")
```

We start by importing the Pandas module and reference it as `pd` instead of `pandas`. This is a personal preference but is commonly seen in tutorials online.

Next, we initialize a variable `url` with the Ransomware Tracker CSV URL. This variable has a data type of `str`.

Finally, we make a function call to `pd.read_csv` with arguments as follows

1. `url` - location where our CSV feed resides (required)
2. `skiprows` - number of rows to skip from the top of the CSV document (in our case the first 8 lines are comments)
3. ​`encoding` - text encoding to be used

Now, we have `df` (our DataFrame) with the data loaded from the URL. Let us validate the data and its structure.

```
df.shape
# (13866, 10)
df.head()
```

![Screen Shot 2019-06-21 at 4.34.48 PM.png](/assets/images/screen-shot-2019-06-21-at-4.34.48-pm.png)

`df.head()` prints the first 5 rows of the DataFrame by default. You can change this by specifying the required number of rows as an argument. Hence, `df.head(n)` will print the first `n` rows of the DataFrame.

Next, we validate the bottom values of the DataFrame. This is good practice for large datasets such as Ransomware Tracker with over 13,000 rows of data.

```
df.tail()
```

![Screen Shot 2019-06-22 at 12.00.58 PM.png](/assets/images/screen-shot-2019-06-22-at-12.00.58-pm.png)

In our output, we can confirm the following facts:

1. The DataFrame recognized the header names
2. All fields are parsed correctly and unavailable fields are replaced with `NaN` value
3. The last row is a comment and needs to be removed

To remove the last row of the DataFrame, we can use a simple one-liner from Pandas:

```
df.drop(df.tail(1).index, inplace=True)
df.tail()
```

![Screen Shot 2019-06-22 at 12.09.31 PM.png](/assets/images/screen-shot-2019-06-22-at-12.09.31-pm.png)

Great!

Now, the `df.shape` command should return `(13865, 10)` since we removed the last row of the DataFrame.

### Data Transformation

The next step involves manipulating and transforming the data in our DataFrame.

Let's start with fixing the header names (also known as _column_ _names_) of the DataFrame. To do this, we start by retrieving the list of existing header names.
```
list(df.columns)
'''
['# Firstseen (UTC)',
 'Threat',
 'Malware',
 'Host',
 'URL',
 'Status',
 'Registrar',
 'IP address(es)',
 'ASN(s)',
 'Country']
'''
```

I decided to make the DataFrame easier to read and comprehend with the following header name changes.

- **Old:** `# Firstseen (UTC)`
- **New:** `Firstseen`
- **Old:** `IP address(es)`
- **New:** `IPs`
- **Old:** `ASN(s)`
- **New:** `ASNs`

To accomplish this, we can use the `df.rename` function as follows.

```
columns = {'# Firstseen (UTC)': 'Firstseen', 'IP address(es)': 'IPs', 'ASN(s)':'ASNs'}
df = df.rename(columns=columns)
df.head()
```

![Screen Shot 2019-06-21 at 5.50.48 PM](/assets/images/screen-shot-2019-06-21-at-5.50.48-pm.png)

The `Firstseen` column in our DataFrame can provide us with a treasure of knowledge.

However, the values available consist of a date and time. We simply want the date. This requires a transformation of the values in the `Firstseen` column in our DataFrame.

Before we apply the solution in the context of the DataFrame, let us shift perspective. Consider a value from the `Firsteen` column. For example - `2018-08-12 00:46:13`

```
s_dt = '2018-08-12 00:46:13'
type(s_dt)
# str
```

The goal is to transform this value into our desired format. I choose to change the format to `12-08-2018`. How can we do this?

Python provides us with a useful module called `datetime` for this exact purpose. We can leverage the `datetime.strptime` function to convert `s_dt` (a `str` object) to a `datetime` object as follows.

```
import datetime
o_dt = datetime.datetime.strptime(s_dt,'%Y-%m-%d %H:%M:%S')
type(o_dt)
# datetime.datetime
```

Now, we construct our desired format `DD-MM-YYYY` using the `datetime.strftime` function and `o_dt` (the `datetime` object) as follows.

```
s1_dt = o_dt.strftime("%d-%m-%Y")
s1_dt
# '12-08-2018'
type(s1_dt)
# str
```

Easy! We successfully transformed one string but what about an entire DataFrame column?

To achieve this, we can use the `df.apply` function which applies a function along an axis of the DataFrame. For the function aspect, I choose to construct a [lambda function](https://www.w3schools.com/python/python_lambda.asp) (popularly known as _anonymous functions_).

```
df['Firstseen'] = df['Firstseen'].apply(lambda x: datetime.datetime.strptime(x,'%Y-%m-%d %H:%M:%S').strftime("%d-%m-%Y"))
df.head()
```

![Screen Shot 2019-06-22 at 12.58.25 PM.png](/assets/images/screen-shot-2019-06-22-at-12.58.25-pm.png)

Voila! Let us dissect the above command...

```
df['Firstseen'].apply(lambda x: datetime.datetime.strptime(x,'%Y-%m-%d %H:%M:%S').strftime("%d-%m-%Y"))
```
Here:
1. `df['Firsteen']` refers to the column `Firstseen` in the DataFrame `df`
2. `lambda x: datetime.datetime.strptime(x,'%Y-%m-%d %H:%M:%S').strftime("%d-%m-%Y")` is our _lambda function_
    - The `x` in `lambda x` references _each_ element in the `Firsteen` column
    - `datetime.datetime.strptime(x,'%Y-%m-%d %H:%M:%S')` converts each `x` (`str` object) to a `datetime` object using the provided format
    - `strftime("%d-%m-%Y")` then converts each `datetime` object back to `str` in the provided format (`DD-MM-YYYY`)
3. We apply this _lambda function_ across the entire `Firstseen` column using `df.apply` function

The biggest takeaway is to always achieve the desired transformation at the element-level before attempting to manipulate the DataFrame.

### Querying

The next step is to query the DataFrame and generate valuable insights. In this step, I aim to use Pandas to perform operations on the DataFrame, extract output, and visualize the results.

#### Query 1

I decided to define query 1 as follows: _Number of entries per threat_ 

In this query, we want to categorize our dataset based on the `Threat` field. This basically involves a group by operation followed by aggregation and sorting. I write the query as follows.

```
df.groupby('Threat').size().sort_values(ascending=False)
'''
Threat
Distribution Site    11297
Payment Site          1660
C2                     908
dtype: int64
'''
```

Interesting! The output indicates the existence of 3 threats - `Distribution Site`, `Payment Site` and `C2` (Command and Control Site). As seen in the Python query, we utilize a variety of Pandas functions to manipulate the data.

Now, how about a visualization?

Visualization of data in Python can be achieved with a variety of libraries such as Matplotlib, Seaborn, and ggplot. Read more [here](https://www.fusioncharts.com/blog/best-python-data-visualization-libraries/).

Pandas comes with an in-built `df.plot` function exposing useful plotting abilities. In fact, `df.plot` basically refers to Matplotlib in the backend for visualization.

Let's create a simple horizontal bar graph to illustrate the different categories of threats and their counts. The query is as follows.

```
df.groupby(['Threat']).size().sort_values(ascending=False).plot(kind='barh')
```

![screen-shot-2019-07-12-at-5.26.37-pm.png](/assets/images/screen-shot-2019-07-12-at-5.26.37-pm-e1563370050540.png)

The `df.plot` function is an effective tool to generate useful graphs. In our simple example above, we specified the argument `kind=barh` to indicate a **horizontal bar graph.**

#### Query 2

For the next query, I decided to play with the `Firstseen` field of the DataFrame. A valuable tip is to always attempt trend analysis if the dataset contains date/time fields.

I decided to define query 2 as follows: _Yearly_ _trend in malware_

This query is slightly more complex as compared to the previous one. The first transformation involves creating a new DataFrame column called `Firstseen_year` in which the "year" from the `Firstseen` element is captured and stored. We accomplish this by using a custom defined lambda function.

```
df['Firstseen_year'] = df['Firstseen'].apply(lambda x: datetime.datetime.strptime(x,'%d-%m-%Y').strftime("%Y"))
```

Before we continue, let us understand the `dtypes` or data types of elements within our DataFrame using the following command.

```
df.dtypes
'''
Firstseen         object
Threat            object
Malware           object
Host              object
URL               object
Status            object
Registrar         object
IPs               object
ASNs              object
Country           object
Firstseen_year    object
dtype: object
'''
```

As seen above, **all** the elements are of `object` data type which is equivalent to `str` data type in Python. When working with date/time elements, it is **strongly recommended** to ensure a suitable data type. This especially matters for operations such as **sorting**.

One mechanism to change the `dtype` of a column is to use the `df.astype` function as follows.

```
df['Firstseen_year'] = df['Firstseen_year'].astype('datetime64[ns]')
df.dtypes
'''
Firstseen                 object
Threat                    object
Malware                   object
Host                      object
URL                       object
Status                    object
Registrar                 object
IPs                       object
ASNs                      object
Country                   object
Firstseen_year    datetime64[ns]
dtype: object
'''
```

Great! Our DataFrame column `Firstseen_year` now has data type as `datetime64[ns]`.

Although this is the correct way to work with date/time elements, it is important to note that _side-effects_ are plenty. Let us take a look at the contents of the DataFrame `df`.

```
df[['Firstseen','Firstseen_year']].head()
```

![Screen Shot 2019-07-14 at 11.30.11 AM.png](/assets/images/screen-shot-2019-07-14-at-11.30.11-am.png)

As we can see, once we extract `2018` from `12-08-2018` and convert it to the `datetime64[ns]` data type, we end up with `2018-01-01`.

While it makes sense... it does not meet our desired format i.e., **year** only. This means that we absolutely require `2018` instead of `2018-01-01` and the like. But how?

Simple!

Since `df['Firstseen_year']` is of the data type `datetime64[ns]`, we can extract the "year" part of the date/time object as follows.

```
df['Firstseen_year'] = df['Firstseen_year'].dt.year
df[['Firstseen','Firstseen_year']].head()
```

![Screen Shot 2019-07-14 at 10.21.19 PM.png](/assets/images/screen-shot-2019-07-14-at-10.21.19-pm.png)

Wait, what about the data types?

```
df.dtypes
'''
Firstseen         object
Threat            object
Malware           object
Host              object
URL               object
Status            object
Registrar         object
IPs               object
ASNs              object
Country           object
Firstseen_year     int64
dtype: object
'''
```

As we can see, `Firstseen_year` column has `int64` values. Now, operations such as **sorting** can be achieved accurately. Back to the query!

```
ax = df[['Firstseen_year','Malware']].groupby('Firstseen_year').count().sort_values(by='Firstseen_year', ascending=False).plot(kind='area', figsize=(20,5))
ax.set_xlabel("Firstseen Year")
ax.set_ylabel("Number of Malware")
ax.set_title("Yearly Malware Trend - Ransomware Tracker")
```

![Screen Shot 2019-07-14 at 10.44.51 PM.png](/assets/images/screen-shot-2019-07-14-at-10.44.51-pm.png)

The above query includes many useful features of the `df.plot` function. This is an example of an **area** graph. The `figsize=(20,5)` argument indicates the size of the graph produced as output.

No graph is complete without appropriate `x` and `y` labels. The `set_xlabel` and `set_ylabel` functions play a significant role in helping us define these labels.

#### Query 3

For the next query, I decided to focus on a slightly more complex query. This time, I decided to utilize two fields - `Firstseen_year` and `Threat`.

Query 3 can be defined as follows: _Number of malware per threat per year_ 

 To achieve this query, we simply require two _group-by_ instructions followed by aggregation.

```
df.groupby(['Firstseen_year','Threat']).size()
'''
Firstseen_year  Threat           
2015            C2                      37
2016            C2                     709
                Distribution Site    10441
                Payment Site          1346
2017            C2                     140
                Distribution Site      843
                Payment Site           314
2018            C2                      22
                Distribution Site       13
dtype: int64
'''
```

The insights generated here is extremely valuable. Finding _correlations_ between different columns and fields is typically achieved using the `df.groupby` function. Visualizing the results would be the icing on the cake!

Let's visualize the data as follows.

```
ax = df.groupby(['Firstseen_year','Threat']).size().unstack().plot(kind='area',stacked=True,figsize=(20,5))
ax.set_xlabel("Firstseen Year")
ax.set_ylabel("Number of Malware")
ax.set_title("Malware per Threat per Year - Ransomware Tracker")
```

![Screen Shot 2019-07-17 at 5.26.57 PM.png](/assets/images/screen-shot-2019-07-17-at-5.26.57-pm.png)

The above query showcases an area plot described by `kind='area'` as argument to the function `df.plot`. The _stacking_ is achieved with the argument `stacked=True` and makes the graph easier to visualize.

Again, we utilize the `set_xlabel` and `set_ylabel` functions to correctly label the graph. This is always recommended!

### Conclusion

In this tutorial, we explored [Pandas](https://pandas.pydata.org/) - the _defacto_ Python module in a Data Analyst's toolkit.

Using the practical example of [Ransomware Tracker data](https://ransomwaretracker.abuse.ch/), we went through the steps involved in ingesting, cleaning, parsing, querying, and visualizing data to generate powerful insights.

> You can view and download a Jupyter Notebook with everything highlighted in this tutorial from [here](https://nbviewer.jupyter.org/github/arjuntherajeev/jupyter_notebooks/blob/master/Ransomware_Tracker_Tutorial.ipynb).

Key takeaways include:

1. Always be curious about data. In Cyber Security, we are surrounded by tons of valuable data - logs, threat intelligence, etc. You never know what you will find.

2. Leverage modern technologies such as Python, Jupyter Notebooks, GitHub, etc. to write code, visualize graphs, and share with others.

3. Within Python, explore numerous visualization libraries and modules such as Seaborn, Plotly, Bokeh, Matplotlib, etc. Depending on the scenario, one of them could provide much more value over the other.

4. Try to correlate with various datasets. For more advanced analytics, play with multiple datasets. In our example, we used only one dataset - Ransomware Tracker feed. In the real-world, you might face multiple datasets. As challenging as it sounds, the reward (insights generated) are usually worth it.

I hope you enjoyed reading this. Please leave a comment or email me with questions.
