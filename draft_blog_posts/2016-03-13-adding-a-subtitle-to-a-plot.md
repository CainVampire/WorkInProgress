---
layout: post
title: "Adding a sub-title to a ggplot2 plot"
date: 2016-03-13
---


## Plot the age distribution in British Columbia, 1971-2015

### data

Source file:  (./data_source/cansim-0510001-eng-8333626124709750278.csv)

The CANSIM extract was cleaned up to produce a wide table that contained only total population by age group. (The source table also contains columns by male and female.) This table was saved as `data_agedistrib01.csv`.

Note that the table originally tabulated the last category as "90 years and over"; starting with 2001 this category was expanded to single years 90 through 99, and a "100 years and over" category was added. Since we want to see the change over time, the post-2000 detail has been omitted in the source data.

The table is then melted into long format.


```{r, echo=FALSE, message=FALSE}

# load the necessary packages
# data manipulation
library(tidyr)
library(dplyr)
# graphics
library(ggplot2)
library(scales)
# tables
library(xtable)

# or, for the development version of ggplot2...
# install.packages("devtools")
devtools::install_github("hadley/ggplot2")


```

#### Read and summarize the data

```{r, echo=FALSE, message=FALSE}

# read the data
data_agedistrib01 <- read.csv("./data_source/data_agedistrib01.csv", header = TRUE)

# rename the columns
colnames(data_agedistrib01) <- c("year", "All.ages", 
                                 "00", "01", "02", "03", "04", 
                                 "05", "06", "07", "08", "09",
                                 seq(10, 89, by = 1), "90+")

# turn the wide version of data_medianage into long format, add pctage column
data_agedistrib01_long <- data_agedistrib01 %>%
  gather(age, popul, -year) %>%
  filter(age != "All.ages") %>%
  group_by(year, age) %>%
  summarise(popul = sum(popul)) %>%
  mutate(year_pct = (popul / sum(popul) * 100)) 

head(data_agedistrib01_long)

# create a version of the table with only the annual total population (i.e. no age breakdown)
data_totalpop_19712015 <- data_agedistrib01 %>%
  gather(age, popul, -year) %>%
  filter(age == "All.ages")

head(data_totalpop_19712015)

```


#### B.C. annual population total plot

A plot showing the on-going increase in population in British Columbia, from 1971 through 2015.

The first version of the plot is a basic version of the plot. The second version adds an assortment of formatting:

* axis length and grid specifications
* axis titles
* chart title

Note that the X axis scale is such that it follows the five year increments of the Census of Canada.

The total population of British Columbia has increased from 2.2 million in 1971 to 4.7 million in 2015. The population surpassed 3 million in 1986, and 4 million in 1999.

```{r, echo=FALSE, message=FALSE}

# basic
plot_totalpop <- ggplot(data_totalpop_19712015, aes(x=year, y=popul, colour=age)) +
  geom_line(size=1.5)

# basic plot
plot_totalpop

# fix the formatting
plot_totalpop2 <- plot_totalpop +
  theme_classic() +
  ggtitle("British Columbia, Total population, 1971-2015") +
  ylab("population") +
  scale_x_continuous(breaks = seq(1971, 2016, by = 5)) +
  scale_y_continuous(limits = c(0, 5000000), breaks = seq(0, 5000000, by = 1000000),
                     labels = comma) +
  # remove the legend (not much point when there's only one line!)
  theme(legend.position="none")

plot_totalpop2

# save the plot as a png file
png("plot_totalpop2.png", width=1024, height=768, res=120)
plot(plot_totalpop2)
dev.off()

```

#### Now with a subtitle


```{r}

# code taken from
# http://rud.is/b/2016/03/12/ggplot2%E3%81%A7%E5%AD%97%E5%B9%95-subtitles-in-ggplot2/
#
# NOTE: "gg" is the existing plot object

library(grid)
library(gtable)

ggplot_with_subtitle <- function(gg, 
                                 label="", 
                                 fontfamily=NULL,
                                 fontsize=10,
                                 hjust=0, vjust=0, 
                                 bottom_margin=5.5,
                                 newpage=is.null(vp),
                                 vp=NULL,
                                 ...) {
 
  if (is.null(fontfamily)) {
    gpr <- gpar(fontsize=fontsize, ...)
  } else {
    gpr <- gpar(fontfamily=fontfamily, fontsize=fontsize, ...)
  }
 
  subtitle <- textGrob(label, x=unit(hjust, "npc"), y=unit(hjust, "npc"), 
                       hjust=hjust, vjust=vjust,
                       gp=gpr)
 
  data <- ggplot_build(gg)
 
  gt <- ggplot_gtable(data)
  gt <- gtable_add_rows(gt, grobHeight(subtitle), 2)
  gt <- gtable_add_grob(gt, subtitle, 3, 4, 3, 4, 8, "off", "subtitle")
  gt <- gtable_add_rows(gt, grid::unit(bottom_margin, "pt"), 3)
 
  if (newpage) grid.newpage()
 
  if (is.null(vp)) {
    grid.draw(gt)
  } else {
    if (is.character(vp)) seekViewport(vp) else pushViewport(vp)
    grid.draw(gt)
    upViewport()
  }
 
  invisible(data)
 
}



# -----------------

# set the name of the current plot object to "gg"
gg <- plot_totalpop

# define the subtitle text
subtitle <- "The population of B.C. has risen from 2.2 million in 1971 to 4.7 million in 2015"
 
ggplot_with_subtitle(gg, subtitle,
                     fontfamily="FranklinGothic-Book",
                     bottom_margin=20, lineheight=0.9)



```

#### Alternate approach to the sub-title, using the dev version of ggplot2

(the dev version of ggplot2 is not yet ready for prime time)


-30-
