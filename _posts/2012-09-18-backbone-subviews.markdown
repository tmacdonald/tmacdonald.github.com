---
layout: post
title: Backbone Subviews
permalink: /2012/09/backbone-subviews
---

#Backbone Subviews

Something I've noticed bouncing around the Backbone community these days is the idea of how to structure subviews. I'm going to outline here a basic example of how I look to include subviews.

Note that I really haven't dug into some of the frameworks that claim to solve this problem, notably Backbone.LayoutManager from Tim Branyen.

In this example, we have two Backbone Views: ParentView and ChildView.

	var ParentView = Backbone.View.extend({
		
	});

	var ChildView = Backbone.View.extend({
		
	});

The ParentView and ChildView will have fairly simplistic templates

	<script type="text/template" id="parentTemplate">
		<p>This is a parent</p>
		<div id="child"></div>
	</script>

	<script type="text/template" id="childTemplate">
		<p>This is a child</p>
	</script>

The ChildView will simply render it's template into the default div element

	var ChildView = Backbone.View.extend({

		template: _.template($('#childTemplate').html()),

		render: function() {
			this.$el.html(this.template());
			return this;
		}
	});

Now the `ParentView` will initialize the `ChildView` in it's initialize method, then render the `ChildView` in it's render method

	var ParentView = Backbone.View.extend({
		template: _.template($('#parentTemplate').html()),

		render: function() {
			this.$el.html(this.template());

			this.$('#child').html(this.childView.el);
			this.childView.render();

			return this;	
		}
	});

I imagine at this point you're thinking, *this doesn't buy me anything*, and you're right. The next step is to notice that if `ChildView` becomes more advanced, say, it will render anytime a collection is reset, we would move the line

			this.childView.render()

from `ParentView` and add an initialize method to the `ChildView`

		initialize: function() {
			this.collection.on('reset', this.render, this);
		}

For some extra context, here were some other approaches I tried:

## Approach 1

In this approach I initialized the `ChildView` in the `ParentView`'s render function

	var ParentView = Backbone.View.extend({
		template: _.template($('#parentTemplate').html()),

		render: function() {
			this.$el.html(this.template());

			this.childView = new ChildView({el: this.$('#child')});

			return this;	
		}
	});

Something about initializing the `ChildView` outside of the `ParentView initialize` method didn't feel right. Also, re-rendering the `ParentView` would make it necessary to clean up after `this.childView`. The other issue that may pop up is that we won't be able to use Backbone's `tagName` and `className` properties, as the `el` is being passed into the `ChildView`.

## Approach 2

In this approach, I initialized the `ChildView` in the `ParentView`'s initialize method, but render it differently

	var ParentView = Backbone.View.extend({
		template: _.template($('#parentTemplate').html()),

		initialize: function() {
			this.childView = new ChildView();
		},

		render: function() {
			this.$el.html(this.template());

			this.$('#child').html(this.childView.render().el);

			return this;
		}
	});

The problem here is that it keeps the `ChildView` from being able to render when a collection is reset, for example, without the `ParentView` knowing about the collection, which may be outside of its responsibilities.