---
title: "Discrimination in Europe"
date: 2019-11-16
tags: [European Social Survey, discrimination, France, England, gganimate, ggplot2, R]
header:
  image: "/images/euro-discr/france_islam_banner.jpg"
  excerpt: "European Social Survey, discrimination, France, England, gganimate, ggthemr, ggplot2, R"
  teaser: /images/euro-discr/france_islam_banner.jpg
mathjax: "true"
---

I am working on a project right now for a class at Sciences Po about discrimination. I figured I could look at how people have perceived being discriminated against over time using each round of the European Social Survey and practice my animation and visualization skills. I was really happy with [ggthemr](https://github.com/cttobin/ggthemr) who came through with the beautiful *"flat dark"* theme to match the website theme. 

I decided to plot the proportion of ethnic minorities who feel discriminated against in their country for France and England over time. The plots animate over time, and over different forms of discrimination. Bars represent 95% confidence intervals. 

| Round | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|
| **Year** | 2002 | 2004 | 2006 | 2008 | 2010 | 2012 | 2014 | 2016 | 2018 |

# Discrimination of Ethnic Minorities

## Discrimination of Minorities in France
![](https://i.imgur.com/UTiWXh9.gif)

![](https://i.imgur.com/jfRPIKY.gif)

The trends in discrimination perceptions for ethnic minorities in France seem to follow a U-shaped trend, dipping in round 4-6 of the ESS, or in 2008-2012. The average proportion is the highest for racial discrimination, which makes sense, given I subset for ethnic minorities in this plot. Teh U-shaped trend in France may have something to do with hysteria about immigrants and Muslims. 9/11 happened a year before the first round, and the Charlie Hebdo and Bataclan attacks in France happened between rounds 7 and 8, in 2015. These events may have caused an effect on people's perceptions. France's recurring secularism debate also must cause some variation.

## Discrimination of Minorities in England
![](https://i.imgur.com/hACryzm.gif)

![](https://i.imgur.com/qKC0Uz7.gif)

In England, there is U-shaped curved for racial discrimination that is much more pronounced than in France.

# Discrimination of Muslims
I decided to repeat the above step, but by subsetting for Muslims only. In some rounds of the survey, no respondents were Muslim. 

## Discrimination of Muslims in France
![](https://i.imgur.com/kQK7Gbx.gif)

![](https://i.imgur.com/bue5Bcw.gif)

The trends for racial discrimination and religious discrimination for Muslims in France are opposite, with relgious discrimination rising to its highest in 2018 and racial discrimination lowering signficantly over time. Over time, people have been to say they are discriminated against more because of their religion than their race. 

The average proportion for racial discrimination and religious dirscrimination (represented by the horizontal dashed lines) are almost equal among French Muslims. In the plots for England below, Muslims report much higher religious didcrimination on average than racial discrimination. For both France and England, Muslims report being discriminated against more because of their religion than their race in 2016-2018.

## Discrimination of Muslims in England
![](https://i.imgur.com/pY47OiF.gif)

![](https://i.imgur.com/ENS4uHT.gif)

We're missing data for 2004 and 2006 which is when the London bombings happened, so we can't say too much about the trend over time, and the impact of traumatic events in England, but we do see the similar dip in 2010-2012 that we see in first plots.

## Impactful Events, Discrimination, and Perceived Discrimination

The curves dip after 9/11 and before the last 2 rounds which makes me think about the impact of 9/11 and the Charlie Hebdo and Bataclan events. It's a shame we're missing data for Muslims in England. Nevertheless, these plots highlight the possible effect of political climate on discrimination and perceived discrimination.
