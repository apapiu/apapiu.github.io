---
layout: post
title: "Strategies for the Board Game Risk!"
---

The game of Risk is a turn-based startegy game where players battle each other to take over the wolrd. Your aim is to control as many of the forty-two territories with the armies at your disposal. The way you gain ground is by attacking enemy territories via rolling dice. Here is a (very) basic rundown of the battle mechanics:  Battles occur in rounds, with an attacking player typically rolling 3 dice, and a defending player 2 dice. The dice are then ordered and compared and the person with the lowest die loses some number of armies. There a quite a few other rules; you can find them explained [here](http://www.hasbro.com/common/instruct/risk.pdf).


From a game theoretic point of view the first question is who has the advantage, the attacker or the defender? On one hand the defender only rolls 2 dice - one less than the defender, on the other hand the attacker loses armies in the case of equality. It turns out the computation is not so easy. My first instinct as a combinatorialist is to simply count or list all the possibilities. Doing this by hand however is quite tricky - there are a total of 7776 cases (this is just $$6^5$$) and it's very easy to overcount cases. In fact one of the first published [papers](https://www.researchgate.net/publication/266313658_Markov_chains_and_the_RISK_board_game) on the topic got the computations wrong because of a false independence assumption. Yep this was in a published paper in a peer-review journal (on expository math but still). Perhaps even more comical is that checking your answers can be done with like 5 lines of code to list all the posibilities. A correct version of the computations by Jason Osborne can be found [here](http://www4.stat.ncsu.edu/~jaosborn/research/RISK.pdf). Once the battle probabilities are found one can use recursion or Markov Chains to figure out probabilities of winning between mutiple armies. So if you want to say see what the probability of winning in a 9 versus 7 battle you can look at Table 3 in Jason's paper in see that it's $$72.6%$$.

However the probabilities themselves won't be that helpful during realy play. Many times in the game you'll be in the following situation: you have, say 28 armies and want to take over four of your enemy's territory and he has 10, 8 ,2 and 1 armies in these territories respectively. Now you're not only intersted in what the probability of winning after a battle but you are really intersted in the distribution of the number of armies you are left with after you finish the attack. In particular you are intersted in the **expected value** and **variance** of this distribution. In more lay-mans terms we will try to explore the following two questions:

- how many armies will you have (on average) once the battle is done?
- how _certain_ are you about the expected value the number of armies you have remaining? 

We could do the markov chain aprroach again but let's try something else instead. Our approach, while not necessarily the most precise but is certainly the most natural: just roll the corresponding dice over and over again and see what the outcomes are. Luckily computers can generate ([pseudo](https://en.wikipedia.org/wiki/Pseudorandom_number_generator)) random rolls of dice so we don't have to do this ourselves. Here's a function `Rolls` written in R that simulates a battle netween a number of attacking armies `atarm` here versus a number of defending armies called `defarm` and stops when either one player or the other runs out of pieces for the specific territory. Below I run the 14 versus 12 armies battle a few times. The first `[1] 1 0` means the attacker is left with one army after and the defender with 0. As you can see even from these random examples, there seems to be a lot of variability.

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

To get a better sense of the distribution we'll basically write some code to repeat the `Rolls(14, 12)` function a whole bunch of times (100000 times to be precise) and then look at the distribution of how many armies the attacker is left with.  
![](/img/riskplot1.jpg)



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




