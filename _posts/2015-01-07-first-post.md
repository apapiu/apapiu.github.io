---
layout: post
title: "Strategies for the Board Game Risk!"
---

<span> $$x^2$$ <span>

$$\begin{aligned}
\mathrm{d}n_1 = r n_1 \left(1 -  \frac{n_1 + C(x_1, x_2) n_2}{K(x_1) } \right) \mathrm{d}t + \frac{1}{\sqrt{K_o} } \sqrt{r n_1 \left(1 +  \frac{n_1 + C(x_1, x_2) n_2}{K(x_1) } \right) } \mathrm{d}W_1\end{aligned}$$


The game of Risk is a turn-based startegy game where players battle each other to take over the wolrd. Your aim is to control as many of the forty-two territories with the armies at your disposal. The way you gain ground is by attacking enemy territories via rolling dice. Here is a basic rundown of the battle mechanics:  Battles occur in rounds, with an attacking player typically rolling (up to) 3 dice, and a defending player (up to) 2 dice. After rolling, dice are paired up (Highest rolled attacker die versus highest rolled defender die, then next highest rolled pair if required).
The highest rolled number wins (eliminating one opponent army), with ties resulting in a win by the defender. There a quite a few other rules; you can find them explained [here]().


The first step in getting insight is figuring out who has the probabilistic advantage: on one hand the defender only rolls 2 dice - one less than the defender, on the other hand the attacker loses armies in the case of equality. It turns out the computation is not so easy. My first instinct as a combinatorialist is to simply count or list all the possibilities. Doing this by hand however is quite tricky - there are a total of 7776 cases (this is just 6 to the fifth) and it's very easy to overcount cases. In fact one of the first published [papers](https://www.researchgate.net/publication/266313658_Markov_chains_and_the_RISK_board_game) on the topic got the computations wrong because of a false independence assumption. Yep this was in a published paper in a peer-review journal (on expository math but still). Perhaps even more comical is that checking your answers can be done with like 5 lines of code to list all the posibilities. The corrected version by John Osborne can be found [here]. He uses these probabilities of winning a single round and Markox Chains to compute different probabilities of winning for various army sizes. So for example if you have 9 armies and the defender has 7 you have a 72% chance of winning i.e. of eliminating all 7 defending armies. 

Many times in the game you'll be in the following situation: you have, say 28 armies and want to take over four of your enemy's territory and he has 10, 8 ,2 and 1 armies in these territories respectively. Now you're not only intersted in what the probability of winning after a battle is but you more interested in two things: 

- how many armies will you have (on average) once the battle is done?
- how _certain_ are you about the expected value the number of armies you have remaining? In other words how big of a role does luck play in the game? In other words what's the variance of the random variable number of armies remaining.
- what order should you attack the territories in? 

We could do the markov chain aprroach again but let's try something else instead. Our approach, while not necessarily the most precise but is certainly the most natural: just roll the corresponding dice over and over again and see what the outcomes are. Luckily computers can generate ([pseudo](https://en.wikipedia.org/wiki/Pseudorandom_number_generator)) random rolls of dice so we don't have to do this ourselves. Here's a function `Rolls` written in R that simulates a battle netween a number of attacking armies `atarm` here versus a number of defending armies called `defarm` and stops when either one player or the other runs out of pieces for the specific territory. Below I run the 14 versus 12 armies battle a few times. The first `[1] 1 0` means the attacker is left with one army after and the defender with 0. As you can see even from these random examples, there is a lot of variability. To get a better sense of the variance we'll basically write some code to repeat the `Rolls(14, 12)` function a whole bunch of times (100000 times to be precise) and then look at the distribution of how many armies the attacker is left with.  

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




![](/img/Rplot.png)

We can see the histogram _here_. 

