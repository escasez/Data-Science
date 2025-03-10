The Impact of Natural Events on the American Population and Economy
========================================================

<p>
The goal of this analysis is to understand and analyze the impact of natural events on the US economy and its population. The dataset at hand consist of observations taken through out America for the last couple of decades. The analysis pertains to understanding the impact of these events on humans and the econonmy as a whole. Damages to humans consist of both fatal and non fatal observations. Where as the damage to the economy is categorized into property and crop damage. The analysis is to find out:
<ul>
  <li>which event has caused the most number of human infliction (both fatal and non fatal)</li>
  <li>which event has caused the most impact on the economy (both in terms of property and crop damage)</li>
</ul>
</p>




## Data Processing
<p>
The data was retrieved from the provided dataset 'repdata-data-StormData.csv' with importance given to the following four variables:
<ul>
  <li>EVENTYPE - Event Types</li>
  <li>FATALITIES - Number of fatalities</li>
  <li>INJURIES - Number of injuries</li>
  <li>PROPDMG - Property Damage</li>
  <li>PROPDMGEXP - Magnitude of property damage</li>
  <li>CROPDMG - Crop Damage</li>
  <li>CROPDMGEXP - Magnitude of Crop damage</li>
</ul>

Processing was only done to these two columns: PROPDMG and CROPDMG. Processing consisted of iterating through each observation and pre-process the
property damage expenses - PROPDMG by taking into account the expense magnitude 'PROPDMGEXP'. Similary, for the crop damage expense variable - CROPDMG  processing was done by taking into account the expense magnitude 'CROPDMGEXP'.
</p>

```r
# Read the csv file contents
df = read.table("C:/Users/pbtmq/Desktop/Reproducible Research/Week 4/repdata-data-StormData.csv", 
    sep = ",", header = TRUE)
```


<h3>1: Analyze the impact of events on humans and record the most harmful event</h3>

```r

# Group event type by total number of fatalities and injuries. Also include
# a new column which will be the sum of total fatalities and total injuries
# for each event.
gdf = ddply(df, c("EVTYPE"), function(x) c(FAT = sum(x$FATALITIES), INJ = sum(x$INJURIES), 
    TOTAL = sum(sum(x$FATALITIES) + sum(x$INJURIES))))

# Order the dataset by the total number of human afflictions
gdf = gdf[order(gdf$TOTAL, decreasing = TRUE), ]

# Show only those events that have atleast one human affliction and take the
# top 10 of such events
gdfTopTen = gdf[which(gdf$TOTAL > 0), ][1:10, ]

# Rearrange the factor levels
gdfTopTen$EVTYPE = factor(gdfTopTen$EVTYPE, levels = gdfTopTen$EVTYPE)

# show a bar graph using ggplot
strwr <- function(str) gsub(" ", "\n", str)
p1 = ggplot(gdfTopTen, aes(x = EVTYPE, y = TOTAL)) + geom_bar(stat = "identity") + 
    scale_x_discrete("Events", breaks = levels(gdfTopTen$EVTYPE), labels = strwr(levels(gdfTopTen$EVTYPE))) + 
    xlab("Events") + ylab("Total Human Injuries & Fatalities") + ggtitle("Top 10 Events in USA with most Human Injury/Fatality")


p2 = ggplot(gdfTopTen, aes(x = EVTYPE, y = TOTAL)) + geom_bar(stat = "identity") + 
    scale_x_discrete("Events", breaks = levels(gdfTopTen$EVTYPE), labels = strwr(levels(gdfTopTen$EVTYPE))) + 
    scale_y_log10() + xlab("Events") + ylab("Total Human Injuries & Fatalities") + 
    ggtitle("Top 10 Events in USA with most Human Injury/Fatality (Log10)")


grid.arrange(p1, p2, ncol = 1, nrow = 2)
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 



<h3>2: Analyze the impact of events on US economy and record the most harmful event.</h3>

```r
# Include only those columns that we are interested in
gdfExp = df[, (c("EVTYPE", "PROPDMG", "PROPDMGEXP", "CROPDMG", "CROPDMGEXP"))]

