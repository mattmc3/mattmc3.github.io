+++
title = 'jQuery to the rescue!'
date = 2009-06-29T12:52:00-04:00
categories = 'Tutorial'
tags = ["jquery"]
+++

I had a thought today as I was posting - I like to link out from my blog, but it's really inconvenient for my readers to click on those links and have it open that link over top of the post they're reading. And I don't want to have to go back to my previous posts to update the links manually. Blogger should just handle it for me.

A quick Google search brought me to [these](http://blogging.nitecruzr.net/2009/02/make-links-open-in-new-window.html) [posts](http://blogging.nitecruzr.net/2008/02/make-all-links-open-in-new-window.html), but that isn't quite what I wanted. That would change every single link on the page. There are some links that hop between other pages or actions on my blog, and opening those in a new window would be annoying. So, jQuery to the rescue. I just added the following snippet to my blogger template:

```js
<script src='http://ajax.googleapis.com/ajax/libs/jquery/1.3.2/jquery.min.js' type='text/javascript'/>
<script type='text/javascript'>
$(document).ready(function() {

// Make external links open in a new window
$("a[href^='http:']").not("[href*='blogspot.com']").click(function(){
    this.target = "_blank";
});

});
</script>
```

What this says (in semi-cryptic jQuery-ese) is that when the page has finished rendering (`document.ready()`), then associate my `click()` function with all anchor tags that href to an external website. So, with this simple modification to my blogger template, I can now make blogger handle all the links to external sites in all my posts for all time by opening them in a new window with no additional actions on my part. Ever. Nice...
