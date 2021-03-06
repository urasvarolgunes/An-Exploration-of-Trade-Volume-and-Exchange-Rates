ETC1010 PROJECT
================
Uras Varolgunes, Nicholas Wong, Anuka Atapattu, Wai Chung Man, Naradha Bandara
11 Ekim 2017

An Exploration of Trade Volume and Exchange Rates
=================================================

Today, we are going to make a general analysis of some economic data for a group of countries with a more specific goal of finding a connection between trade volume and exchange rates.

We have data of the exchange rates for the currencies of selected countries in US dollars, export and import data of the US with other countries, GDP data from World Bank and export and import numbers for all countries.

We start by tidying our data. First, we will tidy the GDP data.

``` r
knitr::opts_chunk$set(echo = TRUE)
library(readr)
library(tidyverse)
library(readxl)
GDP <- read_csv("GDP.csv", skip = 4)
#First, gather the years. Then, use select to get the relevant columns. Lastly, we filter the countries and the time period we want to use for our analysis.
GDPtidy <- GDP %>%
  gather(as.character(1960:2016), key = "year", value = "GDP")%>%
  select(`Country Code`, year, GDP)

countries <- c("AUS", "CAN", "CHN", "GBR", "JPN")
GDP_final <- GDPtidy %>%
  filter(`Country Code` %in% countries) %>%
  filter(year > 1992, year<2016)

#We now tidy the exchange rate data.
USDrates <- read_csv("USDrates.csv")
USDrates_final<- USDrates %>%
  gather ("ZWD", "AUD", "CAD", "CNY", "EUR", "GBP", "JPY", key = "Currency Pair", value = "Exchange Rate") %>%
filter(`Currency Pair` != "ZWD") %>%
  filter(`Currency Pair` != "EUR") %>%
  filter(Year>1992)%>%
  arrange(Year)

#Tidy the export and import data of all countries. Choose the countries and the time period, then filter out  the percentage of GDP measure denoted by "PC_GDP" from the measure column. 
NX <- read_csv("ExportImport.csv")
NX_final <- NX %>%
  filter(LOCATION %in% countries) %>%
  select(LOCATION, SUBJECT, MEASURE, TIME, Value) %>%
  filter(TIME > 1992, TIME<2016) %>%
  filter(MEASURE== "PC_GDP")%>%
  spread(SUBJECT, Value)%>%
  arrange(TIME)

#Tidy the trade data of the US with other countries. Filter countries and year. Select the relevant columns for the analysis.
UStrade <- read_excel("UStrade.xlsx")
country <- c("Australia", "Canada", "China", "Japan", "United Kingdom")
UStrade_final <- UStrade %>%
  filter(`Partner Name` %in% country) %>%
  filter(Year > 1992) %>%
  select(`Partner Name`, Year, `Export (US$ Thousand)`, `Import (US$ Thousand)`, `Export Partner Share (%)`, `Import Partner Share (%)`)
```

Data Graphs
===========

We now take a general look at our data. Plot the GDP data against time.

``` r
library(ggplot2)
ggplot(GDP_final, aes(x = year, y = GDP, colour = `Country Code`)) +
  geom_point()+
  theme(axis.text.x=element_text(angle=90,hjust=1,vjust=0.5))
```

<img src="etcproject_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-3-1.png" style="display: block; margin: auto;" />

Graphing the GDP data shows a stable upward trend across 4 of the 5 countries. The difference between Australia, Canada, Japan and Great Britain can largely be explained by the relative size of the country. China's growth trend stands out, with rapid and sustained growth over the period.

``` r
ggplot(NX_final, aes(x = TIME, y = EXP, colour = LOCATION )) +
  geom_line() + geom_point()
```

<img src="etcproject_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-4-1.png" style="display: block; margin: auto;" />

