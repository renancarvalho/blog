---
layout: post
title:  "PUBLISHING ON NPM"
date:   2015-05-30 15:05:00
categories: Javascript
---



Recently I had to publish a [package](https://github.com/renancarvalho/CsvToCollection "csvtocollection") on npm. Then I realized, why not write a quick post about it?

You probably agree with me that [npm](www.npmjs.com) is awesome and saves our lives when the subject is project dependencies. To follow this article, I recommend you to first understand how npm install works, before reading this post, if you are familiar with that go ahead.

###Why should you publish your package?

I have one thing in mind:
> If one thing saves your life, it can save other lives too.

How many packages, scripts, tools and everything else have been saving your life since you started as a developer? So yes man, go ahead and publish it, don't be scared if it isn't the best code. Of course you should worry about the code, but let the community help you, why not? 


###Registering on npm

There are two ways to do that, the normal way via [npm website](https://www.npmjs.com/signup "npm web site"), or you can do it as a developer using command line :)

	npm set init.author.name "Your Name"
	npm set init.author.email "Your Email"
	npm set init.author.url "Your Url"
	
	
So far so good, we told npm that we want to use that information to create the user.


	npm adduser
	


After run `npm adduser`, npm will ask for an username, password and email. Of course you need to choose one username that is not already registered. If you fill out all the 'fields' correctly your account will be created. To make sure, go to npm site and log in using the same username and password. That should work.
	
	
###Creating your package

To create your package you need to run `npm init` and fill out all the steps that npm requests. Some steps have a default value, so if you don't tell npm anything, it will assume the default value in parentheses.


###The icing on the cake

After completing your code, you'll be ready to publish it on npm. To do that you need to run:


	npm publish
	
It will take a while depending on the connection and package size. But once it is finished, your package will be ready to help other developers.


**Important note:**

When you create the package with `npm init`, npm will ask for the package name, you need to use a new name, if not you wont be able to publish that, but don't worry you can always update the package.json file.


Welcome to the open source world.
