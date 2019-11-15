---
title: "Homework 5"
author: "Tim Roy"
date: "December 1, 2018"
output: github_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

```{r}
#setwd(dirname(rstudioapi::getActiveDocumentContext()$path))
#install.packages("pacman")
pacman::p_load(plyr,
               tidyverse,
               texreg,
               reshape2,
               lmtest,
               separationplot,
               pROC,
               Zelig)
cyyoung <- read_csv("cyyoung.csv")
```

## a. fit a logistic regression with ERA and Win %
```{r}
# Putting data in a vector of y's and matrix of x's
y <- cyyoung$cy
x <- cbind(cyyoung$era, cyyoung$winpct)

# Likelihood function for logit
llk.logit <- function(param,y,x) {
  os <- rep(1,length(x[,1]))
  x <- cbind(os,x)
  b <- param[ 1 : ncol(x) ]
  xb <- x%*%b
  sum( y*log(1+exp(-xb)) + (1-y)*log(1+exp(xb)));
  # optim is a minimizer, so min -ln L(param|y)
}

# Fit logit model using optim
ls.result <- lm(data = cyyoung, cy ~ era + winpct) # use ls estimates as starting values
stval <- ls.result$coefficients # initial guesses
logit.result.opt <- optim(stval,llk.logit,method="BFGS",
                          hessian=T,y=y,x=x)
# call minimizer procedure
pe.opt <- logit.result.opt$par ; pe.opt # point estimates
```
Points estimates are printed above.
```{r}
vc.opt <- solve(logit.result.opt$hessian) # var-cov matrix
se.opt <- sqrt(diag(vc.opt)) ; se.opt # standard errors
```
Standard errors are printed above.
```{r}
ll.opt <- -logit.result.opt$value ; ll.opt # likelihood at maximum
```
Likelihood at maximum is printed above.

```{r}
# Now with glm()
baseline <- cy ~ winpct + era
logit.cy <- logit.cy.1 <- glm(data = cyyoung, baseline, family = "binomial")
screenreg(logit.cy)
```

As you can tell, the point estimates and standard errors match for the ```optim()``` and ```glm()``` methods.

## b. without using a special package, calculate probability pitcher receives award
```{r}
# sequence of era values
era <- seq(1.5, 5, .25)

# Creating matrix of values from which to predict fit values
pred <- expand.grid(era = era,
                    winpct = c(mean(cyyoung$winpct), 0.5, 0.9))

# Predicting our fitvalues
pred[ , c("fit", "se.fit", "residual scale")] <-
  predict(logit.cy,
          newdata = pred,
          type = "response",
          se.fit = T)

# And now we DANCE... woops I meant PLOT
ggplot() +
  geom_line(data = filter(pred, winpct == mean(cyyoung$winpct)),
            aes(x = era, y = fit, col = "mean"), size = 1) +
  geom_line(data = filter(pred, winpct == 0.5),
            aes(x = era, y = fit, col = "0.5"), size = 1) +
  geom_line(data = filter(pred, winpct == 0.9),
            aes(x = era, y = fit, col = "0.9"), size = 1) +
  scale_color_manual(name = "Win Percentage",
                     values = c(`mean` = "lightsalmon",
                                `0.5` = "dodgerblue",
                                `0.9` = "red")) +
  ggtitle("Probability of Pitcher Winning Cy Young Award") +
  xlab("Earned Run Average") +
  ylab("Predicted Probability of Winning Cy Young Award") +
  theme(plot.title = element_text(hjust = 0.5)) +
  theme_bw()
```

I predicted probablities using a sequence of ERA from 1.5 to 5 and values of win percentage at 0.5, its mean which is `r round(mean(cyyoung$winpct), 2)`, and 0.9. Without looking at uncertainty, it seems win percentage matters at middle to low values of ERA, but has practically no effect at an ERA of 5. Since no major league pitchers usually get ERAs above 4.5,  it seems as if win percentage is an important predictor of winning the award.


