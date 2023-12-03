## Introduction
### The first step in data exploration is to install and load all the relevant packages. These packages include ggplot2 (to produce graphs), janitor (to help clean the dataset), dplyr (which allows us to use the pipe function), and palmerpenguins (which provides the dataset). This is done below:

```{r Data Exploration}
install.packages("ggplot2")
install.packages("dplyr")
install.packages("palmerpenguins")
install.packages("janitor")

library(ggplot2)
library(dplyr)
library(palmerpenguins)
library(janitor)
```
### The next step is to investigate the raw data set that is already termed penguins_raw from the palmerpenguins package. By using the head() function, the dataset can be observed such that the aspects that require cleaning can be identified. 
```
head(penguins_raw)
```
### From the output, there are multiple noticable aspects of this dataset that need to be cleaned before further analysis. These are:
- The column names need to be changed so they are machine readable (lower and snake case) - this is done through the function 'clean_column_names'. 
- Any empty column or row should be removed - this is done through the function remove_empty_column_names. 
- The final three columns are not relevant and should be removed from the dataset - this is done through the functions select()
- The species names should be shortened - this is done through the function shorten_species()

### All of these cleaning functions are done below through a pipeline, defining a new object 'penguins_clean' which will be the newly cleaned dataset:
```
penguins_clean <- penguins_raw %>%
    clean_column_names() %>%
    shorten_species() %>%
    remove_empty_columns_rows() %>%
    select(-starts_with("Delta")) %>%
    select(-comments)
 
head(penguins_clean)
```
### Now that the dataset has been cleaned, a graph can be plotted to observe the data. Using ggplot, a scatterplot can be made to show the relationship between flipper length and body mass. The code for this is below:
```
ggplot(data = penguins_clean, aes(x = body_mass_g, y = flipper_length_mm, colour = species)) +
  geom_point() +
  labs(title = "Body Mass plotted against Flipper length in all penguins", x = "Body Mass (g)", y = "Flipper Length (mm)") +
  geom_smooth(method = "lm", se = TRUE, aes(group = 1), color = "black") +
  theme_bw()
```

## Hypothesis

Based on the appearance of the exploratory graph and the plotted line of best fit, I formulate the hypothesis that flipper length increases with body mass, in other words, body mass explains some of the variation in flipper. This hypothesis supported by the apparent directly proportional relationship. This can be tested by investigating if there is a positive linear relationship between these variables. As such the hypothesis follows: 
Null hypothesis (H0): R\^2 = 0 - A coefficient of determination = 0 would suggest that body mass explains none of the variation in flipper length. 
Alternative hypothesis (H1): R\^2 \> 0 - A coefficient of determination \> 0 would suggest that body mass explains a non-zero amont of the variation in flipper length.

## Statistical Methods

```{r Statistics}
```
### The appropriate statistical test to investigate this relationship would be finding the coefficient of determination (R^2) in a linear model between these two variables. An F-test of overall significance can then be used to determine whether the found relationship is truly significant. All of these tests are conducted through the linear model function (lm()), 
```
linear_model <- lm(data = penguins_clean, flipper_length_mm ~ body_mass_g)
summary(linear_model)

x <- penguins_clean$body_mass_g
y <- penguins_clean$flipper_length_mm
correlation <- cor.test(x, y)
print(correlation)
```

## Results & Discussion

```{r Plotting Results}

# The summary table from that stats test provides a lot of information on the plotted relationship. However, for the sake of this hypothesis, the most important value is the adjusted R-squared. The adjusted R-sqaured value accounts for any overfitting from the regular R-squared. R-squared is a measure of how well the independent variable in the linear regression explains the variability of the dependent variable. In this context, 'Adjusted R-squared' is used to measure how much of the variation in flipper length is explained by body mass. R-squared values are bound between 0 and 1, higher values indicate stronger relationships. 

# The results of the stat test produce an Adjusted R-squared value of 0.7583. The high value demonstrates a very strong relationship, suggesting that body mass explains a lot of the variation in flipper length. Additionally, the significance of this value is supported by the provided F-test which produces a p-value < 2.2e-16 which is much smaller than 0.05. This allows us to reject the null hypothesis and claim that the relationship is not likely due to chance.

# For full visibility, the initial graph is plotted again, this time only focussing on the tested relationship. The results of the stats test are also projected. The code for the graph is shown below:

ggplot(data = penguins_clean, aes(x = body_mass_g, y = flipper_length_mm)) +
  geom_point() +
  labs(title = "Body Mass plotted against Flipper length in all penguins", x = "Body Mass (g)", y = "Flipper Length (mm)") +
  geom_smooth(method = "lm", se = TRUE, aes(group = 1), color = "black") +
  theme_bw() +
  geom_label(aes(x = 5000, y = 190), hjust = 0, label = paste("Adj R2 = ",signif(summary(linear_model)$adj.r.squared, 5),
                                               "\nIntercept =",signif(linear_model$coef[[1]],5 ),
                                               " \nSlope =",signif(linear_model$coef[[2]], 5),
                                               " \nP =",signif(summary(linear_model)$coef[2,4], 5)))


```

## Conclusion

### This leads to the conclusion that body mass explains variation in flipper length and those penguins with larger body mass are more likely to have longer flippers. This conclusion would be expected as greater body masses usually indicate larger birds and given that both of these measures are of morphological features, it makes sense that flipper length will increase as body mass increases.
