# Exploratory-Data-Analysis-on-Netflix-Dataset
In this project i did EDA on Netflix Dataset using Kaggale:

#https://www.kaggle.com/code/sanketshintre/netflix-dataset-eda

# Table of Content
Data Information
Handling Missing Data
Creating New Columns
Visualization
Conclusion


import os
for dirname, _, filenames in os.walk('/kaggle/input'):
    for filename in filenames:
        print(os.path.join(dirname, filename))

from pandas_profiling import ProfileReport
#from dataprep.eda import create_report

from plotly.subplots import make_subplots
import plotly.graph_objects as go


from plotly.offline import plot, iplot, init_notebook_mode
import plotly.express as px
init_notebook_mode(connected=True)

# Reading the input file
df = pd.read_csv('../input/netflix-shows/netflix_titles.csv')
df.head(10)

# To see the high level data details
df.info()

**Observations:**
1. The above code shows that there are some null values in the data

2. Shows the total rows, name and number of columns and their datatypes

def missing_pct(df):
    # Calculate missing value and their percentage for each column
    missing_count_percent = df.isnull().sum() * 100 / df.shape[0]
    df_missing_count_percent = pd.DataFrame(missing_count_percent).round(2)
    df_missing_count_percent = df_missing_count_percent.reset_index().rename(
                    columns={
                            'index':'Column',
                            0:'Missing_Percentage (%)'
                    }
                   )
    df_missing_value = df.isnull().sum()
    df_missing_value = df_missing_value.reset_index().rename(
                    columns={
                            'index':'Column',
                            0:'Missing_value_count'
                    }
                )
    # Sort the data frame
    #df_missing = df_missing.sort_values('Missing_Percentage (%)', ascending=False)
    Final = df_missing_value.merge(df_missing_count_percent, how = 'inner', left_on = 'Column', right_on = 'Column')
    Final = Final.sort_values(by = 'Missing_Percentage (%)',ascending = False)
    return Final

missing_pct(df)

The function missing_pct takes a data frame as an input and returns a data frame, where each row corresponds to a column in the original dataframe and contains column's name, number of missing values in that column as well as percentage of the missing values.

This is a standard template that I use for every dataset that I want to analyze


# To see the high level data details
df.info()

# Dataset information

# Approach 3
ProfileReport(df)

**Handling the missing data and deleting duplicates**
It is important to handle missing data because any statistical results based on a dataset with non-random missing values could be biased. So you really want to see if these are random or non-random missing values.

Drop the columns which has high number of missing values.
We can impute(filling the missing values using the available information such as mean, median) but we should carefully see the pattern of the column before doing imputation.
For example - You want to fill the height of a person who male. Simpley adding 0 in the missing column would not make sense. So we can take the averega of male height and use that value inplace of missing values.
Rating - manually filling the data usin data from Netflix website
Country - replacing blank countries with the most common country
Cast - replacing null value with "Data not available"
Director - replacing null value with "Data not available"

# Rating data is mentioned incorrectly for few titles in the input file. Hence correcting it by checking the Maturity rating online

df['rating'] = df['rating'].replace({'74 min': 'TV-MA', '84 min': 'TV-MA', '66 min': 'TV-MA'})
df['rating'] = df['rating'].replace({'TV-Y7-FV': 'TV-Y7'})

df['rating'].unique()

# Renaming vaules for Rating for better understanding
# Source : https://help.netflix.com/en/node/2064
df['rating'] = df['rating'].replace({
                'PG-13': 'Teens - Age above 12',
                'TV-MA': 'Adults',
                'PG': 'Kids - with parental guidence',
                'TV-14': 'Teens - Age above 14',
                'TV-PG': 'Kids - with parental guidence',
                'TV-Y': 'Kids',
                'TV-Y7': 'Kids - Age above 7',
                'R': 'Adults',
                'TV-G': 'Kids',
                 'G': 'Kids',
                'NC-17': 'Adults',
                'NR': 'NR',
                'UR' : 'UR'
    
})

