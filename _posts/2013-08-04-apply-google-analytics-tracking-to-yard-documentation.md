---
layout: post
category : tech
tagline: "Inject the script to html files"
tags : [ruby]
---
{% include JB/setup %}

<link href="/blog/assets/css/syntax.css" rel="stylesheet" type="text/css"/>

## Overview
The yard generated documentation does not provide a way to specify the Google
Analytics tracking ID in the configuration. If you want to have the tracking ID
in the generated documentation, you have to go through all the files and inject
the script. Also the files change every time the documentation is generated. I
use the yard-generated documentation as the website for my
[jenkins_api_client](http://github.com/arangamani/jenkins_api_client) project
and want to use my Google Analytics tracking ID to see the usage of the
project. So I decided to create a rake task that will do this for me

## The Rake Task
This simple rake task will go through all html files in the subdirectories of
current directory and inject the google analytics tracking script. Include this
rake task in your `Rakefile` and replace `UA-XXXXXXXX-X` with the google
analytics tracking ID of your website.

{% highlight ruby %}
task :apply_google_analytics do
  # The string to replace in the html document. This is chosen to be the end
  # body </body> tag. So the script can be injected as the last thing in the
  # document body.
  string_to_replace = "</body>"
  # This is the string to replace with. It include the google analytics script
  # as well as the end </body> tag.
  string_to_replace_with = <<-EOF
    <script type="text/javascript">
      var _gaq = _gaq || [];
      _gaq.push(['_setAccount', 'UA-XXXXXXXX-X']);
      _gaq.push(['_trackPageview']);

      (function() {
        var ga = document.createElement('script'); ga.type = 'text/javascript'; ga.async = true;
        ga.src = ('https:' == document.location.protocol ? 'https://ssl' : 'http://www') + '.google-analytics.com/ga.js';
        var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(ga, s);
      })();
    </script>
  </body>
  EOF

  files = Dir.glob("**/*.html")

  files.each do |html_file|
    puts "Processing file: #{html_file}"
    contents = ""
    # Read the file contents
    file =  File.open(html_file)
    file.each { |line| contents << line }
    file.close

    # If the file already has google analytics tracking info, skip it.
    if contents.include?(string_to_replace_with)
      puts "Skipped..."
      next
    end

    # Apply google analytics tracking info to the html file
    contents.gsub!(string_to_replace, string_to_replace_with)

    # Write the contents with the google analytics info to the file
    file =  File.open(html_file, "w")
    file.write(contents)
    file.close
  end
end
{% endhighlight %}

## The Rake task in action

{% highlight bash %}
bundle exec rake apply_google_analytics
{% endhighlight %}

The rake task will go through all
{% highlight bash %}
Processing file: doc/_index.html
Processing file: doc/class_list.html
Processing file: doc/file.README.html
Processing file: doc/file_list.html
Processing file: doc/frames.html
Processing file: doc/index.html
Processing file: doc/JenkinsApi/CLI/Base.html
Processing file: doc/JenkinsApi/CLI/Helper.html
Processing file: doc/JenkinsApi/CLI/Job.html
Processing file: doc/JenkinsApi/CLI/Node.html
Processing file: doc/JenkinsApi/CLI/System.html
Processing file: doc/JenkinsApi/CLI.html
Processing file: doc/JenkinsApi/Client/BuildQueue.html
Processing file: doc/JenkinsApi/Client/Job.html
Processing file: doc/JenkinsApi/Client/Node.html
Processing file: doc/JenkinsApi/Client/System.html
Processing file: doc/JenkinsApi/Client/User.html
Processing file: doc/JenkinsApi/Client/View.html
Processing file: doc/JenkinsApi/Client.html
Processing file: doc/JenkinsApi/Exceptions/ApiException.html
Processing file: doc/JenkinsApi/Exceptions/CLIError.html
Processing file: doc/JenkinsApi/Exceptions/CrumbNotFound.html
Processing file: doc/JenkinsApi/Exceptions/Forbidden.html
Processing file: doc/JenkinsApi/Exceptions/ForbiddenWithCrumb.html
Processing file: doc/JenkinsApi/Exceptions/InternalServerError.html
Processing file: doc/JenkinsApi/Exceptions/JobAlreadyExists.html
Processing file: doc/JenkinsApi/Exceptions/JobNotFound.html
Processing file: doc/JenkinsApi/Exceptions/NodeAlreadyExists.html
Processing file: doc/JenkinsApi/Exceptions/NodeNotFound.html
Processing file: doc/JenkinsApi/Exceptions/NotFound.html
Processing file: doc/JenkinsApi/Exceptions/NothingSubmitted.html
Processing file: doc/JenkinsApi/Exceptions/ServiceUnavailable.html
Processing file: doc/JenkinsApi/Exceptions/Unauthorized.html
Processing file: doc/JenkinsApi/Exceptions/ViewAlreadyExists.html
Processing file: doc/JenkinsApi/Exceptions/ViewNotFound.html
Processing file: doc/JenkinsApi/Exceptions.html
Processing file: doc/JenkinsApi.html
Processing file: doc/method_list.html
Processing file: doc/top-level-namespace.html
{% endhighlight %}

Since this is newly generated documentation, the file has to go through all
files inject the google analytics script in all of them. If the file already
has the analytics script, that file will be skipped. A generated page using
this rake task can be found in the
[jenkins_api_client](http://github.com/arangamani/jenkins_api_client) project
documentation [here](http://www.arangamani.net/jenkins_api_client).
