---
layout: post
title: "IS IT REALLY HARD TO UNDERSTAND CLOSURES?"
date:   2014-07-19 21:06:00
categories: Javascript.
---

What is closure and why should I use it?
When I first started studing Javascript, I saw myself spending a lot of time trying to figured out what was  closure, how it worked and the most important thing, **Why should I use Closures?**.

>The main idea here is try to show the way that I started to understand closure, and also make you spend less time than me.

Let's start it the easy way.

Imagine that we'd like to create a class called Car. `Just make sure that a Javascript class is not like any other language class`. We can have a code like this:



{%highlight javascript %}
	
function Car (pModel, pColor, pYear){
    var model = pModel;
	var color = pColor;
	var year = pYear; 	
	var getTypeOfCar = function(){
		console.log("My new Car is " + _model)
	}	
};
var brandNewCar = new Car("Ford", "Red", 2014);

{%endhighlight%} 


Try it yourself on console, `brandNewCar.getTypeOfCar()`. You'll notice that it is undefined, it doesn't exist. But what happend?

The variables are just available when the function is being executed, after that the browser (more especific, the virtual machine running inside the browser) clean up anything after the function finishes its own execution. That's why you can not access them.

To fix that you have to instead of variables, create attributes inside the function (Class Car). This way you can access the values inside the Class.

{%highlight javascript %}
	
function Car (pModel, pColor, pYear){
    this.model = pModel;
	this.color = pColor;
	this.year = pYear; 	
	this.getTypeOfCar = function(){
		console.log("My new Car is a brand new " + model)
	}	
};
var brandNewCar = new Car("Ford", "Red", 2014);

{%endhighlight%} 



Now if we call the `brandNewCar.getTypeOfCar()` function, "My new Car is a brand new Ford", it will appear on console. Moreover, not only the method is available, but also every attribute is available as well.




###Putting Closure on Scene

Let's make things more useful. Imagine that we don't want to expose all the content from our Class, we'd like to have some private and public variables and/or function.



{%highlight javascript %}
	
function Car (pModel, pColor, pYear){
    var model = pModel;
	var color = pColor;
	var year = pYear; 	
	return { getTypeOfCar : function(){
		console.log("My new Car is a brand new " + model)
		}
	}	
};
var brandNewCar = new Car("Ford", "Red", 2014);

{%endhighlight%} 


Run the code to see that only `brandNewCar.getTypeOfCar()` is available to you, if you try to `brandNewCar.model`, it shows `undefined`. This is because the variable model, as said before, is only available when the function is being executed, now it is a kind of private variable of our Class Car. Ask yourself, how the function `getTypeOfCar()` can access the variable `model`? Closure is the answer.

###What is Closure?

Simple like that: Closure is a function that has access to a private variable on the inner scope function. Is it getting worse? Open one beer and just relax (My favorite one is Blue Moon).


In our example, `getTypeOfCar()` receives a function that has access to the lexical scope, and then use a model variable declared.

The function `getTypeOfCar()` is a kind of closure! Closures are blocks of code that can be executed by keeping the original context that they were created (here the `getTypeOfCar()` is the closure), and this simple example shows one of the benefits of this technique (private and public members).


Other Example of Closure:

 
{%highlight javascript %}
	
function GiftList (){
    var _list = [];
    var orderList = function(){
			return _list.sort();
		}
	return { 
		 	addNewGift: function(gift){
				_list.push(gift);
			},
	     seeMyList: function(){
		return orderList(_list).join(" - ");
	}
  };	
};
var list = new GiftList();
list.addNewGift("Purse");
list.addNewGift("Shoes");
list.addNewGift("Watch");
list.seeMyList();
// "Purse - Shoes - Watch"

{%endhighlight%}



It sounds simple now, but I spent a few days to understand it at first.

See you guys. Take care!