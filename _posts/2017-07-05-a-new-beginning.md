---
layout: post
title: A new beginning
date: 2017-7-5
comments: true
categories: misc
---

As a modern living being it is quite often to be distracted by too many things.
The one downside of this is that I procrastinate a lot.  The other is amnesia,
as I tend to forget things I did and ideas I had.

For this reason, I have always wanted to start a blog, documenting everything
that have crossed my mind.  I also aim to do this frequently, which could be
useful to organize what I have done already and to plan for the next step, and
allow me to focus on work, avoiding the distractions.  Plus, I could perhaps
gain a little audience of my blog in the process.  <!--more-->

# Jekyll

To start this blog I spent some time learning about [Jekyll][jekyll], and how
to publish a Jekyll blog on [GitHub][github].  Initially I started with the
default Cayman theme from GitHub but soon found out I couldn't write blog
posts with it, so discovered a [Cayman blog theme from lorepirri][theme],
which enables blog posts on GitHub pages.  I tweaked a little bit of this
theme to suit my needs better.  Moreover, I enabled [MathJax][mathjax] by
inserting a `<script>` into the `default.html` file (as shown below).

{% highlight html %}
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
{% endhighlight %}

Math constructs in [LaTeX][latex], for example, `$$E = mc^2$$`, can be rendered
directly into a nicely formated version, $$E = mc^2$$.  This will come in handy
as we introduce formulae and equations in subsequent posts.

In the process of setting up this blog, I learned how to add a comment section
to posts using [Disqus][disqus], and also display a comment count on the home
page for individual posts.  This process is surprisingly painless, simply
requiring me to register on Disqus, and adding an embedded comment section is
as simple as pasting this snippet (provided by Disqus) to the template HTML
file:
{% highlight html %}{% raw %}
{% if site.disqus_shortname %}
{% if page.comments %}
<hr/>
<div id="disqus_thread"></div>
<script>
var disqus_config = function () {
    this.page.url = "{{ page.url | prepend: site.github.url }}";
    this.page.identifier = "{{ page.id }}";
};
(function() {
    var d = document, s = d.createElement('script');
    s.src = '//{{ site.disqus_shortname }}.disqus.com/embed.js';
    s.setAttribute('data-timestamp', +new Date());
    (d.head || d.body).appendChild(s);
})();
</script>
<noscript>
    Please enable JavaScript to view the
    <a href="https://disqus.com/?ref_noscript" rel="nofollow">
        comments powered by Disqus.
    </a>
</noscript>
{% endif %}
{% endif %}
{% endraw %}{% endhighlight %}
then I added comment counts to individual posts with:
{% highlight html %}{% raw %}
<a data-disqus-identifier={{ post.id }} href="{{ post.url | relative_url }}#disqus_thread"></a>
<script id="dsq-count-scr" src="//admk.disqus.com/count.js" async></script>
{% endraw %}{% endhighlight %}
and that's it!  So you can expect fast responses from me by commenting
:laughing:.

# Things to write about

I plan to write a few posts about things I did in my PhD, which focuses on
numerical programs, and how we automatically optimize them to run as accurate,
fast and resource-efficient as possible on [FPGAs][fpga].  I expect this to
be quite a dense introduction to the underlying techniques, so this will be a
3-part series at least, in which I will respectively go through the rationale
for doing so, how programs can be analyzed for accuracy, latency and area when
synthesized into circuits, and finally how we can use the information to guide
the optimization process.

I also envision new stuffs to be posted here as work progresses, which I do not
have specifics to share for the moment, except the keywords [FPGA][fpga] and
[deep learning][dl].  I have a lot to learn and a lot of work to be done, I
certainly have high expectations for future updates!

Although this blog will be mostly work related, please anticipate for other
non-technical things (or technical things that are highly uncorrelated with
work), as I master the art of procrastination.  I guess that's it for now, and
I can't wait to update on the things promised to I write about in this post in
the following weeks to come.


[jekyll]: https://jekyllrb.com
[github]: https://www.github.com
[latex]: https://www.latex-project.org
[theme]: https://github.com/lorepirri/cayman-blog
[mathjax]: https://www.mathjax.org
[disqus]: https://www.disqus.com
[fpga]: https://www.xilinx.com/training/fpga/fpga-field-programmable-gate-array.htm
[dl]: http://www.deeplearningbook.org