df['rating'].unique()


df['country'] = df['country'].fillna(df['country'].mode()[0])

df['cast'].replace(np.nan, 'No Data',inplace  = True)
df['director'].replace(np.nan, 'No Data',inplace  = True)
df.dropna(inplace=True)

# Drop Duplicates
df.drop_duplicates(inplace= True)

# splitting the genres in different rows to use it in the viz later

#df_genre = df[df['title'].isin(['Blood & Water', 'Dick Johnson Is Dead', 'Ganglands' ])]
df_genre = df[['show_id', 'title','type', 'listed_in' ]]
df_genre = (df_genre.drop('listed_in', axis=1)
             .join
             (
             df_genre.listed_in
             .str
             .split(', ',expand=True)
             .stack()
             .reset_index(drop=True, level=1)
             .rename('listed_in')           
             ))
          
Creating new columns

# Creating new columns
df['month'] = pd.DatetimeIndex(df['date_added']).month

# Total Shows and movies

df_count = df['show_id'].count().sum()
print(df_count)
# Split of showes and TV
df_type = df.groupby('type')['show_id'].count().reset_index()
df_type = df_type.rename(columns = {"show_id":"count_showids"})

**Visualization**
import plotly.graph_objects as go
fig = go.Figure()
fig.add_trace(go.Indicator(
    value = df_count))

fig = fig.update_layout(
        template = {'data' : {'indicator': [{
        'title': {'text': "Total content on Netflix"},}]
        }})
fig = fig.update_layout(
    #autosize=False,
    #width=500,
    height=100,
    margin=dict(l=50,r=50,b=0,t=1),)

# fig2 = px.pie(df_type, values='count_showids', names='type', color_discrete_sequence=px.colors.sequential.RdBu,
#        title='What type of titles are uploaded more on Netflix' , width=500, height=450)

fig.show()
#fig2.show()

fig = make_subplots(rows=1, cols=2, specs=[[{'type':'bar'}, {'type':'pie'}]])
fig.add_trace(
    
    go.Bar(x= df_type['count_showids'], y= df_type['type'], orientation = 'h', marker=dict(color=["Maroon", "Grey"]), showlegend=False, 
           text = df_type['count_showids'], textposition='auto'),
    row=1, col=1)

fig.add_trace(
    
     go.Pie(labels=df_type['type'], values=df_type['count_showids'], marker_colors= ["Maroon", "Grey"]),
    row=1, col=2)

fig.update_layout(
    title_text="'What type of content is more uploaded more on Netflix?")
fig.show()

<img width="960" alt="2023-03-29 (10)" src="https://user-images.githubusercontent.com/123626990/228470383-433eb2d2-e0f9-410c-bcd1-8050a44ce1c0.png">
<img width="960" alt="2023-03-29 (11)" src="https://user-images.githubusercontent.com/123626990/228470401-8782f9ac-7b3e-4968-97ba-384a276d46be.png">

We observe that there are more movies than TV shows on Netflix

# splitting the countries in different rows 
#df_genre = df[df['title'].isin(['Blood & Water', 'Dick Johnson Is Dead', 'Ganglands' ])]
df_country = df[['show_id', 'title','type', 'country' ]]
df_country = (df_country.drop('country', axis=1)
             .join
             (
             df_country.country
             .str
             .split(', ',expand=True)
             .stack()
             .reset_index(drop=True, level=1)
             .rename('country')           
             ))
             
             df_country_viz_total = df_country[["title", "country"]]
df_country_viz_total = df_country_viz_total.groupby(['country'])["title"].count().reset_index().sort_values('title', ascending= False).head(10)
df_country_viz_total = df_country_viz_total.rename(columns = {"title": "movies_count",})

 
fig1 = px.bar(df_country_viz_total, x='country', y='movies_count', color_discrete_sequence=px.colors.sequential.RdBu,
       title='Top 10 countries with Netflix Content ')
