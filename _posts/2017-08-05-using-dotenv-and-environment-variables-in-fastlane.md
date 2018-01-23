---
layout: episode
episode: "Episode 2"
video_url: https://player.vimeo.com/video/228243145
title:  "Using dotenv and environment variables in fastlane"
date:   2017-08-05 10:00:00 -0600
categories: episodes
tags: android ios
---

Environment variables are a very powerful way to configure a `fastlane` environment. All `fastlane` setups are run off of a set of configurations which are passed into actions and plugins. These configurations can be things such as bundle identifiers, iTunesConnect credentials, and API secret tokens.

Configurations are passed into actions and plugins through code in the `Fastfile`, through configuration files like `Gymfile` or `Deliverfile`, or through environment variables. This tutorial is going to walk through how to setup environment variables using [dotenv](https://github.com/bkeepers/dotenv) to make your `fastlane` codebase flexible and maintainable.

We will cover the the following things:
- Default `.env`
- Multiple confirations with `.env.<config>`
- Secret configurations with `.env.secret` and `.gitignore`

{% highlight shell %}
# .env
STUFF = this is some stuff
{% endhighlight %}

{% highlight shell %}
# .env.config1
CONFIG = this is config 1
{% endhighlight %}

{% highlight shell %}
# .env.config2
CONFIG = this is config 2
{% endhighlight %}

{% highlight shell %}
# .env.secret
SOME_API_TOKEN = 123456789
{% endhighlight %}

{% highlight shell %}
# .gitignore
.env.secret
{% endhighlight %}

{% highlight ruby %}
# Fastfile
fastlane_require 'dotenv'

before_all do
  Dotenv.overload '.env.secret'
end
{% endhighlight %}

{% highlight ruby %}
# Fastfile
fastlane_require 'dotenv'

before_all do
  Dotenv.overload '.env.secret'
end

lane :test do
  # Loaded from .env (by fastlane)
  puts "STUFF: #{ENV['STUFF']}"                  
  
  # Loaded from .env.secret (by us in before_all)
  puts "SOME_API_TOKEN: #{ENV['SOME_API_TOKEN']}"
  
  # Loaded from .env.(<env>) where <env> is passed in from `fastlane test --env <env>` (by fastlane)
  # Ex: `fastlane test --env config` loads .env.config1
  puts "CONFIG: #{ENV['CONFIG']}"
end
{% endhighlight %}

{% highlight shell %}
$: fastlane test
  STUFF: this is some stuff
  SOME_API_TOKEN: 123456789
  CONFIG:
  
$: fastlane test --env config1
  STUFF: this is some stuff
  SOME_API_TOKEN: 123456789
  CONFIG: this is config 1
  
$: fastlane test --env config2
  STUFF: this is some stuff
  SOME_API_TOKEN: 123456789
  CONFIG: this is config 2
{% endhighlight %}
