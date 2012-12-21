---
layout: midnight
title: Functions
permalink: /2012/10/functions
---

#Functions

Many descriptions of Javascript describe functions as first-class objects. What does that mean?
I like to think of first-class objects as anything you can assign to a variable. In many languages, such as Java, you cannot assign a function to a variable.

Here's an example:

{% highlight javascript %}
  var func = function() {
  };
{% endhighlight %}

This could also be written in a different form:

{% highlight javascript %}
  function func() {
  }
{% endhighlight %}

Javascript has a typeof operator that will return to us the type of the variable as a string. It also has an instanceof operator that will return to us whether or not the variable is a certain given type. Both of the approaches from above will behave the same when using these operators.

{% highlight javascript %}
  typeof func
  "function"

  func instanceof Function
  true

  func instanceof Object
  true
{% endhighlight %}

Make sure you pay special attention to that last line. `func` is actually an object (or instance of `Object`). Everything in Javascript that is considered an object (hint: it's effectively everything) has the `Object` class as a parent. That means that functions have methods built in, like toString.

{% highlight javascript %}
  func.toString();
  "function func() {}"
{% endhighlight %}

The `Function` class provides some additional behaviours for functions themselves. I'll talk about that in another article. But first, I want to cement this idea of functions as first-class objects by not only assigning a variable but also moving that variable around.

Most times, a variable is assigned a value and then passed to another function to be processed or returned from the function where it has been created.

## Passing functions as parameters

The classic example of passing a function as a parameter is a calculate function. This calculate function will take as parameters a binary math operator (such as addition or multiplication -- anything that takes two parameters and returns a single value) as well as two parameters (in this case numbers) that will be operated on.

{% highlight javascript %}
  var add = function(x, y) {
    return x + y;
  };

  var subtract = function(x, y) {
    return x - y;
  };

  var calculate = function(operator, x, y) {
    return operator(x, y);
  };
{% endhighlight %}

Some examples of using this code:

{% highlight javascript %}
  calculate(add, 1, 2);
  3

  calculate(subtract, 5, 3);
  2
{% endhighlight %}

As an exercise, try adding multiplication and division operators and pass those to the calculate function. It may seem simple, but remember that the best way to learn is by doing.

If you want a more advanced exercise, try rewriting the calculate function so that it will work with both binary operators and an operator called addThreeNumbers, which (you guessed it) returns the sum of three numeric parameters.

## Returning functions

Now that you've seen functions passed as parameters, I'll show you how to return functions. Let's say that we're writing a very basic math parser where the user passes a symbol string (such as '+' or '*') to a function called getOperator, which will return the appropriate operator. For now we're only going to support addition and subtraction.

{% highlight javascript %}
  var getOperator = function(symbol) {
    var operator;
    if (symbol === '+') {
      operator = add;
    }
    else if (symbol === '-') {
      operator = subtract;
    }
    return operator;
  };
{% endhighlight %}

Note that, in real life, you'd want to have additional checks for unsupported symbols and maybe throw an exception if the getOperator function doesn't support a passed in symbol.

You can do some pretty complex stuff when passing functions as parameters or returning functions from other functions. We've really only scratched the surface.