## c. now with confidence intervals
```{r}
# Using type = "link" to get standard errors
pred[ , c("link", "se.link", "link residual scale")] <-
  predict(logit.cy,
          newdata = pred,
          type = "link",
          se.fit = T)

# plogis gives the distribution function of a logistic distribution
# we need to use plogis to transform the outpust from predict to predicted probs
pred$PP <- plogis(pred$link)

# same things with upper and lower limits
pred$LL <- plogis(pred$link - (1.96 * pred$se.link))
pred$UL <- plogis(pred$link + (1.96 * pred$se.link))

# Defensive Coding to check if predicted probability approximates the fit we estimated prior
stopifnot(pred$fit - pred$PP < 0.001)

# and now we plot
# it's important to plot the geoms in this order to ease visualization
ggplot() +
  # Confidence intervals
  geom_ribbon(data = filter(pred, winpct == 0.5),
              aes(x = era, ymax = UL, ymin = LL, fill = "0.5", col = "0.5"), 
              alpha = 0.2, linetype = 2) +
  geom_ribbon(data = filter(pred, winpct == 0.9),
              aes(x = era, ymax = UL, ymin = LL, fill = "0.9", col = "0.9"), 
              alpha = 0.2, linetype = 2) +
  geom_ribbon(data = filter(pred, winpct == mean(cyyoung$winpct)),
              aes(x = era, ymax = UL, ymin = LL, fill = "mean"), alpha = 0.4) +
  # Predicted probabilities
  geom_line(data = filter(pred, winpct == mean(cyyoung$winpct)),
            aes(x = era, y = PP, col = "mean"), size = 1.2) +
  geom_line(data = filter(pred, winpct == 0.5),
            aes(x = era, y = PP, col = "0.5"), size = 1) +
  geom_line(data = filter(pred, winpct == 0.9),
            aes(x = era, y = PP, col = "0.9"), size = 1) +
  # Line colors
  scale_color_manual(name = "Win Percentage",
                     values = c(`mean` = "lightgreen",
                                `0.5` = "dodgerblue",
                                `0.9` = "red")) +
  # Ribbon colors
  scale_fill_manual(name = "Win Percentage",
                    values = c(`mean` = "lightgreen",
                               `0.5` = "dodgerblue",
                               `0.9` = "red")) +
  ggtitle("Probability of Pitcher Winning Cy Young Award") +
  xlab("Earned Run Average") +
  ylab("Predicted Probability of Winning Cy Young Award") +
  scale_y_continuous(breaks = seq(0, 1, .1)) +
  scale_x_continuous(breaks = seq(1.5, 5, .5)) +
  # Centering title
  theme(plot.title = element_text(hjust = 0.5)) +
  theme_bw()
```

We can be much more certain about our estimates when we hold win percentage at its mean as evidenced by how narrow the confidence intervals are compared to those of the other lines. Both confidence intervals for win percentage equals 0.5 and 0.9 overlap the fitted line when holding win percentage at the mean. For this reason we cannot conclude that the difference between 0.5 and the mean and 0.9 and the mean are statistically significant. The difference between 0.5 and 0.9 does seem to be statistically significant.

## d. find a better model of cy
I want to plot the covariates against the dependent variable to get an idea of the effects going on. I'll use ```geom_point()``` to get an idea of any relationships in the data.
```{r}
# Removing year, pitcher, and team variables for a melted dataframe
cy <- cyyoung[, -c(1, 2, 11)]

# Melting all variables except for cy
melted <- melt(cy, 1:11, c(1:6, 8:11))

# Plotting all variables against cy
ggplot(data = melted, aes(x = as.factor(cy), y = value)) +
  geom_jitter(size = 0.1, width = 0.25) +
  facet_wrap(~ variable, scales = "free") +
  ggtitle("Values of Covariates for Cy Young Winners and non-Winners") +
  theme_bw()
```

Points that are on the left represent observations where the pitcher did not win the award, whereas points on the right of the plot represent observations where the pitcher won the award. Based on these plots, it seems that the variables with the most clear relationships are wins, win percentage, earned run average and strikeouts. If we look at innings there also seems to be a relationship. Walks do not seem to have a strong relationship with winning the award. This makes sense. 
Let's see if ```geom_smooth()``` confirms this. 

```{r}
# Plotting all variables against cy
ggplot(data = melted, aes(x = cy, y = value)) +
  geom_jitter(size = 0.1, width = 0.25) +
  facet_wrap(~ variable, scales = "free") +
  geom_smooth() +
  ggtitle("Values of Covariates for Cy Young Winners and non-Winners") +
  theme_bw()
```

