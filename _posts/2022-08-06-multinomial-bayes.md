---
layout: post
title: Checking if a Die is Fair - A Simple Bayesian Approach
---


If I gave you a die and and asked you if it's a fair die what would you do? Well most likely you would start rolling a few times and look at the data. Yet even with data it's hard to know for sure! Even a fair die can lead to many different outcomes. And also if we're being honest no die is perfectly fair is it. So we're in a bit of a pickle.


**To make things a bit simpler and easier to visualize we will look at a 3 sided die**, worry not however most of what we'll do easily generalizes to a 6 sided die. It's really the jump from 2 to 3 dimensions that's the harder part. Also 3 sided die do actually exist. (share pic)

We are interested in finding information about $\theta = (\theta_1, \theta_2, \theta_3)$ where $\theta_i$ is the probability of rolling an i. Note that $\theta_1+\theta_2+\theta_3 = 1$ so there are some dependencies within $\theta$. A fair die would have $\theta = (1/3,1/3,1/3)$.

The basic setup is that we get some data by rolling the die - say 20 1s, 16 2s, 14 3s and we want to have some sort of idea about the fairness or balancedness of the die. 

#### The Frequentist Approach:

The general frequentist approach is based on Null Hypothesis Statistical Testing and is briefly described below. With this notation our hypothesis becomes:

- $H_0: \theta_1 =  \theta_2 =  \theta_3 = 1/3$
- $H_1$: $H_0$ is not true (i.e. $\theta_i \neq 1/3$ for some i)

The general idea is to obtain some kind of statistic based on the obtained sample that would provide evidence rejecting $H_0$ over $H_1$. 

There are a few issues with this.

- Firstly no one really expects a real life die to be perfectly fair - so we know almost for a fact that $H_0$ is not true! 

- It's not exactly clear how to build a one-dimensional statistic that would be helpful in rejecting the null. It turns out that a certain sum of squares _approximately_ follows a chi-squared distribution under the null hypothesis so we can use that as our statistic. Yet the mathematical justification of all of this if fairly complicated and the statistic isn't in any way "obvious".  

- Thirdly, the degrees of freedom computation involved in testing more complicated hypotheses can add one more layer of confusion especially (see here).

#### A Bayesian Way:
Hopefully I have convinced you that it would be good to at least have an alternative to the approach above. In comes Bayes theorem! Generally Bayesian approaches have a bad rap of being very computationally expensive but this really isn't the case here - in fact both the math and the inference is fairly straightforward. 

Let's again assume we have obtained some data by rolling the die **50 times - 20 1s, 16 2s, 14 3s.** We want to understand the probability distribution of $\theta$ given $x = (20, 16, 14)$. This is the posterior distribution $p(\theta\mid x)$ and we can use Bayes' formula to obtain it:

$$p(\theta\mid x) \propto p(x\mid \theta) p(\theta) $$

Note that $p(\theta\mid x)$ is a probability distribution over probabilities - there's nothing special about it mathematically but it might take a bit to wrap one's head around that idea. Once we plot a concrete example hopefully this becomes more clear.

Let's look at the components:

- $p(\theta)$ will be our prior and will incorporate any prior knowledge we have about the fairness or loadedness of the die. We will for the most part use a uniform prior $p(\theta) \propto 1$ which says that any combination of $\theta_1, \theta_2, \theta_3$ is equally likely as long as they sum up to 1.

- $p(x\mid \theta)$ - the likelihood. For our example this will be 

$$p(x\mid \theta)=\prod p(x_i\mid \theta) = \theta_1^{20}\theta_2^{16}\theta_3^{14}$$ 

This might seem like an odd function but if we take the log we get:

$$log(p(x\mid \theta)) = 20(log(\theta_1))+16(log(\theta_2))+14(log(\theta_3))$$ 

