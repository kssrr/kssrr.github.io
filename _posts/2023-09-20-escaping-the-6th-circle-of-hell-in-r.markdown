---
layout: post
title: "Escaping from the 6th Circle of Hell in R"
date:   2023-09-20 10:51:41 +0200
categories: jekyll update
---

I am a big fan of "The R Inferno" by Patrick Burns, which encourages good coding practices in a humorous way. However, I am afraid some of the "heretics imprisoned in flaming tombs" inhabiting the 6th circle of hell according to Burns may be caught there unjustly. 

The chapter scolds what Burns calls "global assignment" using the `<<-` operator. Consider this code:

{% highlight r %}
x <- 1
y <- 2

fun <- function() {
  x <- 2
  y <<- 100 
}

fun()

x
#>[1] 1
y
#>[1] 100
{% endhighlight %}

This is a function with side effects: when called, it modifies a value in the global environment. This may be unexpected and lead to confusing results. However, there are a few points to consider about this.

First: the operator is the _superassignment_, not "global" assignment, operator; i.e. `<<-` does not automatically assign in the global environment, it assigns in the _parent_ environment. While this will often be the global environment, this is still an important distinction!

For example in a function factory, a function that returns a function, the parent environment of the manufactured function is not the global environment; it is the execution environment of the factory used to produce it! Sounds confusing? It really is not. Take a look at this function[^1]:

{% highlight r %}
new_counter <- function() {
  i <- 0

  function() {
    i <<- i + 1
    i
  }
}
{% endhighlight %}

[^1]: This idea is taken from Hadley Wickham's "Advanced R", p. 253. 

This is a function that returns a function, which we can use to create counters:

{% highlight r %}
counter_one <- new_counter()
counter_two <- new_counter()
{% endhighlight %}

Here, we use the superassignment to maintain state across function executions - the functions remember how often they have been called:

{% highlight r %}
counter_one()
#>[1] 1
counter_one()
#>[1] 2
counter_two()
#>[1] 1
{% endhighlight %}

The side effects are contained within the execution environment of the manufacturing function and do not affect values in the global environment. 

This is not to say that stateful functions are a great idea either. However, regarding superassignment, I think it is important to remember that often, it is not the tool that is problematic, but how you use it. I can use a hammer to put a nail into my wall and hang a beautiful painting, or I can use it to clobber someone to death. Much in the same way, superassignment can be used for smart "hacks" like stateful functions or memoization[^2], or it can be used to create extremely confusing side effects that can wreck your analysis.

[^2]: Memoization is also mentioned by Burns as a potential use case.
