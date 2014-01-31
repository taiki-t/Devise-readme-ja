Devise-readme-ja
================
<div class="bs-callout bs-callout-info">
Devise を使いたいので、勉強のためにとりあえずREADMEを訳します。
誤訳等ございましたらご指摘願います。Deviseを使用するのに十分な量だけ順次訳したいと思います。
バージョンは 3.2.2です。
すでにいくつも日本語訳や解説が存在します。
こちらの記事もご覧ください。
<ul>
<li>[http://easyramble.com/devise-on-rails.html:title]</li>
<li>[http://babie.hatenablog.com/entry/20100729/1280381392:title]</li>
<ul>
</div>
-----

* Rackベース
* Railsエンジンに基づいた完全な MVC のシステム
* 同時に複数のモデルにサインインさせることができる。
* モジュラー構造。好きな部分だけ使える。

次の10のモジュールから構成される。

| モジュール名 | 説明 |  
| ------------ | ---- |  
| [Database Authenticatable](http://rubydoc.info/github/plataformatec/devise/master/Devise/Models/DatabaseAuthenticatable) | encrypts and stores a password in the database to validate the authenticity of a user while signing in. The authentication can be done both through POST requests or HTTP Basic Authentication.  |  
| [Omniauthable](http://rubydoc.info/github/plataformatec/devise/master/Devise/Models/Omniauthable) |  adds Omniauth (https://github.com/intridea/omniauth) support; |
| [Confirmable](http://rubydoc.info/github/plataformatec/devise/master/Devise/Models/Confirmable) |  sends emails with confirmation instructions and verifies whether an account is already confirmed during sign in. |
| [Recoverable](http://rubydoc.info/github/plataformatec/devise/master/Devise/Models/Recoverable) |  resets the user password and sends reset instructions. |
| [Registerable](http://rubydoc.info/github/plataformatec/devise/master/Devise/Models/Registerable) |  handles signing up users through a registration process, also allowing them to edit and destroy their account. |
| [Rememberable](http://rubydoc.info/github/plataformatec/devise/master/Devise/Models/Rememberable) |  manages generating and clearing a token for remembering the user from a saved cookie. |
| [Trackable](http://rubydoc.info/github/plataformatec/devise/master/Devise/Models/Trackable) |  tracks sign in count, timestamps and IP address. |
| [Timeoutable](http://rubydoc.info/github/plataformatec/devise/master/Devise/Models/Timeoutable) |  expires sessions that have no activity in a specified period of time. |
| [Validatable](http://rubydoc.info/github/plataformatec/devise/master/Devise/Models/Validatable) |  provides validations of email and password. It's optional and can be customized, so you're able to define your own validations. |
| [Lockable](http://rubydoc.info/github/plataformatec/devise/master/Devise/Models/Lockable) |  locks an account after a specified number of failed sign-in attempts. Can unlock via email or after a specified time period. |



Devise は YARV 上でスレッドセーフであることが保証されています。JRubyでのスレッドセーフサポートは進行中です。

## はじめる。

Devise 3.0 は Rails3.2以上で動作します。Gemfileに以下を追加してください。

```ruby
gem 'devise'
```

`bundle`コマンドを実行してインストールしてください。

DeviseをGemfileに追加し、インストールした後に、ジェネレータを実行して下さい。

```console
rails generate devise:install
```

ジェネレータはイニシャライザをインストールします。イニシャライザにはDeviseの全ての設定オプションが記載されているので、__必ず__見て下さい。完了後、ジェネレータを使用して好きなモデルにDeviseをインストールします。

```console
rails generate devise MODEL
```

 MODEL を、アプリケーションのユーザに使われているクラス名に置き換えてください。大方の場合`User`ですが、`Admin`の場合もあるでしょう。これによってモデルが（存在しなければ）作成され、デフォルトのDeviseのモジュールが設定されます。
ORMが対応している場合、ジェネレータはマイグレーションファイルも作成するので、次は、通常 `rake db:migrate`を実行します。また、このジェネレータはconfig/routes/rbファイルの設定も行い、Deviseコントローラを指し示すようにします。

ここで、アプリケーションを再起動する必要があることに注意してください。再起動をしないと、ログインができなかったり、ルートヘルパーが定義されていないなどのエラーが発生します。

### コントローラフィルタとヘルパー
Deviseはコントローラとビューで使用されるヘルパーをいくつか作成します。ユーザの認証をコントローラに設定するには、次の before_filterを加えるだけで大丈夫です。:

```ruby
before_filter :authenticate_user!
```
ユーザがサインインしているか確認するためには、次のヘルパーを使用してください。:

```ruby
user_signed_in?
```
現在サインインしているユーザには、次のヘルパーが使えます。:

```ruby
current_user
```
次のスコープでセッションにアクセスできます。:

```ruby
user_session
```
ユーザがサインインした後や、アカウントの確認またはパスワードの確認後、Deviseはリダイレクトの為にスコープされたルートパスを探しにいきます。
例：:userリソースの場合、`user_root_path`が存在するときはそれを使い、ない場合はデフォルトの`root_path`が使われます。これは、ルートを定義する必要あることを意味します：

```ruby
root to: "home#index"
```
また、リダイレクトフックをカスタマイズするには、`after_sign_in_path_for` や`after_sign_out_path_for`を上書きます。  

最後に、それぞれの環境におけるメーラーのデフォルトurlオプションを設定する必要があります。"config/environments/development.rb"のための設定は次のようになります:  

```ruby
config.action_mailer.default_url_options = { :host => 'localhost:3000' }
```
もしdeviseモデル名に "user" ではなく "member" を使用している場合、ヘルパーは次のようにする必要があることに注意してください：

```ruby
before_filter :authenticate_member!

member_signed_in?

current_member

member_session
```

### モデルの設定

モデル内のdeviseメソッドは、モデルのモジュールを設定するためにいくつかのオプションを受け取ります。
例えば、暗号化アルゴリズムのコストは次のように設定します：

```ruby
devise :database_authenticatable, :registerable, :confirmable, :recoverable, :stretches => 20
```
:stretchesの他に、 :pepper, :encryptor, :confirm_within, :remember_for, :timeout_in, :unlock_in に加え、他の値も定義することができます。
詳細は、`devise:install`を実行したときに作成されるinitializerファイルを参照してください。


### ストロングパラメータ
ビューをカスタマイズするとき、フォームに新しい属性を追加することがあるとおもいます。Rails 4 はパラメータのサニタイズをモデルからコントローラに移したので、
Deviseでも同様にこの問題をコントローラで処理することになりました。

Deviseには、パラメータの任意のセットをモデルにまで引き渡すことができる（したがってサニタイズが必要な）アクションは3つしかありません。
それらのアクション名とデフォルトで許可されるパラメータを次に挙げます：

| アクション | 説明 |  
| ---------- | --- |  
| `sign_in` (`Devise::SessionsController#new`) | 認証キーのみ許容します。 (`email` 等) |  
| `sign_up` (`Devise::RegistrationsController#create`) | 認証キーに加え、 `password` と `password_confirmation`を許容します。 |  
| `account_update` (`Devise::RegistrationsController#update`) | 認証キーに加え、 `password`, `password_confirmation` , `current_password`を許容します。 |  

追加のパラメータを（遅延評価で）許容したい場合は、`ApplicationController`内に簡単なフィルターを加えることで可能になります。

 `ApplicationController`:

```ruby
class ApplicationController < ActionController::Base
  before_filter :configure_permitted_parameters, if: :devise_controller?

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.for(:sign_up) << :username
  end
end
```

Deviseのデフォルトを完全に変更したり、カスタムした振舞を呼び出したい場合は、ブロックを渡すこともできます。

```ruby
def configure_permitted_parameters
  devise_parameter_sanitizer.for(:sign_in) { |u| u.permit(:username, :email) }
end
```
複数のDeviseモデルがあり、それぞれのモデルについて異なるパラメータサニタイザを設定したい場合について書きます。
この場合、`Devise::ParameterSanitizer` から継承し、自分のロジックを追加することをお勧めします：

```ruby
class User::ParameterSanitizer < Devise::ParameterSanitizer
  def sign_in
    default_params.permit(:username, :email)
  end
end
```

その後、それを使用できるようにコントローラを設定します。

```ruby
class ApplicationController < ActionController::Base
  protected

  def devise_parameter_sanitizer
    if resource_class == User
      User::ParameterSanitizer.new(User, :user, params)
    else
      super # Use the default one
    end
  end
end
```

上記の例では、ユーザが`:username` と `:email`のどちらでも大丈夫なように許可されたパラメータを上書きしています。遅延評価でない方法でパラメータを設定する場合は、上記の before filter をカスタムコントローラに定義することになるでしょう。
コントローラをカスタマイズし、設定する方法の詳細は今後紹介します。

### ビューの設定

認証機能をもつアプリケーションの素早い開発をサポートするためにDeviseは作成されましたが、
カスタマイズをする場合には、Deviseの手順に従ってもらわなければなりません。  

Deviseはエンジンなので、すべてのビューはgem内に収まっています。
それらのビューは始める時には役立つでしょうが、後々アプリケーションによって変更したくなったときはどうすればよいのでしょうか。その場合、次に紹介するジェネレータを実行して、アプリケーション側にすべてのビューをコピーする必要があります。

```console
rails generate devise:views
```
もし一つ以上のDeviseのモデルがあるとき（"User" や "Admin" 等）、Devisがすべてのモデルに同じビューを使用していることに気付くでしょう。幸いにも、Deviseでは簡単な方法でビューをカスタマイズすることができます。 "config/initializers/devise.rb"内
で"config.scoped_views = true" を設定するだけで良いのです。

設定し終えると、"users/sessions/new" や "admins/sessions/new"といった役割に基づいたビューを持つことができます。
スコープ内にビューが見つからない場合は、Deviseは"devise/sessions/new"にあるデフォルトのビューを使用します。
スコープされたビューを生成するために、ジェネレータを使用することも出来ます。

```console
rails generate devise:views users
```

### コントローラの設定

ビューレベルでのカスタマイズでは十分でない場合、次の手順に従ってそれぞれのコントローラをカスタマイズすることができます。

1. 自前のコントローラを作成します。例えば、`Admins::SessionsController`の場合:

    ```ruby
    class Admins::SessionsController < Devise::SessionsController
    end
    ```
    上記の例の場合、コントローラは`app/controller/admins/` ディレクトリ内に作成される必要があることに注意して下さい

2. ルータにこのコントローラを使うよう指示します。

    ```ruby
    devise_for :admins, :controllers => { :sessions => "admins/sessions" }
    ```

3. そして、コントローラを変更したので、`"devise/sessions"` ビューは使われません。なので、忘れずに `"devise/sessions"` を `"admins/sessions"`にコピーして下さい。

    留意して欲しいのは、Deviseはフラッシュメッセージを使用してユーザのサインインが成功したか失敗したか伝えるということです。Deviseはアプリケーションが`"flash[:notice]"` と `"flash[:alert]"`を呼び出すことを前提としています。フラッシュの全体のハッシュは表示しないで下さい。特定のキーのみ表示するようにし、少なくとも`:timedout`キーをハッシュから取り除いて下さい。Deviseはこのキーを幾つかの状況でで追加しますが、このキーが表示されることは意図されていません。

### ルートの設定

Deviseはデフォルトのルートが設定された状態でインストールされます。それらをカスタマイズしたい場合は、
大方 devise_for メソッドを通して行えるでしょう。そのメソッドは :class_name, :path_prefix、といったようないくつかのオプションを受け取ります。I18nのためのパス名の変更も含みます。：

```ruby
devise_for :users, :path => "auth", :path_names => { :sign_in => 'login', :sign_out => 'logout', :password => 'secret', :confirmation => 'verification', :unlock => 'unblock', :registration => 'register', :sign_up => 'cmon_let_me_in' }
```
より詳細を知りたい場合は`devise_for` ドキュメンテーションを参照することを忘れないで下さい。
[http://rubydoc.info/github/plataformatec/devise/master/ActionDispatch/Routing/Mapper:devise_for:title]
Be sure to check `devise_for` documentation for details.

より深いカスタマイズが必要な場合、例えば "/users/sign_in"の他に"/sign_in" を許可したい場合、
やるべきことは、ルータ内で通常通りルートを定義して、`devise_scope` でラップすることだけです。

```ruby
devise_scope :user do
  get "sign_in", :to => "devise/sessions#new"
end
```

この方法で"/sign_in" にアクセスされたとき、deviseに :user スコープを使用するように指示できます。
ルータ内で、`devise_scope`には`as`というエイリアスがあることに注意して下さい。  

http://rubydoc.info/github/plataformatec/devise/master/ActionDispatch/Routing/Mapper:devise_scope

### I18n

DeviseはフラッシュメッセージにI18nを使用していて、フラッシュキーは :notice と :alert です。アプリケーションをカスタマイズするために、独自のロケールファイルを設定することができます。：

```yaml
en:
  devise:
    sessions:
      signed_in: 'Signed in successfully.'
```
また、ルートで定義された単数形の名前を使用して設定したリソースに基づいて個別のメッセージを作成することもできます。

```yaml
en:
  devise:
    sessions:
      user:
        signed_in: 'Welcome user, you are signed in.'
      admin:
        signed_in: 'Hello admin!'
```
Devise mailer はサブジェクトメッセージを作成するために似たようなパターンを使います。：

```yaml
en:
  devise:
    mailer:
      confirmation_instructions:
        subject: 'Hello everybody!'
        user_subject: 'Hello User! Please confirm your email'
      reset_password_instructions:
        subject: 'Reset instructions'
```
利用可能なメッセージを確認するために、Deviseのローカルファイルを参照してください。wikiにある多くの翻訳も参照するとよいでしょう。

https://github.com/plataformatec/devise/wiki/I18n

### テストヘルパー

Deviseは機能仕様のためにいくつかテストヘルパーを備えています。それらを使うためには、機能テスト内にDeviseをインクルードする必要があります。
次のコードを `test/test_helper.rb`ファイルの一番下に追加してください。

```ruby
class ActionController::TestCase
  include Devise::TestHelpers
end
```

RSpecを使用している場合は、次のコードを`spec/support/devise.rb` ファイル内におくことができます。:

```ruby
RSpec.configure do |config|
  config.include Devise::TestHelpers, :type => :controller
end
```

これで`sign_in` と `sign_out`　メソッドを使う用意ができました。
そのようなメソッドはコントローラ内で使う場合と同様の特徴を持ちます。

```ruby
sign_in :user, @user   # sign_in(scope, resource)
sign_in @user          # sign_in(resource)

sign_out :user         # sign_out(scope)
sign_out @user         # sign_out(resource)
```

２つほど、覚えておく大事なことがあります。

1. これらのヘルパーはCapybaraやWebratによって駆動される統合テストでは動作しません。機能テストのみでの使用を意図したものです。代わりに、セッションを使用して明示的にユーザを設定するか、フォームの記入をしてください。

2.  Devise内部のコントローラ、またはDeviseのものから継承しているコントローラをテストする場合は、リクエストの前に、どのマッピングが使用されるべきかをDeviseに伝える必要があります。これが必要な理由は、Deviseはこの情報をルータから得ますが、機能テストはルータを通らないからです。よって明示的に伝えられる必要があります。例えば、ユーザスコープをテストする場合は単純に次のようにしてください。：
    ```ruby
    @request.env["devise.mapping"] = Devise.mappings[:user]
    get :new
    ```

### Omniauth

Devise はそのままでOmniauthをサポートしており、他のプロバイダを使って認証するために使えます。`config/initializers/devise.rb` にOmniauthの設定を指定するだけで使えます。：
```ruby
config.omniauth :github, 'APP_ID', 'APP_SECRET', :scope => 'user,public_repo'
```

Omniauth サポートについての詳細はwikiを参照してください：

* https://github.com/plataformatec/devise/wiki/OmniAuth:-Overview

### 複数モデルの設定

Deviseでは好きなだけDeviseモデルを設定することができます。上記のUserモデルに加え、認証とタイムアウト機能だけをもつAdminモデルを設定したい場合、次のように実行してください。：

```ruby
# 必要なフィールドのマイグレーションを作成する。
create_table :admins do |t|
  t.string :email
  t.string :encrypted_password
  t.timestamps
end

# Admin モデル内
devise :database_authenticatable, :timeoutable

# ルートファイル内
devise_for :admins

# 保護された（protected）コントローラ内
before_filter :authenticate_admin!

# コントローラとビュー内
admin_signed_in?
current_admin
admin_session
```

もしくは、単にDeviseジェネレータを実行しても良いです。

覚えておいて欲しいのは、それらのモデルは完全に異なるルートを持つということです。それらはサインイン、サインアウトや、その他の動作のために同じコントローラを共有**しない**し、**できません**。もし異なる役割に同じアクションを共有させたい場合、role columnを用意したり、もしくは[CanCan](https://github.com/ryanb/cancan)を使い、役割に基づいたアプローチをとることをおすすめします。

### 他の ORMs

Devise はActiveRecord (デフォルト) と Mongoid をサポートします。使用するORMを選択するためには、イニシャライザファイル内でrequireしてください。

## Additional information

### Heroku

Using devise on Heroku with Ruby on Rails 3.1 requires setting:

```ruby
config.assets.initialize_on_precompile = false
```

Read more about the potential issues at http://guides.rubyonrails.org/asset_pipeline.html

### Warden

Devise is based on Warden, which is a general Rack authentication framework created by Daniel Neighman. We encourage you to read more about Warden here:

https://github.com/hassox/warden

### Contributors

We have a long list of valued contributors. Check them all at:

https://github.com/plataformatec/devise/graphs/contributors

## License

MIT License. Copyright 2009-2014 Plataformatec. http://plataformatec.com.br

You are not granted rights or licenses to the trademarks of the Plataformatec, including without limitation the Devise name or logo.
