---
layout: post
title: First post!
---

4.4 Notation: All the summations in this exercise denoted by $\sum_q$ will be from $q = 0$ to $Q$. 

a) $\sigma$ will be the standard deviation of the noise $\sigma \epsilon_n$. We need to normalize $f$ since otherwise $f(x_n)$ could have different values and then $\sigma$ would have a different impact for each $y_n$. So basically normalizing distributes the noise evenly among our data points. Let's see what we need to normalize by. Let $f(x) = \sum_q a_q L_q(x)$, where $L_q$ is the $q$-th Legendre polynomial. Now $$E_x[f^2] = E_x[(\sum_q a_q L_q(x)^2] = .5\int_{-1}^{1}(\sum_q a_q L_q(x))^2 dx =\sum_q \frac{a_q^2}{2q+1}$$ where the last equality follows from 4.3 e) after we do the squaring. 
So $$E_{a,x}[f^2] = E_a[\sum_q \frac{a_q^2}{2q+1}]=\sum_q \frac{1}{2q+1}$$ since $E_a[a_q^2] = 1$. So in order to normalize $f$ we need to divide by $(\sum_q \frac{1}{2q+1})^{1/2}$.

b) Given $N$ data points of the form $(x_n, y_n)$ we create a matrix $X$ where the $i$-th row in $X$ is $(L_0(x_i), L_1(x_i),...,L_Q(x_i))$ and we let $y = (y_1,...,y_n)$ Then $g_Q(x) = \sum_q w_q L_q(x)$ where $w_q$ is the $q$-th entry in $w = ((X^TX)^{-1}X^T)y$. Basically we are doing linear regression in the $z$-space where the basis is made up of the Legendre polynomials of degree less than or equal to $Q$. Take $Q = 2$ or $Q = 10$ to obtain $g_2$ and $g_{10}$ respectively. We actually implement this in part d).

c) $E_{out}(g_{10}) = E_x[(g_{10} - f(x) - \sigma\epsilon)^2]$ where $f(x)$ is a degree $Q$ polynomial written as $f(x) = \sum_q a_q L_q(x)$ and $g_{10} = \sum_q b_q L_q(x)$ where $b_q$ is 0 for $q\geq 11$ - we write $g_{10}$ this way since it makes computations easier.
Now $$E_x[(g_{10} - f(x) - \sigma\epsilon)^2] = E_x[(g_{10}(x)-f(x))^2]+E_x[(\sigma \epsilon)^2] = E_x[(g_{10}(x)-f(x))^2] +\sigma^2 = \sum_q \frac{(b_q-a_q)^2}{2q+1} +\sigma^2$$ where again the last equality follows from the orthogonaility of Legendre polynomials in 4.3 e).

d) Let $Q$ be the degree of our target polynomial $f$, $N$ - the number of noisy examples generated and $\sigma$ the standard deviation of the noise. We will now fit a second degree polynomial $g_2$ and a tenth degree polynomial $g_{10}$ to the data we generated while varying $Q$, $N$, and $\sigma$ and see what insights we can get into overfitting. For a fixed triple $(Q, N, \sigma)$ the overfit measure will  be $E_{out}(H_{10}) - E_{out}(H_2)$ - the difference in out of sample error in fitting the data with a tenth and a second degree polynomial averaged over $500$ runs - the averaging is necessary since, as we shall see, the variance of $E_{out}(g_2)$ and $E_{out}(g_{10})$ can be high.         
    To begin let's choose $Q = 10, N = 80, \sigma^2 = .1$. The results of out fit can be seen in **Figure 1**. We see that $g_{10}$ performs better than $g_2$ -- we get $E_{out}(g_2) = 0.0970$ and $E_{out}(g_{10}) = 0.0221$. For the second degree polyomial fit the main contributor to $E_{out}$ is the bias - the model cannot capture the complexity of the target function - it does however do a good job of not fitting the noise. The opposite is true for $g_{10}$. In this case the model does have enough complexity to fit the data but since noise is present it fits some of the noise in as well - thus the variance of the model increases.

```{r, echo=FALSE, fig.cap= "The black curve is the original taget function, the blue circles are the noisy data points, the green curve is the fitted second degree polynomial $g_2$ and the red curve is $g_{10}$. Since noise is minimal $g_{10}$ does a good job in capturing the signal while $g_2$ has high bias."}
library(polynom)
set.seed(127)
q = 10 #degree of poly
n = 80 #number of examples
s = .1 #this is really sigma squared

x <- runif(n, min = -1, max = 1)
    
poly <- polynomial(rnorm(n = q+1)) #poly

epsi <- rnorm(n) #noise

y <- predict(poly, x) + sqrt(s)*epsi #values of poly +noise
df <- data.frame(x, y)

model_2 <- lm(y ~ poly(x,2), data = df)
model_10 <- lm(y ~ poly(x, 10), data = df)


#now we want E_out
test <- runif(1000, min = -1, max = 1)
testdf <- data.frame(test)
colnames(testdf)[1]<-"x"
plot(test, predict(poly,test), ylim = c(-2,2), xlab = "x",
     ylab = "y", main = "Fig 1: Not so Noisy 5th Degree Poly with 80 data points")
newy_2 <- predict(model_2, newdata = testdf)
newy_10 <- predict(model_10, newdata = testdf)
points(test, newy_10, col = "red")
points(test, newy_2, col = "green")
points(x, y, col = "blue")

```

One would thus expect $g_{10}$ to perform more poorly as amount of noise increases and the number of examples decreases and this will in fact be the case. Let's do an examples with $Q = 5, N = 40, \sigma^2 = 1$. This is seen in **Figure 2**. In this case we get $E_{out}(g_2) = .061$ and $E_{out}(g_{10} = 1.12$. $g_2$ is the clear winner here - it still showcases some bias but it deals with the noise much better. $g_{10}$ fits too much of the noise leading to an increase in the variance and thus an increase in $E_{out}$.

