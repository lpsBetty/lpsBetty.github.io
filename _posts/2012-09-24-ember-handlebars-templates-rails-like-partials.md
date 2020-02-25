---
layout: post
title: "Ember Handlebars templates (Rails-like partials)"
redirect_from: /ember-handlebars-templates-rails-like-partials/
date: 2012-09-24 10:39:01
comments: true
description: "A small tutorial how to embed a Handlebars template like a Rails partial"
keywords: "ember, rails, handlebars, partials, template"
tags:
- emberjs
- handlebars
- javascript
- ruby on rails
category: Javascript
banner_preview: home2.jpg
banner_image: home2.jpg
---

I was looking for Handlebars partials (or templates that i could embed) that I can render like partials in Rails:

{% highlight ruby %}{% raw %}
render :partial => "partial_name"
{% endraw %}{% endhighlight %}

#### 1. registerPartial in js

You can either register a partial for a **small inline code snippet**:

{% highlight javascript %}{% raw %}
/* partials.js */
Handlebars.registerPartial("time",
  '<time datetime="{{created_at}}">{{created_at}}</time>'
);
/* file.handlebars */
some text created at: {{> time}}
{% endraw %}{% endhighlight %}

#### 2. Template File (.handlebars)

or the more convenient way: create a new template file (eg. *time.handlebars*) and include it with Ember.Handlebars helper „template“, like so:

{% highlight javascript %}{% raw %}
/* time.handlebars */
<time datetime="{{created_at}}">{{created_at}}</time>

/* file.handlebars */
some text created at: {{template "path/to/file/time"}}
{% endraw %}{% endhighlight %}

That’s it, pay attention to the right path otherwise Ember will not find the template (just see the console for the error message)!