This confirms our previous thoughts. Since a pitcher who pitches a lot of games will have more walks than a bad pitcher that played a few games,m aybe we should make variables to measure strikeouts and walks per inning and see if there looks to be a relationship with these variables.

**Checking for a relationship in walks per inning**
```{r}
cyyoung$strikeouts_per_inning <- cyyoung$strikeout/cyyoung$innings
cyyoung$walks_per_inning <- cyyoung$walks/cyyoung$innings

ggplot(data = cyyoung, aes(x = cy, y = walks_per_inning)) +
  geom_jitter() +
  ggtitle("Walks per inning") +
  geom_smooth() +
  theme_bw()

ggplot(data = cyyoung, aes(x = cy, y = strikeouts_per_inning)) +
  geom_point() +
  ggtitle("Srikeouts per inning") +
  geom_smooth() +
  theme_bw()
```

There is clearly a much greater relationship with strikeouts than with walks per inning. The data points for walks per inning for award winners are concentrated around the mean, meaning winners usually have average walks per inning, and that this variable is not a great predictor of winning the award. 

I don't think there is a need to include either of these variables. Voters will value a starting pitcher that has played more and thrown more strikeouts than a player that has a better ratio. Walks per innings matters just as little as walks, so I won't include it.  

**Looking at differences in means of variables**
```{r}
# Calculating means of Cy Young winners and non-winners
means <- ddply(melted, .(cy, variable), summarise, value=mean(value))

# Subtracting winners means from non-winners means
diffs <- filter(means, cy == 1)$value - filter(means, cy == 0)$value

# Calculating standard deviations for each variable of interest
# Only keeping variables we care about
stdevs <- cy[ , c(1:6, 8:11)] %>% sapply(sd)  # Applying sd() to all columns

# Calculating standardized differences in means
standardized.diffs <- abs(diffs/stdevs) # Taking absolute value to ease visualization
standardized.diffs.df <- data.frame(standardized.diffs) 

# Plotting
ggplot(standardized.diffs.df, aes(y = standardized.diffs, 
                                  x = rownames(standardized.diffs.df))) +
  geom_point(color = "red", size = 1.5) +
  geom_hline(yintercept = 0, linetype = 2) +
  geom_errorbar(ymin = 0, ymax = standardized.diffs, width = 0) +
  xlab("Variable") +
  ylab("Standardized Difference in Means") +
  ggtitle("Difference in Means Between Cy Young Winners and non-Winners") +
  theme_bw()
```

This plot confirms our conclusions from the earlier plot that looks at the difference between winner and non-winners. ERA, strikeouts, win percentage, innings and wins have the largest difference between the means of either group. I should include these variables in my model. I should choose between win and win percentage given they measure the same thing.

**Checking out how the distributions look**
```{r}
ggplot(data = melted, aes(x = value)) +
  geom_histogram(color = "black", fill = "white") +
  facet_wrap(~variable, scales = "free") +
  theme_bw() +
  ggtitle("Distributions of Variables of Interest")
```

The distributions look normal for all non-binary variables except for team rank. Observations are concentrated at a rank of 1. So it seems our sample is made of of mostly pitchers from number one ranked teams. If we wanted to test the hypothesis that pitchers from better teams win the award, we would need more data. I may want to include team win percentage in the model to get at this question, but given the lack of data, I won't be able to offer a sufficient answer. Moreover, our difference in means plot tells us there isn't much differences in team win percentage between winners and non-winners right off the bat.

**Looking at whether time has an effect on the number of innings a pitcher plays**
```{r}
a <- ggplot(data = cyyoung, aes(x = year, y = innings)) +
  geom_point() +
  geom_smooth() +
  ggtitle("Amount of Innings Pitched Over Time")
 b <- ggplot(data = cyyoung, aes(x = year, y = innings, col = factor(cy), fill = factor(cy))) +
  geom_point() +
  geom_smooth() +
  geom_smooth(formula = cyyoung$year ~ cyyoung$innings) +
  ggtitle("Amount of Innings Pitched Over Time")
 
 cor(cyyoung$year, cyyoung$innings)
 
 gridExtra::grid.arrange(a, b)
```

