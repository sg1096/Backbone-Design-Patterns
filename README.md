# Backbone.js Design Patterns
Design Patterns
==============================

### Re-using code with extensions

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



