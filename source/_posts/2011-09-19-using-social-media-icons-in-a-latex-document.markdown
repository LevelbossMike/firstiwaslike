---
layout: post
title: "Using Social Media Icons in a LaTeX Document"
date: 2011-09-19 10:59
comments: true
categories: LaTeX
---

Recently I wrote a letter of application to a local software firm. With
my letter of application I sent a CV and because I was applying for
a position as a Ruby/Rails/Web developer writing the application in 
Word or Pages was not an option and geek cred aside, in my opinion everything of
importance you write should be written in LaTeX just because it looks a lot nicer
 than your everyday WYSIWYG-Editor.

![Latex a beautiful thing.](http://img836.imageshack.us/img836/6889/latexr.jpg "LaTeX, the most beautiful thing")

I ended up using [moderncv](http://www.ctan.org/tex-archive/macros/latex/contrib/moderncv "moderncv") (the classic version) for my cv and scrlttr2 out of [komascript](http://www.ctan.org/tex-archive/macros/latex/contrib/moderncv) for my application
letter.

moderncv comes with a header by default where you can put your contact
information.

![default moderncv header](http://img4.imageshack.us/img4/2234/moderncvheader.png "Classic moderncv header")

I thought it would be nice to also put my GitHub and Twitter
account data in the header. With corresponding icons of course. Surprisingly despite its serious degree of
awesome LaTeX does not support GitHub
or Twitter icons out of the box so I had to find a way to insert them
manually. 

Turns out that's pretty simple. LaTeX supports a way to include images
and even vector graphics in text inline. You just have to specify which
image you want to insert and the respective font-size to make it look
seamless.

Assuming you have a font-size of 12pt in your text and want to
include github's octocat (saved as octocatvector.eps) and a link to your github repository:

{% codeblock github-icon inline lang:latex %}
\includegraphics[height=12pt]{octocatvector.eps}~<link to your github repository> 
{% endcodeblock %}

My result looks like this.

![result moderncv](http://img17.imageshack.us/img17/4136/githubkl.png "moderncv with github and twitter")

That's really everything there is to it. You can do this with any image,
LaTeX will scale it to any size you specify but naturally vector
graphics will be the best option because you can scale them to any size
you want to without loss of quality.

Here's what my document looks like when zoomed in very close to my github-line.

![result scaled](http://img225.imageshack.us/img225/8650/githubgr.png
"github icon scaled")



