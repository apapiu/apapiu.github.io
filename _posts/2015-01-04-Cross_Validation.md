---
layout: post
title: Cross Validation Error Pitfalls
---

<iframe src="https://drive.google.com/file/d/0B8Utq7NDKZdCWFBwRWpHNjJFLTg" width="640" height="480"></iframe>

Let's say you have 10 models that you want to test and roughly all models have the same cross validation error distribution: the Cross Validation Mean Squared Error is normally distributed with mean = 3 and standard deviation equal to .2. Since CV error is an average of a bunch of errors the normality assumption will always hold roughly speaking.   

In order to pick the best model we will look at the cross validation errors and then pick the one that gives the smallest error. **One of the mistakes I made early however was to assume that this error is an unbiased estimate of the true test error.** After a bit of thinking this is clearly not true. By choosing the minimum test error every time we do cross-validation on the 10 models we shift the distribution of the real test error to the left. 

In more formal terms: say you have a normally distributed random variable $X$. We can look at two other Random variables: 
$$\overline X = \frac{X_1+X_2+...+X_n}{n} $$

and $$ minX = min\{X_1, X_2, ..., X_n\} $$

<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"/>
<script src="your_map_files/htmlwidgets-0.5/htmlwidgets.js"></script>
<script src="your_map_files/jquery-1.11.1/jquery.min.js"></script>
<link href="your_map_files/leaflet-0.7.3/leaflet.css" rel="stylesheet" />
<script src="your_map_files/leaflet-0.7.3/leaflet.js"></script>
<link href="your_map_files/leafletfix-1.0.0/leafletfix.css" rel="stylesheet" />
<script src="your_map_files/leaflet-binding-1.0.0/leaflet.js"></script>

</head>
<body style="background-color:white;">
<div id="htmlwidget_container">
  <div id="htmlwidget-9329" style="width:100%;height:400px;" class="leaflet"></div>
</div>
<script type="application/json" data-for="htmlwidget-9329">{"x":{"calls":[{"method":"addTiles","args":["http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",null,null,{"minZoom":0,"maxZoom":18,"maxNativeZoom":null,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"continuousWorld":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":null,"unloadInvisibleTiles":null,"updateWhenIdle":null,"detectRetina":false,"reuseTiles":false,"attribution":"&copy; <a href=\"http://openstreetmap.org\">OpenStreetMap</a> contributors, <a href=\"http://creativecommons.org/licenses/by-sa/2.0/\">CC-BY-SA</a>"}]}],"setView":[[42.3601,-71.0589],12,[]]},"evals":[]}</script>
<script type="application/htmlwidget-sizing" data-for="htmlwidget-9329">{"viewer":{"width":"100%","height":400,"padding":0,"fill":true},"browser":{"width":"100%","height":400,"padding":0,"fill":true}}</script>
</body>
</html>


While $$\overline X$$ is an unbiased estimate, $$minX$$ is not, meaning the expected value of $$\overline X $$ will be the expected value of $X$ but the  expected value of $$minX$$ will be lower.


<iframe height="400" id="igraph" scrolling="no" seamless="seamless" src="https://plot.ly/~ouzor/33/count-vs-mpg/" width="450" frameBorder="0"></iframe>

Here's some code and the histograms.

<!DOCTYPE html>
<html>
<head>

</head>
<h1>I am a cat</h1>
</body>
</html>

{% highlight r %}
library(ggplot2)

set.seed(132)
normals <- t(replicate(10000, rnorm(10, mean = 3, sd = .2)))

qplot(apply(normals, 1, mean), binwidth = .005, xlim = c(2.5, 3.5), 
      xlab = "error ")

qplot(apply(normals, 1, min), binwidth = .005, xlim = c(2.5, 3.5),
      xlab = "error")

mean(apply(normals, 1, mean))

mean(apply(normals, 1, min))
{% endhighlight %}

This is the distribution for the average and it is unbiased having the same expected value as the original distribution. The average error in this case is `[1] 2.999636`, almost identical to the real value of 3.
![](/img/CVerror1.png)

This is the left-skewed distribution we get by taking the minimum of the Mean Squared Error. In this case the average is `[1] 2.692245` one underestimate by one standard deviation and a half.
![](/img/cverror2.png)

There are two ways to get an unbiased estimate in this case that come to mind. One is once you choose your model using cross-validation, you will do another cross-validation (or a whole epoch, say) only with the model you have - this way you get a distribution of the cross-validation error for the model you picked. Or the other, more basic option is to simply set aside a test set that is never looked at or touched in any way and then see what error you get on that set.

<div>
    <a href="https://plot.ly/~apapiu/0/" target="_blank" title="Ratio of Colege Educated Single Men to Women &lt;br&gt; (Hover for breakdown)" style="display: block; text-align: center;"><img src="https://plot.ly/~apapiu/0.png" alt="Ratio of Colege Educated Single Men to Women &lt;br&gt; (Hover for breakdown)" style="max-width: 100%;width: 1188px;"  width="1188" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="apapiu:0"  src="https://plot.ly/embed.js" async></script>
</div>



