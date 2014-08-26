---
layout: post
title:  "USING LOG-ERROR WEB COMPONENT"
date:   2014-08-25 23:28:00
categories: Javascript
---


Hey there, Log-Error is a Web Component (the first one that I made using <a href='http://www.polymer-project.org/' target='_blank'>Google Polymer)</a> that helps you manage 
Javascript errors that maybe is not available for you to debug.

The idea of this Web Component is just "listen" to every error that can happen in your App, then send it to your Api.

#Log-Error Web Component

The code is avaiable on <a href='https://github.com/renancarvalho/log-error' target='_blank'>GitHub</a>

##How to use:

####First import <a href='http://www.polymer-project.org/docs/start/platform.html' target='_blank'>platform.js</a>

<pre><span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"bower_components/platform/platform.js"</span><span class="nt">&gt;&lt;/script&gt;</span>
</pre>


####After that, lets import polymer.js, log-error.html and exceptionHandle.js

<pre><span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"bower_components/polymer/polymer.html"</span><span class="nt">&gt;&lt;/script&gt;</span>
</pre>

<pre><span class="nt">&lt;link</span> <span class="na">rel=</span><span class="s">"import"</span> <span class="na">href=</span><span class="s">"log-error.html"</span><span class="nt">&gt;</span>
</pre>

<pre><span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"exceptionHandle.js"</span><span class="nt">&gt;&lt;/script&gt;</span>
</pre>


####Then use the component

<pre><span class="nt">&lt;log-error</span> <span class="na">api-uri=</span><span class="s">"PathToYourApi"</span><span class="nt">&gt;&lt;/log-error&gt;</span>
</pre>


####Api-Uri

There you need to specify the Uri of your api to receive the errors.

**Note: Be careful with** <a href='http://www.html5rocks.com/en/tutorials/cors' target='_blank'>CORS.</a>

##Parameters:

The Web Component will pass three informations through the ajax POST request.

- referenceError - The error that occurs
- lineError - The line that the error occurred
- targetFileError - The file that has the error




#Contributing

<ol class="task-list">
<li>Fork it!</li>
<li>Create your feature branch: <code>git checkout -b my-new-feature</code>
</li>
<li>Commit your changes: <code>git commit -m 'Add some feature'</code>
</li>
<li>Push to the branch: <code>git push origin my-new-feature</code>
</li>
<li>Submit a pull request \0/.</li>
</ol>