Next we looked at the value of exports by the 5 countries over the time period. This data is expressed as a percentage of GDP so that the countries could easily be compared. There are a few things to note in this graph. Firstly, Canada's exports show strong growth in the 1990s and a correction back towards a growth rate consistent with Australia, Britain, and Japan. Secondly, China's exports grew incredibly quickly up to its peak in 2006-07. This is made more impressive by the fact that their GDP was also growing quickly during this period. Finally, the impact of the global financial crisis can clearly be seen on the graph between 2008 and 2009. We expect that during economic distress, exporters will become more risk averse and are likely to avoid exporting goods when the exchange rates are volatile.

``` r
ggplot(NX_final, aes(x = TIME, y = IMP, colour = LOCATION )) +
  geom_line() + geom_point()
```

<img src="etcproject_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-5-1.png" style="display: block; margin: auto;" />

The import data for the 5 countries shows the same general graph shapes for each individual country as the export data.

``` r
UStrade_final %>%
  ggplot(aes(x = Year, y = `Export (US$ Thousand)`, colour = `Partner Name` )) +
  geom_line() + geom_point()
```

<img src="etcproject_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-6-1.png" style="display: block; margin: auto;" />

``` r
UStrade_final %>%
  ggplot(aes(x = Year, y = `Import (US$ Thousand)`, colour = `Partner Name` )) +
  geom_line() + geom_point()
```

<img src="etcproject_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-7-1.png" style="display: block; margin: auto;" />

The last 2 graphs relate to Export and import data again, but specifically the USD value of America's exports to and imports from each of the 5 individual countries. From the exports data, America appears to export about the same amount of goods to Canada as it does to the other 4 countries combined. Considering the geographic proximity between Canada and the USA, this makes a lot of sense. The import data is a bit more interesting. Once again Canada is expected to have lots of trade with America, but the amount of Chinese imports has grown a staggering amount.

``` r
ggplot(USDrates_final, aes(x = Year, y = `Exchange Rate`, colour = `Currency Pair` )) +
  geom_line() + geom_point()
```

<img src="etcproject_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-8-1.png" style="display: block; margin: auto;" />

We also wanted to see how the value of the currency changed over time to see if there were any patterns similar to the GDP or export and import data. Here we graphed all the exchange rates relative to the US dollar. The exchange rates tell us how many units of the country's currency can be bought with 1 USD. The value of the Japanese Yen makes the Y-axis scale inappropriate for viewing the movement of the other currencies so we filtered it out to get a better scale.

``` r
USDrates_final %>%
filter(`Currency Pair` != "JPY") %>%
  ggplot(aes(x = Year, y = `Exchange Rate`, colour = `Currency Pair` )) +
  geom_line() + geom_point()
```

<img src="etcproject_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-9-1.png" style="display: block; margin: auto;" />

Removing the Yen allows us to see the movement of the Chinese Yuan clearly. Compared to the Yen, the Yuan appears incredibly stable up until 2005 and has relatively low fluctuation from 2005 onwards. This is because the central banks in China implemented a currency peg to the USD, meaning they would buy or sell excess Yuan in foreign exchange markets to keep the exchange rate stable. In 2005 China unpegged the currency and controlled its appreciation to its fair value to prevent the currency from spiking upwards instantly. (Although the graph is going downwards, it shows an appreciation of the Yuan because you can buy less units of Yuan with 1 USD)

``` r
USDrates_final %>%
filter(`Currency Pair` %in% c("AUD", "CAD", "GBP")) %>%
  ggplot(aes(x = Year, y = `Exchange Rate`, colour = `Currency Pair` )) +
  geom_line() + geom_point()
```

<img src="etcproject_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-10-1.png" style="display: block; margin: auto;" />

Filtering out the Yuan allows us to clearly see the exchange rate fluctuations of the remaining 3 currencies. It is quite interesting to see how similarly the AUD and CAD moved over the same time frame, considering how both currencies are freely traded and the strength of the AUD during the mining boom.

The Regression
==============

