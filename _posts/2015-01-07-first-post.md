---
layout: post
title: Strategies for the Board Game Risk!
---

Some Code:

{% highlight r %}

m <- matrix(1:36, nrow = 6)

n <- matrix(numeric(36), nrow = 6)

for (i in 2:5) {
    for (j in 2:5) n[i,j] <- m[i-1,j-1]+m[i-1,j]+m[i-1,j+1]+m[i,j]+m[i+1,j-1]+m[i+1,j]+m[i+1,j+1]
}

{% endhighlight %}


![Image](/img/me.jpg)
