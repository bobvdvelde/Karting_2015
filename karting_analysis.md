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

'T was one of Daan's last days as a bachelor, and he decided he had little too lose: Let's go karting!

But who won, and why? Let's use this as a quick-'n-dirty presentation of R-markdown, the wonderous world of ANCOVA and how people learn how to go-karting!

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
ggplot(kartdata,aes(x=df_num,y=avg, colour=name))+geom_line(aes(group=name),size=3)+geom_point(line="white",size=6)+scale_y_reverse()+xlab("round")+ ylab("inverted time")+ggtitle("Personal round averages throughout the rounds")
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

kable(wide_kart[order(wide_kart$improvement,decreasing=T),c("name","improvement")],caption = "Time improvement between training and round 10 averages")
```



Table: Time improvement between training and round 10 averages

     name        improvement
---  ---------  ------------
3    Steven            12.69
8    Jeroen            11.65
6    Bob               11.23
1    Dietrick          11.16
7    Joost             11.06
5    Ruben             10.90
4    Paul               9.40
2    Daan               9.06
9    Alice              5.71
10   Thijs              4.71

So did the best learner win? It's not quite clear from that little diddy. So we'll throw it through a simple lm, just for the heck of it. But wait, how do the extra rounds figure into this (if your quick, you can do more rounds...)


```r
roundnames <- character()
for(n in paste(".",unique(kartdata$df_num),sep="")) for (ni in paste("X",1:15,n,sep="")) if (ni %in% names(wide_kart)) roundnames <- c(roundnames,ni)
wide_kart$rounds <- apply(wide_kart[roundnames],1,function(x){sum(!is.na(x))})

m <- lm(avg.Finals~avg.Training+improvement+rounds,wide_kart)
summary(m)
```

```
## 
## Call:
## lm(formula = avg.Finals ~ avg.Training + improvement + rounds, 
##     data = wide_kart)
## 
## Residuals:
##        Min         1Q     Median         3Q        Max 
## -2.870e-14  2.114e-15  3.916e-15  5.117e-15  5.754e-15 
## 
## Coefficients:
##                Estimate Std. Error    t value Pr(>|t|)    
## (Intercept)  -1.798e-13  2.745e-13 -6.550e-01    0.537    
## avg.Training  1.000e+00  3.853e-15  2.595e+14   <2e-16 ***
## improvement  -1.000e+00  3.494e-15 -2.862e+14   <2e-16 ***
## rounds        1.103e-15  2.671e-15  4.130e-01    0.694    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.294e-14 on 6 degrees of freedom
## Multiple R-squared:      1,	Adjusted R-squared:      1 
## F-statistic: 2.045e+29 on 3 and 6 DF,  p-value: < 2.2e-16
```

```r
plotreg(m)
```

```
## Model 1: bars denote 0.5 (inner) resp. 0.95 (outer) confidence intervals (computed from standard errors).
```

![](karting_analysis_files/figure-html/unnamed-chunk-5-1.png) 

# The End (?)

That's it: the better your start, the more rounds you do, the better you end! Now theres a first-mover advantage for you!
