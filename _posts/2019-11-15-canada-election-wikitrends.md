---
title: "Canadian Elections Wikipedia Trends"
date: 2019-11-15
tags: [wikipediatrend, gganimate, ggplot2, R, Canada, Canadian Elections]
header:
  image: "/images/canada-wikitrends/canada_leaders_crop.jpg"
  excerpt: "wikipediatrend, gganimate, ggplot2, R, Canada, Canadian Elections"
  teaser: /images/canada-wikitrends/canada_leaders_crop.jpg
mathjax: "true"
---

I was playing with the [wikipediatrend](https://github.com/petermeissner/wikipediatrend0) package in R, and thought it would be cool to compare the searches for each Canadian political party and leader in English and French for the 2011, 2015, and 2019 elections using [gganimate](https://github.com/thomasp85/gganimate).

I downloaded the trends for the 2 weeks leading up to the election day and the day after for the three last elections. I plotted tall and wide barplots side by side and animated over the three elections. Every year, the searches steadily rise in the days leading up to the election, jump after the day of the debate, rise higher on the day of the election, and **EXPLODE** the day after.

### Party Leaders for Each Election

| Party |   2011           |         2015     |        2019      |
|:-----:|:----------------:|:----------------:|:----------------:|
|   BQ  | Gilles Duceppes  | Gilles Duceppes  |   Y-F Blanchet   |
|  NDP  | Jack Layton      |  Tom Mulcair     |  Jagmeet Singh   |
|  Lib  | Michael Ignatief | Justin Trudeau   | Justin Trudeau   |
|  Con  | Stephen Harper   | Stephen Harper   | Andrew Scheer    |
| Green | Elizabeth May    | Elizabeth May    | Elizabeth May    |

## English Wikipedia Searches for Leaders
![](https://i.imgur.com/aGg1uub.gif)

2011 was the most equal election year for English wikipedia searches, with most people interested in the incumbent and the winner Stephen Harper, then Jack Layton, and then Michael Ignatieff. A significant portion were also searching for Gilles Duceppes and Elizabeth May.

2015 sees almost all the searches going to Justin Trudeau. He outshined all the other leaders, which makes sense given he was a new leader and also won the election. He also had most searches in the lead up to the election.

Jagmeet Singh got more searches than Trudeau in 2019 and Andrew Scheer received about half of what they got. This points to Singh's popularity. He's the first leader to lose the election and still receive most wikipedia searches.

[//]: <> (comment, add color: Some Markdown text with <span style="color:blue">some *blue* text</span>.) 

## French Wikipedia Searches for Leaders
![](https://i.imgur.com/WFxqUlW.gif)

French Wikipedia searches are markedly different. Gilles Duceppe in 2011 and Yves-Francois Blanchet in 2019 are searched for more than on English Wikipedia. The NDP leaders are also very popular in 2011 and 2019. Tom Mulcair was severely less popular than Jack Layton and Jagmeet Singh. As on English Wikipedia, Justin Trudeau was searched for most on French Wikipedia in his debut election in 2011.

## English Wikipedia Searches for Parties
![](https://i.imgur.com/h45OIix.gif)

Party searches are correlated with leader searches but are more consistent over time because party searches are not entirely dependent on the party leader. Surprisingly, the Bloc Quebecois was the most searched for party on the day after the 2019 election and shows that these variables aren't always correlated. The Bloc Quebecois leader, Yves-Francois Blanchet, received slighly more than Elizabeth May in 2019, by contrast, the Bloc Quebecois party received several times more searches than the Green Party did. I suspect this had something to do with all the seats they gained. The NDP and Liberal Party have a similar number of searches, but the NDP is the most searched for party leading up to the election. 

## French Wikipedia Searches for Parties
![](https://i.imgur.com/2pGI6xg.gif)

The Bloc Quebecois has an unprecedented several times more searches than all the other parties in the 2019 election on French Wikipedia and may point to a revival in Quebecois pride and separatism. In 2011, the NDP had most searches leading up to the election and the day after, and they also won **A LOT** of seats in Quebec. In 2015, the Conservatives lead in searches before the elections, but the Liberals win the election and end up having most searches the day of the election and after. The NDP also received more searches than the Conservatives in the few days surrounding the election.

## Predicting the Proportional Vote with Wikipedia

It would be interesting to look at whether Wikipedia searches could help predict the proportional representation vote. It is already clear that it isn't so great at predicting the first-past-the-post winner. 


