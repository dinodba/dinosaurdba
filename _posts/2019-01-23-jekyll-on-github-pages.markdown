---
layout: post
title:  "Jekyll on GitHub Pages"
date:   2019-01-23 13:38:03 +0000
permalink: /jekyll-on-github-pages
---

[Skip straight to the steps](#steps)

So the point of this whole blog is to catalogue my learning new technologies for this new cloudy future I find myself in and hopeful help others along the way. It makes sense to me to start with how I got this blog up an running.

I've known I needed to learn more about the technologies the developer community are using for a few years. I've played with, researched, and kind-of actively use some of them - docker, puppet, git, GCP - but for the ones I've played with, I haven't consolidated the knowledge I gained. And the ones I'm using I've learned enough to get me through the stuff I need them to do but don't really understand them and have barely scratched the surface of them.

With a renewed desire to properly learn these new technologies I figured writing blog posts would serve a few purposes:
* Force me to learn things well enough to explain them to others
* Provide a record of the steps taken to allow me to quickly repeat them if and when I need them in the wild
* Perhaps provide help to other dinosaurs like myself

I know myself though. I'm lazy. This ~~probably~~ possibly won't last. Now, I could make myself more accountable by paying for a blogging service but I'm too tight for that. I needed a free blogging platform. I've kept a blog before. It's a personal one about my kid. I wrote it for family that live too far away to see us regularly. I kept it updated for almost a decade. It was hosted on blogger. I knew I didn't want to use blogger for this. I did a bit of googling and eventually decided on wordpress.com. It's free. It seemed popular and easy. It was in every site's top 5. I'm planning to learn about hosting and stuff and I didn't want putting together a working blog to take up the first few months of my journey.

But then I talked to a colleague. He'd just started using Jekyll on GitHub pages. He knows about this stuff so I could trust his opinion. But he knows about this stuff - so he didn't have so much to learn! He ported his blog from a self-hosted ghost platform. He explained Jekyll was clunky at best. But the argument that sold me was once I got into wordpress it would be hard to port the blog elsewhere if I wanted to move. I always expected to move - once I'd learned more about hosting. I thought as long as I had the content written it'd just end up being a bit of copy and paste. But maybe it's not going to be that easy.

So I had a look at Jekyll and GitHub pages. Jekyll doesn't use a database. As a database professional we weren't off to a good start :). But it's just flat files. That I can write in markdown (that's one of the things I want to learn) and upload to github (that's another thing I want to learn), and host on GitHub Pages (that's free). So why not give it a go? And here we are. And here are the steps.

### <a name="steps"></a>The Steps

I don't believe in repeating other people's work if it's useful so I'll link to the resources I've used and provide updates where necessary.

So firstly, [Mike Dane](https://www.youtube.com/playlist?list=PLLAZ4kZ9dFpOPV5C5Ay0pHaa0RJFhcmcB)'s Jekyll's playlist got me mostly up and running. He described Jekyll really well and his instructions for getting it up on GitHub pages were great - much easier and a bit different from what I'd expected when reading the GitHub docs. He also helped me understand git branching a little better.

Things not covered in his videos...

His Mac instructions didn't work for me. I did as he explained but Jekyll didn't install. After a bit of googling I discovered that's really not the best way to make changes to your Ruby enviornment - those steps will be affecting the OS's Ruby rather than the user's. I found some steps on stackoverflow that got me farther but still didn't work. Eventually I found the [Jekyll docs](https://jekyllrb.com/docs/installation/macos/) themselves have actually been updated with the correct steps.
Once I exported my new path (also in the docs) it worked correctly.

Mike's final video is the one for GitHub pages. He mentions that you need to update baseurl in the _config.yml file. This works fine but it breaks local testing. I've been adding it for versions I'm pushing to git and removing it again for local editing.

I don't know if this is really Jekyll or just HTML but I had so much trouble getting anchor links within posts to work (the link at the top of this page to The Steps section). The format that worked for me is to put the anchor first:

```### <a name="steps"></a>The Steps```

And then it's called as follows:

```[Skip straight to the steps](#steps)```

