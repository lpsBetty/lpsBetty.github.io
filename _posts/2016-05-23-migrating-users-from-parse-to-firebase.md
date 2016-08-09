---
layout: post
title: "Migrating users from Parse to Firebase"
permalink: migrating-users-from-parse-to-firebase
date: 2016-05-23 23:23:23
comments: true
description: "Migrating users from Parse to Firebase"
keywords: "parse, firebase, migration, javascript, angular"
tags:
- parse
- firebase
- migration
- javascript
- angularjs
---

As some of you may know the [Parse](https://www.parse.com) hosted service is shutting down on January 28, 2017. I was looking for an alternative because I don't want to self-host a Parse Server. First of all I read some blog posts and revised the features of Parse that are used by my small app [showmaniac](http://showmaniac.org). 

Feature Requirements:

* Saving Data to an User Object/Record
* Authentication with
  * Email / Password
  * Facebook oAuth

<br>

### Migrate to Firebase 

[Firebase](https://www.firebase.com/) seems like the easiest and most popular alternative in my case. I was really quick rewriting the front-end JS code and everything, but the most difficult task was migrating the small user base from Parse to Firebase:

I had 2 different types of user authentications: Email with Password and Facebook oAuth. When exporting the User Data from Parse this looks like this: 

{% highlight javascript %}{% raw %}
{  // 'normal' user
  "bcryptPassword": "$2a$10$8XeeJNTVXjgyMFqqKGQD1Q/IYNVAzGse",
  "email": "betty@example.com",
  "username": "betty",
  ...
}, { // facebook user
  "authData": {
    "facebook": {
      "access_token": "CAAGFLXc9SkMBACyvT3QG1JQZDZD",
      "expiration_date": "2016-03-09T15:00:00.622Z",
      "id": "1000000000"
    }
  },
  "bcryptPassword": "$2a$10$uoRqTIiResUUiWR869ULNuUoVYGhxUrO",
  "username": "YVEy6wiGg11jemrEUgJRojyHx",
  ...  
}
{% endraw %}{% endhighlight %}


Unfortunately Firebase doesn’t allow importing users, so I just found this [stackoverflow question](http://stackoverflow.com/questions/16053273/firebase-import-users-from-existing-app), which can't do the trick, but it got me thinking!

Here are the 2 solutions, for the 2 types of users I had. 

<br>

#### 1. Users with Facebook Authentication

It was quite easy to write a small JS script by using the [Firebase Web API](https://www.firebase.com/docs/web/api/firebase/authwithoauthtoken.html). This script will authenticate with Facebook using an existing OAuth 2.0 access token - the access token was saved (and updated when the user logs in) in your Parse Database. 

The downside, it does only work for active users, so when the expiration date is expired, the login will fail :(   --  That's why I added a check for not making an extra call to Firebase:

{% highlight javascript %}{% raw %}
var parseUsers = [{ "authData": { "facebook": { "access_token": "CAAGFLXc9SkMBACyvT3QG1JQZDZD", "expiration_date": "2016-03-09T15:00:00.622Z" }} }];
var ref = new Firebase('https://<YOUR-FIREBASE-APP>.firebaseio.com');

parseUsers.forEach(function(user) {
  var isNotExpired = user.authData && new Date(user.authData.facebook.expiration_date) > new Date();
  if(isNotExpired) {
    ref.authWithOAuthToken('facebook', user.authData.facebook.access_token, function(error, authData) {
      if (!error) {
        console.log('Authenticated successfully with payload:', authData);
        var refUser = new Firebase('https://<YOUR-FIREBASE-APP>.firebaseio.com/users/').child(authData.uid);
        refUser.set({username: user.username}); // set user attribute
      }
    });
  }
});
{% endraw %}{% endhighlight %}

If you want to run this JS script [here](http://js.do/code/migrate-parse-users-to-firebase), just replace the `parseUsers` array and the firebase URL.

<br>

#### 2. Users with Email & bcyrpted Password or MIGRATE THEM ALL!

![Migrate them all!](https://cdn.meme.am/instances/200x/68515726.jpg){: .pull-right}

The **REAL** question is: How can you import users from Parse to Firebase, when the password is bcyrpted? 

The only answer and solution that worked for me was to:

1. do Parse and Firebase authentication side by side
2. put Parse authentication 'in the background'
3. get user data from Parse to Firebase behind the scenes

Let's go in more detail and show some code examples. When the user comes to your app and the Parse user is still logged in, you need them to **logout and login again** to create a Firebase authentication. 

{% highlight javascript %}{% raw %}
var currentParseUser = Parse.User.current();
if(currentParseUser) {
   // help them login again with their Parse credentials (when no facebook auth)
  if(!currentParseUser.get('authData')) {
    user.username = currentParseUser.get('username');
  }
  Parse.User.logOut();
}
{% endraw %}{% endhighlight %}


My login form is also my register form (was always like this), which makes it easier now. So when the login function (see below) is called this happens:

1. Check if Firebase authentication exists (login)
  * yes ⇾ login ✓
  * no  ⇾ sign up ⇾ see 2.
2. Create Firebase authentication (sign up) - after success:
  * Parse user exists ⇾ get Parse data ⇾ login ✓
  * Parse user doesn't exist ⇾ login ✓

I currently use AngularFire, but you can also accomplish this with the latest [Firebase SDK](https://firebase.google.com/docs/).

{% highlight javascript %}{% raw %}
var login = function(user) {
  // Auth = $firebaseAuth
  Auth.$authWithPassword(user).catch(function(error) {
    if(error.code === 'INVALID_USER') {
      Auth.$createUser(user).then(function() {
        // after creating Firebase auth, get user data from Parse
        Parse.User.logIn(user.email, user.password, {
          success: function(parseUser) {
            user.username = parseUser.get('username'); // set user data
            login(user);
            Parse.User.logOut();
          }, error: function () {
            login(user);
          }
        });
      }).catch(showError);
    } else {
      showError(error);
    }
  });
};
{% endraw %}{% endhighlight %}



For **Facebook Authentications** you can do the same, but there will be 2 pop-up authentication windows, which is not very nice. That's why I would recommend only getting the *FB Parse user*, when the necessary data is not yet set for the *FB Firebase user*. This code gets called after the user has been authenticated via Facebook to Firebase:


{% highlight javascript %}{% raw %}
if(firebaseUser.authData.provider === 'facebook' && !firebaseUser.username.length) {
  // check if parse user has username
  Parse.FacebookUtils.logIn(null, {
    success: function(parseUser) {
      if (parseUser.existed()) {
        firebaseUser.username = parseUser.get('username');
      }
      Parse.User.logOut();
    }
  });
}
{% endraw %}{% endhighlight %}

Next time the user logs in via Email/Password or Facebook, no additional request will be made to Parse, which means: 

A successful sign up to your Firebase project **completes the migration** of this user!
{: .txt-center}

<br>

All active users can be safely moved to Firebase, without them even noticing. If they do not login until Parse shuts down, they will lose their data & account - so it may be nice to notify your users, that they should login (once) in the next months. ;)


---
  
##### References

[Firebase.authWithOAuthToken()](https://www.firebase.com/docs/web/api/firebase/authwithoauthtoken.html)  
[AngularFire - Users and Authentication](https://www.firebase.com/docs/web/libraries/angular/api.html#angularfire-users-and-authentication-createusercredentials)  
[StackOverflow - Q1](http://stackoverflow.com/questions/36185483/how-to-migrate-data-from-parse-com-to-firebase)  
[StackOverflow - Q2](http://stackoverflow.com/questions/16053273/firebase-import-users-from-existing-app)    
