# Rails入門13

## 今回のテーマ

以下のような、ブログ作成用の新規画面を作成する

[![https://diveintocode.gyazo.com/f661a7b7cdf1e4e058077ccef3e20a65](https://t.gyazo.com/teams/diveintocode/f661a7b7cdf1e4e058077ccef3e20a65.png)](https://diveintocode.gyazo.com/f661a7b7cdf1e4e058077ccef3e20a65)

## 章の内容
ブログ作成用の新規画面（`new`）に必要な

- ルーティングを`RESTful`に沿って実装する。
- アクション(`new`)を実装する。
- フォームを`form_with`メソッドを使用して実装する。

## RESTfulとは

RESTfulを直訳すると、「RESTな性質をもつ」といった意味になります。

ここで「REST」とは、Webを構築するときの考え方の一つで、`リソースとなるURLをHTTPメソッドに対応させ、自由にCRUDできるものとして扱う考え方`といえます。

よって「RESTful」とは、`上記のような決まりごとを守っているもの`、と考えられます。

さっそく、RESTfulな観点を意識し、`new`を実装していきましょう。

## 【new】

Newには、次の役割があります。

- 新規画面を表示する
- 入力された値をcreateに送る

コードの実装は、Routing → Controller → Viewの順で進めます。今回Modelの実装はありません。

### Routing

まずは、ルーティング機能から作成しましょう。

新規画面のルーティングを作成するために、`resources`メソッドを使用します。

`config/routes.rb`

```rb
Rails.application.routes.draw do
  # 追記する
  resources :blogs
end
```

#### resourcesメソッド

`resourcesメソッドについて（Railsガイド）`

https://railsguides.jp/routing.html#crud%E3%80%81%E5%8B%95%E8%A9%9E%E3%80%81%E3%82%A2%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3

resourcesメソッドとは、RESTfulなルーティングを自動生成するメソッドです。

`config/routes.rb`に`resources :blogs`を追記したあと、実際に`rails routes`コマンドをターミナルで実行して、確かめてみましょう。

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

resourcesメソッドを使用することで、RESTfulな7つのルーティングが自動的に生成されます。

※updateへのroutingが2個生成されているのは、後の仕様でpatchが追加されたためです。特に気にする必要はありません。

ここで、`rails routes`コマンドで表示された、結果の見方を説明します。

`Prefix`は、URLを簡単に表現した名称のことです。
Prefixを使用すると、link_toメソッド、renderメソッド、redirectメソッドが必要とするパスを簡単に記述できるというメリットがあります。

`Verb`は、リクエストで送信されるHTTPメソッドを示しています

`URI Pattern`は、リクエストURLを示しています。

`Controller#Action`は、関連付けされたコントローラ名とアクション名を示しています。

例えば、HTTPメソッドが`GET`、URLが`/blogs/new`のリクエストが送られてきた場合、送られてきたリクエストは、Blogsコントローラーのnewアクションへつなぐ準備ができた、という見方ができます。（実際には、Blogsコントローラーのnewアクションはまだ作成されていないので、つながってはいません。あくまで、つなぐ準備ができた段階です。）

[![https://diveintocode.gyazo.com/7831da463339ea7bd5bc5290da60fe51](https://t.gyazo.com/teams/diveintocode/7831da463339ea7bd5bc5290da60fe51.png)](https://diveintocode.gyazo.com/7831da463339ea7bd5bc5290da60fe51)

**resourcesメソッドを使用するメリット**

resourcesメソッドを使用しない場合、URLとアクションを一つ一つ紐付ける必要があります。紐付けるコードを実装するのは手間ですし、実装されたコードも煩雑になります。

`resourcesメソッドを使用しない場合のroutes.rb`

```
get ‘/blogs’,     to: ‘blogs#index’
get ‘/blogs/new’, to: ‘blogs#new’
get ‘/blogs/:id’, to: ‘blogs#show'
・
・
・
```

resourcesメソッドを使用した場合は、`resources :blogs`の１行記述するのみでルーティングが作成されますので、簡潔に簡単に実装できるというメリットが、resourcesメソッドにはあります。

これで、新規画面に必要なルーティングを実装することができました。

### Controller

ルーティングを作成することができたので、次は、コントラーラに`new`アクションを定義します。

`app/controllers/blogs_controller.rb`

```rb
class BlogsController < ApplicationController
  def index
  end

  # 追記する
  def new
    @blog = Blog.new
  end
end
```

### View

最後に、Viewを作成します。

BlogsControllerのnewアクションが呼ばれた場合、Viewはデフォルトで`blogs/new.html.erb`を呼び出すことになっています。そのため、`views/blogs`に`new.html.erb`ファイルを作成する必要があります。

### form_withメソッド

`new.html.erb`ファイルが作成できたら、実際にフォームを作成していきます。
ここで、モデルと関連付けたフォームを作成するには、`form_with`メソッドを使用します。

`form_with`メソッドは、ヘルパーメソッドの一つで、簡単にフォームを作成できます。

```
Rails 5.1より前のバージョンでは、【form_for】と【form_tag】の2種類のメソッドがありましたが、
Rails 5.1ではこの2つのメソッドを【form_with】に統合してますので、今後は【form_with】メソッドを使用しましょう。
```

`【form_for】と【form_tag】の【form_with】への統合（Railsガイド）`

https://railsguides.jp/5_1_release_notes.html#form-for%E3%81%A8form-tag%E3%81%AEform-with%E3%81%B8%E3%81%AE%E7%B5%B1%E5%90%88


例えば、次のフォームを作成しようとすると、

[![https://diveintocode.gyazo.com/29f94c13d79371f31700c02cf95675e5](https://t.gyazo.com/teams/diveintocode/29f94c13d79371f31700c02cf95675e5.png)](https://diveintocode.gyazo.com/29f94c13d79371f31700c02cf95675e5)

本来、次のようなHTMLを作成することになりますが、
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

`form_with`メソッドを使用すると、次のように簡単に記述でき、処理が実行されると上記で作成したHTMLを自動生成してくれます。

```html
<%= form_with(model: @blog) do |form| %>
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

`form_with`メソッドは、次のようにmodelオプションを設定し、その引数にモデルのインスタンスを設定しています。

```
<%= form_with(model: @blog) %>
```

これは、Railsが、モデルオプションに設定したインスタンスを元に、行き先（どのようなリクエストを送るか）を自動で推測してくれます。

逆にmodelオプションを使用しない場合を考えると、次のように、`行き先URL`と`HTTPメソッド`を個別に指定する必要があります。

```
<%= form_with(url: blogs_path, method: post) %>
```

modelオプションを使用した方が、簡単でわかりやすいといったメリットがあります。

### フォーム作成

今までのことを踏まえ、`blogs/new.html.erb`にブログの新規画面フォームを作成してみましょう。

`app/views/blogs/new.html.erb`

```html
<%= form_with(model: @blog, local: true) do |form| %>
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

上から順にコードを見ていきましょう。

`<%= form_with(model: @blog, local: true) do |form| %>`
form_withメソッドのオプションには、modelオプションを指定します。
引数として@blogを指定することで、Railsは自動的に`POST`メソッドで`/blogs`URLにリクエストを送信するフォームを作成します。

また、`form_with`メソッドは、デフォルトでJavaScript用のリクエストが発生してしまうので、` local: true`とすることで、HTML用のリクエストを生成するようにします。(難しいので、この時点で理解できなくても問題ありません。)

`do ~ end`の中には、ラベルを生成する`form.label`、ボタンを生成する`form.submit`を使用することで、フォームに結びついたパーツを作成することができます。

最後に、formタグとしてのHTMLコードを生成するために、`form_with`メソッドを`<%=`で囲みます。`<%=`で囲まないと、HTMLコードは作成されず、フォームが画面に表示されなくなります。

#### 各フォームパーツのメソッドについて

`form.label`はHTMLのlabelタグを生成するメソッドです。

`form.text_field`は、属性(type)をtextに指定した、HTMLのinputタグを作成するメソッドです。

`form.submit`は、属性(type)をsubmitに指定した、HTMLのinputタグを作成するメソッドです。value(ボタンに表示する名称)は、form_withに渡されたモデルクラスによって代わります。valueを変更する場合、オプションを指定する必要があります。

それぞれをdivタグで囲んでいるのは、改行するためです。divタグもしくはbrタグで囲わないと横一列になります。

### サーバーを起動し、確認する

作成したビューを確認しましょう。

ターミナルでサーバー起動コマンドを実行し、`blogs/new`にアクセスします。

`rails s -b $IP -p $PORT`

[![https://diveintocode.gyazo.com/2c2c54bb0e53568dfc30b969c3c58f68](https://t.gyazo.com/teams/diveintocode/2c2c54bb0e53568dfc30b969c3c58f68.png)](https://diveintocode.gyazo.com/2c2c54bb0e53568dfc30b969c3c58f68)

このような画面が表示されれば、`new`の設定は完了です。

### まとめ

```
resourcesメソッドを使用すると、
簡単に簡潔に必要なルーティング設定をすることができる。

ヘルパーメソッドの一つであるform_withメソッドを使用することで、
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
