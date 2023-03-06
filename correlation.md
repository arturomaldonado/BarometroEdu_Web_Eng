---
title: "Correlation with the AmericasBarometer"
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



<style type="text/css">
.columns {display: flex;}
h1 {color: #3366CC;}
</style>

# Introducction

The previous sections corresponding to the [t-test](https://arturomaldonado.github.io/BarometroEdu_Web_Eng/ttest.html) and to the [ANOVA test](https://arturomaldonado.github.io/BarometroEdu_Web_Eng/anova.html) are about the relationship of a numerical variable with a categorical variable, in such a way that the goal is to compare and extrapolate the means of the numerical variable by groups of the categorical variable.
In the section about [cross-tables](https://arturomaldonado.github.io/BarometroEdu_Web_Eng/chi.html) we analyze bivariate relationships between two categorical variables (or factor variables in R terminology).
This evaluation is done using cross-tables (or contingency tables) and is evaluated using the chi-square test.

In this section we will look at the bivariate relationship between two numerical variables, using a scatterplot for visual inspection and Pearson's correlation coefficient for evaluation.

# About the dataset

The data we are going to use should be cited as follows: Source: AmericasBarometer by the Latin American Public Opinion Project (LAPOP), wwww.LapopSurveys.org.
You can download the data freely [here](http://datasets.americasbarometer.org/database/login.php).

This document reloads a trimmed database, originally in SPSS (.sav) format.
It is recommended to clean the Environment before starting this section.


```r
library(rio) 
lapop18 = import("https://raw.github.com/lapop-central/materials_edu/main/LAPOP_AB_Merge_2018_v1.0.sav")
lapop18 = subset(lapop18, pais<=35)
```

We also load the data for the 2021 round.


```r
lapop21 = import("https://raw.github.com/lapop-central/materials_edu/main/lapop21.RData") 
lapop21 = subset(lapop21, pais<=35)
```

# Support for democracy and level of democracy

In this section we will continue to use the report *The Pulse of Democracy*, available [here](https://www.vanderbilt.edu/lapop/ab2018/2018-19_AmericasBarometer_Regional_Report_10.13.19.pdf), where the main findings of the 2018/19 round of the AmericasBarometer are presented.
In this report, Figure 1.3 is presented.
This is a scatterplot that relates the variable support for democracy (from the AmericasBarometer) to the Electoral Democracy Index from the project [V-Dem](https://www.v-dem.net/en/).
This figure shows "the relationship between the level of support for democracy and the rating of democracy in each country" (p. 12).

![](Figure1.3.JPG){width="505"}

To reproduce this figure, we must add the results of the variable ING4 by country.
ING4.
"Changing the subject, democracy may have problems, but it is better than any other form of government. To what extent do you agree or disagree with this statement?" People could respond on a scale of 1 to 7, where 1 means "strongly disagree" and 7 means "strongly agree." The report indicates that the original question is recoded into a dummy variable, where responses between 5 and 7 are considered supporters of democracy.
The X-axis of Figure 1.3 shows the percentage of people who support democracy by country (that is, those who answer between 5 and 7 in each country).

Then, on the V-Dem project website, we can calculate the Electoral Democracy Index scores for each country (see [here](https://www.v-dem.net/en/analysis/VariableGraph/)).
So, data can be collected for the 18 countries that are part of the report "The Pulse of Democracy".
This data can then be downloaded in .csv format.
The Y-axis of Figure 1.3 shows the V-Dem Electoral Democracy Index scores on a scale of 0 to 1.
For this section, data from the 2018 and 2019 Electoral Democracy Index have been collected for the 18 countries analyzed in the report, including the country code, in order to merge the data later.
This dataset is also hosted in the "materials_edu" repository of the LAPOP account on GitHub.
We load the data.


```r
vdem = import("https://raw.github.com/lapop-central/materials_edu/main/vdem.xlsx")
vdem
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["country"],"name":[1],"type":["chr"],"align":["left"]},{"label":["pais"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["vdem2018"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["vdem2019"],"name":[4],"type":["dbl"],"align":["right"]}],"data":[{"1":"Mexico","2":"1","3":"0.725","4":"0.710","_rn_":"1"},{"1":"Guatemala","2":"2","3":"0.615","4":"0.594","_rn_":"2"},{"1":"El Salvador","2":"3","3":"0.641","4":"0.631","_rn_":"3"},{"1":"Honduras","2":"4","3":"0.366","4":"0.360","_rn_":"4"},{"1":"Nicaragua","2":"5","3":"0.244","4":"0.245","_rn_":"5"},{"1":"Costa Rica","2":"6","3":"0.879","4":"0.889","_rn_":"6"},{"1":"Panama","2":"7","3":"0.758","4":"0.783","_rn_":"7"},{"1":"Colombia","2":"8","3":"0.680","4":"0.667","_rn_":"8"},{"1":"Ecuador","2":"9","3":"0.637","4":"0.673","_rn_":"9"},{"1":"Bolivia","2":"10","3":"0.587","4":"0.537","_rn_":"10"},{"1":"Peru","2":"11","3":"0.779","4":"0.784","_rn_":"11"},{"1":"Paraguay","2":"12","3":"0.587","4":"0.601","_rn_":"12"},{"1":"Chile","2":"13","3":"0.852","4":"0.773","_rn_":"13"},{"1":"Uruguay","2":"14","3":"0.853","4":"0.858","_rn_":"14"},{"1":"Brasil","2":"15","3":"0.737","4":"0.674","_rn_":"15"},{"1":"Argentina","2":"17","3":"0.834","4":"0.812","_rn_":"16"},{"1":"Rep. Dom.","2":"21","3":"0.536","4":"0.598","_rn_":"17"},{"1":"Jamaica","2":"23","3":"0.799","4":"0.810","_rn_":"18"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

To reproduce Figure 1.3 we have to recode the variable ING4 according to the rule indicated in the report:

-   ING4 values from 1-4 = 0 in the new variable "support"

-   ING4 values from 5-7 = 100 in the new variable "support"


```r
library(car)
lapop18$support = car::recode(lapop18$ing4, "1:4=0; 5:7=100")
table(lapop18$support)
```

```
## 
##     0   100 
## 11463 15623
```

With this new variable "support", we now have to add the data of this variable by country and save this information in a new dataframe "df".
For this we will use the command `summarySE` that reports the descriptive statistics of the "support" variable by country.
The N of each country, the average (which would be the percentage), the standard deviation, the standard error and the size of the confidence interval are included.
In this case we only require the data on average.
Looking at the table, we see that Uruguay is the country that reports a higher proportion of citizens who support democracy and also has the lowest standard deviation, indicating that there is greater homogeneity of opinions compared to the other 17 countries.


```r
library(Rmisc) #para poder utilizar el comando summarySE
df = summarySE(data=lapop18, measurevar="support", groupvar="pais", na.rm=T)
df
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["pais"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["N"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["support"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["sd"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["se"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["ci"],"name":[6],"type":["dbl"],"align":["right"]}],"data":[{"1":"1","2":"1513","3":"62.72307","4":"48.37013","5":"1.243534","6":"2.439235"},{"1":"2","2":"1524","3":"48.88451","4":"50.00396","5":"1.280890","6":"2.512496"},{"1":"3","2":"1465","3":"58.56655","4":"49.27750","5":"1.287448","6":"2.525440"},{"1":"4","2":"1493","3":"45.01005","4":"49.76705","5":"1.287989","6":"2.526461"},{"1":"5","2":"1496","3":"51.53743","4":"49.99307","5":"1.292540","6":"2.535385"},{"1":"6","2":"1458","3":"72.35940","4":"44.73735","5":"1.171633","6":"2.298267"},{"1":"7","2":"1537","3":"53.80612","4":"49.87115","5":"1.272074","6":"2.495186"},{"1":"8","2":"1619","3":"59.78999","4":"49.04734","5":"1.218967","6":"2.390921"},{"1":"9","2":"1512","3":"54.43122","4":"49.81973","5":"1.281225","6":"2.513169"},{"1":"10","2":"1630","3":"49.14110","4":"50.00796","5":"1.238641","6":"2.429496"},{"1":"11","2":"1496","3":"49.26471","4":"50.01131","5":"1.293012","6":"2.536310"},{"1":"12","2":"1478","3":"51.21786","4":"50.00208","5":"1.300621","6":"2.551262"},{"1":"13","2":"1550","3":"63.87097","4":"48.05295","5":"1.220546","6":"2.394097"},{"1":"14","2":"1529","3":"76.19359","4":"42.60379","5":"1.089543","6":"2.137158"},{"1":"15","2":"1471","3":"59.82325","4":"49.04221","5":"1.278685","6":"2.508243"},{"1":"17","2":"1495","3":"71.10368","4":"45.34325","5":"1.172714","6":"2.300340"},{"1":"21","2":"1474","3":"59.22659","4":"49.15800","5":"1.280400","6":"2.511601"},{"1":"23","2":"1346","3":"51.18871","4":"50.00445","5":"1.362969","6":"2.673777"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

We then use the data from the "vdem2019" column of the "vdem" dataframe to add these data to "df".
We do this with the `cbind` command, where the destination dataframe "df" and the data to de added are indicated, that is `vdem$vdem2019`.
The added column is renamed because by default it is named as the variable.


```r
df = cbind(df, vdem$vdem2019)
colnames(df)[7] = "vdem2019"
df
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["pais"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["N"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["support"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["sd"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["se"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["ci"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["vdem2019"],"name":[7],"type":["dbl"],"align":["right"]}],"data":[{"1":"1","2":"1513","3":"62.72307","4":"48.37013","5":"1.243534","6":"2.439235","7":"0.710"},{"1":"2","2":"1524","3":"48.88451","4":"50.00396","5":"1.280890","6":"2.512496","7":"0.594"},{"1":"3","2":"1465","3":"58.56655","4":"49.27750","5":"1.287448","6":"2.525440","7":"0.631"},{"1":"4","2":"1493","3":"45.01005","4":"49.76705","5":"1.287989","6":"2.526461","7":"0.360"},{"1":"5","2":"1496","3":"51.53743","4":"49.99307","5":"1.292540","6":"2.535385","7":"0.245"},{"1":"6","2":"1458","3":"72.35940","4":"44.73735","5":"1.171633","6":"2.298267","7":"0.889"},{"1":"7","2":"1537","3":"53.80612","4":"49.87115","5":"1.272074","6":"2.495186","7":"0.783"},{"1":"8","2":"1619","3":"59.78999","4":"49.04734","5":"1.218967","6":"2.390921","7":"0.667"},{"1":"9","2":"1512","3":"54.43122","4":"49.81973","5":"1.281225","6":"2.513169","7":"0.673"},{"1":"10","2":"1630","3":"49.14110","4":"50.00796","5":"1.238641","6":"2.429496","7":"0.537"},{"1":"11","2":"1496","3":"49.26471","4":"50.01131","5":"1.293012","6":"2.536310","7":"0.784"},{"1":"12","2":"1478","3":"51.21786","4":"50.00208","5":"1.300621","6":"2.551262","7":"0.601"},{"1":"13","2":"1550","3":"63.87097","4":"48.05295","5":"1.220546","6":"2.394097","7":"0.773"},{"1":"14","2":"1529","3":"76.19359","4":"42.60379","5":"1.089543","6":"2.137158","7":"0.858"},{"1":"15","2":"1471","3":"59.82325","4":"49.04221","5":"1.278685","6":"2.508243","7":"0.674"},{"1":"17","2":"1495","3":"71.10368","4":"45.34325","5":"1.172714","6":"2.300340","7":"0.812"},{"1":"21","2":"1474","3":"59.22659","4":"49.15800","5":"1.280400","6":"2.511601","7":"0.598"},{"1":"23","2":"1346","3":"51.18871","4":"50.00445","5":"1.362969","6":"2.673777","7":"0.810"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

# Diagram of Dispersion

In the dataframe "df" we now have the two variables that are plotted in the scatterplot presented in Figure 1.3.
We can replicate this figure with the `plot` command, where the variable that will go on the X axis is indicated and then the one that will go on the Y axis.
The axes are labeled with `xlab` and `ylab`.
Axis limits are set with `xlim` and `ylim`.
The labels of each point are added with the `text` command, where it is indicated to add the label of the variable `df$pais`.


```r
plot(df$support, df$vdem2019, 
     xlab="support for democracy (%)", 
     ylab="Electoral Democracy Index V_Dem", 
     pch=19, xlim=c(40, 80), ylim=c(0.2, 1))
text(df$support, df$vdem2019, labels=df$pais, cex=0.5, pos=3)
```

![](correlation_files/figure-html/plot-1.png)<!-- -->

However, these labels display the country codes.
To display the country names, we have to transform the variable "pais" into a factor variable "paises" and label with the names.


```r
df$country = as.factor(df$pais)
levels(df$country) = c("Mexico", "Guatemala", "El Salvador", "Honduras", "Nicaragua",
                      "Costa Rica", "Panama", "Colombia", "Ecuador", "Bolivia", "Peru", 
                      "Paraguay", "Chile", "Uruguay", "Brazil", "Argentina", "Dom. Rep.", 
                      "Jamaica")
table(df$country)
```

```
## 
##      Mexico   Guatemala El Salvador    Honduras   Nicaragua  Costa Rica 
##           1           1           1           1           1           1 
##      Panama    Colombia     Ecuador     Bolivia        Peru    Paraguay 
##           1           1           1           1           1           1 
##       Chile     Uruguay      Brazil   Argentina   Dom. Rep.     Jamaica 
##           1           1           1           1           1           1
```

With this new variable we can redo the scatter plot with the country labels.


```r
plot(df$support, df$vdem2019, 
     xlab="Support for democracy (%)", 
     ylab="Electoral Democracy Index VDem", 
     pch=19, xlim=c(40, 80), ylim=c(0.2, 1))
text(df$support, df$vdem2019, labels=df$country, cex=0.5, pos=3)
```

![](correlation_files/figure-html/plot2-1.png)<!-- -->

This same plot can also be reproduced using the library `ggplot`.
First, we define the aesthetics of the graph, that is, the dataframe, which will be "df", and with the specification `aes`, the variables on each axis of the figure.
With the command `geom_point` we indicate that we want to produce a point graph.
One element we can add is the prediction or smooth line, with the command `geom_smooth`.
Within this command it is specified that the linear method is used with `method=lm` and that the confidence interval around the prediction line is not displayed with `se=F`.
Then, with the command `geom_text`, the labels are included for each point, from the variable "country".
The specification `nudge_y` is used to wrap labels vertically and `check_overlap=T` to prevent labels from overlapping.
Finally, the axes are labeled with `labs(...)`, a general theme of the graph is defined, with `theme_light()` and the limits of the axes are defined.


```r
library(ggplot2)
ggplot(df, aes(x=support, y=vdem2019))+
  geom_point()+
  geom_smooth(method=lm, se=F)+ #add trend line
  geom_text(data=df, aes(label=country), cex=2.5, nudge_y = 0.02, check_overlap = T)+ #To label the points, give them a size, location and prevent them from overlapping
  labs(x="Support for democracy", y="Electoral Democracy Index V-Dem ")+ #To label the axes
  theme_light()+
  xlim(40, 80)+
  ylim(0.2, 1)
```

![](correlation_files/figure-html/ggplot-1.png)<!-- -->

As presented in the figure, the distribution of the countries can be summarized with a linear approximation using a straight line.
This straight line has a positive slope, which indicates that there is a direct relationship between both variables: as a country exhibits a higher percentage of citizens who support democracy, a higher score is observed in the electoral democracy index.

# Pearson´s Correlation Coefficient

To evaluate the magnitude of the replationship between both variables, a statistical measure can be added, the Pearsons´s R correlation coefficient.
This coefficient varies between -1 to +1.
The sign indicates the direction of the relationship, while the value indicates the degree of the relationship.
If the coefficient is 0, this indicates an absence of a linear relationship and the closer it is to 1, the greater the linear relationship between the variables.

The report indicates that "in general, there is a positive relationship between the two measures (Pearson's correlation =.64). Although this analysis is descriptive and does not test a causal relationship, the pattern is consistent with previous investigations where it is identified that citizen support for democracy is a central ingredient for the vitality of democracy".

The command `cor.test` can be used to calculate the value of Pearson's coefficient.
Within this command, it is indicated which variable is located on each axis.
By default, the Pearson coefficient is calculated, but with the specification `method="..."`, the Kendall or Spearman coefficient can also be calculated.


```r
cor.test(x = df$support, y = df$vdem2019)
```

```
## 
## 	Pearson's product-moment correlation
## 
## data:  df$support and df$vdem2019
## t = 3.2105, df = 16, p-value = 0.005456
## alternative hypothesis: true correlation is not equal to 0
## 95 percent confidence interval:
##  0.2247472 0.8456754
## sample estimates:
##       cor 
## 0.6259389
```

A coefficient of 0.62 is obtained, which indicates a positive relationship, although the exact value is not the same as that reported in the report because the calculations made in this section do not take into account the effect of survey weights.

# Summary

In this section we have worked on the bivariate relationship between two numerical variables.
The visualization of this relationship has been done through the diagram of dispersion and the evaluation of the relationship has been done through the Pearson´s correlation coefficient.

This is a first step in modeling.
In the following sections, modeling will be introduced using the simple linear regression technique, which is a mathematical expression of what has been seen in this section.
