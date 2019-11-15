---
title: "Canadian Elections Wikipedia Trends"
date: 2019-11-15
tags: [wikipediatrend, gganimate, ggplot2, R, Canada, Canadian Elections]
header:
  image: "/images/canada-wikitrends/canada_leaders_crop.jpg"
excerpt: "wikipediatrend, gganimate, ggplot2, R, Canada, Canadian Elections"
teaser: "/images/canada-wikitrends/canada_leaders_crop.jpg"
mathjax: "true"
---

I was playing with the [wikipediatrend](https://github.com/petermeissner/wikipediatrend0) package in R, and thought it would be cool to compare the searches for each Canadian political party and leader in English and French for the 2011, 2015, and 2019 elections using [gganimate](https://github.com/thomasp85/gganimate).

I downloaded the trends for the 2 weeks leading up to the election day and the day after for the three last elections. I plot tall and wide barplots side by side and animate over the three elections. Every year, the searches steadily rise in the days leading up to the election, jump after the day of the debate, rise higher on the day of the election, and **EXPLODE** the day after. Let's see what we can learn about public interest and politicians and party popularity through these animations.

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

## English Wikipedia Searches for Parties
![](https://i.imgur.com/KIbhCSr.gif)

## French Wikipedia Searches for Parties
![](https://i.imgur.com/2pGI6xg.gif)




