+++
title = 'Source Control Managment'
date = 2009-09-25T21:49:00-04:00
categories = 'Tools'
tags = ['SCM']
+++

I spent most of my day at work today setting up a new development server. SQL Server
2008, IIS, msbuild server and finally, Subversion. Our dev box died a few months ago and
we had cobbled together some things on a bunch of different existing boxes and basically
hobbled along.

So today I got a chance to consolidate everything onto a nice new box. And originally I
wasn't planning on moving our source control onto it. But our SVN is on an old Linux box
and hadn't been updated since 2006. It was humming along nicely the way Linux seems to
do, but that particular brand of aging box has been dying on us quite regularly, so each
day we leave it untouched we take a risk of an inconvenient server failure. And, since
our lone sysadmin has little experience with Linux, the risk seemed even greater. So,
off I went to see if I could get a Windows SVN server up and running.

I installed the Windows version of Subversion from
[here](http://subversion.tigris.org/), but then wound up having to figure out how to set
it up as a Windows service and get it configured. It proved to be [more
complicated](http://blogs.vertigosoftware.com/teamsystem/archive/2006/01/16/Setting_up_a_Subversion_Server_under_Windows.aspx)
than I expected. And I was left with a lot of questions:

- How do I make sure that SVN is running as a Windows service?
- How do I serve out my repositories via HTTPS? With SSL?
- How do I use NT security vs having to manage separate SVN user accounts/passwords?

A quick Google search turned up [VisualSVN Server](http://www.visualsvn.com/server/).
I've used their other product - VisualSVN - but didn't find it worth the cost and I
wound up a happy [Ankh](https://github.com/AmpScm/AnkhSVN) user. I hadn't been back to
their site since, but I wish I had because VisualSVN Server is free and it just works. I
had it up and running and the first 16 of my repositories dumped from SVN 1.3.1 on Linux
and imported to SVN 1.6.5 on Windows all within an hour. Amazingly, I knew the least
about this at the onset of my day, but it took me the least amount of time of any of the
things I had to set up on the dev box today. Highly recommended! 5 full stars!
