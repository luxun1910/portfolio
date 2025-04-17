+++
title = '多言語環境におけるHugoテンプレートでのURL生成方法'
date = 2025-04-17T08:33:36+08:00
draft = false
+++

このホームページはHugoで作成しています。
テンプレートを選ぶ際、ポートフォリオ兼テックブログ、ついでに英語と中国語の練習としての場も兼ね備えたい、と思っていました。
この条件で探したところ、現在も使用している[hugo-profile](https://github.com/gurusabarish/hugo-profile)にたどり着きました。

この[hugo-profile](https://github.com/gurusabarish/hugo-profile)なのですが、テンプレート部分で、`{{ .Site.BaseURL }}`を使用してリンク先としている個所が散見されました。
[公式ドキュメント](https://gohugo.io/methods/site/baseurl/)にも書いてある通り、これはHugoのコンフィグファイル（hugo.yaml）に記載してあるbaseURLの値が埋め込まれます。
このサイトを例に挙げると、`{{ .Site.BaseURL }}#about`は、`https://techblog.luxunworks.com/#about`を指すことになります。

前述した通り、このサイトは英語と中国語の練習の場も兼ねているので、多言語対応となっていて、Hugo.yamlでdefaultContentLanguageInSubdirをtrueにしています。
つまり、URLの形式は「ドメイン（baseURL）+ 言語名 + コンテンツへのパス」となっています。
一方で、`{{ .Site.BaseURL }}`はbaseURLのみを指します。

そのため、例えば中国語版のサイトを開いている時に`https://techblog.luxunworks.com/#about`へアクセスすると、hugo.yamlのdefaultContentLanguageに設定してあるenが自動的に設定されます。

しかし`#about`は無視されてしまうので、結果として`https://techblog.luxunworks.com/en/#about`ではなく`https://techblog.luxunworks.com/en/`にリダイレクトしてしまいます。

デザインは気に入っていただけに、このあたりの挙動をどうしようかと悩んでいたのですが、[ドキュメント](https://gohugo.io/methods/site/baseurl/)を見ると、以下の文言が書かれていました。

>There is almost never a good reason to use this method in your templates. Its usage tends to be fragile due to misconfiguration.
>
>Use the absURL, absLangURL, relURL, or relLangURL functions instead.

つまり、このメソッドはテンプレートで使わず、absURL, absLangURL, relURL, relLangURLを使え！ということですね。
実際に[absLangURLのドキュメント](https://gohugo.io/functions/urls/abslangurl/)を見てみると、どうやら言語サブディレクトリ付きで絶対URLを生成してくれるとのことでした。

早速テンプレートをカスタマイズしたところ、想定した通りの挙動となり、リンク先が`https://techblog.luxunworks.com/en/#about`のように設定されました。

同じような事象に悩まされている方は、是非absURL, absLangURL, relURL, relLangURLを使ってみてください！
