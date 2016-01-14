---
layout: post
title: "Strategies for the Board Game Risk!"
---


The game of Risk is a turn-based startegy game where players battle each other to take over the wolrd. Your aim is to control as many of the forty-two territories with the armies at your disposal. The way you gain ground is by attacking enemy territories via rolling dice. Here is a basic rundown of the battle mechanics: 

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