df_country_viz = df_country[["title", "country"]]
df_country_viz = df_country_viz.groupby(['country'])["title"].count().reset_index().sort_values('title', ascending= False).head(10)

df_country_viz1 = df_country[["title", "type", "country"]]
df_country_viz1 = df_country_viz1.groupby(['country', 'type'])["title"].count().reset_index().sort_values('title', ascending= False)
df_country_viz1 = df_country_viz1.rename(columns = {"title": "movies_count",})
final1 = df_country_viz.merge(df_country_viz1, how = 'left', left_on = 'country', right_on = 'country')
final1['percentage'] = (final1['movies_count']/final1['title'])*100
final1['percentage'] = final1['percentage'].round(1)
final1['percent_string'] = final1['percentage'].astype(str)+ '%'


fig2 = px.bar(final1, x='country', y='percentage', color = 'type',
       title='Top 10 countries with Movie/TV show split ')
       
       fig = go.Figure()
fig.add_trace(
    
go.Bar(x= df_country_viz_total['country'], y= df_country_viz_total['movies_count'], marker_color = 'Maroon',
           text = df_country_viz_total['movies_count'], textposition='auto'))

fig.update_layout(title_text = "Top 10 countries with Netflix Content"
                  , yaxis=dict(title='Movies/TV Shows Count'))
fig.show()

final_movie = final1.query("type == 'Movie'")
final_show = final1.query("type == 'TV Show'")

fig = go.Figure()
fig.add_trace(go.Bar(
    x=  final_movie['country'],
    y= final_movie['percentage'],
    showlegend=True,
    text = final_movie['percent_string'], 
    textposition='auto',
    name='Movie',
    marker_color='Maroon'    
    
))
fig.add_trace(go.Bar(
    x= final_show['country'],
    y= final_show['percentage'],
    showlegend=True,
    text = final_show['percent_string'], 
    textposition='auto',
    name='TV Show',
    marker_color='Grey' 
))



# Here we modify the tickangle of the xaxis, resulting in rotated labels.
fig.update_layout(barmode='stack', title_text = 'Top 10 countries with Movie/TV show split '
                                  , yaxis=dict(title='% Movies/TV Shows Count'))
fig.show()
                  
<img width="960" alt="2023-03-29 (12)" src="https://user-images.githubusercontent.com/123626990/228471290-0bd50fa6-4857-43e6-9b5e-922b065f2ee8.png">
          
 <img width="960" alt="2023-03-29 (13)" src="https://user-images.githubusercontent.com/123626990/228471450-5057dde8-aa86-4b43-8a11-358e610d3d0e.png">
 
 # df_country_viz = df_country[["title", "type", "country"]]
# df_country_viz = df_country_viz.groupby(['country', 'type'])["title"].count().reset_index().sort_values('title', ascending= False)
# df_country_viz = df_country_viz.rename(columns = {"title": "movies_count",})

# df_country_movie = df_country_viz.query("type == 'Movie'").head(10) 
# fig1 = px.bar(df_country_movie, x='country', y='movies_count',color_discrete_sequence=['Maroon'],
#        title='Top 10 countries with the most Netflix movies')
# df_country_movie = df_country_viz.query("type == 'TV Show'").head(10)
# fig2 = px.bar(df_country_movie, x='country', y='movies_count', color_discrete_sequence=['gray'],
#        title='Top 10 countries with the most Netflix TV Shows')

# fig1.show()
# fig2.show()

**United States** is the top leaader in both movie and TV shows. India followed US in the overall content and it seems that it has the most number of movies with very less percentage of TV shows comapred to UK and Japan.

