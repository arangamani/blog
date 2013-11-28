---
layout: post
category: tech
title: Syntax Highlighting in Vim for Gemfile, Berksfile, and Vagrantfile
tagline: ""
tags : [vim, ruby, chef]
---
{% include JB/setup %}

<link href="/blog/assets/css/syntax.css" rel="stylesheet" type="text/css"/>

## The Problem

It all started when I was editing a Vagrantfile and making more customization. It is simply ruby
and having syntax highlighting in it will be easier to edit the file. I always see the following line in
the Vagrantfile:

{% highlight ruby %}
# vi: set ft=ruby :
{% endhighlight %}

but it didn't seem to do anything for the syntax highlighting. I tried to set the filetype for the
Vagrantfile when I was in the editor using

{% highlight ruby %}
:set filetype=ruby
{% endhighlight %}

it worked like a charm. But I don't want to do it every time I enter the file. Also I have the same problem
with the Gemfile as well as the Berksfile.

## The Solution

I did some research and figured out that I can achieve this using the `autocmd` in vimrc. So added the
following lines to my [vimrc](https://github.com/arangamani/vimrc).

{% highlight vim %}
autocmd BufNewFile,BufRead Gemfile set filetype=ruby
autocmd BufNewFile,BufRead Vagrantfile set filetype=ruby
autocmd BufNewFile,BufRead Berksfile set filetype=ruby
{% endhighlight %}

So whenever I open any of these files, they are highlighted as ruby files and I am happy!
