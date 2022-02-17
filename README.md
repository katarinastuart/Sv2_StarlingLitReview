# Sv2_StarlingLitReview

<h2><i>Forthcoming</i></h2>

Compiling the global starling sightings
 
<pre class="r"><code>
starlings <- read.csv("ebd_eursta_prv_relNov-2018.txt", sep="\t", quote = "")
</code></pre>

#think only the top one works??
Starlings <- read.csv("ebd_eursta_prv_relNov-2018.txt", sep="\t", quote = "")
starlings <- read.csv("ebd_eursta_prv_relNov-2018.txt",stringsAsFactors=TRUE,sep="\t")
str(Starlings)

starlingsbrief <- Starlings[c(9,26:27)] # Count and location data
str(starlingsbrief) #3673670 obs. of  47 variables imported

write.csv(starlingsbrief,file="starlingsbrief.csv") #can use this in regular GUI R


# import
starlings <- read.csv("ebd_eursta_prv_relNov-2018.txt",stringsAsFactors=TRUE,sep="\t")
str(starlings)

starlingsfull <- starlings[c(6,8:9,12:13,26:28)] # All important information
str(starlingsfull)
write.csv(starlingsfull,file="starlingsfull.csv")
starlingsfull <- read.csv("starlingsfull.csv",stringsAsFactors=TRUE,sep=",")
str(starlingsfull)

starlingsbrief <- starlings[c(9,26:27)] # Count and location data
str(starlingsbrief)

write.csv(starlingsbrief,file="starlingsbrief.csv")

Starlings <- read.csv("starlingsbrief.csv",stringsAsFactors=TRUE,sep=",")
str(Starlings)



  
##### sorting out native and invasive ranges
Invasive <- read.csv("Invasive.txt", sep="\t", quote = "", header=FALSE)
Invasive <- as.vector(Invasive[,1])
Native <- read.csv("Native.txt", sep="\t", quote = "", header=FALSE)
Native <- as.vector(Native[,1])
ExpandingRange <- read.csv("ExpandingRange.txt", sep="\t", quote = "", header=FALSE)
ExpandingRange <- as.vector(ExpandingRange[,1])

starlingsfull <- read.csv("starlingsfull.csv",stringsAsFactors=TRUE,sep=",")
levels(starlingsfull$COUNTRY)
str(starlingsfull)

ExpandingRangePoints <- starlingsfull[starlingsfull$COUNTRY %in% ExpandingRange,]
InvasivePoints <- starlingsfull[starlingsfull$COUNTRY %in% Invasive,]
NativePoints <- starlingsfull[starlingsfull$COUNTRY %in% Native,]

 

library(maps)
library(mapdata)
library(sp)
library(maptools)
library(Rcpp)
library(colorspace)
library(plyr)
library(scales)
library(ggplot2)
library(ggrepel)
library(dplyr)

 



###EFFORT 3

#map
w2hr <- map_data("worldHires")
dim(w2hr)

#Special points 
labs <- data.frame(
  long = c(-73.9654, 144.9631, 151.2093, 138.6007, 153.0251, 147.3272, 18.4241, -58.3816, -60.6973, 173.2840, 170.1548, 172.6362, 174.7633, 174.7762),
  lat = c(40.7829, -37.8136, -33.8688, -34.9285, -27.4698, -42.8821, -33.9249, -34.6037, -31.6107, -41.2706, -45.4791, -43.5321, -36.8485, -41.2865),
  names = c("US", "Melbourne","New South Wales","South Australia","Queensland","Tasmania","Cape Town","Buenos Ares","Santa Fe","Nelson","Otago","Christchurch","Auckland","Wellington"),
  countries = c("United Statets of America", "Australia","Australia","Australia","Australia","Australia","South Africa","South America","South America","New Zealand","New Zealand","New Zealand","New Zealand","New Zealand"),
  dates = c("1890", "1857","1880","1860","1869","1860","1897","1987","2001","1862","1967","1867","1865","1877"),
  stringsAsFactors = FALSE
)

labsoffset <- data.frame(
  long = c(-65.9654, 151.9631, 158.2093, 129.6007, 163.0251, 137.3272, 18.4241, -50.3816, -51.6973, 165.2840, 180.1548, 182.6362, 182.7633, 180.7762),
  lat = c(40.7829, -37.8136, -33.8688, -34.9285, -27.4698, -42.8821, -37.9249, -36.6037, -31.6107, -41.2706, -48.4791, -44.5321, -35.8485, -41.2865),
  names = c("US", "Melbourne","New South Wales","South Australia","Queensland","Tasmania","Cape Town","Buenos Ares","Santa Fe","Nelson","Otago","Christchurch","Auckland","Wellington"),
  dates = c("1890", "1857","1880","1860","1869","1860","1897","1987","2001","1862","1967","1867","1865","1877"),
  stringsAsFactors = FALSE
) 

#black outlined, grey filled
gg1 <- ggplot() + 
  geom_polygon(data = w2hr, aes(x=long, y = lat, group = group), fill = "gray80", color = "gray80") + 
  coord_fixed(1.3)
  
#Trying to add starlings
gg2 <- gg1 +
  geom_point(data = ExpandingRangePoints, aes(x = LONGITUDE, y = LATITUDE), color = "#008080", size = 0.25, alpha = 1) +
  geom_point(data = InvasivePoints, aes(x = LONGITUDE, y = LATITUDE), color = "maroon", size = 0.25, alpha = 1) +
  geom_point(data = NativePoints, aes(x = LONGITUDE, y = LATITUDE), color = "#008080", size = 0.25, alpha = 1) +
  geom_point(data = labs, aes(x = long, y = lat), color = "midnightblue", size = 2) +
  geom_point(data = labs, aes(x = long, y = lat), color = "honeydew3", size = 1) +
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank(),
          panel.background = element_blank(), axis.line = element_blank())

#adding labels
gg3 <- gg2 +
  geom_text(data = labsoffset, aes(x = long, y = lat, label=dates),hjust=0.5, vjust=0.5, size = 5) 
 
gg3

 

#Trying to add starlings
gg2 <- gg1 +
  geom_point(data = ExpandingRangePoints, aes(x = LONGITUDE, y = LATITUDE), color = "cadetblue2", size = 0.25, alpha = 1) +
  geom_point(data = InvasivePoints, aes(x = LONGITUDE, y = LATITUDE), color = "lightpink2", size = 0.25, alpha = 1) +
  geom_point(data = NativePoints, aes(x = LONGITUDE, y = LATITUDE), color = "cadetblue2", size = 0.25, alpha = 1) +
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank(),
          panel.background = element_blank(), axis.line = element_blank())

 

png('StarlingGlobal_GSApres.png', width = 1050, height = 650)
gg2
dev.off()

jpeg('StarlingGlobal_GSApres.jpeg')
gg2
dev.off()

 

# Open a pdf file
pdf("StarlingGlobal2.pdf") 
# 2. Create a plot
gg3
# Close the pdf file
dev.off() 

jpeg('StarlingGlobal.jpeg')
gg3
dev.off()

png('StarlingGlobal.png', width = 1050, height = 650)
gg3
dev.off()

<pre class="r"><code>
</code></pre>

<pre class="r"><code>
</code></pre>
