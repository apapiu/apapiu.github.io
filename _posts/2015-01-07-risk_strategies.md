---
layout: post
title: "Strategies for the Board Game Risk"
---

The game of Risk is a turn-based strategy game where players battle each other to take over the world. Your aim is to control as many of the forty-two territories with the armies at your disposal. The way you gain ground is by attacking enemy territories via rolling dice. Here is a (very) basic rundown of the battle mechanics:  Battles occur in rounds, with an attacking player typically rolling 3 dice, and a defending player 2 dice. The dice are then ordered and compared and the person with the lowest die loses some number of armies. There a quite a few other rules; you can find them explained [here](http://www.hasbro.com/common/instruct/risk.pdf).

The board is an _approximate_ map of the world and players can attack territories adjacent to them. What's neat about this is that the actual layout doesn't matter that much, all that matters is how the teritories are connected with each other! So let's ignore the structure of the map itself and look at the structure we are really interested in: the graph where the territories are the nodes and where we draw edges between adjacent territories. Since I couldn't quite find a satisfactory map online I decided to make my own. Not only that but I made it interactive! I used the amazing [networkD3](https://christophergandrud.github.io/networkD3/) package - **see [here](http://rpubs.com/apapiu/riskgraph) for the interactive map.** - you can click and drag the nodes around! Since I couldn't figure out how to embed the interactive map in this post we'll have to be satisfied with a boring old static image.


![](/img/graph1.png)



From a game theoretic point of view the first question is who has the advantage, the attacker or the defender? On one hand the defender only rolls 2 dice - one less than the defender, on the other hand the attacker loses armies in the case of equal die. It turns out the computation is not so easy, it's easy to overcount cases (see [here](https://www.researchgate.net/publication/266313658_Markov_chains_and_the_RISK_board_game) for an incorrect.  A correct version of the computations by Jason Osborne can be found [here](http://www4.stat.ncsu.edu/~jaosborn/research/RISK.pdf). Once the battle probabilities are found one can use recursion or Markov Chains to figure out probabilities of winning between mutiple armies. So if you want to say see what the probability of winning in a 9 versus 7 battle you can look at Table 3 in Jason's paper in see that it's $$72.6\%$$.

However the probabilities themselves won't be that helpful during real play. Many times in the game you'll be in the following situation: you have, say 28 armies and want to take over four of your enemy's territory and he has 10, 8 ,2 and 1 armies in these territories respectively. **Now you're not only interested in what the probability of winning after a battle but you are really interested in the distribution of the number of armies you are left with after you finish the attack.** In particular you are interested in the **expected value** and **variance** of this distribution - as far as I could tell no one has addressed these issues directly. So the purpose of this post will be to do just that! In more layman's terms we will try to explore the following two questions:

- how many armies will you have (on average) once the battle is done?
- how _certain_ are you about the expected value the number of armies you have remaining? 

Our approach, while not necessarily the most precise but is certainly the most natural: just roll the corresponding dice over and over again and see what the outcomes are. Luckily computers can generate ([pseudo](https://en.wikipedia.org/wiki/Pseudorandom_number_generator)) random rolls of dice so we don't have to do this ourselves. We've built a function `Rolls` written in R (scroll down a bit to see it) that simulates a battle between a number of attacking armies `atarm` here versus a number of defending armies called `defarm` and stops when either one player or the other runs out of pieces for the specific territory. Below I run the 14 versus 12 armies battle a few times. The first `[1] 1 0` means the attacker is left with one army after and the defender with 0. As you can see even from these random examples, there seems to be a lot of variability.


{% highlight r %}
Rolls <- function(atarm, defarm) { # here atarm is the number of guys -1 - the guys you are actually attacking with
        while (atarm >=2 & defarm >= 1) {
            at <- sort(sample(6, min(3, atarm), replace = TRUE), decreasing = TRUE) #ordered attack dice
            def <- sample(6, min(2, defarm) , replace = TRUE) # defense dice
            
            
            if (at[1]>max(def) & at[2]>min(def)) {defarm <- defarm - min(2, defarm)}
            else if (at[1]<= max(def) & at[2] <= min(def)) 
                {atarm <- atarm - min(2, defarm)} #this is t
            else {defarm <- defarm -1; atarm <- atarm - 1}
        }
        
        while (atarm >=1 & defarm >=1) { #taking care of the case with one attacker
            at <- sample(6,1)
            def <- sample(6, min(2, defarm) , replace = TRUE)
            if (at > max(def)) {defarm <- defarm -1}
                else {atarm <- atarm -1}
        }
    return(c(atarm, defarm))
}
{% endhighlight %}

{% highlight r %}
> Rolls(14, 12)
[1] 1 0
> Rolls(14, 12)
[1] 0 7
> Rolls(14, 12)
[1] 8 0
> Rolls(14, 12)
[1] 0 3
{% endhighlight %}

To get a better sense of the distribution we'll basically write some code to repeat the `Rolls(14, 12)` function a whole bunch of times (100000 times to be precise) and then look at the distribution of how many armies the attacker is left with. Let's call this function `r sim` 
{% highlight r %}
sim <- function(att, def, reps = 10000) { #simulates the battle reps times 
outcome <- replicate(reps, Rolls(att,def))
expec <- mean(outcome[1,]) 
winningprob <- 1 - table(outcome[1,])[1]/reps
return(list(expec, winningprob, outcome)) #take hist(outcome[1,]) to see the dist
}
{% endhighlight %}

![](/img/riskplot1.jpg)

There are a few observations we can make right away: the tall bar on the left represents the times the attacker loses. This happens roughly 28% of the times in the 14 versus 12 case. When the attacker wins the distribution is roughly bell shaped, in fact there are two curves one can see -- this is because of the game mechanics favoring losing two armies instead of one. The main takeaway however is just how spread out the distribution is. You can expect on average to be left with  4.45071 armies but there is still a good chance you'll have anywhere from 0 to 12 armies once you are done. Now let me state again: these results are not %100 precise. We are using a Monte Carlo simulation so there will be some error to our calculations. However we are repeating this experiment enough times for the error the be insiginificant for practical startegies for the game. Also this approach allows us to be more flexible. We can figure out any aspect of the battles simply by simulating the battle over and over.

Let's try to make these observations a bit more precise. We will get some information in the case when the numbe of attacking and defending armies are equal. 

{% highlight r %}
count <- 1:40
ex <- numeric(40)
prob <- numeric(40)
for (i in count){
    ex[i] <- sim(i,i)[[1]]
    prob[i] <- sim(i,i)[[2]]
}

{% endhighlight %}

The first plot shows the number of armies the attacker expects to be left with on average in an n vs. n battle. We can see that as the number n of attacking and defending armies increases the advantage of the attacker becomes more tangible. For example in a 40 vs. 40 battle the expected value of armies remaining for the attacker is 7.3196 while in a 10 vs. 10 battle the expected value is 2.5528. So in this case **the attacker is at a slight advantage and the advantage increases slightly with the number of attacking armies.**
![](/img/riskplot2.png)

The plot below looks at a different dimension: the probability that the attacker wins in a n vs. n battle. The story is similar to the graph above. If the attacker has 5 armies or more it is worth attacking a defending army of the same size. As the attacking army increases so does the probability of winning. When n = 40 we see that the probability of the attacker winning is 0.7003. So in this case the heuristic is pretty simple: **Attack an army of the same size if you have 5 armies or more.** And even in this case keep in mind that you might be left with very few (or even 0) armies.
![](/img/riskplot3.png)


Now lets keep the attacking number fixed at 20 and vary the number of defending armies.

{% highlight r %}

set.seed(124)
ex3 <- numeric(40)
prob3 <- numeric(40)
for (i in count){
    ex3[i] <- sim(20,i)[[1]]
    prob3[i] <- sim(20,i)[[2]]
}
{% endhighlight %}

The graph below shows the expected values of the armies the attacker (you) is left with as we vary the number of defending armies. 
![](/img/riskplot4.png)

The graph below shows the probability of wininng with 20 armies against a varying number of defending armies. There is quite an interesting takeway here: **the probability the attacker wins changes very steeply around n = 20**. This basically means that you should roughly attack armies that are slighly less numerous than yours but should _never_ attack armies that are more numerous than yours. In the case of 25 armies the probability the attacker wins aganst 15 defending armies is 0.8603% which is pretty good! But try to attack with 20 vs. 25 and your chance of success drops to a meek  0.3832. 

![](/img/riskplot5.png)

Finally lets move on to analyzing the variance of the game.

{% highlight r %}

set.seed(124)
simulation <- sim(20, 10)
dist <- simulation[[3]][1,]
distance <- abs(dist - mean(dist)) 
length(distance[distance <8])/10000
{% endhighlight %}


Remember the situation: you have 20 armies as the attacker and we are attacking a defending army of 10. Based on the code above, we get a probability of .9382 for distance = 8 so this means you are in between (3.5 and 19.5) 93.8% of the time. For the player this is not that great : it basically says you are 93% confident that you'll end up with anywhere in between 3.5 and 19.5 armies - a very wide interval. This cuts down to the main issue a lot of plaer have with risk - it's too high variance - luck playes to big of a role. So even if you play very well you'll only see the benefits in the very long run. However let's try to summarise the srategies we discovered using your Monte Carlo simulations:

**1. Risk is a high variance game. Even if you play optimally you will lose many times.**

**2. The Atacker has a slight advantage and the advantage become bigger as the number of attacking armies increases.**

**3. You gain most by attacking armies that are slighly weaker than yours. Never attack armies that are stronger than yours.**

**4. Attack an a defending force of the samze size only if you have 5 or more armies to attack with.**








