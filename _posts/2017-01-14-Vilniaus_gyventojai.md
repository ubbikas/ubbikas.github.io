---
title: Vilniaus gyventojai
subtitle: Vilniaus gyventojų tankumo žemėlapis
output: md_document
htmlwidgets: TRUE
layout: post
image: /img/Vilniaus_gyventojai/thumbnail.png
---



## Vilniaus registruotų gyventojų duomenys

Perskaitęs Povilo Poderskio postą ["Kaip sutaupyti 2.8 milijonus eurų"](https://medium.com/@povilaspoderskis/kaip-sutaupyti-2-8-milijonus-eurų-36363cbdcf46), aplankiau ten minimą Lietuvos atvirų duomenų portalą [www.opendata.lt](http://www.opendata.lt), nes viešai prieinama statistika - visuomet sveikintinas reiškinys. Tuo tarpu mano neatsispirimas pasižaidimams su duomenimis nulėmė tai, jog nutariau panagrinėti patį pirmą įkeltą duomenų pluoštą - Vilniaus miesto savivaldybės registruotų gyventojų sarašą. Kadangi gyvenu Vilniuje ir šis miestas man tikrai patinka, pamaniau, pasirinkimas neturėtų nuvilti. Pagrindinis šio posto klausimas - kur Vilniuje yra tankiausiai gyvenama, o kartu su gautais pastebėjimais pateikiu analizės kelią: keletą duomenų manipuliavimo instrukcijų, komentarų ir galiausiai visą R kodą.

Pirmiausia pakrauname reikalingas bibliotekas, ir, jei dar to nepadarėme, atsisiunčiame  reikalingus duomenis:


{% highlight r %}
library(magrittr)
library(dplyr)
library(readr)
library(plotly)
library(ggplot2)
library(knitr)
library(rvest)
library(stringr)
library(methods)
library(pander)
library(ggmap)
library(viridis)

Sys.setlocale("LC_ALL", 'Lithuanian')

data_file <- "./data/Vilniaus_gyventojai/registered_people_n_streets.csv"

if (!file.exists(data_file)) {
  url <- "https://raw.githubusercontent.com/vilnius/gyventojai/master/data/registered_people_n_streets.csv"
  download.file(url, destfile = data_file)
}

people <- read_delim("./data/Vilniaus_gyventojai/registered_people_n_streets.csv", delim = ";")
{% endhighlight %}

Dabar galime peržvelgti, kokius duomenis nagrinėsime:


{% highlight r %}
head(people) %>%
  pander(style = 'rmarkdown', split.table = Inf)
{% endhighlight %}



|  GIMIMO_METAI  |  GIMIMO_VALSTYBE  |  LYTIS  |  SEIMOS_PADETIS  |  KIEK_TURI_VAIKU  |    SENIUNIJA    |        GATVE        |
|:--------------:|:-----------------:|:-------:|:----------------:|:-----------------:|:---------------:|:-------------------:|
|      1975      |        LTU        |    M    |        I         |         0         | Naujoji  Vilnia | A. Kojelavičiaus g. |
|      1949      |        LTU        |    M    |        V         |         0         |   Pašilaičiai   |    Pašilaičių g.    |
|      1938      |        LTU        |    M    |        I         |         0         |     Šeškinė     |     Ukmergės g.     |
|      1981      |        LTU        |    M    |        V         |         1         |     Verkiai     |   Kazio Ulvydo g.   |
|      1990      |        LTU        |    M    |        NA        |         0         |  Fabijoniškės   | Salomėjos Nėries g. |
|      1998      |        LTU        |    M    |        NA        |         0         |  Fabijoniškės   |     L. Giros g.     |

Paskaičiuojame visą gyventojų kiekį:


{% highlight r %}
# sumuojame visus gyventojus
people_sum <- nrow(people)
{% endhighlight %}

Iš viso registruotų gyventojų turime 575125, o daugiausia gimusių yra šiose valstybėse:


{% highlight r %}
people %>% 
  group_by(GIMIMO_VALSTYBE) %>% 
  summarise(Viso = n(),
            Procentas = round(Viso/people_sum*100,1)) %>% 
  arrange(desc(Viso)) %>% 
  head(10) %>% 
  pander(style = 'rmarkdown')
{% endhighlight %}



|  GIMIMO_VALSTYBE  |  Viso  |  Procentas  |
|:-----------------:|:------:|:-----------:|
|        LTU        | 508548 |    88.4     |
|        BLR        | 23495  |     4.1     |
|        RUS        | 21882  |     3.8     |
|        UKR        |  6783  |     1.2     |
|        GBR        |  1682  |     0.3     |
|        KAZ        |  1518  |     0.3     |
|        LVA        |  1345  |     0.2     |
|        POL        |  845   |     0.1     |
|        DEU        |  835   |     0.1     |
|        AZE        |  538   |     0.1     |

## Gyventojai pagal seniūnijas

Kadangi mūsų tyrinėjamuose duomenyse Vilniaus gyventojai yra suskirstyti pagal seniūnijas, tai pagal jas ir pažvelgsime į gyventojų išsibarstymą.

Pirmiausia - žemėlapis.

![seniūnijų žemėlapis](/img/Vilniaus_gyventojai/VilniausMiestoSeniunijos.png)

Nors seniūnijų plotai smarkiai skiriasi, vis tiek galime pažiūrėti, kurios iš jų yra labiausiai apgyvendintos:


{% highlight r %}
people %>% 
  filter(SENIUNIJA != "NULL") %>% 
  group_by(SENIUNIJA) %>% 
  summarise(Gyventojai = n()) %>% 
  arrange(desc(Gyventojai)) %>% 
  mutate(SENIUNIJA = reorder(factor(SENIUNIJA), Gyventojai)) -> sen_people

sen_people %>% 
  ggplot(aes(x = SENIUNIJA, y = Gyventojai)) +
  geom_bar(stat = "identity") +
  coord_flip() +
  xlab("Seniūnija") + 
  theme_minimal() -> sen_people_plot

ggplotly(sen_people_plot)
{% endhighlight %}

<!--html_preserve--><div id="htmlwidget-bbee147534c265f8d1f9" style="width:504px;height:648px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-bbee147534c265f8d1f9">{"x":{"data":[{"x":[8740,11335,11429,12204,14241,15912,20094,20891,22751,26370,27124,29003,30419,30882,31451,33628,36558,37178,38524,44848,46091],"y":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21],"text":["SENIUNIJA: Paneriai<br>Gyventojai: 8740","SENIUNIJA: Grigiškės<br>Gyventojai: 11335","SENIUNIJA: Rasos<br>Gyventojai: 11429","SENIUNIJA: Žvėrynas<br>Gyventojai: 12204","SENIUNIJA: Viršuliškės<br>Gyventojai: 14241","SENIUNIJA: Šnipiškės<br>Gyventojai: 15912","SENIUNIJA: Vilkpėdė<br>Gyventojai: 20094","SENIUNIJA: Senamiestis<br>Gyventojai: 20891","SENIUNIJA: Pilaitė<br>Gyventojai: 22751","SENIUNIJA: Karoliniškės<br>Gyventojai: 26370","SENIUNIJA: Justiniškės<br>Gyventojai: 27124","SENIUNIJA: Naujamiestis<br>Gyventojai: 29003","SENIUNIJA: Šeškinė<br>Gyventojai: 30419","SENIUNIJA: Naujininkai<br>Gyventojai: 30882","SENIUNIJA: Lazdynai<br>Gyventojai: 31451","SENIUNIJA: Naujoji  Vilnia<br>Gyventojai: 33628","SENIUNIJA: Pašilaičiai<br>Gyventojai: 36558","SENIUNIJA: Antakalnis<br>Gyventojai: 37178","SENIUNIJA: Fabijoniškės<br>Gyventojai: 38524","SENIUNIJA: Žirmūnai<br>Gyventojai: 44848","SENIUNIJA: Verkiai<br>Gyventojai: 46091"],"key":null,"type":"bar","marker":{"autocolorscale":false,"color":"rgba(89,89,89,1)","line":{"width":1.88976377952756,"color":"transparent"}},"showlegend":false,"xaxis":"x","yaxis":"y","hoverinfo":"text","orientation":"h","name":""}],"layout":{"margin":{"t":21.6823947234906,"r":7.30593607305936,"b":35.636732623034,"l":113.24200913242},"font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"xaxis":{"domain":[0,1],"type":"linear","autorange":false,"tickmode":"array","range":[-2304.55,48395.55],"ticktext":["0","10000","20000","30000","40000"],"tickvals":[0,10000,20000,30000,40000],"ticks":"","tickcolor":null,"ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(235,235,235,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"y","title":"Gyventojai","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"type":"linear","autorange":false,"tickmode":"array","range":[0.4,21.6],"ticktext":["Paneriai","Grigiškės","Rasos","Žvėrynas","Viršuliškės","Šnipiškės","Vilkpėdė","Senamiestis","Pilaitė","Karoliniškės","Justiniškės","Naujamiestis","Šeškinė","Naujininkai","Lazdynai","Naujoji  Vilnia","Pašilaičiai","Antakalnis","Fabijoniškės","Žirmūnai","Verkiai"],"tickvals":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21],"ticks":"","tickcolor":null,"ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(235,235,235,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"x","title":"Seniūnija","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":null,"line":{"color":null,"width":0,"linetype":[]},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":false,"legend":{"bgcolor":null,"bordercolor":null,"borderwidth":0,"font":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895}},"barmode":"stack","hovermode":"closest"},"source":"A","config":{"modeBarButtonsToAdd":[{"name":"Collaborate","icon":{"width":1000,"ascent":500,"descent":-50,"path":"M487 375c7-10 9-23 5-36l-79-259c-3-12-11-23-22-31-11-8-22-12-35-12l-263 0c-15 0-29 5-43 15-13 10-23 23-28 37-5 13-5 25-1 37 0 0 0 3 1 7 1 5 1 8 1 11 0 2 0 4-1 6 0 3-1 5-1 6 1 2 2 4 3 6 1 2 2 4 4 6 2 3 4 5 5 7 5 7 9 16 13 26 4 10 7 19 9 26 0 2 0 5 0 9-1 4-1 6 0 8 0 2 2 5 4 8 3 3 5 5 5 7 4 6 8 15 12 26 4 11 7 19 7 26 1 1 0 4 0 9-1 4-1 7 0 8 1 2 3 5 6 8 4 4 6 6 6 7 4 5 8 13 13 24 4 11 7 20 7 28 1 1 0 4 0 7-1 3-1 6-1 7 0 2 1 4 3 6 1 1 3 4 5 6 2 3 3 5 5 6 1 2 3 5 4 9 2 3 3 7 5 10 1 3 2 6 4 10 2 4 4 7 6 9 2 3 4 5 7 7 3 2 7 3 11 3 3 0 8 0 13-1l0-1c7 2 12 2 14 2l218 0c14 0 25-5 32-16 8-10 10-23 6-37l-79-259c-7-22-13-37-20-43-7-7-19-10-37-10l-248 0c-5 0-9-2-11-5-2-3-2-7 0-12 4-13 18-20 41-20l264 0c5 0 10 2 16 5 5 3 8 6 10 11l85 282c2 5 2 10 2 17 7-3 13-7 17-13z m-304 0c-1-3-1-5 0-7 1-1 3-2 6-2l174 0c2 0 4 1 7 2 2 2 4 4 5 7l6 18c0 3 0 5-1 7-1 1-3 2-6 2l-173 0c-3 0-5-1-8-2-2-2-4-4-4-7z m-24-73c-1-3-1-5 0-7 2-2 3-2 6-2l174 0c2 0 5 0 7 2 3 2 4 4 5 7l6 18c1 2 0 5-1 6-1 2-3 3-5 3l-174 0c-3 0-5-1-7-3-3-1-4-4-5-6z"},"click":"function(gd) { \n        // is this being viewed in RStudio?\n        if (location.search == '?viewer_pane=1') {\n          alert('To learn about plotly for collaboration, visit:\\n https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html');\n        } else {\n          window.open('https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html', '_blank');\n        }\n      }"}],"modeBarButtonsToRemove":["sendDataToCloud"]},"base_url":"https://plot.ly"},"evals":["config.modeBarButtonsToAdd.0.click"],"jsHooks":[]}</script><!--/html_preserve-->

Norėdami įvertinti tankiausiai apgyvendintas seniūnijas, turime išsitraukti jų plotus. Šią informaciją galime rasti [čia](https://lt.wikipedia.org/wiki/Vilniaus_miesto_savivaldyb%C4%97).


{% highlight r %}
# nuskaitome lentelę
"https://lt.wikipedia.org/wiki/Vilniaus_miesto_savivaldyb%C4%97" %>% 
  read_html() %>% 
  html_table(fill = TRUE) %>% 
  .[[7]] -> areas_table

# suvienodiname seniūnijų pavadinimus su prieš tai turėtais
sen_list <- c("Verkiai", "Antakalnis", "Pašilaičiai", "Fabijoniškės", "Pilaitė",
              "Justiniškės", "Viršuliškės", "Šeškinė", "Šnipiškės", "Žirmūnai",
              "Karoliniškės", "Žvėrynas", "Grigiškės", "Lazdynai", "Vilkpėdė",
              "Naujamiestis", "Senamiestis", "Naujoji  Vilnia", "Paneriai", 
              "Naujininkai", "Rasos")
areas_table$SENIUNIJA <- as.factor(sen_list)
areas_table <- select(areas_table, SENIUNIJA, Plotas)

# sutvarkome plotus
areas_table$Plotas %>% 
  str_replace_all("km²", "") %>% 
  str_trim() %>% 
  str_replace_all(",", ".") %>% 
  as.numeric() -> areas_table$Plotas

# sujungiame gyventojų skaičių ir plotų lenteles pagal seniūniją, apskaičiuojame
# gyventojų tankį, t.y. žmonių skaičių kvadratiniame kilometre
merge(areas_table, sen_people, by = "SENIUNIJA") %>% 
  mutate(Tankis = round(Gyventojai / Plotas, 2)) %>% 
  arrange(desc(Tankis)) %>% 
  mutate(SENIUNIJA = reorder(factor(SENIUNIJA), Tankis)) -> sen_area_people

sen_area_people %>% 
  ggplot(aes(x = SENIUNIJA, y = Tankis, 
             text = paste("Gyventojai:", Gyventojai, "<br>", "Plotas:", Plotas))) +
  geom_bar(stat = "identity") +
  coord_flip() +
  xlab("Seniūnija") + 
  theme_minimal() -> sen_area_people_plot

ggplotly(sen_area_people_plot)
{% endhighlight %}

<!--html_preserve--><div id="htmlwidget-b68b18dd67f0377a38c4" style="width:504px;height:648px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-b68b18dd67f0377a38c4">{"x":{"data":[{"x":[102.9,481.58,751.39,828.23,855.67,899.92,1596.48,1648.62,1950.87,3053.5,4458.29,4520,4642.44,5100,5276.24,5696.4,6042.29,6592.5,6913.41,9102.01,9396.1],"y":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21],"text":["SENIUNIJA: Paneriai<br>Tankis: 102.9<br>Gyventojai: 8740 <br> Plotas: 84.94","SENIUNIJA: Antakalnis<br>Tankis: 481.58<br>Gyventojai: 37178 <br> Plotas: 77.2","SENIUNIJA: Naujininkai<br>Tankis: 751.39<br>Gyventojai: 30882 <br> Plotas: 41.1","SENIUNIJA: Verkiai<br>Tankis: 828.23<br>Gyventojai: 46091 <br> Plotas: 55.65","SENIUNIJA: Naujoji  Vilnia<br>Tankis: 855.67<br>Gyventojai: 33628 <br> Plotas: 39.3","SENIUNIJA: Rasos<br>Tankis: 899.92<br>Gyventojai: 11429 <br> Plotas: 12.7","SENIUNIJA: Grigiškės<br>Tankis: 1596.48<br>Gyventojai: 11335 <br> Plotas: 7.1","SENIUNIJA: Pilaitė<br>Tankis: 1648.62<br>Gyventojai: 22751 <br> Plotas: 13.8","SENIUNIJA: Vilkpėdė<br>Tankis: 1950.87<br>Gyventojai: 20094 <br> Plotas: 10.3","SENIUNIJA: Lazdynai<br>Tankis: 3053.5<br>Gyventojai: 31451 <br> Plotas: 10.3","SENIUNIJA: Pašilaičiai<br>Tankis: 4458.29<br>Gyventojai: 36558 <br> Plotas: 8.2","SENIUNIJA: Žvėrynas<br>Tankis: 4520<br>Gyventojai: 12204 <br> Plotas: 2.7","SENIUNIJA: Senamiestis<br>Tankis: 4642.44<br>Gyventojai: 20891 <br> Plotas: 4.5","SENIUNIJA: Šnipiškės<br>Tankis: 5100<br>Gyventojai: 15912 <br> Plotas: 3.12","SENIUNIJA: Žirmūnai<br>Tankis: 5276.24<br>Gyventojai: 44848 <br> Plotas: 8.5","SENIUNIJA: Viršuliškės<br>Tankis: 5696.4<br>Gyventojai: 14241 <br> Plotas: 2.5","SENIUNIJA: Naujamiestis<br>Tankis: 6042.29<br>Gyventojai: 29003 <br> Plotas: 4.8","SENIUNIJA: Karoliniškės<br>Tankis: 6592.5<br>Gyventojai: 26370 <br> Plotas: 4","SENIUNIJA: Šeškinė<br>Tankis: 6913.41<br>Gyventojai: 30419 <br> Plotas: 4.4","SENIUNIJA: Justiniškės<br>Tankis: 9102.01<br>Gyventojai: 27124 <br> Plotas: 2.98","SENIUNIJA: Fabijoniškės<br>Tankis: 9396.1<br>Gyventojai: 38524 <br> Plotas: 4.1"],"key":null,"type":"bar","marker":{"autocolorscale":false,"color":"rgba(89,89,89,1)","line":{"width":1.88976377952756,"color":"transparent"}},"showlegend":false,"xaxis":"x","yaxis":"y","hoverinfo":"text","orientation":"h","name":""}],"layout":{"margin":{"t":21.6823947234906,"r":7.30593607305936,"b":35.636732623034,"l":113.24200913242},"font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"xaxis":{"domain":[0,1],"type":"linear","autorange":false,"tickmode":"array","range":[-469.805,9865.905],"ticktext":["0","2500","5000","7500"],"tickvals":[0,2500,5000,7500],"ticks":"","tickcolor":null,"ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(235,235,235,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"y","title":"Tankis","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"type":"linear","autorange":false,"tickmode":"array","range":[0.4,21.6],"ticktext":["Paneriai","Antakalnis","Naujininkai","Verkiai","Naujoji  Vilnia","Rasos","Grigiškės","Pilaitė","Vilkpėdė","Lazdynai","Pašilaičiai","Žvėrynas","Senamiestis","Šnipiškės","Žirmūnai","Viršuliškės","Naujamiestis","Karoliniškės","Šeškinė","Justiniškės","Fabijoniškės"],"tickvals":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21],"ticks":"","tickcolor":null,"ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(235,235,235,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"x","title":"Seniūnija","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":null,"line":{"color":null,"width":0,"linetype":[]},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":false,"legend":{"bgcolor":null,"bordercolor":null,"borderwidth":0,"font":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895}},"barmode":"stack","hovermode":"closest"},"source":"A","config":{"modeBarButtonsToAdd":[{"name":"Collaborate","icon":{"width":1000,"ascent":500,"descent":-50,"path":"M487 375c7-10 9-23 5-36l-79-259c-3-12-11-23-22-31-11-8-22-12-35-12l-263 0c-15 0-29 5-43 15-13 10-23 23-28 37-5 13-5 25-1 37 0 0 0 3 1 7 1 5 1 8 1 11 0 2 0 4-1 6 0 3-1 5-1 6 1 2 2 4 3 6 1 2 2 4 4 6 2 3 4 5 5 7 5 7 9 16 13 26 4 10 7 19 9 26 0 2 0 5 0 9-1 4-1 6 0 8 0 2 2 5 4 8 3 3 5 5 5 7 4 6 8 15 12 26 4 11 7 19 7 26 1 1 0 4 0 9-1 4-1 7 0 8 1 2 3 5 6 8 4 4 6 6 6 7 4 5 8 13 13 24 4 11 7 20 7 28 1 1 0 4 0 7-1 3-1 6-1 7 0 2 1 4 3 6 1 1 3 4 5 6 2 3 3 5 5 6 1 2 3 5 4 9 2 3 3 7 5 10 1 3 2 6 4 10 2 4 4 7 6 9 2 3 4 5 7 7 3 2 7 3 11 3 3 0 8 0 13-1l0-1c7 2 12 2 14 2l218 0c14 0 25-5 32-16 8-10 10-23 6-37l-79-259c-7-22-13-37-20-43-7-7-19-10-37-10l-248 0c-5 0-9-2-11-5-2-3-2-7 0-12 4-13 18-20 41-20l264 0c5 0 10 2 16 5 5 3 8 6 10 11l85 282c2 5 2 10 2 17 7-3 13-7 17-13z m-304 0c-1-3-1-5 0-7 1-1 3-2 6-2l174 0c2 0 4 1 7 2 2 2 4 4 5 7l6 18c0 3 0 5-1 7-1 1-3 2-6 2l-173 0c-3 0-5-1-8-2-2-2-4-4-4-7z m-24-73c-1-3-1-5 0-7 2-2 3-2 6-2l174 0c2 0 5 0 7 2 3 2 4 4 5 7l6 18c1 2 0 5-1 6-1 2-3 3-5 3l-174 0c-3 0-5-1-7-3-3-1-4-4-5-6z"},"click":"function(gd) { \n        // is this being viewed in RStudio?\n        if (location.search == '?viewer_pane=1') {\n          alert('To learn about plotly for collaboration, visit:\\n https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html');\n        } else {\n          window.open('https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html', '_blank');\n        }\n      }"}],"modeBarButtonsToRemove":["sendDataToCloud"]},"base_url":"https://plot.ly"},"evals":["config.modeBarButtonsToAdd.0.click"],"jsHooks":[]}</script><!--/html_preserve-->

Kaip matome, tankiausiai apgyvendinti yra miegamieji rajonai, o rečiausiai apgyvendintos yra salyginai gerokai didesnį plotą turinčios seniūnijos.

Toliau pažvelkime į žmonių, gimusių kitose valstybėse, procentinį kiekį skirtingose seniūnijose:


{% highlight r %}
people %>% 
  filter(GIMIMO_VALSTYBE != "LTU",
         SENIUNIJA != "NULL") %>% 
  group_by(SENIUNIJA) %>% 
  summarise(Ne_LTU = n()) %>%
  left_join(sen_people, by = "SENIUNIJA") %>% 
  mutate(Ne_LTU_Procentas = 100*Ne_LTU/Gyventojai,
         SENIUNIJA = reorder(factor(SENIUNIJA), Ne_LTU_Procentas)) %>%
  ggplot(aes(x = SENIUNIJA, y = Ne_LTU_Procentas)) +
  geom_bar(stat = "identity") +
  coord_flip() +
  xlab("Seniūnija") + 
  ylab("Gyventojų, gimusių ne Lietuvoje, procentas") +
  theme_minimal() -> sen_people_notLTU_plot

ggplotly(sen_people_notLTU_plot)
{% endhighlight %}

<!--html_preserve--><div id="htmlwidget-4311143f27b0d569ac25" style="width:504px;height:648px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-4311143f27b0d569ac25">{"x":{"data":[{"x":[7.99071402226031,9.16093084234677,9.26199463865638,9.51606522790207,9.93329388347948,10.3589710978983,10.3857335686845,10.9020911715811,11.3475177304965,11.5300570816982,11.7976699601829,12.1103569632981,12.1897498274105,12.2076241664178,12.6621160409556,13.2124263007275,13.3837537695658,15.015219221553,15.3661327231121,15.6506482692994,17.1680635200706],"y":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21],"text":["SENIUNIJA: Verkiai<br>Ne_LTU_Procentas: 7.99","SENIUNIJA: Žvėrynas<br>Ne_LTU_Procentas: 9.16","SENIUNIJA: Pašilaičiai<br>Ne_LTU_Procentas: 9.26","SENIUNIJA: Pilaitė<br>Ne_LTU_Procentas: 9.52","SENIUNIJA: Antakalnis<br>Ne_LTU_Procentas: 9.93","SENIUNIJA: Lazdynai<br>Ne_LTU_Procentas: 10.36","SENIUNIJA: Fabijoniškės<br>Ne_LTU_Procentas: 10.39","SENIUNIJA: Rasos<br>Ne_LTU_Procentas: 10.9","SENIUNIJA: Viršuliškės<br>Ne_LTU_Procentas: 11.35","SENIUNIJA: Žirmūnai<br>Ne_LTU_Procentas: 11.53","SENIUNIJA: Justiniškės<br>Ne_LTU_Procentas: 11.8","SENIUNIJA: Šnipiškės<br>Ne_LTU_Procentas: 12.11","SENIUNIJA: Šeškinė<br>Ne_LTU_Procentas: 12.19","SENIUNIJA: Vilkpėdė<br>Ne_LTU_Procentas: 12.21","SENIUNIJA: Karoliniškės<br>Ne_LTU_Procentas: 12.66","SENIUNIJA: Naujamiestis<br>Ne_LTU_Procentas: 13.21","SENIUNIJA: Senamiestis<br>Ne_LTU_Procentas: 13.38","SENIUNIJA: Naujininkai<br>Ne_LTU_Procentas: 15.02","SENIUNIJA: Paneriai<br>Ne_LTU_Procentas: 15.37","SENIUNIJA: Naujoji  Vilnia<br>Ne_LTU_Procentas: 15.65","SENIUNIJA: Grigiškės<br>Ne_LTU_Procentas: 17.17"],"key":null,"type":"bar","marker":{"autocolorscale":false,"color":"rgba(89,89,89,1)","line":{"width":1.88976377952756,"color":"transparent"}},"showlegend":false,"xaxis":"x","yaxis":"y","hoverinfo":"text","orientation":"h","name":""}],"layout":{"margin":{"t":21.6823947234906,"r":7.30593607305936,"b":35.636732623034,"l":113.24200913242},"font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"xaxis":{"domain":[0,1],"type":"linear","autorange":false,"tickmode":"array","range":[-0.858403176003529,18.0264666960741],"ticktext":["0","5","10","15"],"tickvals":[-1.11022302462516e-016,5,10,15],"ticks":"","tickcolor":null,"ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(235,235,235,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"y","title":"Gyventojų, gimusių ne Lietuvoje, procentas","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"type":"linear","autorange":false,"tickmode":"array","range":[0.4,21.6],"ticktext":["Verkiai","Žvėrynas","Pašilaičiai","Pilaitė","Antakalnis","Lazdynai","Fabijoniškės","Rasos","Viršuliškės","Žirmūnai","Justiniškės","Šnipiškės","Šeškinė","Vilkpėdė","Karoliniškės","Naujamiestis","Senamiestis","Naujininkai","Paneriai","Naujoji  Vilnia","Grigiškės"],"tickvals":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21],"ticks":"","tickcolor":null,"ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(235,235,235,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"x","title":"Seniūnija","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":null,"line":{"color":null,"width":0,"linetype":[]},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":false,"legend":{"bgcolor":null,"bordercolor":null,"borderwidth":0,"font":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895}},"barmode":"stack","hovermode":"closest"},"source":"A","config":{"modeBarButtonsToAdd":[{"name":"Collaborate","icon":{"width":1000,"ascent":500,"descent":-50,"path":"M487 375c7-10 9-23 5-36l-79-259c-3-12-11-23-22-31-11-8-22-12-35-12l-263 0c-15 0-29 5-43 15-13 10-23 23-28 37-5 13-5 25-1 37 0 0 0 3 1 7 1 5 1 8 1 11 0 2 0 4-1 6 0 3-1 5-1 6 1 2 2 4 3 6 1 2 2 4 4 6 2 3 4 5 5 7 5 7 9 16 13 26 4 10 7 19 9 26 0 2 0 5 0 9-1 4-1 6 0 8 0 2 2 5 4 8 3 3 5 5 5 7 4 6 8 15 12 26 4 11 7 19 7 26 1 1 0 4 0 9-1 4-1 7 0 8 1 2 3 5 6 8 4 4 6 6 6 7 4 5 8 13 13 24 4 11 7 20 7 28 1 1 0 4 0 7-1 3-1 6-1 7 0 2 1 4 3 6 1 1 3 4 5 6 2 3 3 5 5 6 1 2 3 5 4 9 2 3 3 7 5 10 1 3 2 6 4 10 2 4 4 7 6 9 2 3 4 5 7 7 3 2 7 3 11 3 3 0 8 0 13-1l0-1c7 2 12 2 14 2l218 0c14 0 25-5 32-16 8-10 10-23 6-37l-79-259c-7-22-13-37-20-43-7-7-19-10-37-10l-248 0c-5 0-9-2-11-5-2-3-2-7 0-12 4-13 18-20 41-20l264 0c5 0 10 2 16 5 5 3 8 6 10 11l85 282c2 5 2 10 2 17 7-3 13-7 17-13z m-304 0c-1-3-1-5 0-7 1-1 3-2 6-2l174 0c2 0 4 1 7 2 2 2 4 4 5 7l6 18c0 3 0 5-1 7-1 1-3 2-6 2l-173 0c-3 0-5-1-8-2-2-2-4-4-4-7z m-24-73c-1-3-1-5 0-7 2-2 3-2 6-2l174 0c2 0 5 0 7 2 3 2 4 4 5 7l6 18c1 2 0 5-1 6-1 2-3 3-5 3l-174 0c-3 0-5-1-7-3-3-1-4-4-5-6z"},"click":"function(gd) { \n        // is this being viewed in RStudio?\n        if (location.search == '?viewer_pane=1') {\n          alert('To learn about plotly for collaboration, visit:\\n https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html');\n        } else {\n          window.open('https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html', '_blank');\n        }\n      }"}],"modeBarButtonsToRemove":["sendDataToCloud"]},"base_url":"https://plot.ly"},"evals":["config.modeBarButtonsToAdd.0.click"],"jsHooks":[]}</script><!--/html_preserve-->

Galime pastebėti, jog daugiausia iš jų yra apsistoję pietinėse Vilniaus seniūnijose.

## Gyventojai pagal gatves

Nors ir žinome tankiausiai apgyvendintas seniūnijas, jos nėra vientisai apgyvendintos, ypač turinčios didesnį plotą, todėl norėdami tiksliau pamatyti Vilniaus gyventojų koncentraciją patyrinėsime jų skaičius gatvių lygmenyje. Žinoma, ir šis rodiklis nėra idealus, nes gatvių ilgiai smarkiai skiriasi - nuo trumpų senamiesčio gatvelių iki kelis rajonus jungiančių prospektų. Taip pat iškyla problemų norint gyventojų skaičių pavaizduoti žemėlapyje: kaip išdalinti gyventojus per visą gatvės ilgį, kaip įsivertinti tankį iš šalia esančių gatvių ir t.t. Taigi pirmiausia - 15 daugiausiai gyventojų turinčių gatvių:


{% highlight r %}
people %>% 
  filter(GATVE != "NULL") %>% 
  group_by(GATVE) %>% 
  summarise(Gyventoju_gatveje = n()) -> gatves_gyv
  
gatves_gyv %>% 
  top_n(15) %>%
  mutate(GATVE = reorder(factor(GATVE),Gyventoju_gatveje)) %>% 
  ggplot(aes(x = GATVE, y = Gyventoju_gatveje)) +
  geom_bar(stat = "identity") +
  coord_flip() +
  xlab("Gatvė") + 
  ylab("Gyventojų skaičius") +
  theme_minimal() -> gatves_gyv_plot

ggplotly(gatves_gyv_plot)
{% endhighlight %}

<!--html_preserve--><div id="htmlwidget-3778ed326457cda55881" style="width:504px;height:648px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-3778ed326457cda55881">{"x":{"data":[{"x":[5386,5741,6020,6219,6320,7113,7133,7403,7699,8549,10231,10836,16776,17516,18187],"y":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15],"text":["GATVE: Žemynos g.<br>Gyventoju_gatveje: 5386","GATVE: Ukmergės g.<br>Gyventoju_gatveje: 5741","GATVE: Gabijos g.<br>Gyventoju_gatveje: 6020","GATVE: Gelvonų g.<br>Gyventoju_gatveje: 6219","GATVE: Savanorių pr.<br>Gyventoju_gatveje: 6320","GATVE: Salomėjos Nėries g.<br>Gyventoju_gatveje: 7113","GATVE: Justiniškių g.<br>Gyventoju_gatveje: 7133","GATVE: Laisvės pr.<br>Gyventoju_gatveje: 7403","GATVE: Fabijoniškių g.<br>Gyventoju_gatveje: 7699","GATVE: Viršuliškių g.<br>Gyventoju_gatveje: 8549","GATVE: S. Stanevičiaus g.<br>Gyventoju_gatveje: 10231","GATVE: Kalvarijų g.<br>Gyventoju_gatveje: 10836","GATVE: Architektų g.<br>Gyventoju_gatveje: 16776","GATVE: Žirmūnų g.<br>Gyventoju_gatveje: 17516","GATVE: Taikos g.<br>Gyventoju_gatveje: 18187"],"key":null,"type":"bar","marker":{"autocolorscale":false,"color":"rgba(89,89,89,1)","line":{"width":1.88976377952756,"color":"transparent"}},"showlegend":false,"xaxis":"x","yaxis":"y","hoverinfo":"text","orientation":"h","name":""}],"layout":{"margin":{"t":21.6823947234906,"r":7.30593607305936,"b":35.636732623034,"l":136.62100456621},"font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"xaxis":{"domain":[0,1],"type":"linear","autorange":false,"tickmode":"array","range":[-909.35,19096.35],"ticktext":["0","5000","10000","15000"],"tickvals":[0,5000,10000,15000],"ticks":"","tickcolor":null,"ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(235,235,235,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"y","title":"Gyventojų skaičius","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"type":"linear","autorange":false,"tickmode":"array","range":[0.4,15.6],"ticktext":["Žemynos g.","Ukmergės g.","Gabijos g.","Gelvonų g.","Savanorių pr.","Salomėjos Nėries g.","Justiniškių g.","Laisvės pr.","Fabijoniškių g.","Viršuliškių g.","S. Stanevičiaus g.","Kalvarijų g.","Architektų g.","Žirmūnų g.","Taikos g."],"tickvals":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15],"ticks":"","tickcolor":null,"ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(235,235,235,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"x","title":"Gatvė","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":null,"line":{"color":null,"width":0,"linetype":[]},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":false,"legend":{"bgcolor":null,"bordercolor":null,"borderwidth":0,"font":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895}},"barmode":"stack","hovermode":"closest"},"source":"A","config":{"modeBarButtonsToAdd":[{"name":"Collaborate","icon":{"width":1000,"ascent":500,"descent":-50,"path":"M487 375c7-10 9-23 5-36l-79-259c-3-12-11-23-22-31-11-8-22-12-35-12l-263 0c-15 0-29 5-43 15-13 10-23 23-28 37-5 13-5 25-1 37 0 0 0 3 1 7 1 5 1 8 1 11 0 2 0 4-1 6 0 3-1 5-1 6 1 2 2 4 3 6 1 2 2 4 4 6 2 3 4 5 5 7 5 7 9 16 13 26 4 10 7 19 9 26 0 2 0 5 0 9-1 4-1 6 0 8 0 2 2 5 4 8 3 3 5 5 5 7 4 6 8 15 12 26 4 11 7 19 7 26 1 1 0 4 0 9-1 4-1 7 0 8 1 2 3 5 6 8 4 4 6 6 6 7 4 5 8 13 13 24 4 11 7 20 7 28 1 1 0 4 0 7-1 3-1 6-1 7 0 2 1 4 3 6 1 1 3 4 5 6 2 3 3 5 5 6 1 2 3 5 4 9 2 3 3 7 5 10 1 3 2 6 4 10 2 4 4 7 6 9 2 3 4 5 7 7 3 2 7 3 11 3 3 0 8 0 13-1l0-1c7 2 12 2 14 2l218 0c14 0 25-5 32-16 8-10 10-23 6-37l-79-259c-7-22-13-37-20-43-7-7-19-10-37-10l-248 0c-5 0-9-2-11-5-2-3-2-7 0-12 4-13 18-20 41-20l264 0c5 0 10 2 16 5 5 3 8 6 10 11l85 282c2 5 2 10 2 17 7-3 13-7 17-13z m-304 0c-1-3-1-5 0-7 1-1 3-2 6-2l174 0c2 0 4 1 7 2 2 2 4 4 5 7l6 18c0 3 0 5-1 7-1 1-3 2-6 2l-173 0c-3 0-5-1-8-2-2-2-4-4-4-7z m-24-73c-1-3-1-5 0-7 2-2 3-2 6-2l174 0c2 0 5 0 7 2 3 2 4 4 5 7l6 18c1 2 0 5-1 6-1 2-3 3-5 3l-174 0c-3 0-5-1-7-3-3-1-4-4-5-6z"},"click":"function(gd) { \n        // is this being viewed in RStudio?\n        if (location.search == '?viewer_pane=1') {\n          alert('To learn about plotly for collaboration, visit:\\n https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html');\n        } else {\n          window.open('https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html', '_blank');\n        }\n      }"}],"modeBarButtonsToRemove":["sendDataToCloud"]},"base_url":"https://plot.ly"},"evals":["config.modeBarButtonsToAdd.0.click"],"jsHooks":[]}</script><!--/html_preserve-->

Norėdami panaudoti visą turimą informaciją, ją turime pavaizduoti žemėlapyje, o tam reikės gatvės koordinačių, kurias ištrauksime su **geocode()** iš [ggmap](https://cran.r-project.org/web/packages/ggmap/index.html) paketo. Ši leidžia pasinaudoti Google Maps API ir rasti ieškomo objekto (šiuo atveju - visų turimų gatvių) koordinates. Nenorėdami suerzinti Google, užklausas siųsime vienos sekundės intervalu.


{% highlight r %}
gatves_data_file <- "./data/Vilniaus_gyventojai/gatves_loc_df.csv"

if (!file.exists(gatves_data_file)) {
  gatves <- gatves_gyv$GATVE
  gatves_loc <- data.frame(GATVE = gatves, Lon = NA, Lat = NA)
  gatves_len <- length(gatves)
  gatves_inc <- 0
  
  for (gatve in gatves) {
    adresas <- paste(gatve, "Vilnius", "Lithuania", sep = ",")
    tryCatch({
      geo <- geocode(adresas)
    }, error = function(e) {
      print(e)
      geo$lon <- NA
      geo$lat <- NA
      print(paste(adresas, "nerastas!"))
    })
    gatves_loc[gatves_loc$GATVE == gatve, "Lon"] <- geo$lon
    gatves_loc[gatves_loc$GATVE == gatve, "Lat"] <- geo$lat
    gatves_inc <- gatves_inc + 1
    print(paste0(round(100*gatves_inc/gatves_len, 1), "% finished"))
    Sys.sleep(1)
  }
  
  # Pastebime, jog yra gatvių, kurių koordinatės nebuvo rastos. Viena iš priežasčių yra
  # žmogaus pilnu vardu pavadintos gatvės, o Google Maps vardus dažniausiai sutrumpina,
  # todėl mėginsime nerastas gatves sutrumpinti iki vieno žodžio ilgio ir pakartoti paieškos
  # algoritmą.

  gatves_loc %>%
  filter(is.na(Lon)) %>%
  .[[1]] -> nerastos_gatves

  lapply(nerastos_gatves, function(gatve) {
    g_split <- str_split(gatve, pattern = " ") %>% .[[1]]
    g_zodziai <- length(g_split)
  
    if (g_zodziai > 2) {
      gatve_short <- paste(g_split[(g_zodziai-1):g_zodziai], collapse = " ")
      adresas_short <- paste(gatve_short, "Vilnius", "Lithuania", sep = ",")
  
      tryCatch({
        geo <- geocode(adresas_short)
      }, error = function(e) {
        print(e)
        geo$lon <- NA
        geo$lat <- NA
        print(paste(adresas, "nerastas!"))
      })
  
      gatves_loc[gatves_loc$GATVE == gatve, "Lon"] <<- geo$lon
      gatves_loc[gatves_loc$GATVE == gatve, "Lat"] <<- geo$lat
      Sys.sleep(1)
    }
    return(NULL)
  })
  
  write.csv(gatves_loc, 
            file = gatves_data_file,
            row.names = FALSE)
}

gatves_loc <- read.csv(gatves_data_file, stringsAsFactors = FALSE)
{% endhighlight %}


{% highlight text %}
## [1] "Skirtingų gatvių skaičius: 2157"
{% endhighlight %}



{% highlight text %}
## [1] "Nerasta koordinačių: 76 gatvės"
{% endhighlight %}

Taigi gatvių, kurių koordinatės nebuvo rastos, skaičius yra santykinai mažas, o jose gyvenančių gyventojų yra:


{% highlight r %}
gatves_loc_gyv <- merge(gatves_gyv, gatves_loc, by = "GATVE")
people_no_loc <- filter(gatves_loc_gyv, is.na(Lon)) %>% 
                   .[["Gyventoju_gatveje"]] %>% 
                   sum()
gatves_loc_gyv <- na.omit(gatves_loc_gyv)
{% endhighlight %}


{% highlight text %}
## [1] "1299 gyventojai gyvena gatvėse be rastų koordinačių (0.24%)"
{% endhighlight %}

Šie gyventojai yra pašalinami iš duomenų tolimesnėje analizėje įtakos neturės. Pirmiausia pavaizduosime surinktas gatvių koordinates ir jų tankį žemėlapyje. Nereikia pamiršti, jog gatvių kiekis nenusako gyventojų kiekio - tam turėsime nagrinėti kiekvieną registruotą gyventoją atskirai.


{% highlight r %}
map <- get_map(location = 'Vilnius', zoom = 11, maptype = 'roadmap')
{% endhighlight %}


{% highlight r %}
ggmap(map,extent = "device") +
  geom_point(aes(x = Lon, y = Lat), 
             colour = "red", 
             alpha = 1, 
             size = 1.5, 
             data = gatves_loc_gyv) +
  geom_density2d(aes(x = Lon, y = Lat), 
                 size = 0.5,
                 data = gatves_loc_gyv) +
  stat_density2d(aes(x = Lon, y = Lat, fill = ..level.., alpha = ..level..), 
                 size = 0.1, 
                 bins = 7,
                 geom = "polygon",
                 data = gatves_loc_gyv) +
  scale_fill_viridis(guide = FALSE) +
  scale_alpha(range = c(0.15, 0.25), 
              guide = FALSE) +
  labs(x = NULL, y = NULL)
{% endhighlight %}

![plot of chunk unnamed-chunk-14](/figures/source/2017-01-14-Vilniaus_gyventojai/unnamed-chunk-14-1.png)

## Vilniaus gyventojų tankumo žemėlapis

Kadangi gatvės centrinis taškas reikštų, kad visi gyventojai ten ir gyvena, sugeneruosime dirbtinį žmonių išsibastymą: kiekvienas gyventojas bus paslinktas atstumu iki 425 metrų atsitiktine kryptimi.


{% highlight r %}
people_loc_file <- "./data/Vilniaus_gyventojai/people_loc_df.csv"
if (!file.exists(people_loc_file)) {
  people_loc <- merge(people, gatves_loc, by = "GATVE")
  write.csv(people_loc, people_loc_file, row.names = FALSE)
}
people_loc <- read.csv(people_loc_file)

# 0.0046 ilgumos ir 0.0027 platumos pokyčiai, išreikšti dešimtainiais laipsniais, atitinka 300 metrų poslinkį atitinkamomis kryptimis
people_loc %>% 
  filter(!is.na(Lon),!is.na(Lat)) %>% 
  mutate(LonN = Lon + runif(n(), -0.0046, 0.0046),
         LatN = Lat + runif(n(), -0.0027, 0.0027)) -> people_loc
{% endhighlight %}

Pagaliau galime pavaizduoti visų registruotų Vilniaus gyventojų tankumo žemėlapį.


{% highlight r %}
map2 <- get_map(location = 'Vilnius', zoom = 12, maptype = 'roadmap')
{% endhighlight %}


{% highlight r %}
ggmap(map2, extent = "device") +
  geom_density2d(aes(x = LonN, y = LatN),
                 size = 0.5,
                 bins = 15,
                 data = people_loc) +
  stat_density2d(aes(x = LonN, y = LatN, fill = ..level.., alpha = ..level..),
                 size = 0.1,
                 bins = 170,
                 geom = "polygon",
                 data = people_loc) +
  scale_fill_viridis(guide = FALSE) +
  scale_alpha(range = c(0.02, 0.07),
              guide = FALSE) +
  labs(x = NULL, y = NULL)
{% endhighlight %}

![plot of chunk unnamed-chunk-17](/figures/source/2017-01-14-Vilniaus_gyventojai/unnamed-chunk-17-1.png)

Rezultatai nėra netikėti. Tankiausiai apgyvendintos miegamųjų rajonų zonos: Justiniškės, Pašilaičiai, Fabijoniškės, Žirmūnai, Lazdynai ir t.t., o senesni Vilniaus rajonai, tokie kaip Senamiestis, Naujamiestis, Užupis ar Naujininkai, yra apgyvendinti rečiau.

## Vilniaus gyventojų, gimusių ne Lietuvoje, tankumo žemėlapis

Galiausiai pažvelgsime į ne Lietuvoje gimusių gyventojų išsibarstymą Vilniuje ir pamėginsime įžvelgti galimas tendencijas. Aptarsime keturias populiariausias valstybes: Balturusiją, Rusiją, Ukrainą ir Jungtinę Karalystę.

#### Baltarusija

Be standartiškai tankiai apgyvendintų miegamųjų rajonų, gimusieji Baltarusijoje taip pat kaip gyvenamąją vietą dažniau renkasi Naująją Vilnią ir Naujininkus.


{% highlight r %}
ggmap(map, extent = "device") +
  geom_density2d(aes(x = LonN, y = LatN),
                 size = 0.5,
                 bins = 10,
                 data = filter(people_loc,
                               GIMIMO_VALSTYBE == "BLR")) +
  stat_density2d(aes(x = LonN, y = LatN, fill = ..level.., alpha = ..level..), 
                 size = 0.1, 
                 bins = 50,
                 geom = "polygon",
                 data = filter(people_loc,
                               GIMIMO_VALSTYBE == "BLR")) +
  scale_fill_viridis(guide = FALSE) +
  scale_alpha(range = c(0.03, 0.03), 
              guide = FALSE) +
  labs(x = NULL, y = NULL)
{% endhighlight %}

![plot of chunk unnamed-chunk-18](/figures/source/2017-01-14-Vilniaus_gyventojai/unnamed-chunk-18-1.png)

#### Rusija

Tuo tarpu Vilniaus gyventojai, gimę Rusijoje, išsiskiria Naujamiesčio, Lazdynų, Naujosios Vilnios pasirinkimu.


{% highlight r %}
ggmap(map, extent = "device") +
  geom_density2d(aes(x = LonN, y = LatN),
                 size = 0.5,
                 bins = 10,
                 data = filter(people_loc,
                               GIMIMO_VALSTYBE == "RUS")) +
  stat_density2d(aes(x = LonN, y = LatN, fill = ..level.., alpha = ..level..), 
                 size = 0.1, 
                 bins = 50,
                 geom = "polygon",
                 data = filter(people_loc,
                               GIMIMO_VALSTYBE == "RUS")) +
  scale_fill_viridis(guide = FALSE) +
  scale_alpha(range = c(0.03, 0.03), 
              guide = FALSE) +
  labs(x = NULL, y = NULL)
{% endhighlight %}

![plot of chunk unnamed-chunk-19](/figures/source/2017-01-14-Vilniaus_gyventojai/unnamed-chunk-19-1.png)

#### Ukraina

Gimusieji Ukrainoje pasižymi tankiu gyventoju kiekiu Paneriuose ir Naujamiestyje, ypač stoties rajone.


{% highlight r %}
ggmap(map, extent = "device") +
  geom_density2d(aes(x = LonN, y = LatN),
                 size = 0.5,
                 bins = 10,
                 data = filter(people_loc,
                               GIMIMO_VALSTYBE == "UKR")) +
  stat_density2d(aes(x = LonN, y = LatN, fill = ..level.., alpha = ..level..), 
                 size = 0.1, 
                 bins = 50,
                 geom = "polygon",
                 data = filter(people_loc,
                               GIMIMO_VALSTYBE == "UKR")) +
  scale_fill_viridis(guide = FALSE) +
  scale_alpha(range = c(0.03, 0.04), 
              guide = FALSE) +
  labs(x = NULL, y = NULL)
{% endhighlight %}

![plot of chunk unnamed-chunk-20](/figures/source/2017-01-14-Vilniaus_gyventojai/unnamed-chunk-20-1.png)

#### Jungtinė Karalystė

O kaip gimimo valstybę Jungtinę Karalystę nurodę gyventojai dažniausiai renkasi Pašilaičius ir Fabijoniškes.


{% highlight r %}
ggmap(map, extent = "device") +
  geom_density2d(aes(x = LonN, y = LatN),
                 size = 0.5,
                 bins = 10,
                 data = filter(people_loc,
                               GIMIMO_VALSTYBE == "GBR")) +
  stat_density2d(aes(x = LonN, y = LatN, fill = ..level.., alpha = ..level..), 
                 size = 0.1, 
                 bins = 70,
                 geom = "polygon",
                 data = filter(people_loc,
                               GIMIMO_VALSTYBE == "GBR")) +
  scale_fill_viridis(guide = FALSE) +
  scale_alpha(range = c(0.03, 0.03), 
              guide = FALSE) +
  labs(x = NULL, y = NULL)
{% endhighlight %}

![plot of chunk unnamed-chunk-21](/figures/source/2017-01-14-Vilniaus_gyventojai/unnamed-chunk-21-1.png)

## Pabaigai

Norėdami tikslesnio Viniaus gyventojų tankumo žemėlapio turėtume:

- Turėti kiekvieno gyventojo tikslią registruotą gyvenamąją vietą (idealus variantas).
- Gauti namų, priklausančiu atitinkamoms gatvėms, koordinates ir gatvės gyventojų skaičių išdalinti kiekvienam namui.
- Išsitraukti tikslias viso ilgio koordinates kiekvienai gatvei ir gyventojus paskirstyti aplink visą gatvės ruožą.

------
