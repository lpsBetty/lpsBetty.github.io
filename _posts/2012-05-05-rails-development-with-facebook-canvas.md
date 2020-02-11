---
layout: post
title: "Rails Development with Facebook Canvas"
date: 2012-05-05 15:23
comments: true
description: "Run your Rails development environment inside facebook canvas."
keywords: "facebook, rails, development, localhost"
tags:
- facebook
- development
- ruby on rails
category: Rails
banner_preview: home2.jpg
banner_image: home2.jpg
---

I was trying to make an existing app run as a Facebook App!

The following instructions/solutions will help you to access your [http://localhost:3000/](http://localhost:3000/) through Facebook. In addition I would like to get the Facebook user with the [fb_graph gem](https://github.com/nov/fb_graph/) and save it in my database.

First of all you have to go to [developers.facebook.com/apps](https://developers.facebook.com/apps/) to create a new Facebook test app.

The only important setting for now is this one:

![Canvas Setting](/images/posts/canvas_setting.png)
{: .txt-center}

I have also enabled the sandbox mode in the advanced settings, so that only I can access the Facebook app.



My app is really small so I’ve decided to only make an **before_filter** if the user uses Facebook to access my app:


{% highlight ruby %}{% raw %}
class HomeController < ApplicationController
  before_filter :facebook_authorize

  private

  def facebook_authorize
    return unless params[:signed_request]
    @auth = FbGraph::Auth.new FACEBOOK[:key], FACEBOOK[:secret]
    @auth = @auth.from_signed_request(params[:signed_request])
    if @auth.authorized?
      f = Facebook.find_or_initialize_by_identifier(@auth.user.identifier.try(:to_s))
      f.access_token = @auth.user.access_token.access_token
      f.save!
      session[:current_user] = f.id
    else
      render :authorize
    end
  end
end
{% endraw %}{% endhighlight %}


My **Facebook Model** only has two attributes: *identifier* and *access_token*.

If there is no **params[:signed_request]** nothing will happen because it is not a Facebook Canvas. Otherwise an unauthorized user (every user has to give explicit permission to use the app) have to get redirect to authorize the app:

The `authorize.html.erb` looks like this:

{% highlight html %}{% raw %}
<script>
  top.location.href = '<%= @auth.authorize_uri("https://apps.facebook.com/YOUR_APP_NAME/").html_safe %>';
</script>
{% endraw %}{% endhighlight %}



### API Error Code: 191

API Error Description: The specified URL is not owned by the application
Error Message: Invalid redirect_uri: Given URL is not allowed by the Application configuration.

I got this error when redirecting to the authorize url. The problem was my canvas url! I’ve used this:

{% highlight ruby %}{% raw %}
@auth.authorize_uri("https://apps.facebook.com/12345/").html_safe
{% endraw %}{% endhighlight %}


This does not work with the **App ID**, you have to specify an **App Namespace** in the basic settings – then the error will disappear, like so (App Namespace is „app_localhost“):

{% highlight ruby %}{% raw %}
@auth.authorize_uri("https://apps.facebook.com/app_localhost/").html_safe
{% endraw %}{% endhighlight %}