There is a clear decrease in innings played on average in our sample over time. I am assuming starting pitchers are taken out of games earlier than they used to be and that closers are brought in earlier as well. Too much time and money is spent on pitchers that it is not worth risking ruining their throwing arm. We can therefore theorize an interaction effect between year and innings. An increase in the number of innings should increase the probability of winning the Cy Young award conditional on the year. In later years, innings played should weigh more relative to prior years. 

**Looking at whether the league has an effect on ERA**
```{r}
ggplot(data = cyyoung, aes(x = natleag, y = era, col = factor(cy))) +
  geom_jitter() +
  geom_smooth() +
  ggtitle("Difference in ERAs between American(0) and National(1) Leaguers")

mean(filter(cyyoung, natleag == 0)$era) - mean(filter(cyyoung, natleag == 1)$era)
```

ERA is on average `r round(mean(filter(cyyoung, natleag == 0)$era) - mean(filter(cyyoung, natleag == 1)$era), 2)` higher in the American league than in the National league. This leads me to conclude that we should interact ERA and the league.

```{r}
logit.cy.1 <- glm(data = cyyoung, cy ~ winpct + 
                    era, 
                  family = "binomial",
                  control = glm.control(maxit = 500))
logit.cy.2 <- glm(data = cyyoung, cy ~ winpct + 
                    strikeout + 
                    era, 
                  family = "binomial",
                  control = glm.control(maxit = 500))
logit.cy.3 <- glm(data = cyyoung, cy ~ 
                    winpct + 
                    strikeout + 
                    natleag * 
                    era, 
                  family = "binomial",
                  control = glm.control(maxit = 500))
logit.cy.4 <- glm(data = cyyoung, cy ~ winpct + 
                    innings + 
                    natleag*era + 
                    year +
                    strikeout,
                  family = "binomial",
                  control = glm.control(maxit = 500))
logit.cy.5 <- glm(data = cyyoung, cy ~ 
                    winpct + 
                    strikeout +
                    
                    natleag*era + 
                    year +
                    innings*year, 
                  family = binomial(link = "logit"),
                  control = glm.control(maxit = 500))

m <- cy ~ winpct + strikeout + era*natleag + innings*year

screenreg(list(logit.cy.1, 
               logit.cy.2, 
               logit.cy.3,
               logit.cy.4,
               logit.cy.5))
```

Because designated hitters are allowed in the American league, it is more difficult for pitchers to achieve a lower ERA. In the natonal the league the pitchers have to go up to bat (and pitchers are usually bad hitters), however in the American league the pitcher is always replaced at bat by a designated hitter. For this reason it makes sense to add an interaction effect between the league and ERA. A decrease in ERA should yield an increase in probabilty of winning the award that is conditional on the league. That is, a lower ERA in the American league should predict winning better, or count for more than a lower ERA in the national league.

BIC and AIC are both penalized-likelihood criteria. BIC penalizes more complex models with many terms harsher than does AIC. If they disagree with each other it is because AIC is choosing a larger model. BIC increases and is harshly penalizing it the models as I add covariates, however my AIC improves. Adding a variable improves likelihood, deviance, and AIC, but not BIC. I decided to keep the last model.

**Likelihood Ratio Test**
```{r}
lrt <- lrtest(logit.cy, logit.cy.5)
p.val <- lrt$`Pr(>Chisq)`
lrt
p.val[2]
```

Given that our p-value is `r round(p.val[2], 4)` and is far below the standard of 0.05, we can reject the null hypothesis that likelihood is the same for our restricted and unrestricted models.

**ROC Curve**
```{r}
# Predicting fit with real data
cyyoung[ , "m1_fit"] <- predict(logit.cy, newdata = cyyoung, type = "response")
cyyoung[ , "m2_fit"] <- predict(logit.cy.5, newdata = cyyoung, type = "response")

# ROC cruve
ROC1 <- roc(cyyoung$cy, cyyoung$m1_fit)
ROC2 <- roc(cyyoung$cy, cyyoung$m2_fit)

# Plotting
ggroc(ROC1, color = "black", size = 1) +
  geom_abline(intercept = 1, slope = 1) +
  ggtitle("ROC Curve for Old Model")+
  theme_bw()

# Area under the curve
ROC1$auc

# Repeating for new model
ggroc(ROC2, color = "black", size = 1) +
  geom_abline(intercept = 1, slope = 1) +
  ggtitle("ROC Curve for New Model") +
  theme_bw()

ROC2$auc
```
The curve for our respecified model lies further from the 45 degree line than does the line for the original model. The area under the curve thus larger. This statistic provides us with the probability that randomly selected pairs of winners and non-winners are correctly ordered by the test. Our new model correctly orders winners and non-winners about `r round(ROC2$auc, 2)*100`% of the time, whereas our old model correctly orders only about `r round(ROC1$auc, 2)*100`% of the time.

