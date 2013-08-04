---
layout: post
category : tech
tagline: "Simple syntax checker"
tags : [ruby, json]
---
{% include JB/setup %}

<link href="/blog/assets/css/syntax.css" rel="stylesheet" type="text/css"/>

## Overview

It is easy to make mistakes in a JSON file by simply missing a comma or other
punctuation mark. A simple command line tool to check the syntax will be
helpful to make sure the syntax is correct before even starting to use the
file.

## The tool

This simple tool is written in ruby and uses the "json" rubygem. Any version
should be fine. Make sure the gem is installed before attempting to use this
tool.

{% highlight bash %}
gem install json
{% endhighlight %}

Once the gem is installed, place the contents  of the following block into a
file (example: /usr/local/bin/jsonlint). Once the file is saved, make the file
executable.

{% highlight bash %}
chmod +x /usr/local/bin/jsonlint
{% endhighlight %}

_Note:_ If you do not have permission to edit/modify files in this location,
execute the commands using `sudo`.

If you prefer saving the file in a different location, make sure the file is in
your PATH or add the location to the PATH.

{% highlight ruby %}
#!/usr/bin/env ruby

require "rubygems"
require "json"

if ARGV.size < 1
  puts "Please provide a JSON file"
  exit 1
end

file_name = ARGV.shift

begin
  JSON.load(File.read(file_name))
  puts "Syntax OK"
  exit 0
rescue JSON::ParserError
  puts "Syntax Error!"
  exit 1
end
{% endhighlight %}

## Tool in action

Here is a simple valid JSON file:

{% highlight ruby %}
{
    "example": {
        "title": "This is an example"
    }
}
{% endhighlight %}

Running the lint tool against this file gives:

{% highlight bash %}
Kannans-MacBook-Pro:~ kannanmanickam$ jsonlint test.json
Syntax OK
{% endhighlight %}

Here is a JSON file that misses a "}"

{% highlight ruby %}
{
    "example": {
        "title": "This is an example"
}
{% endhighlight %}

Running the list tool against this file gives:

{% highlight bash %}
Kannans-MacBook-Pro:~ kannanmanickam$ jsonlint test.json
Syntax Error!
{% endhighlight %}
