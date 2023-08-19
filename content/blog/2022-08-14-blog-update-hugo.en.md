---
title: "Migrating to Hugo"
date: 2022-08-14T14:49:32+02:00
tags: ["blog", "hugo"]
author: Ander Granado
draft: false
---

Hello, aspaldiko! It's been a while since I last posted anything, despite having several ideas in mind and others that I left unfinished (like the series on malware analysis, for example), so it was about time to pick it up again.

First things first, I think it was time to give the blog a little refresh. A while back, I migrated it from Jekyll to Gatsby to modernize it a bit, but I'd been wanting to use something simpler for a while. Not that Gatsby is bad in itself, but it seems to generate too many files for something that should be straightforward, so I looked for alternatives.

I've known about [Hugo](https://gohugo.io/) for a while, which is advertised as the world's fastest website generator, using Go behind the scenes. Honestly, speed isn't my main concern, but if it generates fewer files and is simpler to use, then it's better. Also, diving into Gatsby without knowing much about React felt a bit daunting.

One of the great things about Hugo is its theme system. You can change themes by cloning repos or adding them as submodules, and it's much easier to handle than in Gatsby. Moreover, the "terminal" theme I've chosen looks pretty good, it's minimalist, has that retro touch I like, and still remains modern. If I want to change the look in the future, it's as simple as adding another theme, and that's it. I'm not much of a frontend dev, so being able to maintain a good appearance without having to tinker too much is a big advantage.

On the other hand, another advantage is that you just need to set up your GitHub Action in the repository to build it and push it to the GitHub Pages branch, making it really quick. Just push the changes and forget about it.

Still, all of this is just a facelift. The most important thing about a blog is always its content, so this is just a tune-up to get back into blogging, which was long overdue.

In the coming days, I'll be uploading a few things to get back into "regular" posting.

See you soon!

Happy hacking!