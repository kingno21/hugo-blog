+++
title = 'ブログ書きましょう・Hugo・netlify'
slug = 'post1'
image = 'images/letsblog/hugo.png'
date = '2018-08-02T0:00:00'
description = 'ブログの始めよう！'
+++
## はじめに

> ブログを書くことに何の意味がある？

勉強したことの整理になるし、ほかの人がより簡単に情報収集することに貢献できるかつフォロワと議論もできるし、間違いなとをただしてもらえる

> どこで、書けばいいの
>> エンジニアなので、エンジニア事情しか知らなくて申し訳ないです。🙇‍♂️

* 日本: qiita, esa
* アメリカ: medium, 個人ブログ(の方が多い), stackoverflow, reddit
* 中国: 博客园, github, 個人ブログ
* 韓国: 企業ブログ(네이버, 카카오), 個人ブログ

とかで、結構必要な情報は集まるはずです。見てわかるですが、個人ブログがまだ多数をしめている。

> 書いていいどころはある？

単純に考えて、エンジニアであればコードで採用とかしていると思いますが、githubで貢献できているエンジニアがまず少ないし、企業で貢献していると個人githubはコミットできなくなる傾向がある。次に、判断する材料としてブログは実績を見せられる媒体となっているので、いい武器になる

## 本題

個人ブログを作る場合、qiitaもいい選択だし、英語だったらmediumとかもあるのでいいじゃんになるが、あくまでも、まとまる場所なので個人の色と性格を出しにくい

海外のエンジニアを見てみるとやはり、個人のブログを運営して記事を書くのが多い。単純に個性を出しやすいしエンジニアリング力も見せられることができる

### 早速自分のブログを作りましょう

#### Hugo

[Hugo](https://gohugo.io/)とのサイト作成プラットフォームがを見つけた(結構いろんな人が使ってブログ書いてたので気になって試した)。2013年から始まったみたいで、27632個の星をもらってるのでかなりいけてるみたい

> step1: first impressions

ドキュメントなが！！！、量も多い

> step2: 耐えて読み進める

さっぱり、眠い、訳わからないものを説明している

> step3: ダメだ、サンプル触ってみよう

... dirが多い、[HugoDoc](https://github.com/gohugoio/hugo/tree/master/examples/blog)さすが作書使いこなしている

> step4: 新しいサイト作ってみよう

```
// install hugo
brew install hugo
```

```
// create new site
hugo new site blogSite

// install themes
// https://themes.gohugo.io/
// select hello-friend theme

cd blogSite
git clone https://github.com/panr/hugo-theme-hello-friend.git themes/hello-friend

// run site
hugo server -t hello-friend

// open chrome enter: http://localhost:1313
// wala
```

> step5: ...

できた、サイトがレンダリングされた。で、どうするか

#### 実装

いろいろ調べならが進んで、個人的にいいと思ったのは、まず、theme/exampleSiteを見てみましょう(今回はblogの構築に重点を置くので、ほかの機能は説明しません、知りたい時には自分に聞いてくださいできるだけ答えます)

- contents
- data

この二つのdirに必要な情報が全て乗っている。とりあえず、theme/exampleSite/contentsの中身を全部rootにコピーしましょう、サイトで見てみよう

- contents
	- post
		- hello.md

なので、http://localhost:1313/post/hello/をみると、mdファイルの中身がレンダーされている!!!

- data

は、あれば移動して必要に編集して見てください。

#### 編集

sampleの最上部に区切られてる部分がると思いますが、レンダーに必要な情報を記述する部分になっているので、毎回変えれば簡単にタイトルと説明文が変わりますがテーマが壊れることがない

実装を読んでみるとわかりますが、全てhtmlとcssで形を整ってから、templateエンジンを渡って変数を解釈する動作をしている。また、情報後から参照するために最適化された構成になっていることもわかる

これからは、気合いと好みでcssを編集したり、HTMLを追加したりするのもありだし、themeを1から作るこの可能なので有志はやってください(freeで満足している)

### 作り終わった、出そう!

#### github.io

エンジニアであれば、githubは持っていると思うので一番簡単になるはず

「<user_name>.github.io」と、レポジトリを作って`blogSite/public`をデプロイしまよう

詳しくは[Github.io deploy](https://gohugo.io/hosting-and-deployment/hosting-on-github/)

#### netlify

サイトのデプロイを簡単にしてくれるサイト、ドメインも購入できてデプロイも通知してくれるように設定可能

説明いらないぐらい簡単[netlify deply](https://gohugo.io/hosting-and-deployment/hosting-on-netlify/)


## 最後に

ブログ書きましょう！！！


