---
title: "Replication Check In"
author: "Tim Roy"
date: "October 24, 2018"
output: github_document
---

I seek to replicate and expand on findings from Corstange and York (2018), *Sectarian Framing in the Syrian Civil War*. The authors use a sample of 2000 Syrian refugees in Lebanon in order to find the effect of framing using sectarian rhetoric on the perceived importance of sectarianism in the conflict. 



Corstange and York assess the effect of different frames of the Syrian conflict on the perceived importance of eight different possible reasons for the conflict. The outcome variables are measured on a four point scale and are the following:   
    1. Democratic freedoms,     
    2. Sectarian differences,   
    3. International Rivalries,      
    4. The role of religion in politics,      
    5. Minority rights,      
    6. Terrorist activity,       
    7. Declining living standards, and      
    8. Corruption.

The authors treat a control group to no frame with the following prompt: "People have explained the Syrian conflict to us in a number of different ways." They treat some subjects to a single frame of the conflict and others to two frames. People treated to one frame were fed this prompt: "People have explained the Syrian conflict to us in a number of different ways. For example, many people have described it as a conflict between *FRAME*".   
The frames were as follows:   
1. "Sunnis and Alawis" (_Sectarianism_),  
2. "democracy and dictatorship" (_Democracy_),  
3. "religion and secularism" (_Secularism_), or  
4. "foreign forces fought on Syrian soil" (_Foreigners_).   

When treating subjects to two frames, the authors placed the "Sectarianism" frame against the others while randomizing the order in which they were told. The prompts were phrased as follows: "People have explained the Syrian conflict to us in a number of different ways. For example, many people have described it as a conflict between *FRAME 1*, and a few people have described it as a conflict between *FRAME 2*". 

They find that when sectarian discourse is deployed alone only government supporters attribute greater importance to sectarian differences, however, When placed alongside a competing frame there is no effect. 

I have not yet been able to decide on a manner to expand on the research yet.

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Dataset
```{r}
library(tidyverse)

Syria <- read.csv("framing-ajps-replication.csv")

## Sample is split between government and opposition (fsa + islamist)
Syria$govt <-
    with(Syria, ifelse(faction == "govt", 1, 0))
Syria$oppn <-
    with(Syria, ifelse(faction %in% c("fsa", "islamist"), 1, 0))

## Sample is split between control and treatment groups (different frames)
Syria$tr_ctrl <-
    with(Syria,
         ifelse(treat == "control",
                1, 0))

## PLotting treatment and control group
ggplot(Syria, aes(x = tr_ctrl)) +
  geom_bar(fill = "orange", 
           alpha = 0.6) +
  scale_x_discrete(breaks = seq(0, 1, 1)) +
  labs(title = "Sample Distribution",
       y = "Count") +
  xlab("Treatment                                  Control") +
  ggtitle("Sample Distribution")


```

## Distributions of some covariates
```{r}
# Preparing data to plot distributions of sample
data_gathered <- Syria %>% gather(- case_id, -age, -female, -sunni, -kurd, -leb_rd, -syr_rd, -faction, -treat, -fight_dem, -fight_sect, -fight_foreign, - fight_rel, -fight_minority, -fight_terror, -fight_econ, -fight_corruption, -id_syrian, -govt, -oppn, -tr_ctrl, key = "Measure", value = "Value")

data_gathered$Value <-
    with(data_gathered,
         factor(Value,
                levels = c("Low",
                           "Medium",
                           "High")))

data_gathered_tr <- subset(data_gathered, subset = tr_ctrl == 0)
data_gathered_ctrl <- subset(data_gathered, subset = tr_ctrl == 1)

# Plotting distributions of education , engagement, and religiosity
library(wesanderson)

eer_ctrl  <- ggplot(data_gathered_ctrl, aes(x = Value, fill = Measure)) +
  geom_bar(position = position_dodge(width = 0.6),
           alpha = 0.6) +  
  labs(title = "Control Distribution of Education, Engagement and Religiosity",
       y = "Frequency",
       x = "Score") +
  scale_fill_manual(values = wes_palette(n = 3, name = "GrandBudapest1"), 
                    labels = c("Education", "Engagement", "Religiosity"))              

eer_tr <- ggplot(data_gathered_tr, aes(x = Value, fill = Measure)) +
  geom_bar(position = position_dodge(width = 0.6),
           alpha = 0.6) +                  
  labs(title = "Treatment Distribution of Education, Engagement and Religiosity",
       y = "Frequency",
       x = "Score") +
  scale_fill_manual(values = wes_palette(n = 3, name = "GrandBudapest1"), 
                    labels = c("Education", "Engagement", "Religiosity"))
  

library(gridExtra)
grid.arrange(eer_ctrl, eer_tr, nrow = 2)

```
```{r echo = F}
age_tr <- subset(Syria, subset = tr_ctrl == 0)
age_ctrl <- subset(Syria, subset = tr_ctrl == 1)

agectrl <- ggplot(age_ctrl, aes(x = age)) + 
  geom_density(aes(fill = "orange", alpha = 0.5)) + 
  theme(legend.position = "none") +
  ylab("Density") +
  xlab("Age") +
  ggtitle("Distribution of Control Group's Ages")

agetr <- ggplot(age_tr, aes(x = age)) + 
  geom_density(aes(fill = "orange", alpha = 0.5)) + 
  theme(legend.position = "none") +
  ylab("Density") +
  xlab("Age") +
  ggtitle("Distribution of Treatment Groups' Ages")

grid.arrange(agectrl, agetr, nrow = 2)
```