**Separation plot**
```{r}
# Separation plot's don't print or knit - I had to write them to the disk

# old model
separationplot(pred = cyyoung$m1_fit, 
               actual = cyyoung$cy, 
               type = "line", 
               heading = "Separation Plot for Old Model", 
               height = 2,
               file = "old sep plot.pdf")

# new model
separationplot(pred = cyyoung$m2_fit, 
               actual = cyyoung$cy, 
               type = "line", 
               heading = "Separation Plot for New Model", 
               height = 2,
               file = "new sep plot.pdf")
```

Run this code in RStudio and take a look at the PDFs that were written to the RProject's folder. For some reason they don't print when I knit so I decided to do it this way. 

The separation plots tell us the new model has better predictive power. The light and dark lines are more separated, and the dark lines do not occur as frequently on the left-hand side of the new model's plot than they do in the old model's separation plot. In the old model, the dark lines occur more evenly across the plot too. The comparison of the separation plots suggest the new model predicts better than the old model.

**Percent Correctly Predicted**
```{r}
cyyoung_pcp.1 <- mutate(cyyoung,
                       out = ifelse(m1_fit >= 0.5 , 1, 0),
                       correct = ifelse(out == cy , 1 , 0))

pcp.old <- round(mean(cyyoung_pcp.1$correct, na.rm = TRUE), 3)

cyyoung_pcp.2 <- mutate(cyyoung,
                       out = ifelse(m2_fit >= 0.5, 1, 0),
                       correct = ifelse(out == cy , 1 , 0))

pcp.new <- round(mean(cyyoung_pcp.2$correct, na.rm = TRUE), 3)


# Using function created by Chris Adolph to check code above (defensive coding)
pcp.glm <- function(res, y, type="model") { # other types:  null, improve
  
  pcp <- mean(round(predict(res,type="response"))==y)
  pcpNull <- max(mean(y), mean(1-y))
  pcpImprove <- (pcp-pcpNull)/(1-pcpNull)

  if (type=="model")
    return(pcp)
  if (type=="null")
    return(pcpNull)
  if (type=="improve")
    return(pcpImprove)
}

# old model
chris.pcp.old <- round(pcp.glm(logit.cy,
        cyyoung$cy,
        type = "model"), 3)


# New model
chris.pcp.new <- round(pcp.glm(logit.cy.5,
        cyyoung$cy,
        type = "model"), 3)

stopifnot(chris.pcp.new == pcp.new & chris.pcp.old == pcp.old)

pcp.old
pcp.new
pcp.new - pcp.old
```

When we hold the cut off point at 0.5, the new model has only `r (pcp.new-pcp.old)*100` percent more correct predictions than does the old model. I am curious to see what happens if we hold the cutoff point at 0.75.

```{r}
cyyoung_pcp.1 <- mutate(cyyoung,
                       out = ifelse(m1_fit >= 0.75 , 1, 0),
                       correct = ifelse(out == cy , 1 , 0))

pcp.old <- round(mean(cyyoung_pcp.1$correct, na.rm = TRUE), 3)

cyyoung_pcp.2 <- mutate(cyyoung,
                       out = ifelse(m2_fit >= 0.75, 1, 0),
                       correct = ifelse(out == cy , 1 , 0))

pcp.new <- round(mean(cyyoung_pcp.2$correct, na.rm = TRUE), 3)

pcp.old
pcp.new
pcp.new - pcp.old
```

When we hold the cut off point at 0.75, the new model has `r (pcp.new-pcp.old)*100` percent more correct predictions. The new model's higher predictions are thus more accurate than the old model's, but the accuracy is about the same for predictions that are lower.

