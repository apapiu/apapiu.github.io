---
title: "Minst"
output: html_document
---

We will be looking at the MNIST data set on Kaggle. The goal in this competition is to take an image of a handwritten single digit, and determine what that digit is.

Let's load up the data from the Kaggle  [competition]("https://www.kaggle.com/c/digit-recognizer/data"):


{% highlight r %}
library(ggplot2)
library(gridExtra)
train <- read.csv("train.csv")
{% endhighlight %}



{% highlight text %}
## Warning in file(file, "rt"): cannot open file 'train.csv': No such file or
## directory
{% endhighlight %}



{% highlight text %}
## Error in file(file, "rt"): cannot open the connection
{% endhighlight %}

Let's look at the dimension of train.


{% highlight r %}
dim(train)
{% endhighlight %}



{% highlight text %}
## [1] 42000   786
{% endhighlight %}



{% highlight r %}
head(train[1:8])
{% endhighlight %}



{% highlight text %}
##   label pixel0 pixel1 pixel2 pixel3 pixel4 pixel5 pixel6
## 1     1      0      0      0      0      0      0      0
## 2     0      0      0      0      0      0      0      0
## 3     1      0      0      0      0      0      0      0
## 4     4      0      0      0      0      0      0      0
## 5     0      0      0      0      0      0      0      0
## 6     0      0      0      0      0      0      0      0
{% endhighlight %}



{% highlight r %}
train$label <- as.factor(train$label)
{% endhighlight %}

So we basically have 42000 examples with each example having 785 features - pixels in this case and a label - the digit the image represent. It's actually pretty easy to reconstruct the image from the vector of pixels and we do this below. First I construct a square matrix form the vector and then use `image` to diplay it.


{% highlight r %}
digit <- matrix(as.numeric(train[8,-1]), nrow = 28) #look at one digit
{% endhighlight %}



{% highlight text %}
## Warning in matrix(as.numeric(train[8, -1]), nrow = 28): data length [785]
## is not a sub-multiple or multiple of the number of rows [28]
{% endhighlight %}



{% highlight r %}
image(digit, col = grey.colors(255))
{% endhighlight %}

![center](/Users/alexpapiu/GitHub/apapiu.github.iofigs/MINST/Users/alexpapiu/GitHub/apapiu.github.iounnamed-chunk-3-1.png) 

Looks like a 3! 

Since I want this to be a self-contained reproducible post I will split the training set into a test set and a training set just so I don't have to log into Kaggle to test the results. 



Let's play around and see if we can extract any features from the pixels that can be more informative. First I'd like to know more about average intensity - that is the average value of a pixel in an image for the different digits. Intuition tells me that the digit "1" will have on average have less intensity than say an "8". Below I create a new feature that is the average of the pixel intensity and then use the R function `aggregate` (extermely useful in these situations) to look at averages by label. Then I plot the average intensity using `qplot` frmo the `ggplot2` package.


{% highlight r %}
train$intensity <- apply(train[,-1], 1, mean) #takes the mean of each row in train

intbylabel <- aggregate (train$intensity, by = list(train$label), FUN = mean)

plot <- ggplot(data=intbylabel, aes(x=Group.1, y = x)) +
    geom_bar(stat="identity")
plot + scale_x_discrete(limits=0:9) + xlab("digit label") + 
    ylab("average intensity")
{% endhighlight %}

![center](/Users/alexpapiu/GitHub/apapiu.github.iofigs/MINST/Users/alexpapiu/GitHub/apapiu.github.iounnamed-chunk-4-1.png) 

As we can see there are some differences in intensity. The digit "1" is the less intense while the digit "0" is the most intense. So this new feature seems to have some predictive value if you wanted to know if say your digit is a "1" or no. But the problem of course is that **different peple write their digits differently**. We can get a sense of this by plotting the distribution of the average intensity by label.


{% highlight r %}
ggplot(train, aes(x=intensity)) + 
    geom_density(aes(group= label, fill=as.factor(label)), alpha=0.3)
{% endhighlight %}

![center](/Users/alexpapiu/GitHub/apapiu.github.iofigs/MINST/Users/alexpapiu/GitHub/apapiu.github.iounnamed-chunk-5-1.png) 

What can we observe from the histograms above? Well most intensity distributions seem roughly normally distributed but some have higher variance than others. The digit "1" seems to be the one people write most consistently across the board. Other than that the intensity feature isn't all that helpful. _However_ there is one little quirk hidden in the diagram above. Every distribution is unimodal (i.e. has one maxima) _except_ for the 7. Even before I analyize the data more closely I have a hunch as to why this might be: people write "7" in two different way. I know because I am one of the weirdos (well from an American perspective) who puts a little line through the middle - that's how I was taught attending school in Romania. I'll bet that second bump is caused by weirdos like me who line their sevens.


{% highlight r %}
p1 <- qplot(subset(train, label ==1)$intensity, binwidth = .75, 
            xlab = "Intensity Histogram for 1")

p2 <- qplot(subset(train, label ==4)$intensity, binwidth = .75,
            xlab = "Intensity Histogram for 4")

p3 <- qplot(subset(train, label ==7)$intensity, binwidth = .75,
            xlab = "Intensity Histogram for 7")

p4 <- qplot(subset(train, label ==9)$intensity, binwidth = .75,
            xlab = "Intensity Histogram for 9")

grid.arrange(p1, p2, p3,p4, ncol = 2)
{% endhighlight %}

![center](/Users/alexpapiu/GitHub/apapiu.github.iofigs/MINST/Users/alexpapiu/GitHub/apapiu.github.iounnamed-chunk-6-1.png) 


