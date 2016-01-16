---
layout: post
title: First post!
---

Let's say you have 10 models that you want to test and roughly all models have the same cross validation error distribution: the CVMSE is normally distributed with mean = 3 and standard deviation equal to .2. Since CV error is an average of a bunch of errors the normaility assumption will always hold roughly speaking.   

In order to pick the best model we will look at the cross validation errors and then pick the one that gives the smallest error. **One of the mistakes I made early however was to assume that this error is an unbiased estimate of the true test error.** After a bit of thinking this is clearly not true. By choosing the minimum test error every time we do cross-validation on the 10 models we shift the distribution of the real test error to the left. 

In more formal terms: say you have a normally distributed random variable $X$. We can look at two other Random variables: 
$$\overline X = \frac{X_1+X_2+...+X_n}{n} $$

and $$ minX = min\{X_1, X_2, ..., X_n\} $$

While $$\overline X$$ is an unbiased estimate, $$minX$$ is not, meaning the expected value of $$\overline X $$ will be the expected value of $X$ but the  expected value of $$minX$$ will be lower.


Here's some code and the histograms. 

{% highlight r %}
set.seed(132)
normals <- t(replicate(1000, rnorm(10, mean = 3, sd = .2)))

hist(apply(normals, 1, mean), xlab = "Error")

hist(apply(normals, 1, min), xlab = "Error")

mean(apply(normals, 1, mean))

mean(apply(normals, 1, min))
{% endhighlight %}


As we can see always taking the minimum MSE does not correctly estimate the true MSE of 3. In fact it siginificatly underestimates it roughly by one standard deviation and a half (??is there a way to get this in general??). Therefore it is important to set a validation set aside and only run the model on it once in order to get an unbiased estimate of the real test error. 