# Iterate through each observation and pre-process the property damage
# expenses - PROPDMG. Here we are taking into account the magnitude
# 'PROPDMGEXP' and assigning the correct expense in 'PROPDMG'
processPropExpense = function(x) {
    
    if (toupper(x[3]) == "M") {
        x[2] = as.numeric(x[2]) * 1e+06
    } else {
        if (toupper(x[3]) == "K") {
            x[2] = as.numeric(x[2]) * 1000
        } else {
            if (toupper(x[3]) == "H") {
                x[2] = as.numeric(x[2]) * 100
            } else {
                if (toupper(x[3]) == "B") {
                  x[2] = as.numeric(x[2]) * 1e+09
                } else {
                  if (toupper(x[3]) == "8") {
                    x[2] = as.numeric(x[2]) * 1e+08
                  } else {
                    if (toupper(x[3]) == "7") {
                      x[2] = as.numeric(x[2]) * 1e+07
                    } else {
                      if (toupper(x[3]) == "6") {
                        x[2] = as.numeric(x[2]) * 1e+06
                      } else {
                        if (toupper(x[3]) == "5") {
                          x[2] = as.numeric(x[2]) * 1e+05
                        } else {
                          if (toupper(x[3]) == "4") {
                            x[2] = as.numeric(x[2]) * 10000
                          } else {
                            if (toupper(x[3]) == "3") {
                              x[2] = as.numeric(x[2]) * 1000
                            } else {
                              if (toupper(x[3]) == "2") {
                                x[2] = as.numeric(x[2]) * 100
                              } else {
                                if (toupper(x[3]) == "1") {
                                  x[2] = as.numeric(x[2]) * 10
                                } else {
                                  if (toupper(x[3]) == "-" | toupper(x[3]) == 
                                    "?" | toupper(x[3]) == "+" | toupper(x[3]) == 
                                    "0") {
                                    x[2] = as.numeric(x[2]) * 1
                                  }
                                }
                              }
                            }
                          }
                        }
                      }
                    }
                  }
                }
            }
        }
    }
    
    return(as.numeric(x[2]))
}

# Iterate through each observation and pre-process the crop damage expenses
# - CROPDMG. Here we are taking into account the magnitude 'CROPDMGEXP' and
# assigning the correct expense in 'CROPDMG'
processCropExpense = function(x) {
    if (toupper(x[5]) == "M") {
        x[4] = as.numeric(x[4]) * 1e+06
    } else {
        if (toupper(x[5]) == "K") {
            x[4] = as.numeric(x[4]) * 1000
        } else {
            if (toupper(x[5]) == "B") {
                x[4] = as.numeric(x[4]) * 1e+09
            } else {
                if (toupper(x[5]) == "2") {
                  x[4] = as.numeric(x[4]) * 100
                } else {
                  if (toupper(x[5]) == "?" | toupper(x[5]) == "0") {
                    x[4] = as.numeric(x[4]) * 1
                  }
                }
            }
        }
    }
    
    return(as.numeric(x[4]))
}

# Apply the preprocessing algorithm to the property expense - PROPDMG
gdfExp$PROPDMG = apply(gdfExp, 1, processPropExpense)

# Apply the preprocessing algorithm to the property expense - CROPDMG
gdfExp$CROPDMG = apply(gdfExp, 1, processCropExpense)

# Group event type by total crop and property damage. Also include a new
# column that contains the sum of total crop and total property damage for
# each event
gdfExpPropGrp = ddply(gdfExp, c("EVTYPE"), function(x) c(PROP = sum(x$PROPDMG), 
    CROP = sum(x$CROPDMG), TOTAL = sum(sum(x$PROPDMG) + sum(x$CROPDMG))))

# Order the dataset descreasingly by the total damage expense
gdfExpPropGrp = gdfExpPropGrp[order(gdfExpPropGrp$TOTAL, decreasing = TRUE), 
    ]

# Take the top 10 events with the most amount of damage expense
gdfExpPropGrp = gdfExpPropGrp[1:10, ]

# Rearrange the factor levels for graphing
gdfExpPropGrp$EVTYPE = factor(gdfExpPropGrp$EVTYPE, levels = gdfExpPropGrp$EVTYPE)

# show a bar using ggplot
strwr <- function(str) {
    str = gsub(" ", "\n", str)
    str = gsub("/", "\n", str)
    return(str)
}
ggplot(gdfExpPropGrp, aes(x = EVTYPE, y = TOTAL)) + geom_bar(stat = "identity") + 
    scale_x_discrete("Events", breaks = levels(gdfExpPropGrp$EVTYPE), labels = strwr(levels(gdfExpPropGrp$EVTYPE))) + 
    xlab("Events") + ylab("Total Damage Expense") + ggtitle("Top 10 Events in USA with most Damage Expense")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

```r

```



## Results
From the first two graphs it is evident that the most destructive event on humans is Tornado, followed by Excessive Heat. From the last graph it is evident that the most amount of damage was done to the economy by Flood, followed by Hurricane/Typhoons.
