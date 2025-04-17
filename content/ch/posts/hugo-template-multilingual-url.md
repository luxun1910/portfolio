+++
title = '多语言环境下Hugo模板中的URL生成方法'
date = 2025-04-17T08:33:36+08:00
draft = false
+++

这个主页是用Hugo创建的。
在选择模板时，我想要一个作品集兼技术博客，同时也希望它成为练习英语和中文的场所。
根据这些条件搜索后，我找到了目前正在使用的[hugo-profile](https://github.com/gurusabarish/hugo-profile)。

在这个[hugo-profile](https://github.com/gurusabarish/hugo-profile)模板中，我注意到在许多地方都使用了`{{ .Site.BaseURL }}`作为链接。
正如[官方文档](https://gohugo.io/methods/site/baseurl/)中所述，这会嵌入Hugo配置文件（hugo.yaml）中指定的baseURL值。
以本站为例，`{{ .Site.BaseURL }}#about`会指向`https://techblog.luxunworks.com/#about`。

如前所述，这个网站也是用来练习英语和中文的场所，所以它支持多种语言，我在Hugo.yaml中将defaultContentLanguageInSubdir设置为true。
这意味着URL格式变成了"域名（baseURL）+ 语言名称 + 内容路径"。
然而，`{{ .Site.BaseURL }}`仅指向baseURL本身。

因此，在查看中文版网站的同时访问`https://techblog.luxunworks.com/#about`时，会自动应用hugo.yaml中设置的defaultContentLanguage（即"en"）。

但由于`#about`部分被忽略，它会重定向到`https://techblog.luxunworks.com/en/`而不是`https://techblog.luxunworks.com/en/#about`。

虽然我非常喜欢[hugo-profile](https://github.com/gurusabarish/hugo-profile)的设计，但是我一直在思考如何处理这种行为，但查看[文档](https://gohugo.io/methods/site/baseurl/)时，我发现以下声明：

>There is almost never a good reason to use this method in your templates. Its usage tends to be fragile due to misconfiguration.
>
>Use the absURL, absLangURL, relURL, or relLangURL functions instead.

换句话说，不要在模板中使用这个方法，而应该使用absURL, absLangURL, relURL或relLangURL!
查看[absLangURL文档](https://gohugo.io/functions/urls/abslangurl/)后发现，它可以生成带有语言子目录的绝对URL。

我立即自定义了模板，它按预期运行，链接设置为类似`https://techblog.luxunworks.com/en/#about`的形式。

如果你也遇到类似的问题，请尝试使用absURL, absLangURL, relURL或relLangURL!

