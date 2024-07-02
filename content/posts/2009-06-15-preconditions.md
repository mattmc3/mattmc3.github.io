+++
title = 'Preconditions'
date = 2009-06-15T23:52:00-04:00
+++

Google's been doing some interesting things lately. Some attract lots of attention, some very little. One that you may not have heard of is [Google Collections](http://google-collections.googlecode.com/). It's just a series of Java helper classes. At first it may seem like a non-event, and mostly for us .NET folks that's true. But I read [this post](http://publicobject.com/2007/09/coding-in-small-with-google-collections_08.html) about a year ago, and my interest was piqued by their [Preconditions](http://google-collections.googlecode.com/svn/trunk/javadoc/com/google/common/base/Preconditions.html) class. It's nothing significant - it just throws various argument exceptions if you don't pass "true" to its methods. But, after writing and using a .NET implementation of it for over a year now, I'm a huge fan of the concept. Formalizing "preconditions" does a few things:

1. Causes me to actually bother to check method parameters (because it's easy) rather than just start using them and risk a failure
1. Takes all the code for checking arguments and makes it very readable. It's really clear that I'm defining a contract for what you can pass to my methods, and I'll throw an exception early if you violate that contract. It's also clear where your eyes need to skip to to get past the house keeping stuff and see the meat of the method.
1. I now notice when I haven't checked params because it's obvious when my methods don't begin with a Preconditions call or two. This is a good thing when _code smells_ are obvious
1. Creates a standard convention for how method argument verification is handled in a project

DbC, or [design-by-contract](http://en.wikipedia.org/wiki/Design_by_contract), is a concept I was first introduced to in a comparative languages course in college. Eiffel is a language that actually requires DbC with it's syntax. With the Preconditions class, you can accomplish the same sort of thing in C#. [Try it out](http://groups.google.com/group/mattmc3/web/Preconditions.zip).

As for a _Postconditions_ class, well... lets just say that I'll do what I say I'll do in a method, and there's no need for me to check that I did it for you. If my method doesn't do what it says, file a bug report :-)
