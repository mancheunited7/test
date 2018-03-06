# Rails入門13

## 今回のテーマ

```
以下のようなフォームを作成できること

[![https://diveintocode.gyazo.com/f661a7b7cdf1e4e058077ccef3e20a65](https://t.gyazo.com/teams/diveintocode/f661a7b7cdf1e4e058077ccef3e20a65.png)](https://diveintocode.gyazo.com/f661a7b7cdf1e4e058077ccef3e20a65)
```

## 章の内容

- ブログを作成するための新規画面（`new`）をRESTfulに沿って実装する。
- 新規画面のフォームを`form_withメソッド`を使用して実装する。

## RESTfulとは

RESTfulを「REST」と「ful」に分けて考えると、

「REST」は、CRUD機能をHTTPメソッドと組み合わせて自由に操作できるようにする、ことを意味しています。

「ful」は、◯◯の性質を持っている、という意味になります。

よって、RESTfulとは、`CRUD機能を自由に操作できる性質を持った考え方`、ということが言えます。

さっそく、RESTfulな観点を意識し、`new`を実装していきましょう。

## 【new】

Newには、次の役割があります。

- 新規画面を表示させる
- 入力された値をcreateに送る

Newの機能を実現するために、ブラウザからリクエストがきて処理を行うRouting→Controller→Viewの順でコードを実装していきます。今回Modelの実装はありません。

### Routing

まずは、ルーティング機能から作成しましょう。

新規作成画面のルーティングを作成するために、`resourcesメソッド`を使用します。
routes.rbファイルに、以下のように記述します。

`config/routes.rb`

```rb
Rails.application.routes.draw do
  # 追記する
  resources :blogs
end
```

#### resources

`resourcesメソッドについて（Railsガイド）`
https://railsguides.jp/routing.html#crud%E3%80%81%E5%8B%95%E8%A9%9E%E3%80%81%E3%82%A2%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3

resourcesメソッドとは、RESTfulなルーティングを自動生成するもの。
実際に`rails routes`コマンドをターミナルで実行して、確かめてみましょう。

```
dive_into_code:~/workspace/sample (master) $ rails routes
   Prefix Verb   URI Pattern               Controller#Action
    blogs GET    /blogs(.:format)          blogs#index
          POST   /blogs(.:format)          blogs#create
 new_blog GET    /blogs/new(.:format)      blogs#new
edit_blog GET    /blogs/:id/edit(.:format) blogs#edit
     blog GET    /blogs/:id(.:format)      blogs#show
          PATCH  /blogs/:id(.:format)      blogs#update
          PUT    /blogs/:id(.:format)      blogs#update
          DELETE /blogs/:id(.:format)      blogs#destroy
```

index/create/new/edit/show/update/destroyそれぞれのアクションへのルーティングが生成されているのが分かります。

このように、resourcesメソッドを使用することで、RESTfulに沿った7つのルーティングが自動的に生成されます。

※updateへのroutingが2個生成されているのは、後の仕様でpatchが追加されたためです。特に気にする必要はありません。

ここで、`rails routes`コマンドで表示された、結果の見方を説明します。

`Prefix`は、URLを簡単に表現した名称

`Prefix`を使用すると、link_toメソッド、renderメソッド、redirectメソッドが必要とするパスを簡単に記述できるというメリットがある。

`Verb`は、HTTPメソッド

`URI Pattern`は、リクエストURL

`Controller#Action`は、関連づけされたコントローラ名とアクション名

例えば、`GET`メソッドで、`/blogs/new`のURLでリクエストが来た場合、Blogsコントローラーのnewアクションを選択するルーティングが作成できた、という見方ができます。

