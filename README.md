---
title: "Final_Project_part1"
author: "Damini Rijhwani"
date: "4/2/2019"
output: rmarkdown::github_document
runtime: shiny
---

```{r}
library(magrittr)
library(dplyr)
library(ggthemes)

library(ggplot2)

set.seed(1)  
map.county <- map_data('county')
county_df <- map_data("county")
counties   <- unique(map.county[,5:6])
#### please change data location and insert the dataset. Dataset is available in final report reference section
dat <- read.csv("C:/Users/Damini/Downloads/air_quality/air_quality.csv", header = TRUE, stringsAsFactors = FALSE)
```

# Map Plot

```{r}
create_graph <- function(data_map, year, title_name, Unit_name, Measure, label_name){
  if(!missing(Measure))
  {
    data_map <- data_map %>% filter(MeasureName==Measure)
  }
  if(!missing(Unit_name))
  {
    data_map <- data_map %>% filter(UnitName == Unit_name )
  }

  data_map <- data_map %>% filter(ReportYear== year)
  county_df$ozone = data_map$Value[match(county_df$subregion ,tolower(data_map$CountyName))]

  map_plot <- ggplot()
  map_plot = map_plot + geom_polygon(data=county_df, aes(x=long, y=lat, group = group, fill=county_df$ozone),colour="white") +
    scale_fill_viridis_c(option = "inferno")
  map_plot <- map_plot + labs(title=substitute(paste("U.S. ", title_name, " by County in " , year)),
                  subtitle="Data obtained: Centers for Disease Control and Prevention")+
  labs(x="", y="", fill=label_name)
  map_plot <- map_plot + theme_map(base_family="serif")
  map_plot <- map_plot + theme(legend.position=c(0.9, 0.25))
  map_plot <- map_plot + theme(plot.title=element_text(margin=margin(b=3), size=16,))
  map_plot <- map_plot + theme(plot.subtitle=element_text(margin=margin(b=-10), size=13, ))
  print(map_plot)
}

```
```{r}
plot1 <- create_graph(dat, "2009", "average ambient concentrations of PM2.5" , Unit = "Micograms per cubic meter", label_name = "NAAQ[%]" )
plot2 <- create_graph(dat, "2003", "average ambient concentrations of PM2.5" , Unit = "Micograms per cubic meter", label_name = "NAAQ [%]")
plot6 <- create_graph(dat, "2011", "average ambient concentrations of PM2.5" , Unit = "Micograms per cubic meter", label_name = "NAAQ[%]" )
plot3 <- create_graph(dat, "2003", "Person-days with max 8-hour average ozone concentration NAAQ" , Measure="Number of person-days with maximum 8-hour average ozone concentration over the National Ambient Air Quality Standard", label_name = "Number of days")
plot4 <- create_graph(dat, "2009", "Person-days with maxi 8-hour average ozone concentration NAAQ" , Measure="Number of person-days with maximum 8-hour average ozone concentration over the National Ambient Air Quality Standard" , label_name = "Number of days")
plot5 <- create_graph(dat, "2011", "Person-days with maxi 8-hour average ozone concentration NAAQ" , Measure="Number of person-days with maximum 8-hour average ozone concentration over the National Ambient Air Quality Standard" , label_name = "Number of days")
```
# Heat Maps
```{r}
build_heatmap <- function(dat_frame)
{
  max_limit_score = max(dat_frame$score)
  print(max_limit_score)
  p <- ggplot(dat_frame, aes(y=StateName, x=ReportYear, fill=score)) +
    geom_tile(colour="white", linewidth=2, width=1, height=1) + scale_fill_gradientn(colours="white", limits=c(0, max_limit_score),
                                                                                       na.value=rgb(246, 246, 246, max=255),guide=guide_colourbar(ticks=T, nbin=50,barheight=.5, label=T, barwidth=10)) +
    scale_fill_viridis_c(option = "plasma") + scale_x_continuous(expand=c(0,0), breaks=seq(1999, 2013, by=2)) + 
    scale_y_discrete(expand=c(0.04,0.04)) +
    labs(x="Year of Reported Data", y="State", fill="NAAQ in %") +
    ggtitle("NAAQ Levels and Major Environmental Timelines")
  return(p)
}


dat_heat <- dat %>% filter(UnitName == "Percent")  %>%
  select(ReportYear, StateName, Value)%>%
  group_by(StateName)%>%
  summarise(score=mean(Value))


dat_fr <- dat %>% filter(UnitName == "Percent") %>%
  select(ReportYear, StateName, Value) %>%
  group_by(ReportYear, StateName)%>%
  summarise(score=mean(Value))
```