If we squint a bit we realize that this a simple case of the [Categorical Crossentropy](https://www.tensorflow.org/api_docs/python/tf/keras/losses/CategoricalCrossentropy) loss that's used to train multi-class MLmodels - which makes sense since this is the MLE estimator.


So assuming a uniform prior we get that the posterior is proportional to the likelihood:

$$p(\theta\mid x) \propto \theta_1^{20}\cdot \theta_2^{16} \cdot \theta_3^{14}$$ a rather simple formula. 

Now to actually get the posterior distribution we could try to sample directly from the unnormalized posterior here. Instead we will choose and use the fact that the posterior is a known distribution.

From $p(\theta\mid x) \propto \theta_1^{20}\theta_2^{16}\theta_3^{14}$ we get that $p(\theta\mid x)$ is a Dirichlet $Dir(1+20, 1+16, 1+14)$ distribution where the Dir distribution is just the normalized version of the likelihood.

More generally if we have a prior = $Dir(\alpha_1, \alpha_2, \alpha_3)$ and roll counts $(k1, k2, k3)$ then the posterior will be $Dir(\alpha_1+k_1, \alpha_2+k_2, \alpha_3+k_3)$.

Here is some code that does these computations for us:
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


Let's use it on the example we had:

```python
x = Diri(prior_alpha=[1,1,1])
x.fit([20,16,14])

posterior = x.sample_posterior(10000)
df_trace = pd.DataFrame(posterior, columns = ["theta1", "theta2", "theta3"])
```

Now `df_trace` is a sample from the posterior distribution. Let's first plot the marginal distributions:

<img width="250" alt="image" src="https://user-images.githubusercontent.com/13619417/183269031-6adc7894-9e28-49ae-addb-869eac16448b.png">

These are informative but might not be enough for us to ascertain the fairness of the die. However they are useful in understanding the uncertainty around the probabilities for each face.

To visualize the joint posterior distribution we will use a trick to use a 2D representation of our three parameters. Due to the condition $\theta_1+\theta_2+\theta_3 = 1$ our thetas will lie on the unit simplex (the green triangle in this case seen below). We can then use barycentric coordinates to visualize the distribution in 2D where we can plot the sample we just got above:

<img width="695" alt="image" src="https://user-images.githubusercontent.com/13619417/183312729-8938952e-2fb0-46ab-9390-2774bc4fed8c.png">

<!-- 
<img width="250" alt="image" src="https://user-images.githubusercontent.com/13619417/183269166-02f0a392-1216-4840-9af2-81e5a1a686bd.png">


<img width="350" alt="image" src="https://user-images.githubusercontent.com/13619417/183269244-2f8aeea7-3112-4130-84fc-816427673a7f.png">

 -->


Ok so now that we have a posterior distribution we still have to use it to decide if the die is fair or not. If we look at $Pr(\theta=(1/3,1/3,1/3)\mid x)$ this will in fact be zero since a point has zero measure. Instead we can define a fair die as one that is within some margin of error of  P=(1/3,1/3,1/3). For example if a die has (0.323,0.342, 0.333) we might consider this die to be good enough for all intents and purposes. 

We will defined our criterion in terms of absolute deviations from P:

$$ \theta_0 = (1/3-x_1, 1/3-x_2, 1/3-x_3) - $$ Then we want $$\sum(\mid 1/3-x_i\mid ) < K$$.

We will pick K to be 0.1 for our purposes. **So we consider a die to be fair if the total absolute deviations from a fair die are less than 10%**. So (0.323,0.342, 0.333) is "fair" while (0.3, 0.3, 0.4). K could be changed as needed.

For our original example this is what the count looks like:

<img width="543" alt="image" src="https://user-images.githubusercontent.com/13619417/183269567-a7aeda6d-b362-49ae-ae03-f4cf7060b1f0.png">


Let's try some more examples:

```python
x = Diri(prior_alpha=[1,1,1])
x.fit([14,3,9])

posterior = x.sample_posterior(5000)
df_trace = pd.DataFrame(posterior, columns = ["theta1", "theta2", "theta3"])
```
<img width="500" alt="image"  class="center" src="https://user-images.githubusercontent.com/13619417/183269478-cebf7ff1-3812-4364-ab9a-60f818ee8203.png">


```python
x = Diri(prior_alpha=[1,1,1])
x.fit([129,132,121])

posterior = x.sample_posterior(5000)
df_trace = pd.DataFrame(posterior, columns = ["theta1", "theta2", "theta3"])
```

<img width="500" alt="image" class="center" src="https://user-images.githubusercontent.com/13619417/183269514-fede31ec-d7e2-4a22-8c90-da56b124112d.png">


