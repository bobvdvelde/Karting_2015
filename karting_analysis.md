# DZWS_karting
rnvdv  
Monday, May 04, 2015  


```
## Version:  1.34
## Date:     2014-10-31
## Author:   Philip Leifeld (University of Konstanz)
## 
## Please cite the JSS article in your publications -- see citation("texreg").
```


# So we went karting....

'T was one of Daan's last days as a bachelor, and he decided he had little to lose: Let's go karting!

But who won, and why? Let's use this as a quick-'n-dirty presentation of R-markdown, the wonderous world of linear modelling and how people learn how to go-karting!

## Let's start with the end: the winners

We'll get it over with: Daan didn't quite win (and neither did I). So who did?


```r
kable(finals[c("pos","name","avg")])
```



 pos  name          avg
----  ---------  ------
   1  Dietrick    38.42
   2  Jeroen      38.82
   3  Bob         39.13
   4  Paul        39.08
   5  Ruben       39.37
   6  Daan        39.47
   7  Joost       40.01
   8  Steven      40.23
   9  Alice       45.89
  10  Thijs       48.33

```r
finals$name <- factor(finals$name,levels=(finals$name[order(finals$pos)]),ordered = T,)
ggplot(finals,aes(x=name,y=avg))+stat_identity(geom="bar")
```

![](karting_analysis_files/figure-html/unnamed-chunk-2-1.png) 

That is a nice little show of the endgame, but what got us there? It's not the 0.479 correlation between our training positions and end positions. So what happened?

# What happened

If we look at the positions starting from our training rounds and leading up to the finals, we get quite a competitive field. Especially between the third qualification and the Finals, things seem to be changing rapidly. 

![](karting_analysis_files/figure-html/unnamed-chunk-3-1.png) 

## Personal improvement

If we look at it from the perspective of personal improvement, what does that give us?


```r
ggplot(kartdata,aes(x=df_num,y=avg, colour=name))+
  geom_line(aes(group=name),size=3)+
  geom_point(line="white",size=6)+
  scale_y_reverse()+xlab("round")+ ylab("inverted time")+ggtitle("Personal round averages throughout the rounds")
```

![](karting_analysis_files/figure-html/unnamed-chunk-4-1.png) 

```r
wide_kart <- reshape(data = kartdata,direction="wide", timevar="df_num",idvar="name")
```

```
## Warning in reshapeWide(data, idvar = idvar, timevar = timevar, varying =
## varying, : multiple rows match for df_num=Training: first taken
```

```r
wide_kart$improvement <- wide_kart$avg.Training - wide_kart$avg.Finals

kable(wide_kart[order(wide_kart$improvement,decreasing=T),c("name","improvement","avg.Training" ,"avg.Finals","pos.Training","pos.Finals")],caption = "Time improvement between training and round 10 averages")
```



Table: Time improvement between training and round 10 averages

     name        improvement   avg.Training   avg.Finals   pos.Training   pos.Finals
---  ---------  ------------  -------------  -----------  -------------  -----------
3    Steven            12.69          52.92        40.23              3            8
8    Jeroen            11.65          50.47        38.82              8            2
6    Bob               11.23          50.36        39.13              6            3
1    Dietrick          11.16          49.58        38.42              1            1
7    Joost             11.06          51.07        40.01              7            7
5    Ruben             10.90          50.27        39.37              5            5
4    Paul               9.40          48.48        39.08              4            4
2    Daan               9.06          48.53        39.47              2            6
9    Alice              5.71          51.60        45.89              9            9
10   Thijs              4.71          53.04        48.33             10           10

So did the best learner win? It's not quite clear from that little diddy. So we'll throw it through a simple lm, just for the heck of it. But wait, how do the extra rounds figure into this (if your quick, you can do more rounds...)


```r
roundnames <- character()
for(n in paste(".",unique(kartdata$df_num),sep="")) for (ni in paste("X",1:15,n,sep="")) if (ni %in% names(wide_kart)) roundnames <- c(roundnames,ni)
wide_kart$rounds <- apply(wide_kart[roundnames],1,function(x){sum(!is.na(x))})

m <- lm(avg.Finals~avg.Training*rounds,wide_kart)
summary(m)
```

```
## 
## Call:
## lm(formula = avg.Finals ~ avg.Training * rounds, data = wide_kart)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.56087 -0.12315  0.06638  0.23176  0.38687 
## 
## Coefficients:
##                       Estimate Std. Error t value Pr(>|t|)    
## (Intercept)         -857.78589  109.01823  -7.868 0.000223 ***
## avg.Training          17.93570    2.09836   8.547 0.000141 ***
## rounds                17.67934    2.19749   8.045 0.000197 ***
## avg.Training:rounds   -0.35355    0.04234  -8.351 0.000160 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.4254 on 6 degrees of freedom
## Multiple R-squared:  0.9894,	Adjusted R-squared:  0.9841 
## F-statistic: 187.1 on 3 and 6 DF,  p-value: 2.578e-06
```

```r
plotreg(m)
```

```
## Model 1: bars denote 0.5 (inner) resp. 0.95 (outer) confidence intervals (computed from standard errors).
```

![](karting_analysis_files/figure-html/unnamed-chunk-5-1.png) 

# The End (?)

That's it: the better your start, the more rounds you do, the better you end! 
Now theres a first-mover advantage for you!