After analysing the overall data, we will go more specific and try to see if there is a connection between the exchange rates and the shares of countries in total imports. The analysis will be from the US perspective. So, we will look at the shares of countries in total US imports and their relative exchange rates with the US.

For this purpose, we quickly create the big data frame that we will use for the regression. Then we filter out the data for each country.

``` r
bigdata <- cbind(UStrade_final, USDrates_final[, 2:3])
bigdata_AUS <- filter(bigdata, `Partner Name`== "Australia")
bigdata_CAN <- filter(bigdata, `Partner Name`== "Canada")
bigdata_CHN <- filter(bigdata, `Partner Name`== "China")
bigdata_GBR <- filter(bigdata, `Partner Name`== "United Kingdom")
bigdata_JPN <- filter(bigdata, `Partner Name` == "Japan")

ggplot(data=bigdata_AUS, aes(x=`Exchange Rate`, y=`Import Partner Share (%)`)) + 
  geom_point() + ggtitle("Australia")+
  geom_smooth(method="lm", se=FALSE)
```

<img src="etcproject_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-11-1.png" style="display: block; margin: auto;" />

``` r

ggplot(data=bigdata_CAN, aes(x=`Exchange Rate`, y=`Import Partner Share (%)`)) + 
  geom_point() + ggtitle("Canada")+
  geom_smooth(method="lm", se=FALSE)
```

<img src="etcproject_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-11-2.png" style="display: block; margin: auto;" />

``` r

ggplot(data=bigdata_CHN, aes(x=`Exchange Rate`, y=`Import Partner Share (%)`)) + 
  geom_point() + ggtitle("China")+
  geom_smooth(method="lm", se=FALSE)
```

<img src="etcproject_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-11-3.png" style="display: block; margin: auto;" />

``` r

ggplot(data=bigdata_GBR, aes(x=`Exchange Rate`, y=`Import Partner Share (%)`)) + 
  geom_point() + ggtitle("United Kingdom")+
  geom_smooth(method="lm", se=FALSE)
```

<img src="etcproject_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-11-4.png" style="display: block; margin: auto;" />

``` r

ggplot(data=bigdata_JPN, aes(x=`Exchange Rate`, y=`Import Partner Share (%)`)) + 
  geom_point() + ggtitle("Japan")+
  geom_smooth(method="lm", se=FALSE)
```

<img src="etcproject_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-11-5.png" style="display: block; margin: auto;" />

``` r

#We save the models and take a look at the R-Squared values.
mod_AUS <- lm(`Import Partner Share (%)`~ `Exchange Rate`, data=bigdata_AUS)
mod_CAN <- lm(`Import Partner Share (%)`~ `Exchange Rate`, data=bigdata_CAN)
mod_CHN <- lm(`Import Partner Share (%)`~ `Exchange Rate`, data=bigdata_CHN)
mod_GBR <- lm(`Import Partner Share (%)`~ `Exchange Rate`, data=bigdata_GBR)
mod_JPN <- lm(`Import Partner Share (%)`~ `Exchange Rate`, data=bigdata_JPN)

tibble(Country= c("Australia","Canada","China","UK","Japan"), Rsquared=c(summary(mod_AUS)$r.squared, summary(mod_CAN)$r.squared, summary(mod_CHN)$r.squared, summary(mod_JPN)$r.squared, summary(mod_JPN)$r.squared))
# A tibble: 5 x 2
    Country   Rsquared
      <chr>      <dbl>
1 Australia 0.59554976
2    Canada 0.64361926
3     China 0.42797846
4        UK 0.07686201
5     Japan 0.07686201
```

After the examination of the R-Squared values we conclude that almost 60 percent of the variation in the shares of Australia and Canada in the total imports of the US is explained by the fluctuations in the exchange rates, while the value is around 40 percent for China and 8 percent for the UK and Japan.

The reason that values differ for each country might be caused by the limitations of the model. We ignored time dependence and based our regression on one explanatory variable for simplicity. Even then, the model is actually useful for explaining the cases for Australia and Canada.