df_2 = df.query("type == 'Movie'")
df_2 = df_2[["title", "rating"]]
df_2 = df_2.groupby(['rating'])["title"].count().reset_index().sort_values('title', ascending = False)
df_2 = df_2.rename(columns = {"title": "movies_count"})
px.bar(df_2, x='ratin<img width="960" alt="2023-03-29 (14)" src="https://user-images.githubusercontent.com/123626990/228472113-f6ecfaed-6d09-4613-860e-02cb8e1556db.png">
g', y='movies_count', color_discrete_sequence=px.colors.sequential.RdBu,
       title='For which category the maximum content(Movies) are uploaded? ')

![Uploading 2023-03-29 (14).png…]()

df_3 = df.query("type == 'TV Show'")
df_3 = df_3[["title", "rating"]]
df_3 = df_3.groupby('rating')["title"].count().reset_index().sort_values('title', ascending = False)
df_3 = df_3.rename(columns = {"title": "movies_count"})
px.bar(df_3, x='rating', y='movies_count', color_discrete_sequence=['grey'],
       title='For which category the maximum content(TV Shows) are uploaded?')
  
<img width="960" alt="2023-03-29 (15)" src="https://user-images.githubusercontent.com/123626990/228472522-dcc7f907-635b-491c-a737-8c8093a03d8d.png">

It seems the most content(TV shows) on Netflix caters to Adults and then teens.

df_5 = df.query("release_year >= 2007")
df_5 = df_5.groupby("release_year")["show_id"].count().reset_index()

fig = px.area(df_5, x='release_year', y='show_id', color_discrete_sequence=px.colors.sequential.RdBu,
      title='Overall content release Trend')
fig.show()
<img width="960" alt="2023-03-29 (17)" src="https://user-images.githubusercontent.com/123626990/228473130-9a1c777a-b71c-4248-a215-05887ec90653.png">

df_4 = df.query("release_year >= 2007")
df_4 = df_4.groupby(["type","release_year"])["show_id"].count().reset_index()
df_4_movie = df_4.query("type == 'Movie'")
df_4_show = df_4.query("type == 'TV Show'")

fig = go.Figure()
fig.add_trace(go.Scatter(
    x=  df_4_movie['release_year'],
    y= df_4_movie['show_id'],
    showlegend=True,
    text = df_4_movie['show_id'], 
      name='Movie',
    marker_color='Maroon'    
    
))
fig.add_trace(go.Scatter(
    x=  df_4_show['release_year'],
    y= df_4_show['show_id'],
    showlegend=True,
    text = df_4_show['show_id'], 
 
    name='TV Show',
    marker_color='Grey' 
))

fig.update_traces( mode='lines+markers')
fig.update_layout(title_text = 'Movies/TV Show release yearly Trend' )
fig.show()
<img width="960" alt="2023-03-29 (18)" src="https://user-images.githubusercontent.com/123626990/228473762-add69021-5583-44f3-ab87-f5d6208c10e8.png">

Conclusion

We did exploratory data analysis on Netflix Movie Data. We found a lot of insights from the data.

We observe that there are more movies than TV shows on Netflix.
Unites States tops the chart followed by India, United Kingdom, and Canada.
Interestingly, the content available in India is heavily skewed towards movies confirming the intuition about big influence of bollywood movie productions.
South Korea has the highest percentage of TV shows
United States is the top leaader in both movie and TV shows. India followed US in the overall content and it seems that it has the most number of movies with very less percentage of TV shows comapred to UK and Japan.
It seems the most content(Movies) on Netflix caters to Adults and then teens.
In 2007, Netflix introduced streaming media and video on demand. We see a slow in the beginning but then it picked up in 2014-2015 and there is a rapid increase till 2018.
By 2018, the content on netlix was 13 times of 2007 year's content. But it has declined since 2019 since the beginning of covid. The other factor could be - In 2019, Disney plus was also launched. Films and television series produced by The Walt Disney Studios and Walt Disney Television, such as Marvel movies moved to Disney plus.
It seems like Netflix focused on movies, and the movie count increases significantly till 2018. There's been a decline in the movies count but a steady growth in the TV shows since 2018.
