---
layout: post
title: Checking if a Die is fair - Bayesian Edition
---

If I gave you a die and and asked you if it's a fair die what would you do? Well most likely you would start rolling a few times and look at the data. Yet even with data it's hard to know for sure! Even a fair die can lead to many different outcomes. And also if we're being honest no die is perfectly fair is it. So we're in a bit of a pickle.

The most common way of dealing with this problem is using hypothesis testing.

- $H_0$: All faces are exactly equally likely
- $H_1$: Not all faces are exactly equally likely

**To make things a bit simpler and easier to visualize we will look at a 3 sided die**, worry not however most of what we'll do easily generalizes to a 6 sided die. It's really the jump from 2 to 3 dimensions that's the harder part. Also 3 sided die do actually exist. (share pic)


#### The Frequentist Approach:
We are interested in finding information about $\theta = (\theta_1, \theta_2, \theta_3)$ where $\theta_1$ is the probability of getting a 1 etc. Note that $\theta_1+\theta_2+\theta_3 = 1$ so there are some dependencies within $\theta$. 

The general frequentist approach is described below. Feel free to skip if it bores you. With this notation our hypothesis becomes:

- $H_0: \theta_1 =  \theta_2 =  \theta_3 = 1/3$
- $H_1$: $H_0$ is not true

The general idea is to obtain some kind of statistic based on the obtained sample that would provide evidence for $H_0$ or $H_1$. 

There are a few issues with this.

- Fristly no one really expects a real life die to be perfectly fair - so we know almost for a fact that $H_0$ is not true! 

- It's not exactly clear how to build a one-dimensional statistic that would be helpful in rejecting the null($H_0$). It turns out that a certain sum of squares _approximately_ follows a chi-squared distribution under the null hypothesis so we can use that as our statistic. The justification for all of that is quite convoluted however - I'd be willing to bet that most data scientists couldn't justify the aymsptotic behavior needed here. 

- Thirdly the degrees of freedom computation involved in testing more complicated hypotheses test can add one more layer of confusion especially (see here).

#### A Bayesian Way:
Hopefully I have convinced that it would be good to at least have an alternative to the aprroach above. In comes Bayes theorem! Generally Bayesian calculations have a bad rap of being very computationally expensive but this really isn't the case here - in fact both the math and the inference is fairly straighforward. Let's look at the formula:

$$p(\theta|x) \propto p(x|\theta) p(\theta) $$

And now assume we have obtained some data by rolling the die **50 times - (20,16,14) which is the counts for 1s, 2s, and 3s respectively.** The vector can be replaces with n, (k1,k2,k3) below - for more generality.

- $p(\theta)$ will be our prior and will incorporate any prior knowledge we have about the fairness or loadedness of the die. We will for the most part use a uniform prior $p(\theta) \propto 1$ which says that any combination of $\theta_1, \theta_2, \theta_3$ is equally likely as long as they sum up to 1.

- $p(x|\theta)$ - the likelihood. For our example this will be $$p(x|\theta)=\prod p(x_i|
\theta) = \theta_1^{20}\theta_2^{16}\theta_3^{14}$$ This might seem like an odd function but if we take the log

$$-log(p(x|\theta)) = 20(log(\theta_1))+16(log(\theta_2))+14(log(\theta_3))$$ - if we squint a bit we realize that this is just the CategoricalCrossentropy that's used a lot as.


So assuming a uniform prior $$p(\theta|x) \propto \theta_1^{20}\cdot \theta_2^{16} \cdot \theta_3^{14}$$ a rather simple formula. Now to actually get the inference on $\theta$ we could use say pymc to do this whole process for us or we could try to sample directly from the unnormalized posterior here. Instead we will choose a middle ground and use the fact that the posterior is a known distrbution.


From $p(\theta|x) \propto \theta_1^{20}\theta_2^{16}\theta_3^{14}$ we get that $p(\theta|x)$ is a Dirichlet $Dir(1+20, 1+16, 1+14)$ distribtion where the Dir distribution is just the normalized version of the likelihood.

Here is some code that does these compuations for us:
```python
import numpy as np
from scipy.stats import dirichlet

class Dir():
    
    def __init__(self, prior):
        self.prior_alpha = prior_alpha
        self.prior = dirichlet(alpha=self.prior_alpha)
        
    def fit(self, y):
        alpha = y + self.prior_alpha
        posterior_alpha = self.prior_alpha + y
        
        self.posterior = dirichlet(alpha=posterior_alpha)
        
    def sample_posterior(self, N):
        return self.posterior.rvs(size = N)
```

There's a decent amount of code here but notice that this is `posterior_alpha = self.prior_alpha + y` is the meat of the computation - and it's a simple addition!

Ok so now that we have a posterior distribution we still have to use it to decide if the die is fair or not. If we look at $Pr(\theta=(1/3,1/3,1/3)|x)$ this will in fact be zero since a point has zero measure. So this approach does provide the commonsensical inference that no die is perfectly fair. Instead we can define a fair die as one that is withing P=(1/3,1/3,1/3) with some margin of error! For example if a die has (0.323,0.342, 0.333) we might consider this die to be good enough for all intents and purposes. 



$$ \theta_0 = (1/3-x_1,1/3-x_2,1/3-x_3) - $$ Then we want $$\sum(|1/3-x_i|) < K$$.

We will pick K to be 0.1 for our purposes. So we consider a die to be fair if the total absolute deviations from a fair die are less than 10%. So (0.323,0.342, 0.333) is "fair" (0.3, 0.3, 0.4) is not fair - the  sum of absolute deviations is 0.2.



<img width="542" alt="image" src="https://user-images.githubusercontent.com/13619417/183262187-3bc6de93-6d5b-47ad-9280-6da9e64b6720.png">


<img width="542" alt="image" src="https://user-images.githubusercontent.com/13619417/183262226-de587b2d-6835-41ef-b17e-68d552274fd9.png">