## e. First differences for change in z for different values of x
Plotting first differences with Zelig for different values of ERA and different values of Win Percentage
```{r}
# Zelig method
# Scenario 1 (win percentage = 50%)
output1 <- cyyoung %>%
  zelig(data = ., m, 
        model = 'logit', cite = FALSE) %>%
  setx(strikeout = quantile(cyyoung$strikeout, 0.25), winpct = .5)%>%
  setx1(strikeout = quantile(cyyoung$strikeout, 0.75), winpct = .5) %>%
  sim(num = 10000) %>%
  get_qi(output, qi = "fd", xvalue = "x1")

# Scenario 2 (win percentage = 75%)
output2 <- cyyoung %>%
  zelig(data = ., m, 
        model = 'logit', cite = FALSE) %>%
  setx(strikeout = quantile(cyyoung$strikeout, 0.25), winpct = .75)%>%
  setx1(strikeout = quantile(cyyoung$strikeout, 0.75), winpct = .75) %>%
  sim(num = 10000) %>%
  get_qi(output, qi = "fd", xvalue = "x1") 

# Binding simulations
sim_fd <- cbind(output1, output2)

# Recording the quantities of interest from those simulations
# I save the median as the first difference as per Aengus' method we learned in lab
final_fd <- data.frame(winpct = as.factor(c(.5, .75)),
                       max = round(apply(sim_fd, 2, function(x) quantile(x, 0.975)),3),
                       min = round(apply(sim_fd, 2, function(x) quantile(x, 0.025)),3),
                       median = round(apply(sim_fd, 2, median), 3)) 

# Trying era
output1 <- cyyoung %>%
  zelig(data = ., m, 
        model = 'logit', cite = FALSE) %>%
  setx(era = quantile(cyyoung$era, 0.75), winpct = .5)%>%
  setx1(era = quantile(cyyoung$era, 0.5), winpct = .5) %>%
  sim(num = 10000) %>%
  get_qi(output, qi = "fd", xvalue = "x1")

# Scenario 2 (win percentage = 75%)
output2 <- cyyoung %>%
  zelig(data = ., m, 
        model = 'logit', cite = FALSE) %>%
  setx(era = quantile(cyyoung$era, 0.75), winpct = .75)%>%
  setx1(era = quantile(cyyoung$era, 0.5), winpct = .75) %>%
  sim(num = 10000) %>%
  get_qi(output, qi = "fd", xvalue = "x1") 

# Binding simulations
sim_fd <- cbind(output1, output2)

# Recording the quantities of interest from those simulations
# I save the median as the first difference as per Aengus' method we learned in lab
final_fd2 <- data.frame(winpct = as.factor(c(.5, .75)),
                       max = round(apply(sim_fd, 2, function(x) quantile(x, 0.975)),3),
                       min = round(apply(sim_fd, 2, function(x) quantile(x, 0.025)),3),
                       median = round(apply(sim_fd, 2, median), 3)) 

# Plotting
final_fd %>%
  ggplot(aes(x = winpct, y = median)) +
  geom_errorbar(aes(ymin=final_fd2$min, ymax=final_fd2$max), 
                width=0.1, color = "dodgerblue", size = 2, alpha = .5) +
  geom_errorbar(aes(ymin=min, ymax=max), 
                width=0.1, color = "red") +
  geom_point(color = "red", size = 3) +
  geom_point(aes(x = final_fd2$winpct, y = final_fd2$median), 
             color = "blue", size = 3) +
  geom_hline(aes(yintercept = 0), linetype = 2) +
  theme_bw() +
  ggtitle("First Difference between 25th and 75th Percentiles for Strikeouts and ERAs") +
  ylab("Difference in Probabilty of Winning Cy Young Award") +
  xlab("Win Percentage") +
  scale_y_continuous(breaks = seq(0, 5, .05)) +
  # Adding value of first differences as text using bquote()
  geom_text(label = bquote(.(final_fd$median)), size = 4,
            # Bringing lines closer together  to ease visualizationand shifting text to the right
            position = position_dodge(width = 5), hjust = -0.5) +
    geom_text(label = bquote(.(final_fd2$median)), aes(y = final_fd2$median), size = 4,
            # Bringing lines closer together  to ease visualization and shifting text to the right
            position = position_dodge(width = 5), hjust = -0.5) +
  # Adding labels for strikeouts and ERA lines
  geom_text(label = "Strikeouts", color = "red", size = 4, aes(x = 0.9, y = final_fd$median[2]), hjust = -0.5) +
  geom_text(label = "ERA", color = "dodgerblue", size = 4, aes(x = 0.9, y = final_fd2$median[2]), hjust = -1.7)

```

