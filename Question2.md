## Introduction

Presented with the palmerpenguins dataset, I begin a data analysis to investigate the relationship between body mass and flipper length, considering the influence of species. The first step in data exploration is to install and load all the relevant packages. These packages include ggplot2 (to produce graphs), janitor (to help clean the dataset), dplyr (which allows us to use the pipe function), and palmerpenguins (which provides the dataset). This is done below:

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

The next step is to investigate the raw data set that is already termed penguins_raw from the palmerpenguins package. By using the head() function, the dataset can be observed such that the aspects that require cleaning can be identified. 
```{r}
head(penguins_raw)
```

From the output, there are multiple noticable aspects of this dataset that need to be cleaned before further analysis. These are:
- The column names need to be changed so they are machine readable (lower and snake case) - this is done through the function 'clean_column_names'. 
- Any empty column or row should be removed - this is done through the function remove_empty_column_names. 
- The final three columns are not relevant and should be removed from the dataset - this is done through the functions select()
- The species names should be shortened - this is done through the function shorten_species()
- In order to only include the data of interest, the other columns can be removed by using subset_columns()

All of these cleaning functions are done below through a pipeline, defining a new object 'penguins_clean' which will be the newly cleaned dataset:

```{r}

penguins_clean <- penguins_raw %>%
    clean_column_names() %>%
    shorten_species() %>%
    remove_empty_columns_rows() %>%
    select(-starts_with("Delta")) %>%
    select(-comments) %>%
    subset_columns(c("species", "body_mass_g", "flipper_length_mm"))
 
head(penguins_clean)
```

Now that the dataset has been cleaned, a graph can be plotted to observe the data. Using ggplot, a scatterplot can be made to show the relationship between flipper length and body mass. The code for this is below:
```{r - Plottoing the exploratory figure}
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

Further I hypothesise that the species influences the relationship between body mass and flipper length as the trend of points seems to differ slightly between each colour distinguished species. An interaction between a categorical value such as 'species' and a covariate such as 'body mass' would determine if the species does influence this relationship. Therefore the hypotehsis can be set as:
Null hypothesis (H0): There is no interaction between species and body mass
Alternative hypothesis (H1): There is a non-zero effect of the interaction between species and body mass.

## Statistical Methods

The appropriate statistical test to investigate this relationship would be finding the coefficient of determination (R^2) in a linear model between these two variables. An F-test of overall significance can then be used to determine whether the found relationship is truly significant. All of these tests are conducted through the linear model function (lm()):

```{r Statistics - Linear Model}
linear_model <- lm(data = penguins_clean, flipper_length_mm ~ body_mass_g)
summary(linear_model)
```

First to investigate the influence of species on this relationship. I run new linear models to find the R^2 values for each different species.

Next, in order to test the significance of the influence of species for this relationship, I can use the r package "car" which allows for the use of ANCOVA tests. The interaction between body mass and species in this model well help determine if the relationship between body mass and flipper length is different between species.

This is done below:

```{r - ANCOVA}

# Creating linear models for each individual penguin species can be achieved by subseting the data further into species specific datasets from which the models can be made. This is done below:

# Creating a linear model for just the Adelie penguin data
adelie_data <- penguins_clean %>% 
  filter_by_species("Adelie")
linear_adelie <- lm(data = adelie_data, flipper_length_mm ~ body_mass_g)
summary(linear_adelie)
# This produces an R^2 value of 0.214 at a significant P-value of 1.343e-09

# Creating a linear model for just the Chinstrap penguins
chinstrap_data <- penguins_clean %>% 
  filter_by_species("Chinstrap")
linear_chinstrap <- lm(data = chinstrap_data, flipper_length_mm ~ body_mass_g)
summary(linear_chinstrap)
# This produces an R^2 value of 0.4027 at a significant P-value of 3.748e-09

# Creating a linear model for just the Gentoo penguins
gentoo_data <- penguins_clean %>% 
  filter_by_species("Gentoo")
linear_gentoo <- lm(data = gentoo_data, flipper_length_mm ~ body_mass_g)
summary(linear_gentoo)
# This produces an R^2 value of 0.4896 at a significant P-value of less than 2.23-16


