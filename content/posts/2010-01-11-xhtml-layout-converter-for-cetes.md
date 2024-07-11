+++
title = 'XHTML layout converter for ceTe’s Dynamic PDF library'
date = 2010-01-11T22:48:00-05:00
categories = 'Tools'
tags = ['XML']
+++

At work, we are using [ceTe’s PDF libraries for .NET](http://www.cete.com/). They have a
pretty easy to use, versatile set of tools for building PDFs. However, after building a
few of them, it became painfully obvious that assembling all but the most anemic PDF
layouts was painful. The Tweak-Compile-Generate-View-Repeat method takes quite some
time. Too much time.

The Tweak-Refresh-Repeat model for HTML+CSS seemed to me to be a **_much_** better
method of building a layout, but the Dynamic PDF API doesn’t support anything like that.
It speaks a little bit of pseudo-HTML3 with `<font>`, `<b>`, `<u>`, `<i>`, and `<p>`
tags. But that is about the extent of it. So, I decided to build a converter that takes
full advantage of our existing tools for XHTML generation and converts those XHTML+CSS
documents into a PDF. It turned out to be surprisingly easy. My converter is less than
600 lines of nicely commented VB, with a few little utility functions for things like
parsing CSS. I was even able to account for the “cascading” part of CSS, where I look to
page level styles, then tag level styles, followed by classes, tag IDs, and finally the
style attribute.

Currently what I have will support tables (including colspans and rowspans), images,
rotated items (using the IE only `{ writing-mode:tb-rl; filter: flipv; }` syntax),
nested <div>s, various positioning techniques, and text alignment and styling. I’m using
NVelocity to dynamically generate the XHTML (though T4 or really anything would do), so
all the dynamic parts of the PDF are done on the front end rather than intermingled with
the proprietary ceTe API calls. Since XHTML is both renderable HTML and parsable XML, I
didn’t need to put in any effort there. The Linq-to-XML functionality in .NET made that
part a snap.

One of the decisions I had to make is what specific subset of XHTML would I support.
Obviously, a PDF is much different than a web page in the way you layout the elements.
One example would be that a PDF isn’t designed to have a flow layout. In other words,
when you resize the PDF, everything stays put rather than wrapping around the way a
properly designed web page would. So, my XHTML assumes you will be absolutely
positioning everything, and the converter throws errors if you don’t. You simply have to
specify the X and Y coordinates, as well as specifying the height and width too.
Actually, that's not entirely true - as a convenience I do auto-calculate height/width
in certain circumstances but mostly you ought to specify. Height and Width are defined
in point units, or “Pt”s, which is most commonly thought of as a unit for fonts, but in
fact is equivalent to the units ceTe uses in its box layout API. This means that your
HTML output will line up size-wise with the PDF that gets generated. Again, I wasn’t
looking to take just any old HTML and make a PDF, but if you tailor your XHTML output to
the PDF layout concept, it’s not too constraining.

There are quite a few things I don’t support. I didn’t do anything too fancy like CSS2
selectors. Another consideration was that the XHTML had to renderable exactly like the
PDF version for this to make any sense, but since you can do lots more in XHTML than I
actually support, you could technically go off the deep end. In addition, I chose to
support only IE as my primary renederer. Firefox will _work_, and looks fairly close,
but my purpose wasn’t really to make a standards compliant HTML output. The idea is to
have a RAD tool for PDF generation, and IE-only was okay for me to meet that goal. Also,
there’s no support for any JavaScript (and really no need for it in a PDF anyway). CSS
that isn’t understood is just ignored, as well as tags or attributes that aren’t
understood. The key parts are that everything has to have an x, y, height,and width
specified in the proper units.

There are some things I still want to add as well. I’d like to add support for the
`<span style="font-family: 'Courier New',Courier,monospace;">!important</span>` CSS
directive. Also, currently I don’t support changing fonts. CeTe’s tool seems to require
quite a bit of config to set up a new font. You have to specify the `<span
style="font-family: 'Courier New',Courier,monospace;">.ttf</span>` file for the bold
version, and the italic version, and the bold/italic version, and then there’s the
underlining iterations – you get the idea. It’s on my TODO list to figure that one out.
I also don’t have support for multiple pages. Currently I’m only set up to do a 1-to-1
of XHTML doc to PDF page. And finally, I need to figure out how to do relative
positioning of items (offset this element from that one by X,Y) as well as doing dynamic
sizing of items to support an undetermined amount of text. It might also be nice to
specify an analyzer that lets you know whether your XHTML input has unsupported tags/css
for debugging purposes.

All and all, this was a _really_ fun side project. There was a lot of up-front work, but
it will pay off in no time. Now we can crank out new PDFs in hours instead of days, and
not have to be in the guts of ceTe’s API. I’d like to highlight some of the more
interesting bits of code, but since I did this for work, I’m not sure what I can/should
showcase. Perhaps I’ll rewrite it in C# – though I’m not inclined to buy the Dynamic
PDF tools for my personal use.
