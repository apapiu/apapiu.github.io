---
layout: post
title: Cross Validation Error Pitfalls
---


<div id="example"></div>

	<script type="text/javascript" src="http://d3js.org/d3.v3.min.js"></script>
	<script type="text/javascript">
	
				var w = 500;
				var h = 500;
				var barPadding = 3;
				
				var dataset = [3, 20, 5, 19, 34, 18, 23, 14];
	          
	          
				//Create SVG element - this is the whole plot
				var svg = d3.select("body")
							.append("svg")
							.attr("width", w)
							.attr("height", h);
	          
	          svg.selectAll("rect")
	            .data(dataset)
	            .enter()
	            .append("rect")
	            .attr("y", function(d, i){return i*35})
	            .attr("x", 0)
	            .attr("width", function(i){return i*10})
	            .attr("height", 30)
	            .attr("fill", "teal")
	            .transition()
	            .duration(2000)
	            .attr("height",15)
	            .transition()
	            .duration(2000)
	            .delay(2000)
	            .attr("x", function(d, i){return i*33})
	            .attr("y", 0)
	            .attr("height", function(i){return i*10})
	            .attr("width", 30)
	            .attr("fill", "black")
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




<div id="example"></div>

	<script type="text/javascript" src="http://d3js.org/d3.v3.min.js"></script>
	<script type="text/javascript">
	
			var w = 500;
			var h = 500;
			var barPadding = 3;
			
			var dataset = [3, 20, 5, 19, 34, 18, 23, 14];
	  
	  
			//Create SVG element - this is the whole plot
			var svg = d3.select("body")
						.append("svg")
						.attr("width", w)
						.attr("height", h);
	  
	  svg.selectAll("rect")
	    .data(dataset)
	    .enter()
	    .append("rect")
	    .attr("y", function(d, i){return i*35})
	    .attr("x", 0)
	    .attr("width", function(i){return i*10})
	    .attr("height", 30)
	    .attr("fill", "teal")
	    .transition()
	    .duration(2000)
	    .attr("height",15)
	    .transition()
	    .duration(2000)
	    .delay(2000)
	    .attr("x", function(d, i){return i*33})
	    .attr("y", 0)
	    .attr("height", function(i){return i*10})
	    .attr("width", 30)
	    .attr("fill", "black")
	</script>