# California Act AB
```{r}
 ggpl <- build_heatmap(dat_fr)
 ggpl <- ggpl + annotate("text", label="California AB 1493", x=2002, y=53, vjust=1, hjust=0, size=I(3), family="serif") + geom_segment(x=2002, xend=2002, y=0, yend=51.5, size=1.7)
ggpl
```
# Without NAAQ values above 5%  
```{r}
dat_frame_low_NAAQ <- dat_fr %>% subset(score < 5)
ggpl_low_NAAQ_5 <- build_heatmap(dat_frame_low_NAAQ)

ggpl_low_NAAQ_5

```
# Without NAAQ values above 2% 
```{r}
dat_frame_low_NAAQ <- dat_fr %>% subset(score < 2)
ggpl_low_NAAQ <- build_heatmap(dat_frame_low_NAAQ) + annotate("text", label="Energy Policy Act", x=2005, y=53, vjust=1, hjust=0.5, size=I(3), family="serif")+
  annotate("text", label="SAFETEA", x=2005, y=53, vjust=54.9, hjust=0.7, size=I(3), family="serif") +
  geom_segment(x=2005, xend=2005, y=0, yend=51.5, size=1.7) +
annotate("text", label="Energy Independence and Security Act", x=2007, y=53, vjust=1, hjust=0.1, size=I(3), family="serif") + 
  geom_segment(x=2007, xend=2007, y=0, yend=51.5, size=1.7)

ggpl_low_NAAQ

```
```{r echo=FALSE, results="hide"}
library(magrittr)
library(dplyr)
library(ggthemes)
library(shiny)
library(ggplot2)
library(ggvis)
set.seed(1)
library(shiny)
dat <- read.csv("C:/Users/Damini/Downloads/air_quality/air_quality.csv", header = TRUE, stringsAsFactors = FALSE)
county_df <- map_data("county")

```
```{r echo = FALSE}
column(12, offset = 2,selectInput(width=535, "value", "Choose a measurment to analyze:",
                                      list(`East Coast` = list("# days with max 8-hour average ozone concentration over the NAAQS", 
                                                               "% of days with PM2.5 levels over the NAAQS",
                                                               "Annual average ambient concentrations of PM2.5"
                                      ))))

sliderInput("bins",
                        "Number of bins:",
                        min = 1999,
                        max = 2013,
                        value = 1, width=1000)

```

```{r echo=FALSE}
reactive({
  
       
        line_df <- dat %>% filter(UnitName == "Micograms per cubic meter")
        if(input$value == "# days with max 8-hour average ozone concentration over the NAAQS")
        {

        line_df <- dat %>% filter(UnitName == "No Units")

        }
        else if(input$value == "% of days with PM2.5 levels over the NAAQS")
        {
            line_df <- dat %>% filter(UnitName == "Percent")
        }
        else if(input$value == "Annual average ambient concentrations of PM2.5")
        {
            line_df <- dat %>% filter(UnitName == "Micograms per cubic meter")
        }
        line_df2 <- line_df %>% filter(ReportYear==input$bins) 
        line_df2 %>% 
            ggvis(x=~StateName, y=~Value, fill:="darkblue") %>% 
            layer_boxplots() %>% add_axis("x", properties = axis_props(labels = list(angle = 45, align = "left", fontSize = 10)), title = " ", ) %>%
            add_axis("y", title = "Average ambient concentrations of PM2.5 in [Âµg/mÂ³]", ) %>% set_options(width = "auto", resizable=FALSE) %>% bind_shiny('plot')

})

ggvisOutput('plot')
```

# Air-Quality-Data-Analysis-


<h1 align="center">
  <br>
  
  <br>
  Damini K Rijhwani
  <br>
</h1>

<h4 align="center"> Air Quality Data Analysis from <a href="https://data.cdc.gov/Environmental-Health-Toxicology/Air-Quality-Measures-on-the-National-Environmental/cjae-szjv">Centers for Disease Control and Prevention</a>.</h4>

<p align="center">
  <a href="https://www.python.org/">
    <img src="https://img.shields.io/badge/Made%20with-Python-1f425f.svg"
         alt="Gitter">
  </a>
  <a href="https://github.com/daminiR/">
  <img src="https://img.shields.io/badge/Ask%20me-anything-1abc9c.svg"></a>
  <a href="https://GitHub.com/Naereen/StrapDown.js/graphs/contributors/">
      <img src="https://img.shields.io/github/contributors/Naereen/StrapDown.js.svg">
  </a>
  <a href="https://pypi.python.org/pypi/ansicolortags/">
    <img src="https://img.shields.io/pypi/l/ansicolortags.svg">
  </a>
</p>

<p align="center">
  <a href="#intro">Intro</a> •
  <a href="#key-features">Key Features</a> •
  <a href="#how-to-use">How To Use</a> •
  <a href="#download">Download</a> •
  <a href="#related">Related</a> •
    <a href="#Requirements">Requirements</a> •
</p>

<div id=”mainDiv”, align="center">




## Intro 

This is an implementation of a research paper, Everybody Dance Now. The main objective of the project is to allow frames of poses to be synthesized which in turn can be used for showcasing dance moves or any movement.
The paper utilizes pix2pixHD generative adversarial model to synthesize an image from semantic pose heat maps. The poses are obtain from openPose framework. Heat maps and part affinity maps are used perform pose estimation.
The pix2pixHD GAN introduces, several separate inputs to the generator and the deliminator, while having multiple decriminators that advance the photo realistic depiction of the synthesized image.Further details about the 
pix2pixHD GAN can read on NVIDIA's <a href="https://github.com/NVIDIA/pix2pixHD">github</a>  or in my <a href="https://thedisruptculture.com/2019/04/20/high-resolution-image-synthesis-and-semantic-manipulation-with-conditional-gans-explained/">blog</a>.   

![](heatmaps_air_quality.png)
## Key Features

Create gifs or videos of synthesized dance moves<br/>
Make people you know dance!<br/>
Simple use of GANs and CNNs<br/>

## How To Use

Clone or download the repo<br/>
Obtain target dataset - the person you want to perform the generated poses on.<br/>
Obtain the test dataset to source the pose estimation from<br/>
Train the GAN model on the target dataset<br/>
Transfer the models learnt outcome to output synthesise images, using generator and the semantic label heat map of the poses.<br/>
Create video from moving frames.<br/>
## Download 
Clone the github repo including source. 


---
> GitHub [@DaminiR](https://github.com/daminiR/) &nbsp;&middot;&nbsp;
> LinkedIn [Damini K Rijhwani](www.linkedin.com/in/drijhwan)



