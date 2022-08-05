# Sv2_StarlingLitReview

<p align="center">

![ScreenShot](/Sv2_vignette/Sv2_Map.png)

</p>

This vignette is from the below publication:

Stuart KC†, Hofmeister NR†, Zichello JM, Rollins LA. **2022** Global invasion history and native decline of the common starling, *Biological Invasions*, under revision. † joint first author

As this project was a literature review, there is no fun genomics analysis for me to write up. However, below I have placed my code for plotting a distribution map of a species from point observation data.

<h2><i>Quick distribution map from observation data</i></h2>

It seems every time I start making a distribution plot, I use a completely different approach and package. Below is just one of the ways you can get it done, and I may in the future add to this document other user friendly R plotting packages I have found. Some of the prettiest distribution maps can be made in more fit for purpose programs like GIS. However this requires extra software and know-how, so for now I shall keep this succinct and only in R.


## Where to find data

Great - you're working on a new species and need to make a plot of where it is located for some figure or presentation. It's actually a very quick proccess. First step is to find a data base where you can pull information about the distribution of your species. There are many options here, some will be quite taxa or country specific. Some good starting points are:

<ul>
<li><a href="https://ebird.org/home">eBird</a></li>
<li><a href="GBIF">https://www.gbif.org/</a></li>
<li><a href="https://www.inaturalist.org/">iNaturalist</a></li>
<li><a href="https://mol.org/">Map of Life</a></li>
</ul>

Find you species and retrieve you data.

## Plotting in R

Packages used in the below code.

<pre class="r"><code>library(dplyr)
library(maps)
library(mapdata)
library(ggplot2)
</code></pre>

You distirbution data will look different depending on where you have pulled it from. The important thing is that each data row is a distinct obervation of an individual, and that two of the supplied column information is the Latitiude and Longitude. In my data I also had a column that contained the country names of obersvation, which allowed me to subset my data to label native and invasive ranges with different colours.

<pre class="r"><code>starling_distribution <- read.csv("distribution_data_starling.txt", sep="\t", quote = "")
levels(starling_distribution$COUNTRY)
</code></pre>

I also created two files, where were lists of country names where the starling was either invasive, or native.

<pre class="r"><code>Invasive <- read.csv("Invasive.txt", sep="\t", quote = "", header=FALSE)
Invasive <- as.vector(Invasive[,1])
Native <- read.csv("Native.txt", sep="\t", quote = "", header=FALSE)
Native <- as.vector(Native[,1])
</code></pre>

Filter the data using the above files.

<pre class="r"><code>InvasivePoints <- starling_distribution[starling_distribution$COUNTRY %in% Invasive,]
NativePoints <- starling_distribution[starling_distribution$COUNTRY %in% Native,]
</code></pre>

I also created a data frame that contained sites of introduction, along with their first arrival date. 

<pre class="r"><code>labs <- data.frame(
  long = c(-73.9654, 144.9631, 151.2093, 138.6007, 153.0251, 147.3272, 18.4241, -58.3816, -60.6973, 173.2840, 170.1548, 172.6362, 174.7633, 174.7762),
  lat = c(40.7829, -37.8136, -33.8688, -34.9285, -27.4698, -42.8821, -33.9249, -34.6037, -31.6107, -41.2706, -45.4791, -43.5321, -36.8485, -41.2865),
  names = c("US", "Melbourne","New South Wales","South Australia","Queensland","Tasmania","Cape Town","Buenos Ares","Santa Fe","Nelson","Otago","Christchurch","Auckland","Wellington"),
  countries = c("United Statets of America", "Australia","Australia","Australia","Australia","Australia","South Africa","South America","South America","New Zealand","New Zealand","New Zealand","New Zealand","New Zealand"),
  dates = c("1890", "1857","1880","1860","1869","1860","1897","1987","2001","1862","1967","1867","1865","1877"),
  stringsAsFactors = FALSE
)</code></pre>

I also created a similar data frame that had slightly offset lat/long for adding date labels. There are ways to achieve this using ggplot (e.g. geom_label_repel() or geom_label(nudge_x = 0, nudge_y = 0), however I had only a few labels and wanted to control the direction of each label offset individually.

<pre class="r"><code>labsoffset <- data.frame(
  long = c(-65.9654, 151.9631, 158.2093, 129.6007, 163.0251, 137.3272, 18.4241, -50.3816, -51.6973, 165.2840, 180.1548, 182.6362, 182.7633, 180.7762),
  lat = c(40.7829, -37.8136, -33.8688, -34.9285, -27.4698, -42.8821, -37.9249, -36.6037, -31.6107, -41.2706, -48.4791, -44.5321, -35.8485, -41.2865),
  names = c("US", "Melbourne","New South Wales","South Australia","Queensland","Tasmania","Cape Town","Buenos Ares","Santa Fe","Nelson","Otago","Christchurch","Auckland","Wellington"),
  dates = c("1890", "1857","1880","1860","1869","1860","1897","1987","2001","1862","1967","1867","1865","1877"),
  stringsAsFactors = FALSE
) 
)</code></pre>

Finally, import your global map data.

<pre class="r"><code>w2hr <- map_data("worldHires")
</code></pre>

Now start plotting your world map.

<pre class="r"><code>gg1 <- ggplot() + 
  geom_polygon(data = w2hr, aes(x=long, y = lat, group = group), fill = "gray80", color = "gray80") + 
  coord_fixed(1.3)
</code></pre>

Add the species distribution points.

<pre class="r"><code>gg2 <- gg1 +
  geom_point(data = ExpandingRangePoints, aes(x = LONGITUDE, y = LATITUDE), color = "#008080", size = 1, alpha = 1) +
  geom_point(data = InvasivePoints, aes(x = LONGITUDE, y = LATITUDE), color = "maroon", size = 1, alpha = 1) +
  geom_point(data = NativePoints, aes(x = LONGITUDE, y = LATITUDE), color = "#008080", size = 1, alpha = 1) +
  geom_point(data = labs, aes(x = long, y = lat), color = "black", size = 6) +
  geom_point(data = labs, aes(x = long, y = lat), color = "midnightblue", size = 5) +
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank(),
          panel.background = element_blank(), axis.line = element_blank())
</code></pre>

Add offset labels and plot

<pre class="r"><code>gg3 <- gg2 +
  geom_text(data = labsoffset, aes(x = long, y = lat, label=dates),hjust=0.5, vjust=0.5, size = 5) 
 
gg3
</code></pre>

Finally, save.

<pre class="r"><code>pdf("StarlingGlobalDisribution.pdf", width = 10, height = 7) 
gg3
dev.off() 
</code></pre>