```{r, echo = FALSE, fig.cap = "The color scheme is the same as in Figure 1. In this case $g_{10}$ overfits the data - it is too sensitive to the noise", fig.height=4}
q= 5 #degree of poly
n = 40 #number of examples
s = 1 #this is really sigma squared
set.seed(152)
x <- runif(n, min = -1, max = 1)
    
poly <- polynomial(rnorm(n = q+1)) #poly

epsi <- rnorm(n) #noise

y <- predict(poly, x) + sqrt(s)*epsi #values of poly +noise
df <- data.frame(x, y)

model_2 <- lm(y ~ poly(x,2), data = df)
model_10 <- lm(y ~ poly(x, 10), data = df)


#now we want E_out
test <- runif(1000, min = -1, max = 1)
testdf <- data.frame(test)
colnames(testdf)[1]<-"x"
plot(test, predict(poly,test), ylim = c(-2,3), xlab = "x",
     ylab = "y", main = "Fig 2: Very Noisy 10th Degree Poly with 40 data points")
newy_2 <- predict(model_2, newdata = testdf)
newy_10 <- predict(model_10, newdata = testdf)
points(test, newy_10, col = "red")
points(test, newy_2, col = "green")
points(x, y, col = "blue")
```


However there will be a lot of variability since we are choosing a new taget function each time. Thus for Given $N, Q, \sigma^2$ we ran 500 linear regressions. In **Figure 3** we show results forfor $Q = 20, N = 80$ and $\sigma^2 = 1$. Looking at the histogram of the overfit measures we see that they are very skewed - the mean is significantly larger than the median. This is because there are a few times when $g_{10}$ messes up really badly and gives large $E_{out}$ ( as high as $40$ for the Mean Squared Error) and this affects the mean but not the median. In particular we get the mean of the overfit measure to be $.389$ and the median $.029$. So $g_2$ performs slightly better than $g_{10}$ in this scenario and $g_{2}$ is also more stable - the maximum $E_{out}(g_2)$ obtained is $3.5$. 

```{r, echo = FALSE, fig.cap = "Skewed Distrubution of the Overfit Measure" }
set.seed(124)
Overfit <- function(q, n, s) {

    x <- runif(n, min = -1, max = 1)
    
    poly <- polynomial(rnorm(n = q+1)) #poly
    
    epsi <- rnorm(n) #noise
    
    y <- predict(poly, x) + sqrt(s)*epsi #values of poly +noise
    df <- data.frame(x, y)
    
    model_2 <- lm(y ~ poly(x,2), data = df)
    model_10 <- lm(y ~ poly(x, 10), data = df)
    
    
    #now we want E_out
    test <- runif(10000, min = -1, max = 1)
    testdf <- data.frame(test)
    colnames(testdf)[1]<-"x"
    newy_2 <- predict(model_2, newdata = testdf)
    newy_10 <- predict(model_10, newdata = testdf)
    
    Eout_2 <- sum((predict(poly,test)-newy_2)^2)/10000
    
    Eout_10 <- sum((predict(poly,test)-newy_10)^2)/10000
    
    return(c(Eout_2, Eout_10, Eout_10 - Eout_2))
}
hist(replicate(500,Overfit(20, 80, 1)[3]), xlab = "Overfit Measure", main = NULL)

```

In **Figure 4** we vary the magnitude of the noise and keep everything else Equal $Q = 10, N = 80$. As we can see - as the ammount of noise increases the overall trend is for the overfit measure to increase as well.

```{r, echo = FALSE, fig.cap= "Varying Noise for Q = 10, N = 80 - An increase in noise leads to overfitting."}
noise = c(0, 0.25, 0.5, 0.75, 1, 1.25, 1.5, 1.75, 2)

overfit = c(-0.160562918, -0.058358514, -0.004537344,  0.173664946,  0.53917195,  1.760951385,  0.742196099,  0.841950389,  0.774870422)

plot(noise, overfit, ylab = "Mean of Overfit Measure")
abline(h = 0)
```

**Figure 5** shows the results of  varying the number of examples with $\sigma^2 = 0.5$ and $Q = 10$. As expected, the more examples we have the more stable the more complex $g_{10}$ model is. In fact if we have more than roughly $100$ examples, $g_{10}$ outperforms $g_2$.

```{r, echo = FALSE, fig.cap= "Varying Number of Exampes for $Q = 10$, $s^2 = .5$ - An increase number of examples leads to a decrease in  the overfit measure."}
N <- c(60, 80 ,100, 120, 140, 160) 
 
overfit <- c(1.32262787, 0.08528971, -0.06047749, -0.08422392, -0.11325889, -0.13347710)

plot (N, overfit, ylab = "Mean of Overfit Measure")
abline(h = 0, v = 0)

```


And lastly we vary degree of $f$ and fix $\sigma^2 = .5$ and $N = 80$. The results can be seen in **Figure 6** In this case there is a lot of variability but overall increasing the degree results in a decrease in the overfit measure. However I think this is not so much because of a decrease in variance for $g_{10}$ as it is a consequence of an increase in bias for $g_2$.

```{r, echo = FALSE, fig.cap= "Varying the degree of f for $N = 80$, $s^2 = .5$"}
 
Q <- c(5, 10, 15, 20, 25, 30)

overfit <- c(0.30510142, 0.26382851,  0.69782394, 0.04901888,  0.08816353, -0.06478407)

plot(Q, overfit, ylab = "Mean of Overfit Measure")
abline(h = 0)
``` 
