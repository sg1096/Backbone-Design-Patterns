# Backbone.js Design Patterns
Design Patterns
==============================

### Re-using code with extensions
-----------------------------------------------
```
var User  = Bacbone.Model.extend({
	defaults : {
		name : 'John Does'
	}
});

var UserItemView = Backbone.View.extend({
	template : '<span><%=name%></span>',
	render : function(){
		var tpl = _.template(this.template),
			html = tpl(this.model.toJSON());
		this.$el.html(html);
		return this;
	}
});

var userItem = new UserItemView({
	model : new User
});

$(document.body).append(userItem.render().el);

```

### Creating a Base Class

```
var BaseView = Backbone.View.extend({
	render : function(){
		var tpl = _template(this.tempate),
			data = (this.model) ? this.model.toJSON() : {},
			html = tpl(data);

		this.$el.html(html);
		return this;
	}
});

var UserItemView = BaseView.extend({
	template : '<span><%=name %></span>'
});

// if we want to add more functionality this can also be written as 
// following

var UserItemView = BaseView.extend({
	tagName : 'div',
	template : '<span><%= name%></span>',
	render : function(){
		//call the parent View's render function
		BaseView.prototype.render.apply(this, arguments);
		this.someFn();
		return this;
	},
	someFn : function(){ }
});

```

### Developing Plugins without extending base class : 
--------------------------------------------------------

Using the concept of constructors and adding methods to its prototype can be a 	better choice than extending the Backbone base class. 

Example in form of pagination is 

```
// Pagination constructor function
var Pagination = function (collection, noOfItemsInPage) {
  	if (!collection) {
    	throw "No collection is passed";
  	}
  	this.currentPage = 1;
  	this.noOfItemsInPage = noOfItemsInPage || 10;
  	this.collection = collection;
}

// Use Underscore's extend method to add properties to your plugin
_.extend(Pagination.prototype, {
  	nextPage: function () {},
  	prevPage: function () {}
});

var User = Backbone.Model.extend({
  	defaults: {
    	name: 'John Doe'
  	}
});

var Users = Backbone.Collection.extend({
  	model: User
});

var paging1 = new Pagination(10, new Users());
var paging2 = new Pagination(20, new Users());

``` 

###Understanding the concept of Javascript Mixins : 
-------------------------------------------------------

Suppose that there is a view names UserItemView which is extending BaseView, there is a requirement where we want the draggable functionality to be added to this view. 

One of the approach is we create a view that encapsulates draggable functionality and extends the BaseView in this case the flow will be UserItemView extends DragabbleView and DraggableView in turn extends BaseView.

If there is only one type of functionality that we want then it is fine, but if there are more than one functionality then the hirearchy becomes difficult to handle. For example if we want the view now to support the sorting functionality also then we will create a SortableView that extends BaseView and we will create a multi-layered inheritance which will be difficult to mantain

To avoid such problems we use the concept of mixin 

Example Code : 
```
// A simple object with some methods
var DraggableMixin = {
	startDrag: function () {
    	// It will have the context of the main class 
    	console.log('Context = ', this);
  	},
  	onDrag: function () {}
}

// UserItemView already extends BaseView
var UserItemView = BaseView.extend({
  	tagName: 'div',
  	template: '<%= name %>'
});

// We just copy the Mixin's properties into the View
_.extend(UserItemView.prototype, DraggableMixin, {
  	otherFn: function () {}
});

var itemView = new UserItemView();

// Call the mixin's method
itemView.startDrag();

```

There are few other ways to create mixins some of them are as follows :

1. functional mixins.
2. Using caching to avoid change of functions.
3. Currying to combine functions and arguments.

## Views in Backbone.js
---------------------------------

### Tips and tricks for Views in Backbone.js
----------------------------------------------
1. Understanding the el property of a View.
2. Understanding various attributes used to create a tag in a View.

### Partially Updating a View
----------------------------------

This is the most common question of most of the developers, a part of the view has to be re-rendered without re-rendering the whole view. for example consider a view which used to a display a lot of complex information and on certain user action only a part of the view has to be updated, if we re-render the whole view for each of the small change it will be a performance blockage. 

```
var UserView = Backbone.View.extend({
	template : _.template('<p><% =name %><% =address%></p>'),
	initialize : function(){
		this.listenTo(this.model, 'change:address', this.showChangedAddress);
	},
	showChangedAddress : function(){
		// we are using the same main view template here though 
  		// another subtemplate for only the address part can 
  		// anyway be used here
  		var html = this.template(this.model.toJSON()),

    	// Selector of the element whose value needs to be updated
    	addressElSelector = ".address",

    	// Get only the element with "address" class
    	addressElement = $(addressElSelector, html);  

  		// Replace only the contents of the .address element
  		this.$(addressElSelector).replaceWith(addressElement);
	}
})
```

###SubViews : 
It is essential to understand when we should use sub views and when we shoudl use render the content in the same view

Refer to example code : 
[Without SubViews](http://jsbin.com/yimori/18/edit?html,js,output)
[With Subviews](http://jsbin.com/vipife/12/edit?html,js,output)

We used jQuery's $.append() method to add the subview elements to the main view. It is found that if there is a large collection of data, appending view elements to the DOM one by one can create a severe performance issue; this will affect the UI responsiveness of the application. The performance hit can be noticed even in modern browsers, since every append causes a DOM reflow and forces the browser to recalculate the size and position of the DOM tree.

Use documentFragment instead : 
[Document Fragment](http://ejohn.org/blog/dom-documentfragments)
This multiple DOM reflow can be avoided by using DocumentFragment, which is described as a lightweight container that can hold DOM nodes. We can collect all of the view elements inside DocumentFragment and then append this fragment to the DOM. This will cause a single reflow for the complete collection, and hence a performance improvement.

```
render: function () {
  // create a document fragment
  var fragment = document.createDocumentFragment();

  this.collection.each(function (model) {
    // add each view element to the document fragment
    fragment.appendChild(new UserItemView({
      model: model
    }).render().el);
  }, this);

  // append the fragment to the DOM
  this.$el.html(fragment);
  return this;
}
```