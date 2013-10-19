---
layout: post
category : tech
tagline: ""
tags : [vagrant, openstack, python]
---
{% include JB/setup %}

<link href="/blog/assets/css/syntax.css" rel="stylesheet" type="text/css"/>

## Problem
I use [Vagrant](http://www.vagrantup.com/) for setting up my local development
environments. Recently, I setup an [OpenStack](http://www.openstack.org/)
development environment on a vagrant virtual machine using the [Devstack](http://devstack.org/)
project. I brought up a Ubuntu 12.04 virtual machine using vagrant and checked out the Devstack
code and started stacking. In the initial attempt everything worked fine and I had a working
OpenStack devlopment environment with the up-to-date code from the master branch of all
projects. The source code for all OpenStack projects will be checked out to `/opt/stack`
directory on the virtual machine. You can simply modify the code on the virtual machine and
restart the appropriate services to see the code change getting reflected. Editing the code
inside a virtual machine using a terminal-based editor can be easier for simple code changes.
But for complex code modification and writing new features, it will good to have a Python IDE.
So I decided to use the [synced folder](http://docs.vagrantup.com/v2/synced-folders/) feature
available in vagrant. This feature allows you to share a folder from the host machine to the
virtual machine as an NFS share. So the files from the host machine can be edited directly using
an IDE and the changes can be seen in the virtual machine. Seems pretty simple, right? That's what
I thought and created a synced folder on my host machine will be mapped to the `/opt/stack`
directory on the virtual machine. After a few weeks, I set this configuration on my Vagrantfile
and reloaded my virtual machine. Everything went fine but when it tried to install and configure
horizon, it just stuck forever. I thought it was a network issue and restarted the stack.sh a
few times and tried to leave it running overngiht but no luck. Then I
terminated the VM and started it on a fresh virtual machine. At this point, I didn't know that
the problem was with the synced folder. This issue was going on for a few weeks and I didn't
take the time to dig deep into the issue to find out the root cause of the problem. While casually
trying to setup the Devstack on another laptop with same configuration and without the synced folder
feature, it worked fine.

## Research/Debugging
After I found out that the problem was with the synced folder feature in vagrant, I ssh-ed into the
virtual machine and killed the command that was running on the horizon screen session of stack. The
command was `python manage.py syncdb --noinput`. Once I killed the process, I got the following
stacktrace.

### The Stacktrace

{% highlight bash %}
2013-10-04 04:25:09 Traceback (most recent call last):
2013-10-04 04:25:09   File "manage.py", line 11, in <module>
2013-10-04 04:25:09     execute_from_command_line(sys.argv)
2013-10-04 04:25:09   File "/usr/local/lib/python2.7/dist-packages/django/core/management/__init__.py", line 453, in execute_from_command_line
2013-10-04 04:25:09     utility.execute()
2013-10-04 04:25:09   File "/usr/local/lib/python2.7/dist-packages/django/core/management/__init__.py", line 392, in execute
2013-10-04 04:25:09     self.fetch_command(subcommand).run_from_argv(self.argv)
2013-10-04 04:25:09   File "/usr/local/lib/python2.7/dist-packages/django/core/management/__init__.py", line 263, in fetch_command
2013-10-04 04:25:09     app_name = get_commands()[subcommand]
2013-10-04 04:25:09   File "/usr/local/lib/python2.7/dist-packages/django/core/management/__init__.py", line 109, in get_commands
2013-10-04 04:25:09     apps = settings.INSTALLED_APPS
2013-10-04 04:25:09   File "/usr/local/lib/python2.7/dist-packages/django/conf/__init__.py", line 53, in __getattr__
2013-10-04 04:25:09     self._setup(name)
2013-10-04 04:25:09   File "/usr/local/lib/python2.7/dist-packages/django/conf/__init__.py", line 48, in _setup
2013-10-04 04:25:09     self._wrapped = Settings(settings_module)
2013-10-04 04:25:09   File "/usr/local/lib/python2.7/dist-packages/django/conf/__init__.py", line 132, in __init__
2013-10-04 04:25:09     mod = importlib.import_module(self.SETTINGS_MODULE)
2013-10-04 04:25:09   File "/usr/local/lib/python2.7/dist-packages/django/utils/importlib.py", line 35, in import_module
2013-10-04 04:25:09     __import__(name)
2013-10-04 04:25:09   File "/opt/stack/horizon/openstack_dashboard/settings.py", line 209, in <module>
2013-10-04 04:25:09     from local.local_settings import *  # noqa
2013-10-04 04:25:09   File "/opt/stack/horizon/openstack_dashboard/local/local_settings.py", line 92, in <module>
2013-10-04 04:25:09     SECRET_KEY = secret_key.generate_or_read_from_file(os.path.join(LOCAL_PATH, '.secret_key_store'))
2013-10-04 04:25:09   File "/opt/stack/horizon/horizon/utils/secret_key.py", line 55, in generate_or_read_from_file
2013-10-04 04:25:09     with lock:
2013-10-04 04:25:09   File "/usr/lib/python2.7/dist-packages/lockfile.py", line 223, in __enter__
2013-10-04 04:25:09     self.acquire()
2013-10-04 04:25:09   File "/usr/lib/python2.7/dist-packages/lockfile.py", line 248, in acquire
2013-10-04 04:25:09     os.link(self.unique_name, self.lock_file)
2013-10-04 04:25:09 KeyboardInterrupt
{% endhighlight %}

By looking at the most recent function that was called, it was waiting to create a lockfile. This
lockfile creation process requires making a symlink and the call to make the symlink `os.link`
appeared to be failed and no Timeout was passed into the function (that was bad, the timeout was None).
There was a loop that caught the exception and retried infinetely. Then I tried to execute the same
python command in an interactive python session and got a 'Operation not permitted' error. When I tried
to manually link the file using `ln` command, it seemed to work just fine but for some reason, trying to
do so using python doesn't work.

### The Python Function

{% highlight python %}
def acquire(self, timeout=None):
    try:
        open(self.unique_name, "wb").close()
    except IOError:
        raise LockFailed("failed to create %s" % self.unique_name)

    end_time = time.time()
    if timeout is not None and timeout > 0:
        end_time += timeout

    while True:
        # Try and create a hard link to it.
        try:
            os.link(self.unique_name, self.lock_file)
        except OSError:
            # Link creation failed.  Maybe we've double-locked?
            nlinks = os.stat(self.unique_name).st_nlink
            if nlinks == 2:
                # The original link plus the one I created == 2.  We're
                # good to go.
                return
            else:
                # Otherwise the lock creation failed.
                if timeout is not None and time.time() > end_time:
                    os.unlink(self.unique_name)
                    if timeout > 0:
                        raise LockTimeout
                    else:
                        raise AlreadyLocked
                time.sleep(timeout is not None and timeout/10 or 0.1)
        else:
            # Link creation succeeded.  We're good to go.
            return
{% endhighlight %}

Since this is a permission issue, I tried to investigate the file
ownership and permission information. They were owned by the `vagrant` user who tried to run the stack.
But since this filesystem is a shared filesystem from the host machine using NFS the ownership and
permission on the host machine also comes into account. Since the UID and GID of the vagrant user and
my local username were different, the virtual machien couldn't properly get access to the filesystem.
That was the root cause!

## Solution

The actual solution to fix the permission issue is to match the UID and GID of the vagrant user to the
one on the local uesrname. I didn't want to mess up my system by changing the UID of my machine.
It might cause some unexpected troubles. I tried to change the UID and GID of the vagrant user on the
virtual machine. It did work but I didn't go with the solution since recreating the vagrant environment
will again have the same issue and everytime I recreate the development environment, I'll have to change
the UID and GID of the vagrant user. Also if I change the host machine, the UID and GID will again change.
So I decided not to use the synced folder approach. Instead, I had a copy of the `/opt/stack` directory
on my host machine and wrote a small script using `rsync` which will update the code in the virtual machine
using the one on my host machine. Once I make the code change, I'll have to restart some services to see the
change anyway. So running just another script didn't seem to be a big deal for me.
Here is the script I used.

### Sample Sync Script

{% highlight bash %}
#!/bin/bash

echo "Syncing files.."
/opt/local/bin/rsync \
    -e ssh \
    -avrh \
    --progress \
    --update \
    --exclude 'stack-volumes-backing-file*' \
    /Users/kannanmanickam/Projects/openstack_dev/vm_code/stack \
    vagrant@192.168.27.100:/opt

echo "Sync complete!"
{% endhighlight %}

Note that in the script, I use the `--update` option. This option makes sure that the new files on the
receiver are unaffected. Only the files locally changed will be updated on the receiver (virtual machine).
For example, there will be new log files in the receiver which we don't have in the host machine. These
files will be unaffected. I also exclude the 'stack-volumes-backing-file'. This file is used for backing
the volumes created by cinder. This is usually 5 GB and there is no need for syncing his file. Before
using this script make sure you have your public key available as an authorized key in the virtual machine
so you don't get prompted for password while running this script.
