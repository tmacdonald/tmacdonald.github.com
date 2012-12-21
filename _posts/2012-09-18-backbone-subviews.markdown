---
layout: post2
title: Backbone Subviews
permalink: /2012/09/backbone-subviews
---

#Backbone Subviews

Something I've noticed bouncing around the Backbone community these days is the idea of how to structure subviews. I'm going to outline some approaches for working with Backbone subviews as well as talk about the pros and cons of each.

Note that I really haven't dug into some of the frameworks that claim to solve this problem, notably Backbone.LayoutManager from Tim Branyen.

For all approaches, I'm going to work with a `ParentView` and `ChildView`.

{% highlight javascript %}
  var ParentView = Backbone.View.extend({
  });

  var ChildView = Backbone.View.extend({
  });
{% endhighlight %}

The `ParentView` and `ChildView` will have the following templates:


  <script type="text/template" id="parentTemplate">
    <p>This is a parent</p>
    <div id="child"></div>
  </script>

  <script type="text/template" id="childTemplate">
    <p>This is a child</p>
  </script>

All approaches will also use a `Collection` class.

{% highlight javascript %}
  var Collection = Backbone.Collection.extend({
  });
{% endhighlight %}

## Approach 1

I initialize the `ChildView` in the `ParentView`'s render function. The `ParentView` will create a `Collection` and pass it to the `ChildView` which will render on a `Collection reset` event.

{% highlight javascript %}
  var ParentView = Backbone.View.extend({
    template: _.template($('#parentTemplate').html()),

    initialize: function() {
      this.collection = new Collection();
    },

    render: function() {
      this.$el.html(this.template());

      this.childView = new ChildView({el: this.$('#child'), collection: this.collection});

      return this;
    }
  });

  var ChildView = Backbone.View.extend({

    template: _.template($('#childTemplate').html()),

    initialize: function() {
      this.collection.on('reset', this.render, this);
    },

    render: function() {
      this.$el.html(this.template());
      return this;
    }
  });
{% endhighlight %}

This approach more or less makes the assumption that the `ParentView`'s `render` method won't be called multiple times. Otherwise, the `render` method will need to ensure there is no memory leak concerning `this.childView`.

This approach also prevents the `ChildView` from using the `Backbone.View` `tagName` or `className` properties.

## Approach 2

In this approach, I initialize the `ChildView` in the `ParentView`'s initialize method, but render it differently.

{% highlight javascript %}
  var ParentView = Backbone.View.extend({
    template: _.template($('#parentTemplate').html()),

    initialize: function() {

      this.collection = new Collection();
      this.collection.on('reset', this.renderCollection, this);
      this.childView = new ChildView();
    },

    render: function() {
      this.$el.html(this.template());

      this.renderCollection();

      return this;
    },

    renderCollection: function() {
      this.$('#child').html(this.childView.render().el);
    }
  });
{% endhighlight %}

The big difference between Approach 1 and Approach 2 is that Approach 2 is using `ParentView` to set up the `reset` event handler. This seems like a disadvantage to me, as the rendering of the `ChildView` bleeds into the responsibilities of the `ParentView`.

We are able to use `Backbone.View` `tagName` and `className` properties.

##Approach 3

The ChildView will simply render it's template into the default div element

The `ParentView` will initialize the `ChildView` in it's initialize method, then let the `ChildView` handle its own rendering.

{% highlight javascript %}
  var ChildView = Backbone.View.extend({
    template: _.teplate($('#childTemplate').html()),

    initialize: function() {
      this.collection.on('reset', this.render, this);
    },

    render: function() {
      this.$el.html(this.template());
    }
  });

  var ParentView = Backbone.View.extend({
    template: _.template($('#parentTemplate').html()),

    initialize: function() {
      this.collection = new Collection();
      this.childView = new ChildView({collection: collection});
    }

    render: function() {
      this.$el.html(this.template());

      this.$('#child').html(this.childView.el);

      return this;
    }
  });
{% endhighlight %}

What I like about this approach is that the `ChildView` instance is initialized in the `ParentView` constructor, the `ChildView` is responsible for setting up the collection `reset` event handler and rendering itself, re-rendering the `ParentView` will not require clean-up code, and `Backbone.View` `tagName` and `className` properties are still usable from the `ChildView`