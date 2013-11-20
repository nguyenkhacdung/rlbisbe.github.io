---
layout: post
title: "Angular.js for Windows Store apps: First look"
date: 2013-11-20 20:00
published: true
comments: true
---

There are a wide variety of javascript frameworks so far, such as Angular, Ember or Knockout, which provide a lot of abstractions, and one feature loved by WPF developers, **Data binding**, and the possibility of using patterns like Model-View-ViewModel (MVVM).

In this article I do a sneak peak at Angular.js, a framework developed by Google that has been trending for the last months. For making it more interesting, I use it within the context of an Windows 8 application, having some extra constraints.

# Installation and configuration

For this project, the first step is to create an empty Windows 8 application with Javascript, you can use Visual Studio Express to do this.

To include **Angular.js**, we could think about linking it from the Angular CDN to our default.html file:

{% highlight html %}<script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.1/angular.min.js">
</script> {% endhighlight %}

However, this cause the following error:

![Using the CDN's Angular JS]({{site.url}}/assets/usingcdn.png)

In Windows Store apps we cannot use a non-local reference to **javascript** files, so we must download them and include them back in our project manually. We could also be tempted to install it using nuget, but in my opinion this causes too many files added to the project.

As a quick note, I recommend to download **angular.js** version, not the minified **angular.min.js**, because we will need to make some changes later on.

Once we have the file added to our project, if we try to compile we will receive a second surprise:

![Exccepcion angle]({{site.url}}/assets/epicexception.png)

There are many ways to fix this error, in my case the fix included debugging the problematic function, and replacing the call to the function **insertBefore** by the following line of code:

{% highlight javascript %}MSApp.execUnsafeLocalFunction(function (){element.insertBefore(child, index);}); {% endhighlight %}

With this slight change, we can now run our application.

# Views

To display information on the screen, rather than build the HTML manually and edit the DOM from javascript code, we simply use certain conventions for embedding data within an HTML template.

{% highlight html %}
{% raw %}
<label>Name:</label>
<input type="text" ng-model="yourName" placeholder="Enter a name here" />
<h1>Hello {{yourName}}!</h1>
{% endraw %}
{% endhighlight %}

In this case, the input generates a variable called **yourName**, and updates the <strong>h1</strong> header field, inside the curly braces. These fields with braces can also be used within html elements, for example, to set the "src" property of an image:

{% highlight javascript %} {% raw %}<img src="{{item}}" /> {% endraw %} {% endhighlight %}

We don't have to worry about the update process, since changing the content of the data causes that what is showing on the screen gets rendered again to show the update. This represents a difference quite interesting with respect to other frameworks such as Knockout, where we needed to define a very specific type of object (**ko.observable**) for having this feature.

# Controllers

The controller is defined as a javascript class, with properties and functions, which are later called from the HTML markup, for example:

{% highlight javascript %}

function TodoCtrl($scope) {
$scope.todos = [];

$scope.addTodo = function () {
$scope.todos.push({ text: $scope.todoText, done: false });
        $scope.todoText = '';
};

};

{% endhighlight %}

This sample defines a controller, a field called **todos**, and a method to store the value defined in the variable in the list **todoText**. The $context variable is similar to the standard _this_ of javascript, although with some variations. The corresponding **view** looks like this:

{% highlight html %}
{% raw %}
<h2>Todo</h2> {{todos.length}}
<form ng-submit="addTodo()">
	<input type="text" ng-model="todoText" placeholder="add new todo here" size="30" />
	<input class="btn-primary" type="submit" value="add" />
</form>
{% endraw %}
{% endhighlight %}

Unlike seen previously, in this code, we define a controller, we access the field length of the object *todos* defined above, and finally we call the method as a result of the form submission to storing the data on the input.

# Loops (and loops within loops)

Another interesting thing that has, is the use of loops for displaying information:

{% highlight html %}
{% raw %}
<ul>
	<li ng-repeat="todo in todos">
		<p>{{todo.name}}</p>
		<span ng-repeat="item in todo.items">
			<img src="{{item}}" /></li>
	</li>
</ul>
{% endraw %}
{% endhighlight %}

In this case represent the structure of **todos**, and assuming that **todo** is a composite structure, these loops are also can be nested, achieving greater freedom when it comes to represent complex data.

# HTTP calls

To make calls to an external server using http we only need the **url** and the **method** by which we will do the request, so we call the method it this way:

{% highlight javascript %}}

$http({method: 'GET', url: 'http://url'}).
success(function (data, status, headers, config) {})

}).
error(function (data, status, headers, config) {})

});

{% endhighlight %}}

We can include this code in our controller, so it's executed when we load it. If we do so without any aditional changes we may will find a nice error as shown below:

![Http error]({{site.url}}/assets/errorHttp.png)

To avoid this error, we'll have to modify the definition of our controller:
{% highlight javascript %} function TodoCtrl ($scope) {% endhighlight %}

So now in addition to $scope, we need to add $http:

{% highlight javascript %} function TodoCtrl ($scope, $http) {% endhighlight %}

# Recap

As I wrote at the beginning, this is just a sneak peek of the things we can do with Angular. It looks very promising, and the available docs make the learning curve easy to handle.

# Additional links

* [Angular.js Homepage] (http://angularjs.org)
* In StackOverflow: [How do you get Angular.js to work in Windows 8 store app?](http://stackoverflow.com/questions/12792383/how-do-you-get-angular-js-to-work-in-a-windows-8-store-app)
* En nettuts+: [Building a Web App From Scratch in AngularJS](http://net.tutsplus.com/tutorials/javascript-ajax/building-a-web-app-from-scratch-in-angularjs/?search_index=6)
