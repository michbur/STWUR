---
layout: post
title: "Skąd się biorą STWURy"
modified:
author: michal
categories: blog
excerpt:
tags: []
image:
  feature:
date: 2017-03-24
output:
  md_document:
    variant: markdown_github
---

### ggmap i dane z Google Analytics

Nasza strona, [stwur.pl](stwur.pl) istnieje już kilka miesięcy i od początku swojego istnienia cieszy się spory zainteresowaniem miłośników R z całego świata. Są dane, warto je zwizualizować, zwłaszcza, że pasują do tematyki naszego następnego spotkania [I ty możesz zostać kartografem!](https://www.meetup.com/Wroclaw-R-Users-Group/events/238204653/). W tej notce skupimy się na pokazaniu ilu użytkowników ze wszystkich krajów świata weszło na naszą stronę od stycznia do marca.


```r
library(ggplot2)
library(ggmap)
library(maps)
library(mapdata)
library(dplyr)
library(reshape2)
library(RColorBrewer) # potrzebujemy ladnych kolorow

# o tym jak pobierać dane z google analytics innym razem - teraz skupimy się na samej wizualizacji
dat <- read.csv("https://raw.githubusercontent.com/STWUR/gdzie-mieszkaja-STWURy/master/STWURy.csv")

# nazwy UK i USA trzeba dostosowac do nazw z pakietu mapdata
summaries_country <- group_by(dat, country) %>% 
  summarise(sessions = sum(sessions),
            users = sum(users),
            newUsers = sum(newUsers)) %>% 
  mutate(region = factor(country, labels = c("Denmark", "Finland", "Germany", "Iceland", 
                                             "Ireland", "Italy", "Netherlands", "Norway", 
                                             "Poland", "Slovenia", "Spain", "Sweden", 
                                             "Switzerland", "UK", "USA"))) %>% 
  select(-country)

# tworzymy mape swiata
world_map <- map_data("world")

other_countries <- unique(world_map[["region"]])[!(unique(world_map[["region"]]) %in% summaries_country[["region"]])]

# jeszcze dluga droga przed STWURem - w tylu krajach o nas nie slyszeli!
print(length(other_countries))
```

```
## [1] 237
```

```r
# dodajmy informacje, ze mamy 0 uzytkownikow w kazdym z krajow ze zmiennej other_countries
country_full <- rbind(summaries_country, 
                  data.frame(sessions = 0,
                             users = 0,
                             newUsers = 0,
                             region = other_countries)) %>% 
  mutate(usersF = cut(users, c(0, 1, 10, 30, 550), include.lowest = TRUE))
# tworzymy zmienna dyskretna userF, aby mapa wygladala ladniej

# to nasi uzytkownicy z calego swiata
ggplot(country_full, aes(map_id = region)) + 
  geom_map(aes(fill = usersF), map = world_map) +
  expand_limits(x = world_map$long, y = world_map$lat) +
  theme_bw() +
  scale_fill_manual("STWURy", values = brewer.pal(5, "GnBu"))
```

![plot of chunk STWUR_map](./figure/STWUR_map-1.png)

```r
# skupmy sie na Europie

ggplot(country_full, aes(map_id = region)) + 
  geom_map(aes(fill = usersF), map = world_map) +
  expand_limits(x = c(-10, 35), y = c(35, 70)) +
  theme_bw() +
  scale_fill_manual("STWURy", values = brewer.pal(5, "GnBu"))
```

![plot of chunk STWUR_map](./figure/STWUR_map-2.png)