+++
title = 'URL Generation Methods in Hugo Templates for Multilingual Environments'
date = 2025-04-17T08:33:36+08:00
draft = false
+++

This homepage is created with Hugo.
When choosing a template, I wanted a portfolio combined with a tech blog, and also a place to practice English and Chinese.
After searching with these conditions, I found [hugo-profile](https://github.com/gurusabarish/hugo-profile), which I'm currently using.

In this [hugo-profile](https://github.com/gurusabarish/hugo-profile) template, I noticed that `{{ .Site.BaseURL }}` is frequently used as the link destination in many places.
As mentioned in the [official documentation](https://gohugo.io/methods/site/baseurl/), this embeds the baseURL value specified in the Hugo config file (hugo.yaml).
Taking this site as an example, `{{ .Site.BaseURL }}#about` would point to `https://techblog.luxunworks.com/#about`.

As mentioned earlier, this site also serves as a place to practice English and Chinese, so it supports multiple languages, and I've set defaultContentLanguageInSubdir to true in Hugo.yaml.
This means the URL format becomes "domain (baseURL) + language name + path to content".
However, `{{ .Site.BaseURL }}` only refers to the baseURL itself.

Therefore, when accessing `https://techblog.luxunworks.com/#about` while viewing the Chinese version of the site, the defaultContentLanguage set in hugo.yaml (which is "en") is automatically applied.

But since the `#about` part is ignored, it redirects to `https://techblog.luxunworks.com/en/` instead of `https://techblog.luxunworks.com/en/#about`.

I liked the design so much that I was wondering how to handle this behavior, but looking at the [documentation](https://gohugo.io/methods/site/baseurl/), I found the following statement:

>There is almost never a good reason to use this method in your templates. Its usage tends to be fragile due to misconfiguration.
>
>Use the absURL, absLangURL, relURL, or relLangURL functions instead.

In other words, don't use this method in templates, but use absURL, absLangURL, relURL, or relLangURL instead!
Looking at the [absLangURL documentation](https://gohugo.io/functions/urls/abslangurl/), it turns out that it generates absolute URLs with language subdirectories.

I immediately customized the template, and it behaved as expected, with links set to something like `https://techblog.luxunworks.com/en/#about`.

If you're struggling with a similar issue, please try using absURL, absLangURL, relURL, or relLangURL!