[![https://diveintocode.gyazo.com/7831da463339ea7bd5bc5290da60fe51](https://t.gyazo.com/teams/diveintocode/7831da463339ea7bd5bc5290da60fe51.png)](https://diveintocode.gyazo.com/7831da463339ea7bd5bc5290da60fe51)

**resourcesメソッドを使用するメリット**

resourcesメソッドを使用しない場合、URLとアクションを一つ一つ紐付ける必要があります。紐付けるコードを実装するのは手間ですし、実装されたコードも煩雑になります。

`resourcesメソッドを使用しない場合`

```
get ‘/blogs’, to: ‘blogs#index’
get ‘/blogs/new’, to: ‘blogs#new’
get ‘/blogs/:id’, to: ‘blogs#show'
```

resourcesメソッドを使用すると、先ほど実装したように、`１行`記述するだけですみます。
簡潔に簡単に実装できるというメリットがresourcesメソッドにはあります。

これで、ブログのCRUD機能に必要な、ルーティングを実装することができました。

### Controller

ルーティングを作成することができたので、次は、コントラーラのアクションを定義します。

`app/controllers/blogs_controller.rb`

```rb
class BlogsController < ApplicationController
  def index
  end

  # 追記する
  def new
  end
end
```

### Viewの作成

最後に、Viewを作成します。

BlogsControllerのnewアクション（メソッド）が呼ばれた場合、Viewはデフォルトで`blogs/new.html.erb`を呼び出すので、`blogs/new.html.erb`を作成します。

### form_with

`blogs/new.html.erb`ファイルが作成できたところで、実際にフォームを作成していきますが、モデルと関連付けたフォームを作成するには、`form_withメソッド`がとても便利です。

`form_withメソッド`は、ビューヘルパーの一種で、フォームの作成を簡単にします。

```
Rails 5.1より前のバージョンでは、`form_for`と`form_tag`の2種類がありましたが、
Rails 5.1ではこの2つを`form_with`に統合してますので、今後は`form_with`メソッドを使用しましょう。

`form_forとform_tagのform_withへの統合（Railsガイド）`
https://railsguides.jp/5_1_release_notes.html#form-for%E3%81%A8form-tag%E3%81%AEform-with%E3%81%B8%E3%81%AE%E7%B5%B1%E5%90%88
```

例えば、次のフォームを作成しようとなると、

[![https://diveintocode.gyazo.com/29f94c13d79371f31700c02cf95675e5](https://t.gyazo.com/teams/diveintocode/29f94c13d79371f31700c02cf95675e5.png)](https://diveintocode.gyazo.com/29f94c13d79371f31700c02cf95675e5)

以下のHTMLを記述することになりますが、
```html
<form action="/blogs" accept-charset="UTF-8" data-remote="true" method="post"><input name="utf8" type="hidden" value="✓"><input type="hidden" name="authenticity_token" value="qbu8RFLhyCuAElg1qr7MLYDEdiEINJdEe1JLQFLr7kyWPFiHCEZKszOrvwON33s9CqGz7njGpBasOzrpYkb8/w==">
  <div class="blog_title">
    <label for="blog_title">Title</label>
    <input type="text" name="blog[title]">
  </div>
  <div class="blog_title">
    <label for="blog_content">Content</label>
    <input type="text" name="blog[content]">
  </div>
  <input type="submit" name="commit" value="Create Blog" data-disable-with="Create Blog">
</form>
```

ビューヘルパーである'form_withメソッド'を使用することで、以下のように簡単に記述できます。

```html
<%= form_with(model: Blog.new) do |form| %>
  <div class="blog_title">
    <%= form.label :title %>
    <%= form.text_field :title %>
  </div>
  <div class="blog_title">
    <%= form.label :content %>
    <%= form.text_field :content %>
  </div>
  <%= form.submit %>
<% end %>
```

#### form_withの引数

先ほど作成したフォームで、'form_withメソッド’は、次のようにmodelオプションを設定し、その引数にモデルのインスタンスを設定しています。

```
<%= form_with(model: Blog.new) %>
```

これは、Railsが、モデルオプションに設定したインスタンスを元に、行き先（どのようなリクエストを送るか）を自動で推測してくれます。

[![https://diveintocode.gyazo.com/c6054cf6b7503342901c91afe5c37da1](https://t.gyazo.com/teams/diveintocode/c6054cf6b7503342901c91afe5c37da1.png)](https://diveintocode.gyazo.com/c6054cf6b7503342901c91afe5c37da1)

Blogモデルのインスタンスであれば、ブログを作成するようなリクエストを、Userモデルのインスタンスであれば、ユーザーを作成するようなリクエストを自動的に作成してくれます。

ここでmodelオプションを使用しない場合を考えると、

```
<%= form_with(url: blogs_path, method: post) %>
```

と記述するように、行き先URLとHTTPメソッドを個別に指定する必要があります。

modelオプションを使用した方が、簡単でわかりやすいといったメリットがあります。

### フォーム作成

今までのことをふまえ、`blogs/new.html.erb`にフォームを作成してみましょう。

`app/views/blogs/new.html.erb`

```html
<%= form_with(model: Blog.new, local: true) do |form| %>
  <div class="blog_title">
    <%= form.label :title %>
    <%= form.text_field :title %>
  </div>
  <div class="blog_title">
    <%= form.label :content %>
    <%= form.text_field :content %>
  </div>
  <%= form.submit %>
<% end %>
```

`<%= form_with(model: Blog.new, local: true) do |form| %>`form_withメソッドには、Blog.new、つまりブログモデルのインスタンスを引数として渡しています。そうすることで、`<form class="new_blog" id="new_blog" action="/blogs" accept-charset="UTF-8" method="post">`のように、POSTメソッドで`/blogs`URLにリクエストを送信するフォームを作成することができます。

また、form_withメソッドは、デフォルトではJavaScript用のリクエストが発生してしまうので、` local: true`とすることで、HTML用のリクエストを発行するようにします。(難しいので、この時点で理解できなくても問題ありません。)

`do ~ end(ブロック)`の中に、用意されたメソッド`form.labelやform.submit`を使用することで、フォームに結びついたテキストフィールドやSubmitボタンを作成することが可能になります。

最後に、formタグとしてHTMLコードを生成するために、form_withメソッドは、`<%=`で囲みます。`<%=`で囲まないと、HTMLコードは作成されず、フォームが画面に表示されなくなります。

#### 各フォームパーツのメソッドについて

`form.label`はHTMLのlabelタグを作成するメソッドです。
`form.text_field`は、属性(type)をtextに指定した、HTMLのinputタグを作成するメソッドです。
`form.submit`は、属性(type)をsubmitに指定した、HTMLのinputタグを作成するメソッドです。value(ボタンに表示する名称)は、form_withに渡されたモデルクラスによって代わります。valueを変更する場合、オプションを指定する必要があります。

それぞれをdivタグで囲んでいるのは、改行するためです。divタグもしくはbrタグで囲わないと横一列になります。

### サーバーを起動し、確認する

作成したビューを確認しましょう。
ターミナルで、サーバー起動コマンドを実行し、`blogs/new`にアクセスします。

`rails s -b $IP -p $PORT`

[![https://diveintocode.gyazo.com/2c2c54bb0e53568dfc30b969c3c58f68](https://t.gyazo.com/teams/diveintocode/2c2c54bb0e53568dfc30b969c3c58f68.png)](https://diveintocode.gyazo.com/2c2c54bb0e53568dfc30b969c3c58f68)

このような画面が表示されれば、`new`の設定は完了です。

### まとめ

```
resourcesメソッドを使用すると、
簡単に簡潔に必要なルーティング設定をすることができる。

ビューヘルパーの一種であるform_withメソッドを使用することで、
必要なHTMLを素早く生成できる。
```

## お疲れ様でした。

#### 質問について

質問はこちらのURLにお願いいたします。

https://diveintocode.jp/diver/questions/new

## テキストフィードバックについて

今後のDIVE INTO CODEの発展のために、お時間があれば
フィードバックのご協力をお願い致します。
以下を参考に、DIVERのコメント欄にご投稿して頂けると有難いです。

・テキストの難易度は適切であったか

・テキストの分量は適切であったか

・説明していない用語はなかったか

・テキストに間違いはなかったか

・テキストのゴールに対して適切な内容であったか

・他に説明を加えてほしいことは無かったか

・やってて楽しいテキストであったか

・有益なテキストであったか

**その他なんでもご意見をご投稿ください**

**DIVER質問機能のコメント欄に記述をお願い致します。**