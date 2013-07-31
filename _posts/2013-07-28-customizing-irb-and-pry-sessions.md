---
layout: post
category : tech
tagline: "Customize it the way you want"
tags : [pry, irb]
---
{% include JB/setup %}

<link href="/blog/assets/css/syntax.css" rel="stylesheet" type="text/css"/>

## The .irbrc file
The `.irbrc` file is usually placed in your home directory (`~/.irbc`). This
file is just a regular ruby file and you can write any arbitrary ruby code that
you want to be executed before the IRB session is loaded.

## The .pryrc file
The `.pryrc` file is very similar to the `.irbrc` file and is placed in your
home directory (`~/.pryrc`) and this file is loaded before the Pry session is
started.

## Customizing the session
If you use a particular gem every time you are in an IRB or a Pry session, you
can simply include that in your `.irbrc` or `.pryrc` file so you do not have to
manually require that gem every time you enter the session. An example for this
would be the pretty print (pp) library. Just add the require statement in the
rc file and that will load this library whenever you enter the session.

{% highlight ruby %}
require "pp"
{% endhighlight %}

_Note:_ The following customizations are IRB specific and Pry has most of them
in-built.

If you use IRB instead of Pry, you will have to install some additional gems to
get the coloring enabled in the output. Pry has this in-built so you do not
have to do any additional work. To enable coloring in IRB, the `wirble` gem
should be installed. The following simple configuration will enable coloring
using the `wirble` gem.

{% highlight ruby %}
require "wirble"

# Initialize Wirble
Wirble.init

# Enable colored output
Wirble.colorize
{% endhighlight %}

Tab completion in IRB can be enabled by
{% highlight ruby %}
require "irb/completion"
{% endhighlight %}

Automatic indentation can be set by
{% highlight ruby %}
IRB.conf[:AUTO_INDENT] = true
{% endhighlight %}

If you would like to save the history between IRB sessions, it can be achieved
by
{% highlight ruby %}
require "irb/ext/save-hostory"
# Number of lines/commands to save

IRB.conf[:SAVE_HISTORY] = 100
# The location to save the history file
IRB.conf[:HISTORY_FILE] = "#{ENV['HOME']}/.irb-save-history"
{% endhighlight %}

## Per-project customization

The `.[irb|pry]rc` file can be customized to automatically load the environment
for your project based on your current working directory.
Whenever I work with Ruby projects, I keep a separate window open for my pry
session. I do not want to load and initialize the project every time I enter te
session. I usually have the credentials saved separately in a YAML file. So in
the `.pryrc` file, I get the current working directory and based on the
directory, I load and initialize my project. The object I initialize in the rc
file will be available inside the irb/pry session, which I can then use for my
interactions.

{% highlight ruby %}
# Obtain the present woking directory
current_dir = Dir.pwd
# Customizations required for my jenkins_api_client project. The block of
# code inside this if block will load the gem project and then initialize
# the client.
if current_dir =~ /^(.*?\/jenkin_api_client)/
  # YAML is required to parse the credentials from the login.yml file
  require"yaml"
  # Obtain the library path of your ruby project
  path = "#{$1}/lib"
  # Insert this path to the front the $LOAD_PATH
  $LOAD_PATH.unshift(path)

  # Load the project as a rubygem
  require "jenkins_api_client"
  # Initialize the client by loading credentials from the yaml file
  @client = JenkinsApi::Client.new(
      YAML.load_file(File.expand_path("~/.jenkins_api_client/login.yml"))
  )
  puts "logged-in to the Jenkins API, use the '@client' variable to use it"
# Custmoizations required for my jenkins_launcher project
elsif current_dir =~ /^(.*?\/jenkins_launcher)/
    ...
elsif current_dir =~ /^(.*?\/some_other_project)/
    ...
end
{% endhighlight %}

Now that I have my rc file setup to prepare my project when I am in my
[jenkins_api_client](http://www.arangamani.net/jenkins_api_client) project
directory, when I enter the Pry session, my project is loaded and ready for me
to play with. If I change my directory to a different project and enter irb/pry
session again, that project will be loaded instead.

{% highlight bash %}
Kannans-MacBook-Pro:jenkins_api_client kannanmanickam$ pry
logged-in to the Jenkins API, use the '@client' variable to use the client
Welcome back to the IRB session, Kannan!
[1] pry(main)>
{% endhighlight %}

It is as simple as adding another condition to match your project and name do
the initializations to load your project. The above method will work even if
you are deep inside your project directory structure as the regex we use to
match th current directory uses non-greedy approach.
