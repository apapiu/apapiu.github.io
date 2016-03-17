---
layout: post
title: Cross Validation Error Pitfalls
---

<!DOCTYPE html>
<meta charset="utf-8">
<style>

.states {
  fill: none;
  stroke: #fff;
  stroke-linejoin: round;
}

</style>
<body>
<script src="//d3js.org/d3.v3.min.js"></script>
<script src="//d3js.org/topojson.v1.min.js"></script>
<script>

var width = 960,
    height = 500;

var fill = d3.scale.log()
    .domain([10, 500])
    .range(["brown", "steelblue"]);

var path = d3.geo.path();

var svg = d3.select("body").append("svg")
    .attr("width", width)
    .attr("height", height);

d3.json("/mbostock/raw/4090846/us.json", function(error, us) {
  if (error) throw error;

  svg.append("g")
      .attr("class", "counties")
    .selectAll("path")
      .data(topojson.feature(us, us.objects.counties).features)
    .enter().append("path")
      .attr("d", path)
      .style("fill", function(d) { return fill(path.area(d)); });

  svg.append("path")
      .datum(topojson.mesh(us, us.objects.states, function(a, b) { return a.id !== b.id; }))
      .attr("class", "states")
      .attr("d", path);
});

</script>

Let's say you have 10 models that you'd want to test and roughly all models have the same cross validation error distribution: the Cross Validation Mean Squared Error is normally distributed with mean = 3 and standard deviation equal to .2. Since CV error is an average of a bunch of errors the normality assumption will always hold roughly speaking.   

In order to pick the best model we will look at the cross validation errors and then pick the one that gives the smallest error. **One of the mistakes I made early however was to assume that this error is an unbiased estimate of the true test error.** After a bit of thinking this is clearly not true. By choosing the minimum test error every time we do cross-validation on the 10 models we shift the distribution of the real test error to the left. 

In more formal terms: say you have a normally distributed random variable $X$. We can look at two other Random variables: 
$$\overline X = \frac{X_1+X_2+...+X_n}{n} $$

and $$ minX = min\{X_1, X_2, ..., X_n\} $$


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




