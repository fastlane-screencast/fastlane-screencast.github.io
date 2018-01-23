---
layout: episode
episode: "Episode 2"
video_url: https://player.vimeo.com/video/228243145
title:  "Using dotenv and environment variables in fastlane"
date:   2017-08-05 10:00:00 -0600
categories: episodes
tags: android ios
slug: 2-using-dotenv-and-environment-variables-in-fastlane
redirect_from:
  - /using-dotenv-and-environment-variables-in-fastlane/
---

Environment variables are a very powerful way to configure a `fastlane` environment. All `fastlane` setups are run off of a set of configurations which are passed into actions and plugins. These configurations can be things such as bundle identifiers, iTunesConnect credentials, and API secret tokens. Actions and plugins allow the configuration through environment variables. This tutorial is going to walk through how to setup environment variables using [dotenv](https://github.com/bkeepers/dotenv) to make your `fastlane` codebase flexible and maintainable. Anything that is likely to change should be pulled out into environment variables.

We will cover the the following things:
- Default `.env`
- Multiple configurations with `.env.<config>`
- Secret configurations with `.env.secret` and `.gitignore`

## Default Environment
`fastlane` has `dotenv` built right in. When any lane is being executed, `dotenv` will run and will automatically load environment variables from a `.env` file. This `.env` file should be placed in your projects `fastlane` directly.

{% highlight shell %}
# fastlane/.env
STUFF = this is some stuff
{% endhighlight %}

## Multiple Environments
`fastlane` also allows for loading of an additional environment through the `--env <environment>` command line option. `fastlane <lane-name> --env <environment>` will attempt to load a `dotenv` file of `.env.<environment>`.

### config1
`fastlane <lane-name> --env config1`

{% highlight shell %}
# fastlane/.env.config1
CONFIG = this is config 1
{% endhighlight %}

### config2
`fastlane <lane-name> --env config2`

{% highlight shell %}
# fastlane/.env.config2
CONFIG = this is config 2
{% endhighlight %}

## Secrets Environment Variables
It is good practice to commit your `.env` and `.env.<environment>` variables to version control so other developers (or CI) can run your lanes with minimal effor but sometimes there may be private or secret environment variables that you may want to be stored in their own `dotenv` file that are not commited to version control.

This custom approach requires three steps:
1. Create a `.env.secret` file
2. Add `.env.secret` to your `.gitignore` file (if you are using git)
3. Manually load `.env.secret` in the `before_all` block in your `Fastfile`

{% highlight shell %}
# fastlane/.env.secret
SOME_API_TOKEN = 123456789
{% endhighlight %}

{% highlight shell %}
# .gitignore
.env.secret
{% endhighlight %}

{% highlight ruby %}
# fastlane/Fastfile
fastlane_require 'dotenv'

before_all do
  Dotenv.overload '.env.secret'
end
{% endhighlight %}

## Fastfile Implementation

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
