TidyTuesday34 - Thanksgiving
================
Arthur Cheib

``` r
library(tidyverse)
library(ggthemes)
```

Read and summary file

``` r
url <- "https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2018-11-20/thanksgiving_meals.csv"

thanks_dataset <- read_csv(url)
```

Defining the scope of analysis: \* Do you celebrate Thanksgiving? \* Which type of pie is typically served at your Thanksgiving dinner? Please select all that apply. \* Do you typically pray before or after the Thanksgiving meal? \* Age \* How much total combined money did all members of your HOUSEHOLD earn last year?

``` r
thx_escope <- thanks_dataset %>% 
  select(1,2, pie1:pie13, prayer, age, family_income) %>%
  filter(celebrate == "Yes")

thx_table <- thx_escope %>% 
  select(prayer, family_income) %>%
  group_by(prayer, family_income) %>% 
  summarize(TOTAL = n()) %>%
  na.omit() %>%
  spread(key = family_income, value = TOTAL) 
  
thx_table[, 2:12] <- round((thx_table[, 2:12]/rowSums(thx_table[, 2:12])), digits = 3)*100

thx_table %>%
  gather(key = family_income, value = TOTAL, -prayer) %>% 
  mutate(TOTAL = case_when(prayer == "No" ~ -TOTAL,
                           TRUE ~ TOTAL),
         family_income = factor(family_income, levels = c("Prefer not to answer", "$0 to $9,999", "$10,000 to $24,999",
                                                          "$25,000 to $49,999", "$50,000 to $74,999", "$75,000 to $99,999",
                                                          "$100,000 to $124,999", "$125,000 to $149,999", "$150,000 to $174,999",
                                                          "$175,000 to $199,999", "$200,000 and up"))) %>% 
  ggplot(aes(x = family_income, y = TOTAL, fill = prayer)) +
  geom_col() +
  geom_label(aes(label = str_c(format(abs(TOTAL)), "%")), fill = "white") +
  coord_flip() +
  scale_y_continuous(limits = c(-20, 20), labels = abs) +
  theme_economist() +
  labs(title = "PERCENTAGE OF RESPONDENTS THAT \n PRAYS IN THANKSGIVING DINNERS",
       subtitle = "Displayed by family combined income",
       y = "",
       x = "") +
  theme(legend.position = "none",
        plot.title = element_text( size = 14))
```

![](first_go_tt34_files/figure-markdown_github/unnamed-chunk-3-1.png)

``` r
thx_escope %>% 
  select(-prayer, - family_income, -celebrate) %>% 
  gather(key = pies, value = types, -age, -id) %>%
  na.omit() %>% 
  group_by(types) %>% 
  mutate(total = n()) %>%
  filter(total > 100) %>%
  ggplot(aes(age, fct_reorder(types, total))) +
  geom_jitter(aes(color = types)) +
  theme_wsj() +
  labs(title = "MOST PREFERRED PIES IN THANKSGIVING DINNERS",
       subtitle = "by age intervals",
       y = "",
       x = "") +
  theme(legend.position = "none",
        plot.title = element_text( size = 16))
```

![](first_go_tt34_files/figure-markdown_github/unnamed-chunk-4-1.png)

``` r
save(... = thx_escope, file = "thx_dataset.RData")
ggsave("prayers.png")
```

    ## Saving 7 x 5 in image

``` r
ggsave("pref_pies.png")
```

    ## Saving 7 x 5 in image