# Below the car package is installed an used to conduct an ANCOVA test for the interaction between species and this linear relationship
install.packages("car")
library(car)
ancova_model <- aov(flipper_length_mm ~ body_mass_g + species, data = penguins_clean)
summary(ancova_model)
```
## Results & Discussion

The summary table from that linear regression stats test provides a lot of information on the plotted relationship. However, for the sake of the first hypothesis, the most important value is the adjusted R-squared. The adjusted R-sqaured value accounts for any overfitting from the regular R-squared. R-squared is a measure of how well the independent variable in the linear regression explains the variability of the dependent variable. In this context, 'Adjusted R-squared' is used to measure how much of the variation in flipper length is explained by body mass. R-squared values are bound between 0 and 1, higher values indicate stronger relationships. 

The results of the linear regression produce an Adjusted R-squared value of 0.7583. The high value demonstrates a very strong relationship, suggesting that body mass explains a lot of the variation in flipper length. Additionally, the significance of this value is supported by the provided F-test which produces a p-value < 2.2e-16 which is much smaller than 0.05. This allows us to reject the null hypothesis and claim that the relationship is not likely due to chance.

For full visibility, the initial graph is plotted again, this time only focussing on the tested relationship. The results of the stats test are also projected. The code for the graph is shown below:

```{r Plotting linear model results}
ggplot(data = penguins_clean, aes(x = body_mass_g, y = flipper_length_mm)) +
  geom_point() +
  labs(title = "Body Mass plotted against Flipper length in all penguins", x = "Body Mass (g)", y = "Flipper Length (mm)") +
  geom_smooth(method = "lm", se = TRUE, aes(group = 1), color = "red") +
  theme_bw() +
  geom_label(aes(x = 5500, y = 188), hjust = 0, label = paste("Adj R2 = ",signif(summary(linear_model)$adj.r.squared, 5),
                                               "\nIntercept =",signif(linear_model$coef[[1]],5 ),
                                               " \nSlope =",signif(linear_model$coef[[2]], 5),
                                               " \nP =",signif(summary(linear_model)$coef[2,4], 5)))

```
The second hypothesis is tested using an ANCOVA statistical test where the interaction between species and body mass is measured. The ANCOVA summary table is interpreted by regarding the significance of the p value of the interaction (in this case, the interaction is considered in the 'species' row). The p value of the interaction < 2e-16. As this value is signficant, there is evidence to suggest that the relationship between body mass and flipper length is different across different species. As such, the null hypothesis can be rejected. 

Likewise, to give a visual demonstration of this result, the graph is plotted once more, this time showing the relationships for each individual species.

```{r Plotting ANCOVA results} 
ggplot(data = penguins_clean, aes(x = body_mass_g, y = flipper_length_mm, colour = species)) +
  geom_point() +
  labs(title = "Body Mass plotted against Flipper length by species", x = "Body Mass (g)", y = "Flipper Length (mm)") +
  geom_smooth(method = "lm", aes(group = species, color = species), se = TRUE) +
  theme_bw() +
  geom_label(aes(x = 5500, y = 182), hjust = 0, label = paste("Adj R2 = ",signif(summary(linear_adelie)$adj.r.squared, 5),
                                               " \nP =",signif(summary(linear_adelie)$coef[2,4], 5)),
             inherit.aes = FALSE, colour = "red") +
  geom_label(aes(x = 5500, y = 190), hjust = 0, label = paste("Adj R2 = ",signif(summary(linear_chinstrap)$adj.r.squared, 5),
                                               " \nP =",signif(summary(linear_chinstrap)$coef[2,4], 5)),
             inherit.aes = FALSE, colour = "green") +
  geom_label(aes(x = 5500, y = 198), hjust = 0, label = paste("Adj R2 = ",signif(summary(linear_gentoo)$adj.r.squared, 5),
                                               " \nP =",signif(summary(linear_gentoo)$coef[2,4], 5)),
             inherit.aes = FALSE, colour = "blue")
```

### Conclusion

## This leads to the conclusion that body mass explains variation in flipper length and those penguins with larger body mass are more likely to have longer flippers. This conclusion would be expected as greater body masses usually indicate larger birds and given that both of these measures are of morphological features, it makes sense that flipper length will increase as body mass increases. This strong positive relationship is observed in all 3 penguin species. However, the results of the ANCOVA test suggest that the there is a significant interaction between species and this relationship. This suggests that the type of species a penguin is, will have a significant influence on the relationship between body mass and flipper length. Likewise, this conclusion would also be expected as different species are expected to have different developmental patterns and growth is likely to be more similar within species than between. 
