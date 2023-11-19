---
layout: default
title: Continuous Deployment - cuz less hassle
permalink: continuous-travis
---

# Travisによる継続的デプロイ

*Created by Floor Drees, [@floordrees](https://twitter.com/floordrees)* / *翻訳者: Masato Yamashita, [@M_Yamashii](https://twitter.com/M_Yamashii)*

### 継続的デプロイとは何か？

継続的デプロイは継続的デリバリーの活動の1つです。継続的デリバリーの背景にある考えは、可能な限りソフトウェアのデリバリープロセスを自動化することです。

継続的デプロイ・チェーンを構築できていれば、Gitを使ったデプロイ（すべてをコミットすることでテストが実行され、全テストが実行完了することでデプロイされること）を強制でき、それによってコラボレーションが容易になり、デプロイが高速化されます。そのため、開発者はアプリケーションをより素晴らしくすることに集中できます！

この継続的デプロイという波に乗って航行している素晴らしい会社があります。このガイドでは、[Travis-ci](http://about.travis-ci.org/)を使って、GitHubからanyninesへのRuby on Railsアプリケーションの継続的デプロイを設定します。

__COACH__: 継続的デプロイのメリットについて話してください。

### Github、Travis CI、anynines

まず必要なのは、GitHubのリポジトリにあるアプリです。これはすでにありますね！次に、anyninesを使ってアプリをデプロイする方法のガイドを、最後まで読んでおきましょう。

<!-- TODO メインディレクトリではなくルートが良いのかも -->
次に、アプリのルートディレクトリに、デプロイに関する情報をもったファイルである`manifest.yml`を作成します。ターミナルで以下を実行してください。

{% highlight sh %}
cf push
{% endhighlight %}

このコマンドを実行すると、anyninesへの初回デプロイが開始されます。cf gemは`manifest.yml`がないことに気づき、アプリのインスタンス数やメモリサイズ、どのサービスをバインドするか、そして一番重要となるこの情報を保存するかどうかなど、標準的な設定に関する質問をしてきます。
この質問に'yes'と答えると、`manifest.yml`が作成されます。

プッシュが成功すると、好きなブラウザでアプリケーションにアクセスできるようになります。これでTravisの設定の準備完了です。

今のところ'本物のテスト'を持ってないので、先に進み、テストが成功したことにするTravisの設定ファイルを作成します。ローカルのアプリディレクトリに移動し、``.travis.yml``ファイルを作成してください。いったん以下の内容を貼り付けておいてください。後でTravis gemを使って、情報を追加します。

{% highlight sh %}
language: ruby
script: 'true'
{% endhighlight %}

アプリにはTravisの設定が含まれていますが、Githubからコードを取得しテストを実行するトリガーのタイミングを、Travisはどうやって知ることができるのでしょうか？ここではGitHubのフックが役に立ちます！

#### Travis CI Githubフックの有効化

変更したコードをコミットしてリポジトリにプッシュし、テストが実行されているかどうかtravis-ci.orgを確認してください。ビルドが成功したことを知らせるメールも届くはずです。

{% highlight sh %}
git add .  
git commit -m "test Travis integration"  
git push origin master
{% endhighlight %}

これでactual deploymentを設定できるようになります。
travis gemを使いましょう。
{% highlight sh %}
gem install travis
{% endhighlight %}

`travis`コマンドを使ってanyninesへのデプロイを設定しましょう。
{% highlight sh %}
travis setup cloudfoundry
{% endhighlight %}

anyninesのURLがわからない場合は、以下コマンドを実行してください。
{% highlight sh %}
cf target
{% endhighlight %}

このコマンドでTravisのセットアップに必要な全ての情報を集められます。この情報には、url、ユーザー名、現在使っている組織とスペースが含まれます。anynines.comにサインアップした後に受け取ったウェルカムメールも参考にしてください。

`travis`コマンドが終了したら、``.travis.yml``は以下のようになっているはずです。
{% highlight sh %}
language: ruby
script: 'true'
deploy:
  provider: cloudfoundry
  target: https://api.de.a9s.eu
  username: jane.doe@example.com
  password:
    secure: your encryped password determined by the travis gem=
  organization: railsgirls
  space: heaven
  on:
    repo: jane/railsgirls
{% endhighlight %}

GitHubリポジトリに変更を反映させるために、`.travis.yml`の変更のコミット、プッシュを忘れないでください。

今後GitHubリポジトリに変更をコミットするたびに、テストが実行されアプリがデプロイされます。その際、Travisはこのようなログを出力します。

{% highlight sh %}
Installing deploy dependencies
Fetching: addressable-2.3.5.gem (100%)
Successfully installed addressable-2.3.5
Fetching: multi_json-1.7.9.gem (100%)
Successfully installed multi_json-1.7.9
Fetching: caldecott-client-0.0.2.gem (100%)
Successfully installed caldecott-client-0.0.2
Fetching: i18n-0.6.5.gem (100%)
Successfully installed i18n-0.6.5
Fetching: tzinfo-0.3.37.gem (100%)
Successfully installed tzinfo-0.3.37
Fetching: minitest-4.7.5.gem (100%)
Successfully installed minitest-4.7.5
Fetching: atomic-1.1.13.gem (100%)
Building native extensions.  This could take a while...
Successfully installed atomic-1.1.13
Fetching: thread_safe-0.1.2.gem (100%)
Successfully installed thread_safe-0.1.2
Fetching: activesupport-4.0.0.gem (100%)
Successfully installed activesupport-4.0.0
Fetching: builder-3.1.4.gem (100%)
Successfully installed builder-3.1.4
Fetching: activemodel-4.0.0.gem (100%)
Successfully installed activemodel-4.0.0
Fetching: cf-uaa-lib-2.0.0.gem (100%)
Successfully installed cf-uaa-lib-2.0.0
Fetching: multipart-post-1.2.0.gem (100%)
Successfully installed multipart-post-1.2.0
Fetching: rubyzip-0.9.9.gem (100%)
Successfully installed rubyzip-0.9.9
Fetching: cfoundry-4.3.6.gem (100%)
Successfully installed cfoundry-4.3.6
Fetching: interact-0.5.2.gem (100%)
Successfully installed interact-0.5.2
Fetching: json_pure-1.8.0.gem (100%)
Successfully installed json_pure-1.8.0
Fetching: mothership-0.5.1.gem (100%)
Successfully installed mothership-0.5.1
Fetching: mime-types-1.25.gem (100%)
Successfully installed mime-types-1.25
Fetching: rest-client-1.6.7.gem (100%)
Successfully installed rest-client-1.6.7
Fetching: uuidtools-2.1.4.gem (100%)
Successfully installed uuidtools-2.1.4
Fetching: cf-5.2.2.gem (100%)
Successfully installed cf-5.2.2
22 gems installed
dpl.2
Preparing deploy
Setting target to https://api.de.a9s.eu...... OK
target: https://api.de.a9s.eu
Authenticating.. .  ... OK
Switching to organization railsgirls... OK
Switching to space heaven... OK
dpl.3
Deploying application
Using manifest file manifest.yml
Uploading railsgirls... OK
Stopping railsgirls... OK
Preparing to start railsgirls... OK
Checking status of app 'railsgirls'...
  0 of 1 instances running (1 starting)
  0 of 1 instances running (1 starting)
  1 of 1 instances running (1 running)
Push successful! App 'railsgirls' available at http://railsgirls.de.a9sapp.eu
Logging out... OK
{% endhighlight %}

これで完了です！