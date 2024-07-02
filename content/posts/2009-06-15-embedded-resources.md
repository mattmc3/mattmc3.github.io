+++
title = 'Embedded Resources'
date = 2009-06-15T10:00:00-04:00
draft = true
+++

![Embedded Resource](/assets/img/EmbeddedResource.gif)

In the .NET world, ORMs are only just now beginning to see the light of day. Though nHibernate has been around for awhile, it has not seen much uptake from mainstream .NET devs. It may have to do with typical FUD stuff, like its roots in the Java world, its reliance on XML for the mappings, its learning curve, or the lack of books and beginner's resources. I'm not judging - I've read [this book](http://www.amazon.com/NHibernate-Action-Pierre-Henri-Kuat%C3%A9/dp/1932394923/ref=sr_1_1?ie=UTF8&amp;s=books&amp;qid=1245024633&amp;sr=8-1) and I think nHibernate's offerings are **the most mature and robust available for .NET devs**, but I've also been looking at Microsoft's offerings via [Linq-to-SQL](http://www.amazon.com/LINQ-Action-Fabrice-Marguerie/dp/1933988169/ref=sr_1_1?ie=UTF8&amp;s=books&amp;qid=1245024741&amp;sr=1-1) and [Entity Framework](http://www.amazon.com/Programming-Entity-Framework-Julia-Lerman/dp/059652028X/ref=sr_1_1?ie=UTF8&amp;s=books&amp;qid=1245024719&amp;sr=1-1) and think that MS is probably going to keep fighting until EF comes out on top. Until this all shakes out, I believe that data access will continue to be in a giant state of flux.

For good or bad, that's the state of things. The ORM space has not matured in the broader .NET world, and ADO.NET is still the most used method of data access for now. That means plain old SQL is still _really_ useful. Actually, plain old SQL will **always** be useful while RDBMs are the anointed choice for data storage.

There are two main sources of trouble though. First, there is the very real issue of how to map tabular data to objects and back again. This is called the [Object Relational Impedance Mismatch](http://en.wikipedia.org/wiki/Object-relational_impedance_mismatch), and is not the subject of today's post. The other issue is one of query management, and it is today's topic. How do you manage your queries? Most people are still using vanilla SQL, though Linq + _some-ORM-tool_ is a very compelling alternative.

For the raw SQL crowd, I think many would agree that it is a **pain** to manage your queries, and no one does it the same way. Do you choose stored procs? How about fancy custom SQL builder classes? Do you use `System.Text.StringBuilder`? Or worse, just regular string concatenation? Pick your poison. Mine is [embedded resources](http://blog.topholt.com/2008/03/18/c-trick-load-embedded-resources-in-a-class-library/"). I keep nearly all my T-SQL in .sql files in my .NET project.

Consider:

- SQL files in your project get syntax highlighting, commented, and formatted. Those things don't happen in string based SQL building

- Stored procedures are detached from the code that uses them. That makes it difficult to know what effects changes in procs will have. Embedded resources allow you to keep SQL near the classes that use it.

- Try doing a simple Find-and-Replace in your stored procs. Not fun. It's a snap with embedded resources in a .NET project

- Stored procs are difficult to version control. I do it via Visual Studio database projects, but not many other devs have every used them. And, MS hasn't done much to develop that project type since VS 2003. AND it's really easy to use SSMS to subvert the process and change procs outside the project. Putting .sql files in your .NET project means that version control is now a no brainer.

- If you have to rollback a release, your SQL being compiled into your project means that a rollback only requires reversing table/view/udf modifications which are typically less extensive than your SQL changes.

- With SQL Server, your .SQL embedded resource files can utilize full T-SQL capabilities. Variables, temp tables, etc. Anything you can put in a stored proc, you can put in your .sql file. The one huge advantage though is that your .sql file _doesn't have to be parsable SQL_. It's still a string when it comes out of the embedded resource, so "`select * from {0}`" can be put through a `String.Format()` transformation prior to execution. It's the best of the stored proc world and the `StringBuilder` world with none of the disadvantages. _PS - don't ever use "`select *`" outside of some pithy example code, and be really careful about SQL injection when using find-and-replace methods._

The one and only drawback I've found is that stored procs are dynamically updateable so you can change a proc on-the-fly, but with embedded resources you have to re-compile. Of course, sql modifications are one of only a handful of change types that can happen dynamically, and just because it can be done that way doesn't mean it should. But, if you need it, you could accomplish this dynamically via some fancy coding too. Just have your embedded resource loader check a directory for a .sql file first.

I'm a big advocate of using the right tool for the job. If ADO.NET and SQL are the right tools for your application, strongly consider putting your SQL in embedded resources. When a `StringBuilder` or a stored proc are appropriate, of course use them instead. But, I've found that those instances to be the exceptions, not the rule.

It may also be easier to get started with a little [helper class](http://groups.google.com/group/mattmc3/web/EmbeddedResourceHelper.zip) for your embedded resources. Hopefully at some point soon the merits of ORM tools will shake out a little more and become more widespread, but until then this may serve you on some real life project _today`. Happy coding!
