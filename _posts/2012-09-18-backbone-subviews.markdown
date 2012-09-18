---
layout: post
title: Backbone Subviews
permalink: /2012/09/backbone-subviews
---

#Backbone Subviews

Something I've noticed bouncing around the Backbone community these days is the idea of how to structure subviews. I'm going to outline here a basic example of how I look to include subviews.

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