## Outcome variables
The code below is used in Corstange and York's replication files which I have provided in a folder titled "Article and Replication Files". I have made small alterations to display the distribution of control and treatment group outcome variables. 
```{r}
# Tweaking code used in the replication files
# to display the distribution of dependent variables
Sumstats_tr <-
    with(subset(Syria, subset = tr_ctrl == 0),
         data.frame(gr = factor(1:2, labels = c("Opposition",
                                                "Government")),
                    dv = rep(factor(1:8,
                                    labels = c("Democracy",
                                               "Sectarianism",
                                               "Foreign Forces",
                                               "Religion",
                                               "Minorities",
                                               "Terrorism",
                                               "Economy",
                                               "Corruption")),
                             each = 2),
                    pe = c(aggregate(fight_dem ~ govt,
                                     FUN  = mean)$fight_dem,
                           aggregate(fight_sect ~ govt,
                                     FUN  = mean)$fight_sect,
                           aggregate(fight_foreign ~ govt,
                                     FUN  = mean)$fight_foreign,
                           aggregate(fight_rel ~ govt,
                                     FUN  = mean)$fight_rel,
                           aggregate(fight_minority ~ govt,
                                     FUN  = mean)$fight_minority,
                           aggregate(fight_terror ~ govt,
                                     FUN  = mean)$fight_terror,
                           aggregate(fight_econ ~ govt,
                                     FUN  = mean)$fight_econ,
                           aggregate(fight_corruption ~ govt,
                                     FUN  = mean)$fight_corruption),
                    # Confidence Intervals
                    se = c(aggregate(fight_dem ~ govt,
                                     FUN  = function(x) sd(x, na.rm = TRUE) / sqrt(sum(!is.na(x))))$fight_dem,
                           aggregate(fight_sect ~ govt,
                                     FUN  = function(x) sd(x, na.rm = TRUE) / sqrt(sum(!is.na(x))))$fight_sect,
                           aggregate(fight_foreign ~ govt,
                                     FUN  = function(x) sd(x, na.rm = TRUE) / sqrt(sum(!is.na(x))))$fight_foreign,
                           aggregate(fight_rel ~ govt,
                                     FUN  = function(x) sd(x, na.rm = TRUE) / sqrt(sum(!is.na(x))))$fight_rel,
                           aggregate(fight_minority ~ govt,
                                     FUN  = function(x) sd(x, na.rm = TRUE) / sqrt(sum(!is.na(x))))$fight_minority,
                           aggregate(fight_terror ~ govt,
                                     FUN  = function(x) sd(x, na.rm = TRUE) / sqrt(sum(!is.na(x))))$fight_terror,
                           aggregate(fight_econ ~ govt,
                                     FUN  = function(x) sd(x, na.rm = TRUE) / sqrt(sum(!is.na(x))))$fight_econ,
                           aggregate(fight_corruption ~ govt,
                                     FUN  = function(x) sd(x, na.rm = TRUE) / sqrt(sum(!is.na(x))))$fight_corruption)))

Sumstats_ctrl <-
    with(subset(Syria, subset = tr_ctrl == 1),
         data.frame(gr = factor(1:2, labels = c("Opposition",
                                                "Government")),
                    dv = rep(factor(1:8,
                                    labels = c("Democracy",
                                               "Sectarianism",
                                               "Foreign Forces",
                                               "Religion",
                                               "Minorities",
                                               "Terrorism",
                                               "Economy",
                                               "Corruption")),
                             each = 2),
                    pe = c(aggregate(fight_dem ~ govt,
                                     FUN  = mean)$fight_dem,
                           aggregate(fight_sect ~ govt,
                                     FUN  = mean)$fight_sect,
                           aggregate(fight_foreign ~ govt,
                                     FUN  = mean)$fight_foreign,
                           aggregate(fight_rel ~ govt,
                                     FUN  = mean)$fight_rel,
                           aggregate(fight_minority ~ govt,
                                     FUN  = mean)$fight_minority,
                           aggregate(fight_terror ~ govt,
                                     FUN  = mean)$fight_terror,
                           aggregate(fight_econ ~ govt,
                                     FUN  = mean)$fight_econ,
                           aggregate(fight_corruption ~ govt,
                                     FUN  = mean)$fight_corruption),
                    se = c(aggregate(fight_dem ~ govt,
                                     FUN  = function(x) sd(x, na.rm = TRUE) / sqrt(sum(!is.na(x))))$fight_dem,
                           aggregate(fight_sect ~ govt,
                                     FUN  = function(x) sd(x, na.rm = TRUE) / sqrt(sum(!is.na(x))))$fight_sect,
                           aggregate(fight_foreign ~ govt,
                                     FUN  = function(x) sd(x, na.rm = TRUE) / sqrt(sum(!is.na(x))))$fight_foreign,
                           aggregate(fight_rel ~ govt,
                                     FUN  = function(x) sd(x, na.rm = TRUE) / sqrt(sum(!is.na(x))))$fight_rel,
                           aggregate(fight_minority ~ govt,
                                     FUN  = function(x) sd(x, na.rm = TRUE) / sqrt(sum(!is.na(x))))$fight_minority,
                           aggregate(fight_terror ~ govt,
                                     FUN  = function(x) sd(x, na.rm = TRUE) / sqrt(sum(!is.na(x))))$fight_terror,
                           aggregate(fight_econ ~ govt,
                                     FUN  = function(x) sd(x, na.rm = TRUE) / sqrt(sum(!is.na(x))))$fight_econ,
                           aggregate(fight_corruption ~ govt,
                                     FUN  = function(x) sd(x, na.rm = TRUE) / sqrt(sum(!is.na(x))))$fight_corruption)))

Sumstats_tr$vis_type <-
    with(Sumstats_tr,
         factor(ifelse(dv %in% c("Sectarianism",
                                 "Religion",
                                 "Minorities"), 1,
                ifelse(dv %in% c("Foreign Forces",
                                 "Terrorism"), 2,
                ifelse(dv %in% c("Democracy",
                                 "Economy",
                                 "Corruption"), 3, 4))),
                labels = c("Sect",
                           "Government",
                           "Opposition")))

Sumstats_ctrl$vis_type <-
    with(Sumstats_ctrl,
         factor(ifelse(dv %in% c("Sectarianism",
                                 "Religion",
                                 "Minorities"), 1,
                ifelse(dv %in% c("Foreign Forces",
                                 "Terrorism"), 2,
                ifelse(dv %in% c("Democracy",
                                 "Economy",
                                 "Corruption"), 3, 4))),
                labels = c("Sect",
                           "Government",
                           "Opposition")))

## government - opposition difference
Sumstats_tr_diff <-
    data.frame(gr = "Treatment",
               dv = levels(Sumstats_tr$dv),
               pe = (subset(Sumstats_tr, gr == "Government",
                            select = pe, drop = TRUE) -
                     subset(Sumstats_tr, gr == "Opposition",
                            select = pe, drop = TRUE)),
               se = sqrt(subset(Sumstats_tr, gr == "Government",
                                select = se, drop = TRUE)^2
                         + subset(Sumstats_tr, gr == "Opposition",
                                  select = se, drop = TRUE)^2))
Sumstats_tr_diff$vis_type <-
    with(Sumstats_tr_diff,
         factor(ifelse(dv %in% c("Sectarianism",
                                 "Religion",
                                 "Minorities"), 1,
                ifelse(dv %in% c("Foreign Forces",
                                 "Terrorism"), 2,
                ifelse(dv %in% c("Democracy",
                                 "Economy",
                                 "Corruption"), 3, 4))),
                labels = c("Sect",
                           "Government",
                           "Opposition")))
Sumstats_tr_diff$dv <-
    with(Sumstats_tr_diff,
         reorder(dv, pe))

Sumstats_ctrl_diff <-
    data.frame(gr = "Control",
               dv = levels(Sumstats_ctrl$dv),
               pe = (subset(Sumstats_tr, gr == "Government",
                            select = pe, drop = TRUE) -
                     subset(Sumstats_ctrl, gr == "Opposition",
                            select = pe, drop = TRUE)),
               se = sqrt(subset(Sumstats_ctrl, gr == "Government",
                                select = se, drop = TRUE)^2
                         + subset(Sumstats_ctrl, gr == "Opposition",
                                  select = se, drop = TRUE)^2))
Sumstats_ctrl_diff$vis_type <-
    with(Sumstats_ctrl_diff,
         factor(ifelse(dv %in% c("Sectarianism",
                                 "Religion",
                                 "Minorities"), 1,
                ifelse(dv %in% c("Foreign Forces",
                                 "Terrorism"), 2,
                ifelse(dv %in% c("Democracy",
                                 "Economy",
                                 "Corruption"), 3, 4))),
                labels = c("Sect",
                           "Government",
                           "Opposition")))
Sumstats_ctrl_diff$dv <-
    with(Sumstats_ctrl_diff,
         reorder(dv, pe))

diffs <- rbind(Sumstats_ctrl_diff, Sumstats_tr_diff)

## Plotting Treatment Sumstats
plot_sumstats_tr <-
  ggplot(data = Sumstats_tr,
         aes(x     = dv,
             y     = pe,
             color = as.factor(ifelse(vis_type == "Sect", 2, 1)))) +
  geom_errorbar(aes(ymin = pe - se * 1.96,
                    ymax = pe + se * 1.96),
                width = 0.15) +
  geom_point(aes(size  = as.factor(ifelse(vis_type == "Sect", 2, 1)))) +
  scale_size_manual(values = c(1.75, 2.75)) +
  scale_color_manual(values = c("grey67", "darkorchid4")) +
  scale_x_discrete(name = NULL) +
  scale_y_continuous(name         = NULL,
                     breaks       = seq(0, 3, 1),
                     labels       = c("Not at All\nImportant",
                                      "Not Very\nImportant",
                                      "Somewhat\nImportant",
                                      "Very\nImportant"),
                     minor_breaks = seq(0.5, 2.5, 1),
                     limits       = c(-0.25, 3.25)) +
  coord_flip() +
  theme_bw() +
  facet_wrap(~ gr) +
  theme(legend.position = "none")

## Plotting control sumstats
plot_sumstats_ctrl <-
  ggplot(data = Sumstats_ctrl,
         aes(x     = dv,
             y     = pe,
             color = as.factor(ifelse(vis_type == "Sect", 2, 1)))) +
  geom_errorbar(aes(ymin = pe - se * 1.96,
                    ymax = pe + se * 1.96),
                width = 0.15) +
  geom_point(aes(size  = as.factor(ifelse(vis_type == "Sect", 2, 1)))) +
  scale_size_manual(values = c(1.75, 2.75)) +
  scale_color_manual(values = c("grey67", "darkorchid4")) +
  scale_x_discrete(name = NULL) +
  scale_y_continuous(name         = NULL,
                     breaks       = seq(0, 3, 1),
                     labels       = c("Not at All\nImportant",
                                      "Not Very\nImportant",
                                      "Somewhat\nImportant",
                                      "Very\nImportant"),
                     minor_breaks = seq(0.5, 2.5, 1),
                     limits       = c(-0.25, 3.25)) +
  coord_flip() +
  theme_bw() +
  facet_wrap(~ gr) +
  theme(legend.position = "none")

## Plotting Differences
plot_diffs <-
    ggplot(data = diffs,
           aes(x     = dv,
               y     = pe,
               color = as.factor(ifelse(vis_type == "Sect", 2, 1)))) +
    geom_errorbar(aes(ymin = pe - se * 1.96,
                      ymax = pe + se * 1.96),
                  width = 0.15) +
    geom_point(aes(size  = as.factor(ifelse(vis_type == "Sect", 2, 1)))) +
    scale_size_manual(values = c(1.75, 2.75)) +
    scale_color_manual(values = c("grey67", "darkorchid4")) +
    geom_hline(yintercept = 0, lty = 2) +
    scale_x_discrete(name = NULL) +
    scale_y_continuous(name         = NULL,
                       breaks       = seq(-3, 3, 1),
                       labels       = c("Opposition\nNarrative",
                                        abs(-2:2),
                                        "  Government\nNarrative"),
                       minor_breaks = seq(-2.5, 2.5, 1),
                       limits       = c(-3.50, 3.50)) +
    coord_flip() +
    facet_wrap(~ gr) +
    theme_bw() +
    theme(legend.position = "none")

```

### Sample Means of Treatment and Control Groups with 95% Confidence Interval
The variables that are proxies for sectarianism are highlighted.
```{r}
plot_sumstats_ctrl + 
  ggtitle("Control")
  
plot_sumstats_tr + 
  ggtitle("Treatment")
```

### Difference in Sample Means of Treatment and Control Groups with 95% Confidence Interval
```{r}
plot_diffs + ggtitle("Differences Between Government and Opposition Sample Means")
```






