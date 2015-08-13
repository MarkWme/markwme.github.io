---
layout: post
title:  "Blogging with Jekyll on Windows"
date:   2015-02-05 23:21:33
categories: jekyll
tags: jekyll windows
---
I've dabbled with blogging a bit in the past and currently run a couple of websites that use popular CMS's.  One of those websites was the previous iteration of the site that now lives at this very URL.  For a while now, I've been meaning to resume my blogging activities and to try and write more often.

The thing that was holding me up was figuring out what platform to use.  This site's predecessor used [Drupal][drupal] which was all well and good.  I liked working with Drupal.  It was a little tough to get going, but once you figured it out it was pretty powerful.  Best of all, it has a good, strong community around it and pretty much any feature you want to add to your site has already been thought of by someone else and published as a module you can download for free.

However, the problem with Drupal and it's cohorts is that it's susceptible to attack.  It's a well known, widely used CMS.  The community are great at finding, reporting and fixing security bugs, but back in October 2014 the world of Drupal was shaken to it's very core.  [Drupageddon happened][drupageddon].  Drupageddon was the result of a highly critical SQL injection vulnerability in Drupal's core.  Just like any other significant security issue it was found and fixed by the Drupal contributors and an update made available.  However, as is always the case, as soon as that vulnerability was patched and publicly disclosed, the bad guys began to exploit it.  And in this case, they moved quickly.  So quick in fact that Drupal's security team were forced to publish a [public service advisory][drupal-psa] which contained the following warning:

>  You should proceed under the assumption that every Drupal 7 website was compromised unless updated or patched before Oct 15th, 11pm UTC, that is 7 hours after the announcement.

*7 hours?*  Wow, for most organisations it would be an acheivement to get a critical patch applied across their systems in 7 days, let alone 7 hours.  And even in the most efficient organisation, 7 hours is a pretty impressive time in which to implement a patch.  The implication then, it that pretty much *every* Drupal 7 site got hacked that day.

By sheer good fortune I escaped Drupageddon on the site I care about the most - an active, production retail website for a small business - I just happened to be browsing through my emails late in the evening, saw the announcement from the Drupal security team and figured as it was rated highly critical and it was late, it would be a good time to get this patch installed.  However, I didn't bother with the other websites, the ones I cared less about.  I thought nothing more of it until a couple of weeks later when my webhost sent me a notification that they'd turned off the mail services on my host as they seemed to be sending rather a high volume of messages.  Oh dear ...

Fortunately none of these websites were important.  A couple were dead and I'd never got around to deleting them properly, so I got rid of those.  My old blog was restored from a not-particularly-recent backup.  And I had a bit of fun poking around the remainder to find out what backdoors had been put in, which if nothing else was highly educational.  However, it got me to thinking why I was using a CMS for websites that were, really, just static pages.

And so here we are.  Blogging with [Jekyll][jekyll] on Windows.  It seemed like the perfect solution.

- No content management system, no server-side stuff needed and no PHP!
- I can create blog posts from a text editor. (I'm a geek - I love using [text editors][sublime])
- It makes me use [git][git] and git is something I need to use more of.
- I can host the site on [github][github-pages]
- I don't need to worry about backups - I've got a copy of the website with me always in my local git repository

And lots more good stuff.  I was a little concerned when I started about the warnings of Jekyll not being supported on Windows, but following the [instructions here][jekyll-win] I was able to get it up and running quickly and it seems to work just fine.

So, as of now, I'm up and running with Jekyll, I have the basic starter theme which is perfect to get going with and I have a simple platform on which to post my babbling :-)

[drupal]:		http://drupal.org
[drupageddon]:	https://www.drupal.org/SA-CORE-2014-005
[drupal-psa]:	https://www.drupal.org/PSA-2014-003
[jekyll]:		http://jekyllrb.com/
[sublime]:		http://www.sublimetext.com/
[git]:			http://git-scm.com/
[github-pages]:	https://pages.github.com/
[jekyll-win]:	http://jekyll-windows.juthilo.com/