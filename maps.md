---
title: "Maps using the AmericasBarometer"
output:
  html_document:
    toc: true
    toc_float: true
    collapsed: false
    number_sections: false
    toc_depth: 1
    code_download: true
    theme: flatly
    df_print: paged
    self_contained: no
    keep_md: yes
editor_options: 
  markdown: 
    wrap: sentence
---



# Introduction

In this section, we will introduce a very used tool nowadays: presentation of data in maps.
We will see a way to calculate average or percentages of some variable of interest and present variation by country in a map with different colors of a palette.

An example of this type of figures is presented in this [tweet](https://twitter.com/participacionpc/status/1524478511788052480?s=21&t=Xa3pZLkl349NzhgneJhfPQ) that LAPOP´s research affiliate in Ecuador [Participación Ciudadana](https://www.participacionciudadana.org/web/) publishes.

![](twitter_FSgJH19XwAExMgt.jpg){width="454"}

This map shows the percentage of citizens that justifies an executive coup in each country using the 2021 round of the AmericasBarometer.
A more intense red means a higher percentage and a pastel color indicates a lower percentage.

In this section, we will se how to replicate this type of map, for which we have to produce information form the dataset of the AmericasBarometer.

# About the dataset

The data we are going to use should be cited as follows: Source: AmericasBarometer by the Latin American Public Opinion Project (LAPOP), wwww.LapopSurveys.org.
You can download the data freely [here](http://datasets.americasbarometer.org/database/login.php).

It is recommended to clean the Environment before starting this section.
In this document, a database in RData format is again loaded.
This format is efficient in terms of storage space.
This database is hosted in the "materials_edu" repository of the LAPOP account on GitHub.
Using the `rio` library and the `import` command, you can import this database from this repository, using the following code.
In this code, we do not elimiate observations from Canada or the United States, countries that do not have information in the map, but that have data about executive coups in the dataset.


```r
library(rio)
lapop21 = import("https://raw.github.com/lapop-central/materials_edu/main/lapop21.RData") 
```

To reproduce the map, we have to calculate the percentage that justifies an executive coup by country.
Figure 1.7 of the 2021 report [The Pulse of Democracy](https://www.vanderbilt.edu/lapop/ab2021/2021_LAPOP_AmericasBarometer_2021_Pulse_of_Democracy.pdf) shows this information.

![](Figure1.7.png){width="579"}

In the section about [confidence intervals](https://arturomaldonado.github.io/BarometroEdu_Web_Eng/IC.html) we see how to build this information using the 2021 dataset of the AmericasBarometer.

# Preparing the data

The variable to replicate this figure is "jc15a" that is worded: Do you believe that when the country is facing very difficult times it is justifiable for the president of the country to close the Congress/Parliament and govern without Congress/Parliament?
The options to answer are:

1.  Yes, it is justified

2.  No, it is not justified

To calculate the percentage, we have to transform the variable in such a way that those who justify to close the congress are assigned a value of 100 and those who do not, the value of 0.
This transformation is saved in a new variable "jc15ar".


```r
lapop21$jc15ar = car::recode(lapop21$jc15a, "1=100; 2=0")
table(lapop21$jc15ar)
```

```
## 
##     0   100 
## 17360  6951
```

Then, we have to calculate the percentage of people who tolerate an executive coup for each country.
When we load the dataset of the AmericasBarometer, variables are generally load as type "numeric" (num in the language of R).

To work with this variable, we have to transform the variable "pais" as a categorical variable (factor in the language of R).
We do this with the command `as.factor`.
We then label this variable with the command `levels`.


```r
lapop21$pais = as.factor(lapop21$pais)
levels(lapop21$pais) = c("Mexico", "Guatemala", "El Salvador", "Honduras",
                        "Nicaragua","Costa Rica", "Panama", "Colombia", 
                        "Ecuador", "Bolivia", "Peru", "Paraguay", "Chile",
                        "Uruguay", "Brazil", "Argentina", "Dom. Rep.",
                        "Haiti", "Jamaica", "Guyana","United States", "Canada")
table(lapop21$pais)
```

```
## 
##        Mexico     Guatemala   El Salvador      Honduras     Nicaragua 
##          2998          3000          3245          2999          2997 
##    Costa Rica        Panama      Colombia       Ecuador       Bolivia 
##          2977          3183          3003          3005          3002 
##          Peru      Paraguay         Chile       Uruguay        Brazil 
##          3038          3004          2954          3009          3016 
##     Argentina     Dom. Rep.         Haiti       Jamaica        Guyana 
##          3011          3000          3088          3121          3011 
## United States        Canada 
##          1500          2201
```

# Executive coups by country

In R there are several ways to reach the same results.
To calculate the percentage of people who justify an executive coup, we follow the same procedure when we calculate confidence intervals.
We use the library `Rmisc` and the command `group.CI`.
We save this infomatiomn in a table "coup".


```r
library(Rmisc)
```

```
## Loading required package: lattice
```

```
## Loading required package: plyr
```

```r
coup = group.CI(jc15ar~pais, lapop21)
coup
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["pais"],"name":[1],"type":["fct"],"align":["left"]},{"label":["jc15ar.upper"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["jc15ar.mean"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["jc15ar.lower"],"name":[4],"type":["dbl"],"align":["right"]}],"data":[{"1":"Mexico","2":"33.88284","3":"30.312500","4":"26.742162"},{"1":"Guatemala","2":"40.41369","3":"36.687307","4":"32.960926"},{"1":"El Salvador","2":"51.78163","3":"48.068670","4":"44.355713"},{"1":"Honduras","2":"25.80600","3":"22.660819","4":"19.515638"},{"1":"Nicaragua","2":"34.61243","3":"31.111111","4":"27.609797"},{"1":"Panama","2":"31.20785","3":"28.962444","4":"26.717039"},{"1":"Colombia","2":"37.61174","3":"35.041447","4":"32.471157"},{"1":"Ecuador","2":"34.35553","3":"31.944444","4":"29.533359"},{"1":"Bolivia","2":"34.56732","3":"32.082414","4":"29.597507"},{"1":"Peru","2":"46.35270","3":"43.805613","4":"41.258529"},{"1":"Paraguay","2":"37.01489","3":"34.459459","4":"31.904033"},{"1":"Chile","2":"18.52143","3":"16.544118","4":"14.566804"},{"1":"Uruguay","2":"10.03646","3":"8.552632","4":"7.068802"},{"1":"Brazil","2":"26.05857","3":"23.862069","4":"21.665568"},{"1":"Argentina","2":"15.62400","3":"13.795620","4":"11.967243"},{"1":"Dom. Rep.","2":"27.81437","3":"25.495959","4":"23.177545"},{"1":"Haiti","2":"48.54708","3":"44.042553","4":"39.538028"},{"1":"Jamaica","2":"32.99455","3":"30.583215","4":"28.171877"},{"1":"United States","2":"15.17602","3":"13.444816","4":"11.713610"},{"1":"Canada","2":"40.35953","3":"38.324989","4":"36.290445"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

The object "coup" save the information of the mean (it is, the percentage) of people who justify an executive coup by country.

Other option is to use the command `compmeans` from the library `descr`.
It calculates the mean, the number of observations and the standard deviation of a variable by groups of other variable.
It also allows to make calculation including survey weights.

These calculations are saved in a data frame "table" with the command `as.data.frame`.
Then, we label the columns with the command `colnames` and we include a column with the names of countries and, finally, we delete the last row, where the mean for the total is calculated.
We do not require this information.

This procedure includes rows for countries where there is no data, such as Costa Rica and Guyana.


```r
library(descr) 
table = as.data.frame(
  compmeans(lapop21$jc15ar, lapop21$pais, lapop21$weight1500, plot=FALSE))
```

```
## Warning in compmeans(lapop21$jc15ar, lapop21$pais, lapop21$weight1500, plot =
## FALSE): 40056 rows with missing values dropped
```

```r
varnames = c("mean_coup", "n_golpe", "sd_golpe")
colnames(table) = varnames
table$country = row.names(table)
table = table[-23, ]
```

We need to add a column that allows to merge of data from "table" with the vectorial data to produce a map.
We call this variable "OBJECTID" and it has a code that we see later in the vectorial files of maps, but it follows an alphabetical order.
After including this variable, we sort "table" from lower to higher in "OBJECTID".
With this sorting, countries are in alphabetical order.

We see in the code lines that start with #.
If we delete #, we include these lines of code and we have all countries in America (including Barbados, Bahamas, Belice, Graneda, Suriname, among others) in alphabetical order.
We maintain \# because these countries are not included in the 2021 round of the AmericasBarometer.

However, we will see that we are going to have vectorial data for these countries.


```r
table$OBJECTID = NA
table = within(table, {
  OBJECTID[country=="Argentina"] = 1
 # OBJECTID[country=="Barbados"]= 2
 # OBJECTID[country=="Bahamas"]= 3
 # OBJECTID[country=="Belice"]=4
  OBJECTID[country=="Bolivia"]=5
  OBJECTID[country=="Brazil"]=6
  OBJECTID[country=="Canada"]=7
  OBJECTID[country=="Chile"]=8
  OBJECTID[country=="Colombia"]=9
  OBJECTID[country=="Costa Rica"]=10
  OBJECTID[country=="Dominica"]=11
  OBJECTID[country=="Dom. Rep."]=12
  OBJECTID[country=="Ecuador"]=13
  OBJECTID[country=="El Salvador"]=14
 # OBJECTID[country=="Granada"]=15
  OBJECTID[country=="Guatemala"]=16
  OBJECTID[country=="Guyana"]=17
  OBJECTID[country=="Haiti"]=18
  OBJECTID[country=="Honduras"]=19
  OBJECTID[country=="Jamaica"]=20
  OBJECTID[country=="Mexico"]=21
  #OBJECTID[country=="Surinam"]=22
  OBJECTID[country=="Nicaragua"]=23
  OBJECTID[country=="Paraguay"]=24
  OBJECTID[country=="Peru"]=25
  OBJECTID[country=="Panama"]=26
  #OBJECTID[country=="San Cristobal and Nieves"]=27
  #OBJECTID[country=="Saint Lucia"]=28
  #OBJECTID[country=="Trinidad and Tobago"]=29
  OBJECTID[country=="Uruguay"]=30
  #OBJECTID[country=="Saint Vicente and the Granedinas"]=31
  #OBJECTID[country=="Venezuela"]=32
  OBJECTID[country=="United States"]=33
})
table = table[order(table$OBJECTID),]
```

# Vector maps

Files to produce maps are vector layers in EESRI Shapefiles format (\*.shp).
There are several repositories in the web where we can find the required files to produce maps.
For example, This [web](https://www.efrainmaps.es/descargas-gratuitas/américa/) has the layers for America as a free download.

After we download and unzip this information, it creates a folder with several files.
All these files are needed to create a map and we have to copy them in our working directory.
From these files, the vector layer that draw the map is called "America.shp".

There are several ways to read vector data in R.
One of them is the library `sf`.
This library includes the command `st_read` that allows to read this information and to work with this data in `ggplot`.
We activate the library and use the command `st_read` to load vector information in R.
We save this information in an object "al".
This object includes a table with 53 observations and two variables.
The 53 observations are all countries in the Americas, that includes, for example, Aruba, Antigua and Barbuda, etc.
The first columns includes the name of countries and the second column saves the geometry to draw a map.


```r
library(sf)
```

```
## Linking to GEOS 3.11.0, GDAL 3.5.3, PROJ 9.1.0; sf_use_s2() is TRUE
```

```r
al = st_read("Americas.shp")
```

```
## Reading layer `Americas' from data source 
##   `/Users/Arturo/Documents/GitHub/BarometroEdu_Web_Eng/Americas.shp' 
##   using driver `ESRI Shapefile'
## Simple feature collection with 53 features and 1 field
## Geometry type: MULTIPOLYGON
## Dimension:     XY
## Bounding box:  xmin: -186.5396 ymin: -58.49861 xmax: -12.15764 ymax: 83.6236
## Geodetic CRS:  WGS 84
```

Because the 2021 round of the AmericasBarometer does not include information for all these countries, we have to delete observations of these countries not required.


```r
al = al[-c(1, 2, 4, 5, 6, 7, 8, 13, 16, 17, 21, 22, 23, 24, 25, 31, 32, 34, 39, 40, 41, 42, 43, 44, 45, 47, 48, 49, 50, 51, 52), ]
```

In this way, the object "al" has the same list of countries that are in "table", which are the countries in the AmericasBarometer.

Following the same procedure as with the object "table", we have to add a column "OBJECTID" to "al".
Because this column is going to coincide with the column in "table", we can do a merge.
Again, this chunk includes lines of code with \# for countries that are not in the AmericasBarometer.


```r
al$OBJECTID = NA
al = within(al, {
  OBJECTID[COUNTRY=="Argentina"] = 1
 # OBJECTID[COUNTRY=="Barbados"]= 2
 # OBJECTID[COUNTRY=="Bahamas"]= 3
 # OBJECTID[COUNTRY=="Belice"]=4
  OBJECTID[COUNTRY=="Bolivia"]=5
  OBJECTID[COUNTRY=="Brazil"]=6
  OBJECTID[COUNTRY=="Canada"]=7
  OBJECTID[COUNTRY=="Chile"]=8
  OBJECTID[COUNTRY=="Colombia"]=9
  OBJECTID[COUNTRY=="Costa Rica"]=10
 # OBJECTID[COUNTRY=="Dominica"]=11
  OBJECTID[COUNTRY=="Dominican Republic"]=12
  OBJECTID[COUNTRY=="Ecuador"]=13
  OBJECTID[COUNTRY=="El Salvador"]=14
 # OBJECTID[COUNTRY=="Granada"]=15
  OBJECTID[COUNTRY=="Guatemala"]=16
  OBJECTID[COUNTRY=="Guyana"]=17
  OBJECTID[COUNTRY=="Haiti"]=18
  OBJECTID[COUNTRY=="Honduras"]=19
  OBJECTID[COUNTRY=="Jamaica"]=20
  OBJECTID[COUNTRY=="Mexico"]=21
  #OBJECTID[COUNTRY=="Surinam"]=22
  OBJECTID[COUNTRY=="Nicaragua"]=23
  OBJECTID[COUNTRY=="Paraguay"]=24
  OBJECTID[COUNTRY=="Peru"]=25
  OBJECTID[COUNTRY=="Panama"]=26
  #OBJECTID[COUNTRY=="San Cristobal y Nieves"]=27
  #OBJECTID[COUNTRY=="Santa Lucía"]=28
  #OBJECTID[COUNTRY=="Trinidad y Tobago"]=29
  OBJECTID[COUNTRY=="Uruguay"]=30
  #OBJECTID[COUNTRY=="San Vicente y las Granadinas"]=31
  #OBJECTID[COUNTRY=="Venezuela"]=32
  OBJECTID[COUNTRY=="United States"]=33
})
al = al[order(al$OBJECTID),]
```

We are going to join the information from "al" and "coup" in a new object "al_data".
We can do this with the command `left_join`, part of the tidyverse.
We indicate that we want to add data from "table" to the object "al".
The variable "OBJECTID" works as the variable to join files by defect.


```r
library(tidyverse)
```

```
## ── Attaching packages ─────────────────
## ✔ ggplot2 3.4.0      ✔ purrr   0.3.5 
## ✔ tibble  3.1.8      ✔ dplyr   1.0.10
## ✔ tidyr   1.2.1      ✔ stringr 1.5.0 
## ✔ readr   2.1.3      ✔ forcats 0.5.2 
## ── Conflicts ──────────────────────────
## ✖ dplyr::arrange()   masks plyr::arrange()
## ✖ purrr::compact()   masks plyr::compact()
## ✖ dplyr::count()     masks plyr::count()
## ✖ tidyr::expand()    masks Matrix::expand()
## ✖ dplyr::failwith()  masks plyr::failwith()
## ✖ dplyr::filter()    masks stats::filter()
## ✖ dplyr::id()        masks plyr::id()
## ✖ dplyr::lag()       masks stats::lag()
## ✖ dplyr::mutate()    masks plyr::mutate()
## ✖ tidyr::pack()      masks Matrix::pack()
## ✖ dplyr::rename()    masks plyr::rename()
## ✖ dplyr::summarise() masks plyr::summarise()
## ✖ dplyr::summarize() masks plyr::summarize()
## ✖ tidyr::unpack()    masks Matrix::unpack()
```

```r
al_data = al %>%
              left_join(table)
```

```
## Joining, by = "OBJECTID"
```

We have the vector data to draw the map and the data from the AmericasBarometer in this new object.

# Map for tolerance to executive coups by country

As we indicate, the library `ggplot2` can work with vector objects.
We specify in the command `ggplot` that we are going to use `data=al_data` and then, we use the command `geom_sf` to specify the variable we want to show.

We are going to start with a basic map.
We use the specification `fill="skyblue3"` in the command `geom_sf` to indicate that all countries have to be colored with color blue.
Also, we specify that country contour have to be colored with color black with `color="black".`


```r
library(ggplot2)
ggplot(data=al_data) +
  geom_sf(fill="skyblue3", color="black")
```

![](maps_files/figure-html/mapa basico-1.png)<!-- -->

Now, we are going to draw data from the variable tolerance to executive coups.
Again, we specify in `ggplot` that data comes from "al_data".
Then, we specify that colors for each country are defined by the variable "mean_coup" in `geom_sf(aes(fill= mean_coup))`.
To create a map where we use a color gradient to indicate a higher or lower percentage, we use the command `scale_fill_gradient` where we define the colors for lower values and higher values in the gradient.
In our case, we use the color "yellow" to show the lower percentages and the color "red" to show the higher percentages.

We add the command `geom_sf_text` where the specification `aes(label=pais)` indicates we want to add text with the labels of each country.
We specify the size of the text with `size=2`.
Finally, we define the title of this figure, the caption, the name of X axis and the legend with `labs`.
We choose a basic theme of black and whites to this figure with `theme_bw()`.


```r
ggplot(al_data) +
  geom_sf(aes(fill = mean_coup))+
  scale_fill_gradient(low = "yellow", high = "red")+
  geom_sf_text(aes(label=country), size=2)+
  labs(title = "Tolerance to executive coups in Latin America",
       caption = "Source: AmericasBarometer, 2021",
       x="Longitude", y="Latitude",
       fill = "% tolerance to executive coups")+
  theme_bw()
```

```
## Warning in st_is_longlat(x): bounding box has potentially an invalid value range
## for longlat data
```

```
## Warning in st_point_on_surface.sfc(sf::st_zm(x)): st_point_on_surface may not
## give correct results for longitude/latitude data
```

![](maps_files/figure-html/mapa completo-1.png)<!-- -->

This code defines breaks in colors in 10 jumps by defect.
If we want to change this breaks, we can use the specification `break=c(…)` within `scale_fill_gradient`.


```r
ggplot(al_data) +
  geom_sf(aes(fill = mean_coup))+
  scale_fill_gradient(low = "yellow", high = "red", breaks=c(15, 30, 45, 60))+
  geom_sf_text(aes(label=country), size=2)+
  labs(title = "Tolerance to executive coups in Latin America",
       caption = "Source: AmericasBarometer, 2021",
       x="Longitude", y="Latitude",
       fill = "% tolerance to executive coups")+
  theme_bw()
```

```
## Warning in st_is_longlat(x): bounding box has potentially an invalid value range
## for longlat data
```

```
## Warning in st_point_on_surface.sfc(sf::st_zm(x)): st_point_on_surface may not
## give correct results for longitude/latitude data
```

![](maps_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

# Summary

In this section, we download vector data for the Americas and we merge data from the AmericasBarometer to draw a map.
This type of map are called "choropleth".
It shows a color gradient depending on the value of a variable.

In our case, we draw a map to show a color gradient for the tolerance to executive coups in Latin America.
