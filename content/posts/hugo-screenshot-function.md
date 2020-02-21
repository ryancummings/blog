+++
title = "my-hugo-screenshot: a simple custom Emacs function"
author = ["Ryan Cummings"]
date = 2020-02-21T09:40:00-05:00
lastmod = 2020-02-21T15:55:36-05:00
tags = ["emacs", "meta"]
categories = ["technical"]
draft = false
+++

## Introduction {#introduction}

This blog is written with an endlessly customizable text editor called [Emacs](https://www.gnu.org/software/emacs/). It is deployed using an Emacs customization called [Ox-Hugo](https://github.com/kaushalmodi/ox-hugo), which exports so-called [org-mode](https://orgmode.org/) files into markdown format for the Hugo static website generator. The static site is pushed to Github for version control and automatically deployed to my domain with a free service offered by Netlify.

There are a lot of different ways to write a blog. Platforms like [WordPress](https://www.wordpress.com) and [Blogger](https://www.blogger.com/) have been around for ages, and I've used quite a few of them. These platforms are excellent, there's no doubt about that. So why go through the trouble of rolling your own blog with Emacs?

In this post, I'll walk through a function I just wrote to instantly add screenshots to posts. That's one beautiful thing about Emacs -- you can literally program your text editor on-the-fly. I realized that adding screenshots was taking up too much time, so I spent 10 minutes writing a function that made it better. How awesome is that?


## TLDR: The function {#tldr-the-function}

```lisp
(defun my-hugo-screenshot (name)
  "Take a screenshot into a time stamped unique-named file with suffix NAME and paste link at point."
  (interactive "sEnter a screenshot name: ")
  (setq imagename (concat (format-time-string "%Y%m%d_%H%M%S")
                          "_" name ".png"))
  (setq filename (concat "~/org/blog/static/img/screenshots/"
                         imagename))
  (setq filelink (concat "/img/screenshots/" imagename))
  (message "waiting 5 seconds while you open the right window")
  (sit-for 5)
  (shell-command (concat "import " filename))
  (insert (concat "[[" filelink "]]")))

(global-set-key "\C-cs" 'my-hugo-screenshot)
```


## Commentary {#commentary}

I won't sugar-coat it. This is an ugly function. Lines are poorly wrapped, too many variables are used, and there is no error checking at all. But it works flawlessly, and it took 10 minutes to research and write. That's all. From start to finish, finding a shell command to take a screenshot on my Linux laptop and writing this little code to take the screenshot, copy it to the static images folder, and add a link in the blog post, took 10 minutes.

For the uninitiated, Emacs is written in a variant of the Lisp programming language ([-> Wikipedia](https://en.wikipedia.org/wiki/Lisp%5F(programming%5Flanguage))) called elisp. Lisp is a beautiful but rarely-used language characterized by all of those parentheses. The language is worth exploring if you have some free time. I believe that every programmer should learn 3 different languages, no matter what their goals are, and learning Lisp will absolutely teach you something about computer science. That said, I have barely scratched the surface, hence the horribly messy function.

On my Arch Linux machine, the command "import" calls an ImageMagick utility that lets you either define a boundary to screenshot, or click on a window to screenshot. The following code tells Emacs to run the command in the shell:

```lisp
(shell-command (concat "import " filename))
```

The `filename` variable is generated pragmatically based on the user's input when the function is first called (using the `interactive` prompt), and the date and time. The link to the screenshot file needs to be relative to the blog file's directory, hence the extra code generating `filelink`. And the `global-set-key` function at the end links the function with the unused key shortcut Ctrl-c s. The whole function lives in my Emacs init file, which loads every time Emacs opens. A detailed explanation of my _init_ is a job for another day.

And that's it! The screenshot is saved in the blog's static screenshots folder and a link is placed in the document. Look, I'll screenshot my writing environment right now:

{{< figure src="/img/screenshots/20200221_093645_sample_screenshot.png" >}}

Let's see WordPress beat that!

Emacs is an incredible piece of software and it makes writing this blog a pleasure. I hope this little example is helpful to someone thinking about trying Emacs or writing basic functions and extensions for it. Learning Emacs can be a long road, but for so many reasons, I can't imagine my life without this software, and that is a unique thing.
