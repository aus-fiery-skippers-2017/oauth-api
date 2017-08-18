# OAuth + Twitter API

## What is an API? ([watch this video](https://www.youtube.com/watch?v=s7wmiS2mSXY))

## What is OAuth?

![oauth diagram](http://www.cubrid.org/files/attach/images/220547/370/421/oauth_10a_authentication_process.png)

## Learning Competencies

* Incorporate third-party gems into a web application using bundler
* Extend the Sinatra application environment with a ruby gem
* Implement user login and authentication in a web application
* Use a third-party API
* Implement OAuth
* Deploy a web application on a hosting server (like Heroku)

## Summary

Let's add support for logging in with Twitter.  This will be our first application that uses OAuth for authentication.

Twitter uses [OAuth version 1][OAuth v1] for API authorization. OAuth (and particularly, OAuth version 1) is hard. As such, you'll find that we've provided a skeleton for you that handles the nasty OAuth bits. Later, if you are feeling adventurous, you can try to code-up your own OAuth conversation.


## Releases

### Release 0 - Run the Skeleton

Create an application that allows a user to sign in via OAuth from Twitter and then send a tweet.

**BEFORE** you start, spend time reading the [Twitter OAuth Documentation][] and familiarize yourself with the basics of OAuth version 1. See if you can relate what you learn to the image below.

#### Configuring Your Environment

Your Twitter "consumer key" and "consumer secret" answer the question, "What application is doing the acting?"

The application skeleton we've provided expects you to have `CONSUMER_KEY` and `CONSUMER_SECRET` environment variables defined for your running server. This is so that we can deploy things securely to Heroku later.

Since these keys are associated with *your account* on Twitter, *they should be kept secret*. That is, don't include them directly in your `environment.rb` file. Instead, you should set `export` the environment variables before you run the server:

```bash
$ export CONSUMER_KEY=<your_twitter_consumer_key>
$ export CONSUMER_SECRET=<your_twitter_consumer_secret>
$ shotgun
```

Since you are using OAuth, you will also need to set a "callback function" which is the route the application will be directed to after the user is authenticated.  When you are using localhost, this callback function is actually set in the request token (see the helper method in your skeleton). **BUT** you still need to set something in the callback field when registering your application with Twitter.  As you can read in [this thread][OAuth Twitter Callback Configuration Discussion] you can actually set this to any valid url and it will be overwritten by the attribute of your request token.

#### How the OAuth (v1) Conversation Works

Unlike your previous Twitter applications, the "access token" and "access token secret" will have to change *depending on what user is currently authenticated*. This key pair answers the question, "On whose behalf is this application acting?"

Twitter needs to answer both of these questions to make sure that the application is valid and that the application can only do what it has permission to do on behalf of an authenticated user.

The core OAuth flow goes like this:

1. Application generates URL to "Sign In with Twitter".
2. Application renders page with "Sign In with Twitter" link
3. User clicks "Sign In with Twitter"
4. User is redirected to Twitter and authorizes the application
5. User is redirected back to the application's callback URL
6. Application verifies the redirection from Twitter is valid
7. If valid, Application takes appropriate action

![OAuth v1 Conversation][OAuth v1 Conversation Diagram]

#### Getting It Running Without Code Changes

Assuming you've configured things properly, the skeleton code provided should just run out of the box. However, the skeleton app doesn't do anything related to creating the user, logging her in, etc. You'll need to implement that part on your own later. For now, though, just make sure the application skeleton works (which will prove that your environment and Twitter application are configured correctly).


### Release 1 - Create Users with Access Tokens in Database

Now you'll need to implement that last step of the OAuth flow. Specifically, you'll need to create the new user, set her as "logged in", store her access token and secret along with her user record, etc. This should happen inside of the `/auth` route in your `controllers/index.rb` file:

```ruby
get '/auth' do
  # the `request_token` method is defined in `app/helpers/oauth.rb`
  @access_token = request_token.get_access_token(:oauth_verifier => params[:oauth_verifier])
  # our request token is only valid until we use it to get an access token, so let's delete it from our session
  session.delete(:request_token)

  # at this point in the code is where you'll need to create your user account and store the access token

  erb :index
end
```

We'll want a `User` model, however the user won't have a password.  Instead, the user will be authenticated via OAuth.  With OAuth there's not necessarily a distinction between signing up and logging in &mdash; the first time a user authenticates via Twitter we can create the `User` object. Instead of a password, you'll want to have columns to store her "access token" and "access token secret".

### Release 2 - Tweet on behalf of your user

Now that you have an authenticated user who has authorized you to use Twitter on their behalf, you should extend your app with similar functionality from the [Tweet Now! 1: Single User Challenge][] so that you can send some tweets on behalf of the user that has been authenticated with OAuth.

### Release 3 - Deploying Securely to Heroku

**STOP** - Go find your laptop and use it to deploy to Heroku - please (no really please) do not reset the SSH keys on DBC machines.

We don't want our confidential information (like application keys and secrets) to be stored in git, especially if we're going to push this to a public repository. If we *were* to store them in a public repository, anyone would be able to pretend to be our application. **NOT good.**

Configuring the `CONSUMER_KEY` and `CONSUMER_SECRET` environment variables on our local machine was easy. [On Heroku, it's slighly more complicated][Heroku Configuration Variables Documentation]:

```bash
$ heroku config:add CONSUMER_KEY=<your_twitter_consumer_key> CONSUMER_SECRET=<your_twitter_consumer_secret>
```

After that, you should be able to deploy! :)

Get to it! On your own machine, you should:

1. Create a new Heroku application using `heroku create`.
2. Add Heroku as a remote destination for git using `git remote add`.
3. Run `heroku config:add` to set up the Twitter key and secret.
4. Deploy with `git push heroku master`.


## Optimize Your Learning

It's almost a guarantee that you'll be working with OAuth at some point in your career. If you found this challenge confusing, frustrating, or unclear, you should keep practicing. Check out the [Oh My Auth! (Google) Challenge][] for more practice with OAuth, this time using version 2.


## Resources

* [OAuth Documentation (version 1)][OAuth v1]
* [Twitter OAuth Documentation][]
* [OAuth Twitter Callback Configuration Discussion][]
* [Heroku Configuration Variables Documentation][]
* [Tweet Now! 1: Single User Challenge][]
* [Oh My Auth! (Google) Challenge][]

[OAuth v1]:http://oauth.net/core/1.0a/
[Twitter OAuth Documentation]:https://dev.twitter.com/docs/auth/oauth
[OAuth Twitter Callback Configuration Discussion]:https://dev.twitter.com/discussions/5749
[Heroku Configuration Variables Documentation]:https://devcenter.heroku.com/articles/config-vars
[OAuth v1 Conversation Diagram]:https://docs.google.com/drawings/d/1E0SMvb5_vL6aqLD3sngHzC1Kn_K_N_P11ooauSf2FKQ/pub?w=960&h=720
[Tweet Now! 1: Single User Challenge]:../../../tweet-now-1-single-user-challenge
[Oh My Auth! (Google) Challenge]:../../../oh-my-auth-google-challenge
