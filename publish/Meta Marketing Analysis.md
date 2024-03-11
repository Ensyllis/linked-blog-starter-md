# Context:
As [Professor Wilson's](https://www.biola.edu/directory/people/jason-wilson) Quantitative Consulting Center began to grow with an abundance of students the professor was more inclined to open up the doors to more clients. One of the strategies I proposed to gain more attention to the [Quantitative Consulting Center](https://www.biola.edu/quantitative-consulting-center) was to propose a digital marketing strategy. We sought to deploy a small scale $20 marketing strategy using videos made by the marketing department of Biola and observe the User Interaction to our ads as well as our target demographic.

One of the metrics we wanted to see was who interacted with our Ads the most


# Code:

```python
import pandas as pd

df = pd.read_csv("The-everything-report (1).csv")

df.head()
```
![[Pasted image 20240310183325.png]]
```python
#Commentary:

#Data Science is the art of asking the right questions, then finding the answers to such questions using data.

#In this case we are trying to find which age ground as well as gender is most interested in our services.

#The easiest way to do this is to examine the amount of time that they watch

import numpy as np
import matplotlib.pyplot as plt


def replace_nan_with_zero(df):
    df.fillna(0, inplace=True)
    return df

replace_nan_with_zero(df)
```
![[Pasted image 20240310183342.png]]
```python
for col in df.columns:

    print(col)
```
![[Pasted image 20240310183356.png]]
```python
import pandas as pd

selected_columns = ["Campaign name", "Video plays at 25%", "Video plays at 50%", "Video plays at 75%", "Video plays at 95%", "Video plays at 100%", "Link clicks"]

new_df = df[selected_columns].copy()

df.plot(x="Campaign Name", y=["Video plays at 25%", "Video plays at 50%", "Video plays at 75%", "Video plays at 95%", "Video plays at 100%", "Link clicks"], kind="bar")
```
![[Pasted image 20240310183436.png]]
```python
# combine age and gender for new campaign name

df['New Campaign Name'] = df['Age'].astype(str) + ' ' + df['Gender']

# drop that campaign

df.drop(columns=['Campaign name'], inplace=True)

# rename it to new campaign name

df.rename(columns={'New Campaign Name': 'Campaign Name'}, inplace=True)
```
From here we can see that the top categories seem to be

1. 24-34 Males

2. 65+ Males

3. 45 - 54 Females

4. 35 - 44 Female

5. 55 - 64 Female

  

To see if this truly works we can do a correlation analysis between Link clicks and Video Plays
```python
# Assuming your DataFrame is called 'df' and you want to sort the 'Column_Name' column in descending order

Links_clicks_sorted_dataframe = df.sort_values(by='Link clicks', ascending=False)

Links_clicks_sorted_dataframe.head()
```
![[Pasted image 20240310183518.png]]
```python
# Something we're encountering is that we need to weigh the data by the amount of time that the video was watched

#TO do this we want to add a new column called Adjuted_watch_time which takes the Video plays column and multiplies it by Video average play time

df['Adjusted_watch_time'] = df['Video plays'] * df['Video average play time']

Vide_Plays = df.sort_values(by='Adjusted_watch_time', ascending=False)

Vide_Plays.head()
```
![[Pasted image 20240310183531.png]]
```python
df.plot(x="New Campaign Name", y=["Page engagement"], kind="bar")
```
![[Pasted image 20240310183556.png]]
```python
Page_engagement_sorted_df = df.sort_values(by='Page engagement', ascending=False)

print(Page_engagement_sorted_df)
```
![[Pasted image 20240310183614.png]]
We can see that the top 5 when we sort by descending are the following.

 Age   Gender  Amount spent (USD)  Link clicks  \

4     65+     male            1.950925          1.0  

1   25-34     male            2.281081          2.0  

7   55-64     male            1.670792          1.0  

2   35-44     male            2.261072          2.0  

3     65+   female            2.241062          1.0

  
  

Just like before
```python
#New Idea we make a three new columns, the ranking for the column ["Page engagement"]", "Link clicks", "Adjusted_watch_time"]

#We then add the three columns together to create a new column called "Total_Rank"

#We then sort the data by the "Total_Rank" column

df['Page engagement rank'] = df['Page engagement'].rank(ascending=False)

df['Link clicks rank'] = df['Link clicks'].rank(ascending=False)

df['Adjusted_watch_time rank'] = df['Adjusted_watch_time'].rank(ascending=False)

df['Total_Rank'] = (df['Page engagement rank'] * df['Link clicks rank'] * df['Adjusted_watch_time rank'])/3

Total_Rank_sorted_df = df.sort_values(by='Total_Rank', ascending=True)

Total_Rank_sorted_df
```
![[Pasted image 20240310183638.png]]
