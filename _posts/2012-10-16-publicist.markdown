---
layout: midnight
title: The Publicist
permalink: /2012/10/publicist
---

Today I thought I'd show you a little scenario that probably has no real-world application but provides a way to show some of the interesting features of Javascript. I'll take a look at object literals and the dynamic nature of the language.

As much as you try to avoid it, our media is full of celebrity gossip. Just imagine being a Hollywood publicist trying to prevent your client from giving the tabloids front-page news.

In this scenario, there is a celebrity and a publicist.

{% highlight javascript %}
  var celebrity = {};

  var publicist = {};
{% endhighlight %}

Both `celebrity` and `publicist` are object literals, meaning that they are objects that have no associated class. These objects can have any number of properties and methods. Note that in javascript, object literal methods are really just properties that have been assigned a function.

Moving along, the `celebrity` object has a number of methods, some which would be considered positive from a publicist's perspective and some which would be considered negative.

{% highlight javascript %}
  var celebrity = {
    attendAuditionForBlockbusterMovie: function(movie) {
      console.log("Attended audition for " + movie);
    },
    exitLimousineWithoutShame: function() {
      console.log("Very public regretful decision");
    },
    appearOnTalkShowToPromoteMovie: function(talkShow, movie) {
      console.log("Appeared on " + talkShow + " to promote " + movie);
    },
    trashHotelRoom: function(hotel, roomNumber) {
      console.log("That's going to cost me");
    }
  };
{% endhighlight %}

The publicist wants to make their client, the celebrity, available to the public without exposing those negative actions.

{% highlight javascript %}
  var publicist = {
    restrictClientActions: function(client) {
    }
  };
{% endhighlight %}

What the publicist wants is for an object to be returned from the `restrictClientActions` method with their positive actions available but their negative actions inaccessible. This would be similar to private and public methods in Java or C++. By the way, Javascript doesn't currently support the traditional idea of private and public methods, but it is possible to simulate them.

The difference in this scenario is that the publicist has no control over the implementation of the object.

I'll start with a naive approach to implementing `restrictClientActions`

{% highlight javascript %}
  restrictActions = function(client) {
    client.exitLimousineWithoutShame = undefined;
    client.trashHotelRoom = undefined;
    return client;
  }
{% endhighlight %}

This does actually prevent anyone from calling these negative methods on the client. However, it modifies the existing client, and thinking you can change someone is probably not a realistic approach.

Here is an implementation of `restrictClientActions` that is nearly the opposite of the first implementation:

{% highlight javascript %}
  restrictActions = function(client) {
    return {
      attendAuditionForBlockbusterMovie: function(movie) {
        client.attendAuditionForBlockbusterMovie(movie);
      },
      appearOnTalkShowToPromoteMovie: function(movie) {
        client.appearOnTalkShowToPromoteMovie(movie);
      }
    };
  }
{% endhighlight %}

What this code is doing is returning a new object literal that wraps each of the positive methods and simply calls the client's wrapped method. Notice that the object literal will maintain a reference to the client. This is called a closure, meaning that the object literal has reference to the environment in which it was created. In this example, the client object is part of the referencing environment of the object literal.

If I run the client through the publicist, the object that comes out will not have the capability to perform negative actions:

{% highlight javascript %}
  var wellBehavedCelebrity = publicist.restrictClientActions(celebrity);
  wellBehavedCelebrity.attendAuditionForBlockbusterMovie("Rocky VII: Adrian's Revenge");
  // output: Attended audition for Rocky VII: Adrian's Revenge
  wellBehavedCelebrity.trashHotelRoom("The Ritz", 109);
  // output: TypeError: Object #<Object> has no method 'trashHotelRoom'
{% endhighlight %}

To add complexity into the scenario, what happens in implementation 2 if the celebrity decides to branch out into the music business?

{% highlight javascript %}
  var celebrity = {
    // existing methods
    performAutotunedDrivelThatEveryoneLoves = function() {
      console.log("La la la");
    }
  };
{% endhighlight %}

Implementation 2 is going to prevent the restrict celebrity from performing this new action, which is bad for the publicist. What if, as part of the `restrictClientActions` method, the approved actions could be included?

For instance:

{% highlight javascript %}
  restrictClientActions: function(client, approvedActions) {
    var restricted = {}, i;
    for (i = 0; i < approvedActions.length; i++) {
      var action = approvedActions[i];

      restricted[action] = function() {
        return client[action].apply(client, arguments);
      };
    }

    return restricted;
  }
{% endhighlight %}

This one is a little bit more complicated. First of all, I declare two variables: `restricted` and `i`. `restricted` will be the object that gets returned. `i` is a number iterator that will be used in the for loop. Next up is the loop itself. What this is doing is that for each string value in the approvedActions list, I'm going to add a property to the

{% highlight javascript %}
  var wellBehavedCelebrity = publicist.restrictClientActions(celebrity);
  wellBehavedCelebrity.attendAuditionForBlockbusterMovie("Rochelle, Rochelle");
  // output: Attended audition for Rochelle, Rochelle
  wellBehavedCelebrity.trashHotelRoom("The Ritz", 109);
  // output: TypeError: Object #<Object> has no method 'trashHotelRoom'
{% endhighlight %}
