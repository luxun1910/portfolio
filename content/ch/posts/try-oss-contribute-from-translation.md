+++
title = '利用AI从文档翻译开始为OSS做贡献'
date = 2025-04-18T23:39:55+08:00
draft = false
+++

## 我的第一次OSS活动

自从大学毕业成为一名Web工程师以来，我一直很钦佩那些为OSS（开源软件）做贡献的人，但总觉得这与自己无关。

因为，即使我平时使用OSS，也发现不了bug，也想不出功能的改进方案。
而且，贡献这个词总感觉门槛很高，需要相当的修炼才行。
我曾认为，OSS活动是一些超级工程师做的事情，对我来说是遥不可及的梦想……

然而有一天，我发现这个博客的主题[hugo-profile](https://github.com/gurusabarish/hugo-profile/pull/183)在切换到暗黑主题时，页脚的Github图标颜色不会改变，导致看不清。
我认为这是一个好机会，于是研究了贡献的方法，并提交了一个[pull request](https://github.com/gurusabarish/hugo-profile/pull/183)，最终被合并了。

![我的第一个Pull Request](/images/try-oss-contribute-from-translation/my-first-pr.jpeg)

从那时起，我开始体会到OSS活动的乐趣。
我不知道用"体验类似志愿者活动"这样的表达是否恰当，但总之，我能感受到自己正在为某件事"贡献"的感觉。
即使不是超级工程师，也可以从小事做起，参与OSS活动。

以此为契机，我开始尽我所能地提交了一些pull request。

## 推荐新手从翻译开始

在这些活动中，我认为对新手来说最容易入手的就是文档翻译。

在日益全球化的世界里，有许多库没有日文版文档。
另一方面，随着以ChatGPT为首的生成式AI、像Cursor这样利用AI的编辑器，以及DeepL等高性能翻译工具的兴起，任何人都可以轻松参与翻译活动的环境已经形成。
与修复bug或添加功能相比，翻译的门槛更低，参与起来相对容易。

当然，最低限度的语言能力是必需的，但是通过使用前面提到的AI和翻译工具，可以相当轻松地完成工作。

如果你想参与OSS活动但觉得门槛很高，不妨先从翻译开始贡献？

## 提交日文文档Pull Request的实际过程

最近，我为一个原本只有外文版的文档添加了日文版，我想分享一下在这个过程中获得的经验和注意事项。

我翻译的是扩展了[MyBatis](https://mybatis.org/mybatis-3/ja/)的库[MyBatis-Plus](https://github.com/baomidou/mybatis-plus)的[文档](https://www.baomidou.com/)。
由于这个库是由中国的开发者创建的，所以文档也只有中文版。

我大约花了3周时间，翻译了大约50页，超过13000行，并最终被合并。
工作日我大概每天工作1小时左右，周末则工作4-5小时。
考虑到这是一个规模较大的库，我认为这个工作速度相当快。但这并不是因为我的语言能力特别出色。
正如我稍后会解释的，使用Cursor，只要具备最低限度的语言能力，就可以飞速地进行翻译工作。
我也会一并解释这一点。

### 1. 寻找仍然是外文的OSS文档

当然，首先需要找到翻译的对象。
回顾一下你在工作或个人项目中一直使用的库可能是最快的方法。
浏览一下Github的[Explore](https://github.com/explore)也是一个不错的选择。

顺便说一下，我接触MyBatis-Plus的契机是因为我现在所在的公司正在使用它。

### 2. 确认是否存在翻译项目

访问相关文档的Github页面后，确认是否已经存在日文翻译项目。
如果存在，则跳到步骤4；如果不存在，则进入步骤3。

### 3. 提交Issue

如果不存在日文翻译，最好先在Issue中询问是否可以创建日文翻译。
可以参考其他人的Issue来决定使用哪种语言编写Issue，但通常应该是英语。

我提交了如下所示的[Issue](https://github.com/baomidou/mybatis-plus-doc/issues/403)，得到了"可以试试看"的回复，于是我立刻开始了工作。

![提交给MyBatis-Plus文档的Issue](/images/try-oss-contribute-from-translation/issue-to-mybatis-plus-document.jpeg)

### 4. 确认规则

有些项目可能对分支的创建等有规定，所以务必一并确认。
通常这些规则会写在README.md文件中。
MyBatis-Plus文档没有特别规定。

### 5. Fork并拉到本地

Fork指的是将他人的远程仓库复制为自己的远程仓库。
可以理解为，对这个复制过来的仓库内容进行修改，最终以pull request的形式提交。

点击仓库页面右上角、Star按钮左侧的Fork按钮，然后按照屏幕上的指示操作即可。

![Fork按钮的位置](/images/try-oss-contribute-from-translation/where-is-fork-button.jpeg)

### 6. 开始翻译工作

从这里开始才是正题！

这次，我决定使用我平时常用的Cursor作为翻译工具。
原因是它可以保持markdown等格式不变，并在清晰显示差异的同时进行翻译。
比起复制粘贴到ChatGPT或DeepL，然后再粘贴回编辑器，这样可以更高效地进行工作。

这样一来，就不需要自己从头开始构思译文，只需要判断"这个日文翻译与原文相比是否恰当"即可。
这个差别非常大。即使不擅长写作，用这种方法也可以像玩找茬游戏一样进行翻译工作。

具体来说，首先将原始文档文件复制粘贴到翻译后文档所在的目录下。
这次，[MyBatis-Plus](https://github.com/baomidou/mybatis-plus)的[文档](https://www.baomidou.com/)是使用名为[Astro Starlight](https://starlight.astro.build/)的静态网站生成器构建的，翻译时放置md文件的位置也是固定的，所以我按照这个要求进行了复制粘贴。

接下来，在聊天窗口中输入并发送类似"{文件路径}是中文文档。请翻译成日语。"的消息。

如果顺利，翻译就会像下面这样完成（因为日文版已经翻译完成，这里以中文翻译成英文为例）。

![使用Cursor进行翻译](/images/try-oss-contribute-from-translation/translation-by-cursor.png)

差异会显示出来，修改翻译不自然的部分后点击Accept。

#### 翻译时的注意事项

以下是翻译时需要注意的几点。

##### 注意点1：每次翻译一个文件

如果试图一次性翻译所有内容，可能会导致译文无法正确应用到文件中。
因此，不要贪心一次性翻译所有文档，最好一个文件一个文件地翻译。

##### 注意点2：使用高性能模型

理所当然，使用性能更高的模型会得到更精确的翻译结果。
在我尝试过的模型中，claude-3.7-sonnet和gemini-2.5-pro-exp的质量相当高。
Cursor可用的模型列表可以在[这里](https://docs.cursor.com/settings/models)找到，请根据需要选择性能看起来更高的模型。

##### 注意点3：翻译长文时，注意后半部分

虽然可以通过调整prompt来改善，但在我的环境中，翻译接近1000行的长文时，越到后面，明显奇怪的翻译就越多。
可能会漏掉整个句子，或者开始使用在日语中不自然、可疑的词语。
因此，在修改时要特别注意后半部分。

##### 注意点4：需要统一的地方手动修改，或使用Project Rules

例如，在这次的文档中，"代码生成器"这个词频繁出现。

"代码"是code，"生成器"是generator的意思，但是AI会生成"コードジェネレータ"、"コードジェネレーター"、"コード生成器"等多种翻译模式。
翻译完成后，对于使用相同术语的地方，要通过替换等方式进行修正。

如果事先知道哪些地方需要统一，最好在Cursor的Project Rules中说明这一点，这样应该能生成更精确的翻译。

### 7. 频繁地commit和push

关于commit和push的粒度有各种说法，但我认为最重要的是要频繁地进行。
正如步骤8中介绍的，之后可以合并commit，而且这样做也可以作为万一发生意外时的备份。

### 8. 合并commit

既然是向他人的源代码提交commit，最好保持commit历史的整洁。

方法有很多，我使用了VSCode扩展GitLens。
具体可以参考[这篇文章](https://qiita.com/sakes9/items/0fe4cba262b3ca95c68b)。

### 9. 创建Pull Request

打开你fork的账户的仓库页面，会看到提示是否提交pull request的文字，按照指示创建pull request即可。
这次我提交了[这个pull request](https://github.com/baomidou/mybatis-plus-doc/pull/406)。

### 10. 等待合并

接下来就是等待了。
有时，对方或AI可能会提出修改意见。

![AI对Pull Request提出的指正](/images/try-oss-contribute-from-translation/pr-point-out.jpeg)

这时，要仔细理解对方的意见，并进行修改。

### 11. 合并

恭喜你！你为OSS做出了贡献！

![Pull Request被合并的瞬间](/images/try-oss-contribute-from-translation/pr-merged.jpeg)

## 总结

怎么样？是不是感觉自己也能做到？
我虽然付费订阅了Cursor，但免费额度也足够使用，而且，虽然稍微麻烦一点，使用DeepL或ChatGPT也能做到类似的事情。

OSS活动不仅仅是代码，可以从很多方面做出贡献。
文档翻译也是OSS活动中很有价值的一环。
你也不妨从自己力所能及的事情开始尝试一下？