The blue points represent the difference in the probability of winning the award between an ERA of `r round(quantile(cyyoung$era, 0.75), 2)` and `r round(quantile(cyyoung$era, 0.25), 2)` and the red points represent the difference between `r round(quantile(cyyoung$strikeout, 0.75), 2)` strikeouts and `r round(quantile(cyyoung$strikeout, 0.25), 2)` strikeouts based on 10,000 simulations, and the bands represent 95% confidence intervals. Although the confidence intervals overlap each other and the opposite first difference, the difference in ERA seems to have a more important impact on the probabilty of winning the award at a 75% win rate than at a 50% win rate. The difference in strikeouts follows the same pattern and has an even larger impact than does ERA.

There is more we can say about ERA since the estimates are more accurate. Essentially, at a 50% win rate, a change in ERA of about 1 is estimated to make very little difference in the probabilty of winnin the award, but has the chance to make a difference of above 0.10. At a win  rate of 0.75, the same change in ERA is estimated to make a 0.15 difference, but has the chance to make a difference of between about .10 and 0.25.

## f. what assumptions of this model are violated by the variable cy?
The model assumes we have enough observations to precisely estimate parameters. Binary outcome variables contain less information per observation than other types of variables, therefore we need a much larger N to correctly estimate parameters. We also should technically see two winners per year but for some years there is only one winner recorded.

```{r}
table(cyyoung$cy, cyyoung$year)
```

In years 1984, 1987, 1989, and 1992 there is only one recorded winner when there should technically have been two. We have no data for 1981. I suspect all of these missing data are closing pitchers who would represent huge outliers had they been includedn in the estimation of our model. They don't have many wins or innings played and have very low ERAs usually.

The fact that only one two pitchers per year can win the award also violates the assumptions of the model, because there is zero probabilty that more than two pitchers can win the award per year. The probability of one pitcher winning depends on the probability of other pitchers winning. The outcomes are not independent of each other. Logistic regression assumes observations to be independent of each other.

## g. How might the exclusion of non-contenders affect our findings?
In reality, some of the pitchers that did not win the award in our dataset should technically have high probabilities of winning the award when compared to the true population. Our model cannot capture this however, since there are only comparisons between good pitchers. If only contenders are in the data, the effect we get is much more random. It is like predicting election outcomes if it is a close race.

A better dataset would include all starting pitchers, because they all have some probability of winning the award. Relief pitchers have very little chance of winning the award.

**Bonus: In Sample Counterfactual**
```{r}
# Creating counterfactual dataset with real data
counterfactual <- cyyoung

# Mutating ERA by subtracting 1 std dev.
counterfactual <- mutate(counterfactual, era = cyyoung$era - sd(cyyoung$era))

# Predicting fitted values with counterfactual dataset
fitted <- predict(logit.cy.5, newdata = counterfactual, type = "response")

# Calculating difference between original and counterfactual fitted values
fd <- fitted - cyyoung$m2_fit 

# Plotting count of differences and avg difference
ggplot(data.frame(fd), aes(x = fd)) +
  geom_dotplot() +
  geom_vline(aes(xintercept = mean(fd)), linetype = "dashed") +
  geom_text(label = bquote(.(round(mean(fd), 3))), aes(x = mean(fd) + 0.02, y = .75)) +
  geom_text(label = "Mean", aes(x = mean(fd) + 0.02, y = 0.8)) +
  ggtitle("First Differences of ERA and ERA + 1 Standard Deviation") +
  xlab("First Differences") +
  ylab("Count") +
  theme_bw()
```

The average difference in the real and counterfactual fitted values (i.e. `r round(mean(fd), 3)`) represents the simulated effect of decreasing ERA by one standard devation (i.e. `r round(sd(cyyoung$era), 3)`). This way of estimating first differences and portraying them makes it easily interpretable in a real world terms. The *observed value* rather than *average-case* approach allows us to draw more accurate predictions from the real data, rather than predictions that may be affected by holding our independent variables constant. When we visualize the differences this way, we can also see the frequency of those differences, which gives us a better idea of the uncertainty, and where we might expect the most likely effect. In the above dotplot we can witness a higher frequency at lower differences.

