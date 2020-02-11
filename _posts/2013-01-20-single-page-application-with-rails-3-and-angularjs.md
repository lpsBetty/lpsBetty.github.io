---
layout: post
title: "Single-Page application with Rails 3 and AngularJS"
redirect_from: /single-page-application-with-rails-3-and-angularjs
date: 2013-01-20 16:39:18
comments: true
description: "A small tutorial how to use AngularJS with Ruby on Rails 3"
keywords: "angular, rails, tutorial"
tags:
- angularjs
- deployment
- javascript
- ruby on rails
category: Javascript
banner_preview: home2.jpg
banner_image: home2.jpg
---

#### What is AngularJS?
AngularJS is an open-source JavaScript framework. It extends HTML’s syntax to express your application’s components clearly and define AngularJS functions, for example:

{% highlight html %}{% raw %}
<ul>
  <li ng-repeat="show in shows">
    <a href={{show.link}} target="_blank">{{show.name}}</a>
  </li>
</ul>
{% endraw %}{% endhighlight %}

AngularJS automatically synchronizes data from your UI (view) with your JavaScript objects (model) through 2-way data binding. It also helps with server-side communication, taming async callbacks with promises and deferreds; and make client-side navigation and deeplinking with hashbang urls or HTML5 pushState a piece of cake.


### Rails 3 and AngularJS – a small tutorial

First of all, create a new Rails app, and change your html-tag to `<html ng-app>`. To setup AngularJS link to the newest version (see [angularjs.org](http://angularjs.org/)) and include your app/assets/javascripts/application.js after that in your layout file (eg: application.html.erb):

{% highlight html %}{% raw %}
<script src="http://ajax.googleapis.com/ajax/libs/angularjs/1.0.3/angular.min.js"></script>
<%= javascript_include_tag "application" -%>
{% endraw %}{% endhighlight %}

You have to intercept html requests to create a single-page application. If you have your root (eg: home#index) you can easily put a before_filter in your ApplicationController to render ‘home/index‘ or even another layout every time the request format is HTML.

{% highlight ruby %}
class HomeController < ApplicationController
  respond_to :html
  def index
  end
end

class ApplicationController < ActionController::Base
  protect_from_forgery
  before_filter :intercept_html_requests

  private
  def intercept_html_requests
    render('home/index') if request.format == Mime::HTML
  end
end
{% endhighlight %}

I recommend to read the [conceptual overview](https://docs.angularjs.org/guide/concepts). Understand Angular’s vocabulary and how all the Angular components work together. Watch the [AngularJS Tutorial](https://docs.angularjs.org/tutorial/index) (with node.js). It explains every major AngularJS feature and also gives a live demo.

Create a new JS File (or use your application.js) and try creating your first AngularJS controller.

JS:
{% highlight javascript %}{% raw %}
function ShowsCtrl($scope) {
  $scope.shows = [{
    id: 1,
    name:'Big Bang Theory',
    seen:true
  }, {
    id: 2,
    name:'Breaking Bad',
    seen:false
  }];
}
{% endraw %}{% endhighlight %}

The data is now hardcoded in the controller, later **$scope.shows** will be defined with data from an external API. (you can also use rails routes and *respond_to :json* if you use Rails Models)

HTML:
{% highlight html %}{% raw %}
<div ng-controller="ShowsCtrl">
  <ul>
    <li ng-repeat="show in shows">
      {{show.name}}
    </li>
  </ul>
</div>
{% endraw %}{% endhighlight %}


As you see, you can define one part of your site to use AngularJS – and you can make many separated controllers.

To load real data by ajax call, you need to add $http in your controller function:

{% highlight javascript %}{% raw %}
function ShowsCtrl($scope, $http) {
  $http.get('/shows.json').success(function(data) {
    $scope.shows = data;
  });
}
{% endraw %}{% endhighlight %}





#### Add a module
For example, you would like to add localstorage functionality to your angular app. First look for an angular module, before you write one by yourself ;) –> [github.com/grevory/angular-local-storage](https://github.com/grevory/angular-local-storage)

Include the external JS file in your app/assets/javascripts folder – you can see this line on the top of the file:

{% highlight javascript %}{% raw %}
var angularLocalStorage = angular.module('LocalStorageModule', []);
{% endraw %}{% endhighlight %}

Now it’s time to define your own app (*appname*) and add this module (*LocalStorageModule*). You can add as many modules as you like. In *'LocalStorageModule'* a service called *'localStorageService'* is defined. To use this service you have to include it in your controller initialization next to *$scope* and *$http*.

{% highlight javascript %}{% raw %}
angular.module('appname', ['LocalStorageModule']);

var ShowsCtrl = ['$scope', '$http', 'localStorageService', function($scope, $http, localStorageService) {
  localStorageService.add('ids', [1,2]);
}
{% endraw %}{% endhighlight %}

Last step: update the html-tag and define the app name. Now you can use *'localStorageService'* in your controller.

{% highlight html %}{% raw %}
<html ng-app="appname">
{% endraw %}{% endhighlight %}



#### Code examples
Below are some code snippets with examples how to use it (in comments above every function).

{% highlight javascript %}{% raw %}
/**
 * Gets number of seen shows.
 * @example
 *   {{seenCount()}} of {{shows.length}} seen
 * @return {Integer} seen count
 */
$scope.seenCount = function() {
  var count = 0;
  angular.forEach($scope.shows, function(show) {
    count += show.seen ? 1 : 0;
  });
  return count;
};

/**
 * Gets called everytime searchId is changed ($watch).
 * If the user chooses a search result the searchId is set and
 * the function adds this show if not null.
 */
$scope.searchId = null;
$scope.$watch('searchId', function() {
  if( $scope.searchId !== null ) {
    $scope.addShow($scope.searchId);
  }
});

/**
 * Adds show to $scope.shows if not already in array.
 * @param {Integer} id show
 */
$scope.addShow = function(id) {
  var show = $scope.loadShow(id);
  if($.inArray(show, $scope.shows) !== -1) { return; }

  $scope.shows.push(show);
};

/**
 * Toggles seen of a show
 * @example
 *   <li ng-click="toggleSeen(show)"></li>
 * @param  {Object} show
 */
$scope.toggleSeen = function(show) {
  show.seen = show.seen ? false : true;
};

/**
 * Removes all special chars, all except numbers, letters and spaces.
 * "Shameless (US)" => "Shameless US"
 * @example
 *   <a href="https://www.google.com/search?q={{parseName(show.name)}}">
 * @param  {String} name tvshow
 * @return {String}      name without special chars
 */
$scope.parseName = function(name) {
  return name.replace(/[^ a-zA-Z0-9]/g,'');
};
{% endraw %}{% endhighlight %}

#### Deployment
There can be a minification problem if you use Rails with AngularJS (the Rails Asset Pipeline minifies all asset files). I got an error when deploying on heroku:

**Uncaught Error: Unknown provider: e**

before I did this: (not wrong, but leads to an error!)

{% highlight javascript %}{% raw %}
app.controller("ShowsCtrl", function($scope, $http) { }
{% endraw %}{% endhighlight %}

According to AngularJS tutorial ([docs.angularjs.org/tutorial/step_05](https://docs.angularjs.org/tutorial/step_05)) you can change this line to the following to prevent minification issues:

{% highlight javascript %}{% raw %}
var ShowsCtrl = ['$scope', '$http', function($scope, $http) { }
{% endraw %}{% endhighlight %}


I hope I’ve helped some of you to get started with AngularJS and Rails.

**Happy coding!**
