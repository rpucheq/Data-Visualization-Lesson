---
title: "Data Visualization"
subtitle: "The Good, The Bad and The Ugly"
author: "Scott Stoltzman"
date: "March 10, 2017"
output: html_document
---



----  

# Data Viz Ain't Easy

In almost every business meeting you'll attend, you will see a poorly designed chart, graph, or other visual representation of data. Most people simply lack the education required to emphasize a point.  

Bad visualizations can be:  

- Difficult or impossible to interpret
- Filled with completely worthless information
- Misleading (intentionally or unintentionally)
- Redundant and boring
- Inaccurate


I have to give credit to [Junk Charts](http://junkcharts.typepad.com/j) - it inspired a lot of this post.

---- 

## Let's take a look at some examples

### Every Death in Shakespeare
**Could you imagine a worse way to show this??**  

<img src="http://i.imgur.com/4kF0q5F.png" title="plot of chunk unnamed-chunk-1" alt="plot of chunk unnamed-chunk-1" style="display: block; margin: auto;" />
    

**Is this not insane!?!?!**  

No one could ever glance at that and possibly want to read it. The only thing that would have made it worse would be if there had been a legend instead of data callouts. The author could easily have used a number of other tools to get the point across. I hate wordles but due to the fact that the article wasn't trying to show the exact proportions of type of deaths, a wordle easily illustrated the point.
  

<img src="http://i.imgur.com/dlDh5H8.png" title="plot of chunk unnamed-chunk-2" alt="plot of chunk unnamed-chunk-2" style="display: block; margin: auto;" />
This example came from this [Junk Charts Article](http://junkcharts.typepad.com/junk_charts/2016/03/which-way-to-die-the-bard-asked-onelesspie.html)

----  

To be clear, I'm not even close to being perfect when it comes to visualizations in my blog. The sizes, shapes, font colors, etc. tend to get out of control and I don't want to take the time in R to tinker with all of the details. However, when it comes to displaying things professionally, it has to be spot on! So, we'll look at the theory and not worry too much about aesthetics (save that for a time when you're getting paid).

Let's load up some libraries and get started.


```r
library(ggplot2)
library(dplyr)
library(tidyr)
library(lubridate)
library(scales)
```

## Decide on what you're trying to accomplish first.  

Ask yourself the following questions to help drive your decision:  

- Are you making a comparison?
- Are you finding a relationship?
- Are you showing a distribution?
- Are you finding a trend over time?
- Are you showing composition?
  
Once you know which question you are asking, it will keep your mind focused on the outcome and will quickly narrow down your charting options.

#### Rule of Thumb  

- **Trend: ** Column, Line  
- **Comparison: ** Area, Bar, Bullet, Column, Line, Scatter  
- **Relationship: ** Line, Scatter  
- **Distribution: ** Bar, Boxplot, Column  
- **Composition: ** Donut, Pie, Stacked Bar, Stacked Column  
  
Obviously, there are choices beyond these and you need to think through your choice wisely. 

Side Note: I ***hate*** donut and pie charts! When used properly, they're terriffic! However, I'm very used to gagging every time one appears on a projector screen due to how frequently they're used inappropriately.

For this project, I'll use some oil production data that I found while digging through http://data.world (pretty great site). The data can be found [here](http://www.eia.gov/dnav/pet/pet_crd_crpdn_adc_mbbl_m.htm)  



```r
#Custom data preparation
#GitHub (linked to at bottom of this post)
source('data_preparation.R')
data = getData()
```


```r
head(data)
```

```
##   Location Month Year ThousandBarrel       Date
## 1  Alabama   Mar 2013            883 2013-03-01
## 2  Alabama   Apr 2013            844 2013-04-01
## 3  Alabama   May 2013            878 2013-05-01
## 4  Alabama   Feb 2013            809 2013-02-01
## 5  Alabama   Mar 1982           1687 1982-03-01
## 6  Alabama   Apr 1982           1567 1982-04-01
```

----  

## Trend - Line Chart

**Objective:** See what the oil production in the US looked like from 1981 - 2016 by year. I want to illustrate the changes over the time period. This is a very high-level view and only shows us a decline followed by a ramp up at the end of the period.

I decided to use a line chart to show the trend over time. When using discrete data you should use a column chart to avoid any confusion that in between these years the data actually was simply linear. However, it paints a much clearer picture this way and is not misleading.

### Which of these views would you rather see?

#### Poor Version  
The x-axis is a disaster and the y-axis isn't formatted well. While it gets the point across, it's still almost worthless.



```r
df = data %>% 
  group_by(Year) %>%
  summarise(ThousandBarrel = sum(ThousandBarrel))

p = ggplot(df,aes(x=Year,y=ThousandBarrel,group=1)) 
p + geom_line(stat='identity') + 
  ggtitle('Oil Production Over Time') + 
  theme(plot.title = element_text(hjust = 0.5),plot.subtitle = element_text(hjust = 0.5)) + 
  xlab('') + ylab('')
```

<img src="http://i.imgur.com/QjuY0mP.png" title="plot of chunk unnamed-chunk-4" alt="plot of chunk unnamed-chunk-4" style="display: block; margin: auto;" />

#### Better Version  
The title gives us a much better understanding of what we're looking at. The chart is slightly wider and the axes are formatted to be legible.


```r
p = ggplot(df,aes(x=Year,y=ThousandBarrel,group=1)) 
p + geom_line(stat='identity') + 
  ggtitle('Thousand Barrel Oil Production By Year in the U.S.') +
  theme(plot.title = element_text(hjust = 0.5),plot.subtitle = element_text(hjust = 0.5)) + 
  theme(axis.text.x = element_text(angle = 90, hjust = 1)) + 
  scale_y_continuous(labels = comma)
```

<img src="http://i.imgur.com/DmOdVOK.png" title="plot of chunk unnamed-chunk-5" alt="plot of chunk unnamed-chunk-5" style="display: block; margin: auto;" />


## Comparison - Line Chart  

**Objective**: Identify which states affected the trend the most. Evaluate them simultaneously in order to paint the picture and compare them.  

From this visual you can see the top states are Alaska, California, Louisiana, Oklahoma, Texas and Wyoming. Texas seems to break the mold quite drastically and drove the spike which occurred after 2010.

### Which of these views would you rather see?

#### Poor Version  
There are far too many colors going on here. Everything at the bottom of the chart is relatively useless and takes our focus away from the big players. 


```r
df = data %>%
  group_by(Location, Year) %>%
  summarise(ThousandBarrel = sum(ThousandBarrel))

df$Year = as.numeric(df$Year)

p = ggplot(df,aes(x=Year,y=ThousandBarrel,col=Location))
p + geom_line(stat='identity') + 
  ggtitle(paste('Oil Production By Year By State in the U.S.')) + 
  theme(plot.title = element_text(hjust = 0.5)) + 
  theme(axis.text.x = element_text(angle = 90, hjust = 1))
```

![plot of chunk unnamed-chunk-6](http://i.imgur.com/LdhZsPF.png)

#### Better Version  
This focuses attention on the top producing states. It compares them to each other and shows the trend per state as well.


```r
n=6 #Arbitrary at first, after trying a few, this made the most sense
topN = data %>%
  group_by(Location) %>%
  summarise(ThousandBarrel = sum(ThousandBarrel)) %>%
  arrange(-ThousandBarrel) %>%
  top_n(n)

df = data %>%
  filter(Location %in% topN$Location) %>%
  group_by(Year,Location) %>%
  summarise(ThousandBarrel = sum(ThousandBarrel))

df$Year = as.numeric(df$Year)
df$Location = as.factor(df$Location)

p = ggplot(df,aes(x=Year,y=ThousandBarrel,group=1))
p + geom_line(stat='identity') + 
  ggtitle(paste('Top',as.character(n),'States - Oil Production By Year in the U.S.')) + 
  theme(plot.title = element_text(hjust = 0.5)) + 
  theme(axis.text.x = element_text(angle = 90, hjust = 1)) + 
  facet_wrap(~Location) + 
  scale_y_continuous(labels = comma) 
```

![plot of chunk unnamed-chunk-7](http://i.imgur.com/Vkxymeb.png)



## Relationship - Scatter Plot

**Objective**: See if Alaska and California data is correlated (This probably isn't important but it allows us to use the same data).

### Which of these views would you rather see?

#### Poor Version  
Lots of completely irrelevant data! Size of the point should have nothing to do with the year.  


```r
statesList = c('Alaska','California')
df = data %>%
  filter(Location %in% statesList) %>%
  spread(Location,ThousandBarrel) %>%
  select(Alaska,California,Month,Year)

p = ggplot(df,aes(x=Alaska,y=California,col=Month,size=Year))
p + geom_point() + 
  scale_y_continuous(labels = comma) +
  scale_x_continuous(labels = comma) +
  ggtitle('Oil Production - CA vs. AK') + 
  theme(plot.title = element_text(hjust = 0.5))
```

![plot of chunk unnamed-chunk-8](http://i.imgur.com/ta0Uv6a.png)

#### Better Version  
The trend line is nice because it helps to visualize the relationship even more. While it can sometimes be misleading, it makes sense with our current data.  


```r
df = data %>%
  filter(Location %in% statesList) %>%
  spread(Location,ThousandBarrel) %>%
  select(Alaska,California,Year)

p = ggplot(df,aes(x=Alaska,y=California))
p + geom_point() + 
  scale_y_continuous(labels = comma) +
  scale_x_continuous(labels = comma) +
  ggtitle('Monthly Thousand Barrel Oil Production 1981-2016 CA vs. AK') + 
  theme(plot.title = element_text(hjust = 0.5)) + 
  geom_smooth(method='lm')
```

![plot of chunk unnamed-chunk-9](http://i.imgur.com/Kxtvyrc.png)




## Distribution - Boxplot 

**Objective**: Examine the range of production by state and year over the time period to give us an idea of the variance.

### Which of these views would you rather see?

#### Poor Version  



```r
df = data %>%
  group_by(Year,Location) %>%
  summarise(ThousandBarrel = sum(ThousandBarrel))

p = ggplot(df,aes(x=Location,y=ThousandBarrel))
p + geom_boxplot() + 
  ggtitle('Distribution of Oil Production by State')
```

![plot of chunk unnamed-chunk-10](http://i.imgur.com/QYyBcNn.png)


#### Better Version  
This gives a nice ranking to the plot while still showing their distributions. While it was semi-apparent in the line charts, the variance of Texas is huge compared to the others! We could take this a step further and separate out the big players from the smaller players.


```r
p = ggplot(df,aes(x=reorder(Location,ThousandBarrel),y=ThousandBarrel))
p + geom_boxplot() + 
  scale_y_continuous(labels = comma) +
  ggtitle('Distribution of Annual Oil Production By State (1981 - 2016)') + 
  coord_flip()
```

![plot of chunk unnamed-chunk-11](http://i.imgur.com/AunupK4.png)


## Composition - Stacked Bar 

**Objective**: Check out the composition of total production by state.

### Which of these views would you rather see?

#### Poor Version  
My favorite, the beautiful pie chart! There's nothing better than this...


```r
df = data %>%
  group_by(Location) %>%
  summarise(ThousandBarrel = sum(ThousandBarrel)) %>%
  mutate(ThousandBarrel = ThousandBarrel/sum(ThousandBarrel))

df$ThousandBarrel = round(100*df$ThousandBarrel,0)

library(plotrix)
pie(x=df$ThousandBarrel,labels=df$Location,explode=0.1,col=rainbow(nrow(df)),main='Percentage of Oil Production by State')
```

![plot of chunk unnamed-chunk-12](http://i.imgur.com/vv1q2FX.png)


#### Better Version  
The 1980's and 2010's will be missing years in terms of a "decade" due to the data provided (and it's only 2017). While the percentage labels are slightly off center, it's certainly much better than the pie chart. It's not quite "apples-to-apples" for a comparison because I created different decades, but you get the idea.

I also created an "Other" category in order to simplify the output. When you are doing comparisons, it's typically a good idea to find a way to reduce the number of variables in the output while not removing data by dropping it completely.


```r
data$Decade = '1980s'
data$Decade[data$Year >= 1990] = '1990s'
data$Decade[data$Year >= 2000] = '2000s'
data$Decade[data$Year >= 2010] = '2010s'
data$Decade = as.factor(data$Decade)

top5 = data %>%
  group_by(Location) %>%
  summarise(ThousandBarrel = sum(ThousandBarrel)) %>%
  arrange(-ThousandBarrel) %>%
  top_n(5) %>%
  select(Location)

top5List = top5$Location

data$State = "Other"

for(i in 1:length(top5List)){
  data$State[data$Location == top5List[i]] = top5List[i]
}

df = data %>%
  group_by(Decade,State) %>%
  summarise(ThousandBarrel = sum(ThousandBarrel)) %>%
  mutate(ThousandBarrel = ThousandBarrel/sum(ThousandBarrel))

df$ThousandBarrel = round(df$ThousandBarrel,3)
df$text = paste(round(100*df$ThousandBarrel,0),'%', sep='')

p = ggplot(df,aes(x=Decade,y=ThousandBarrel,col=reorder(State,ThousandBarrel),fill=reorder(State,ThousandBarrel)))
p + geom_bar(stat='identity') + 
  geom_text(aes(label=text),col='Black',size = 4, hjust = 0.5, vjust = 3, position = "stack") + 
  scale_y_continuous(labels = percent) +
  ggtitle('Percentage of Top Oil Producing States by Decade') + 
  guides(fill=guide_legend(title='State'),col=guide_legend(title='State')) + 
  theme(plot.title = element_text(hjust = 0.5))
```

![plot of chunk unnamed-chunk-13](http://i.imgur.com/I7Cyf6Z.png)




### Some other fun concepts are below!  
Some of them are nice, others are terrible! I won't comment on any of them, but I felt it was necessary to include some other ideas I toyed around with. 

Have fun with your data visualizations. The charts I showed here are extremely simple. Being creative by using things other than R wind up making visuals people can remember. There are plenty of examples around, but they all tend to follow basic principles of design. There are ***A TON*** of good books out there on this topic. 

Now it's your turn!



```r
df = data %>% 
  group_by(Location) %>%
  summarise(ThousandBarrel = sum(ThousandBarrel)) %>%
  arrange(-ThousandBarrel)
p = ggplot(df,aes(x=reorder(Location,ThousandBarrel),y=ThousandBarrel))
p + geom_bar(stat='identity') + 
  ggtitle('Oil Production 1981 - 2016 By Location') + 
  theme(plot.title = element_text(hjust = 0.5)) + 
  coord_flip()
```

![plot of chunk unnamed-chunk-14](http://i.imgur.com/DwJ52D9.png)






```r
top10 = data %>%
  group_by(Location) %>%
  summarise(ThousandBarrel = sum(ThousandBarrel)) %>%
  arrange(-ThousandBarrel) %>%
  top_n(10)
```

```
## Selecting by ThousandBarrel
```

```r
print(top10)
```

```
## # A tibble: 10 × 2
##       Location ThousandBarrel
##          <chr>          <dbl>
## 1        Texas       23447172
## 2       Alaska       15775279
## 3   California        9988225
## 4    Louisiana        4267246
## 5     Oklahoma        3701224
## 6      Wyoming        2894624
## 7       Kansas        1708873
## 8     Colorado        1288643
## 9         Utah         894657
## 10 Mississippi         861999
```

```r
df = data %>% 
  group_by(Location,Year) %>%
  filter(Location %in% top10$Location) %>%
  summarise(ThousandBarrel = sum(ThousandBarrel)) 
p = ggplot(df,aes(x=Year,y=ThousandBarrel,col=Location,fill=Location))
p + geom_bar(stat='identity') + 
  ggtitle('Oil Production - Top 10 States') + 
  theme(plot.title = element_text(hjust = 0.5)) + 
  theme(axis.text.x = element_text(angle = 90, hjust = 1))
```

![plot of chunk unnamed-chunk-15](http://i.imgur.com/Y21hxkY.png)




```r
df = data %>%
  filter(Year == 1990)%>%
  group_by(Location) %>%
  summarise(ThousandBarrel = sum(ThousandBarrel))
df$Location = tolower(df$Location)

#Add States without data
States = data.frame(Location = tolower(as.character(state.name)))
missingStates = States$Location[!(States$Location %in% df$Location)]
appendData = data.frame(Location=missingStates,ThousandBarrel=0)
df = rbind(df,appendData)

states_map <- map_data("state")

ggplot(df, aes(map_id = Location)) + 
    geom_map(aes(fill=ThousandBarrel), map = states_map) +
    expand_limits(x = states_map$long, y = states_map$lat)
```

![plot of chunk unnamed-chunk-16](http://i.imgur.com/jT1XTmF.png)



```r
df = data %>% 
  filter(Location == 'Texas') %>%
  group_by(Year,Month) %>%
  summarise(ThousandBarrel = sum(ThousandBarrel))

p = ggplot(df,aes(x=Month,y=ThousandBarrel))
p + geom_line(stat='identity',aes(group=Year,col=Year)) + 
  ggtitle('Oil Production By Year in the U.S.') + 
  theme(plot.title = element_text(hjust = 0.5)) + 
  theme(axis.text.x = element_text(angle = 90, hjust = 1))
```

![plot of chunk unnamed-chunk-17](http://i.imgur.com/4OwEmjV.png)


As always, the code used in this post is on my [GitHub](https://github.com/stoltzmaniac/Data-Visualization-Lesson)
