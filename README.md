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

The generator will install an initializer which describes ALL Devise's configuration options and you MUST take a look at it. When you are done, you are ready to add Devise to any of your models using the generator:
ジェネレータはイニシャライザをインストールします。イニシャライザにはDeviseの全ての設定オプションが記載されているので、__必ず__見て下さい。完了後、ジェネレータを使用して好きなモデルにDeviseをインストールします。

```console
rails generate devise MODEL
```

 MODEL を、アプリケーションのユーザに使われているクラス名に置き換えてください。大方の場合`User`ですが、`Admin`の場合もあるでしょう。これによってモデルが（存在しなければ）作成され、デフォルトのDeviseのモジュールが設定されます。
ORMが対応している場合、ジェネレータはマイグレーションファイルも作成するので、次は、通常 `rake db:migrate`を実行します。また、このジェネレータはconfig/routes/rbファイルの設定も行い、Deviseコントローラを指し示すようにします。

Replace MODEL by the class name used for the applications users, it's frequently `User` but could also be `Admin`. This will create a model (if one does not exist) and configure it with default Devise modules. Next, you'll usually run `rake db:migrate` as the generator will have created a migration file (if your ORM supports them). This generator also configures your config/routes.rb file to point to the Devise controller.

ここで、アプリケーションを再起動する必要があることに注意してください。再起動をしないと、ログインができなかったり、ルートヘルパーが定義されていないなどのエラーが発生します。

Note that you should re-start your app here if you've already started it. Otherwise you'll run into strange errors like users being unable to login and the route helpers being undefined.

### コントローラフィルタとヘルパー
Deviseはコントローラとビューで使用されるヘルパーをいくつか作成します。ユーザの認証をコントローラに設定するには、次の before_filterを加えるだけで大丈夫です。
Devise will create some helpers to use inside your controllers and views. To set up a controller with user authentication, just add this before_filter:

```ruby
before_filter :authenticate_user!
```
ユーザがサインインしているか確認するためには、次のヘルパーを使用してください。
To verify if a user is signed in, use the following helper:

```ruby
user_signed_in?
```
現在サインインしているユーザには、次のヘルパーが使えます。
For the current signed-in user, this helper is available:

```ruby
current_user
```
次のスコープでセッションにアクセスできます。
You can access the session for this scope:

```ruby
user_session
```
ユーザがサインインした後や、アカウントの確認またはパスワードの確認後、Deviseはリダイレクトの為にスコープされたルートパスを探しにいきます。
例：:userリソースの場合、`user_root_path`が存在するときはそれを使い、ない場合はデフォルトの`root_path`が使われます。これは、ルートを定義する必要あることを意味します：

After signing in a user, confirming the account or updating the password, Devise will look for a scoped root path to redirect. Example: For a :user resource, it will use `user_root_path` if it exists, otherwise default `root_path` will be used. This means that you need to set the root inside your routes:

```ruby
root to: "home#index"
```
また、リダイレクトフックをカスタマイズするには、`after_sign_in_path_for` や`after_sign_out_path_for`を上書きします。
You can also overwrite `after_sign_in_path_for` and `after_sign_out_path_for` to customize your redirect hooks.
最後に、それぞれの環境におけるメーラーのデフォルトurlオプションを設定する必要があります。"config/environments/development.rb"のための設定は次のようになります:
Finally, you need to set up default url options for the mailer in each environment. Here is the configuration for "config/environments/development.rb":

```ruby
config.action_mailer.default_url_options = { :host => 'localhost:3000' }
```
もしdeviseモデル名に "user" ではなく "member" を使用している場合、ヘルパーは次のようにする必要があることに注意してください：
Notice that if your devise model is not called "user" but "member", then the helpers you should use are:

```ruby
before_filter :authenticate_member!

member_signed_in?

current_member

member_session
```

### モデルの設定

モデル内のdeviseメソッドは、モデルのモジュールを設定するためにいくつかのオプションを受け取ります。
例えば、暗号化アルゴリズムのコストは次のように設定します：
The devise method in your models also accepts some options to configure its modules. For example, you can choose the cost of the encryption algorithm with:

```ruby
devise :database_authenticatable, :registerable, :confirmable, :recoverable, :stretches => 20
```
:stretchesの他に、 :pepper, :encryptor, :confirm_within, :remember_for, :timeout_in, :unlock_in に加え、他の値も定義することができます。
詳細は、`devise:install`を実行したときに作成されるinitializerファイルを参照してください。

Besides :stretches, you can define :pepper, :encryptor, :confirm_within, :remember_for, :timeout_in, :unlock_in and other values. For details, see the initializer file that was created when you invoked the "devise:install" generator described above.

### ストロングパラメータ
ビューをカスタマイズするとき、フォームに新しい属性を追加することがあるとおもいます。Rails 4 はパラメータのサニタイズをモデルからコントローラに移したので、
Deviseでも同様にこの問題をコントローラで処理することになりました。
When you customize your own views, you may end up adding new attributes to forms. Rails 4 moved the parameter sanitization from the model to the controller, causing Devise to handle this concern at the controller as well.


Deviseには、パラメータの任意のセットをモデルにまで引き渡すことができる（したがってサニタイズが必要な）アクションは3つしかありません。
それらのアクション名とデフォルトで許可されるパラメータを次に挙げます：
There are just three actions in Devise that allows any set of parameters to be passed down to the model, therefore requiring sanitization. Their names and the permitted parameters by default are:

| アクション | 説明 |  
| ---------- | --- |  
| `sign_in` (`Devise::SessionsController#new`) | 認証キーのみ許容します。 (`email` 等) |  
| `sign_up` (`Devise::RegistrationsController#create`) | 認証キーに加え、 `password` と `password_confirmation`を許容します。 |  
| `account_update` (`Devise::RegistrationsController#update`) | 認証キーに加え、 `password`, `password_confirmation` , `current_password`を許容します。 |  

追加のパラメータを（遅延評価で）許容したい場合は、`ApplicationController`内に簡単なフィルターを加えることで可能になります。
In case you want to permit additional parameters (the lazy way™) you can do with a simple before filter in your `ApplicationController`:

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
To completely change Devise defaults or invoke custom behaviour, you can also pass a block:

```ruby
def configure_permitted_parameters
  devise_parameter_sanitizer.for(:sign_in) { |u| u.permit(:username, :email) }
end
```
複数のDeviseモデルがあり、それぞれのモデルについて異なるパラメータサニタイザを設定したい場合について書きます。
この場合、`Devise::ParameterSanitizer` から継承し、自分のロジックを追加することをお勧めします
If you have multiple Devise models, you may want to set up different parameter sanitizer per model. In this case, we recommend inheriting from `Devise::ParameterSanitizer` and add your own logic:

```ruby
class User::ParameterSanitizer < Devise::ParameterSanitizer
  def sign_in
    default_params.permit(:username, :email)
  end
end
```

And then configure your controllers to use it:
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
The example above overrides the permitted parameters for the user to be both `:username` and `:email`. The non-lazy way to configure parameters would be by defining the before filter above in a custom controller. We detail how to configure and customize controllers in some sections below.

### ビューの設定

We built Devise to help you quickly develop an application that uses authentication. However, we don't want to be in your way when you need to customize it.

Since Devise is an engine, all its views are packaged inside the gem. These views will help you get started, but after some time you may want to change them. If this is the case, you just need to invoke the following generator, and it will copy all views to your application:

認証機能をもつアプリケーションの素早い開発をサポートするためにDeviseは作成されましたが、
カスタマイズをする場合には、Deviseの手順に従ってもらわなければなりません。  

Deviseはエンジンなので、すべてのビューはgem内に収まっています。
それらのビューは始める時には役立つでしょうが、後々アプリケーションによって変更したくなったときはどうすればよいのでしょうか。その場合、次に紹介するジェネレータを実行して、アプリケーション側にすべてのビューをコピーする必要があります。

```console
rails generate devise:views
```
もし一つ以上のDeviseのモデルがあるとき（"User" や "Admin" 等）、Deviseはすべてのモデルに同じビューを使用していることに気付くでしょう。幸いにも、Deviseでは簡単な方法でビューをカスタマイズすることができます。 "config/initializers/devise.rb"内
で"config.scoped_views = true" を設定するだけで良いのです。

If you have more than one Devise model in your application (such as "User" and "Admin"), you will notice that Devise uses the same views for all models. Fortunately, Devise offers an easy way to customize views. All you need to do is set "config.scoped_views = true" inside "config/initializers/devise.rb".

設定し終えると、"users/sessions/new" や "admins/sessions/new"といった役割に基づいたビューを持つことができます。
スコープ内にビューが見つからない場合は、Deviseは"devise/sessions/new"にあるデフォルトのビューを使用します。
スコープされたビューを生成するために、ジェネレータを使用することも出来ます。


After doing so, you will be able to have views based on the role like "users/sessions/new" and "admins/sessions/new". If no view is found within the scope, Devise will use the default view at "devise/sessions/new". You can also use the generator to generate scoped views:

```console
rails generate devise:views users
```

### コントローラの設定

If the customization at the views level is not enough, you can customize each controller by following these steps:

ビューレベルでのカスタマイズでは十分でない場合、次の手順に従ってそれぞれのコントローラをカスタマイズすることができます。

1. 自前のコントローラを作成します。例えば、`Admins::SessionsController`の場合:

    ```ruby
    class Admins::SessionsController < Devise::SessionsController
    end
    ```
    上記の例の場合、コントローラは`app/controller/admins/` ディレクトリ内に作成される必要があることに注意して下さい
    Note that in the above example, the controller needs to be created in the `app/controller/admins/` directory.

2. ルータにこのコントローラを使うよう指示します。

    ```ruby
    devise_for :admins, :controllers => { :sessions => "admins/sessions" }
    ```

3. そして、コントローラを変更したので、`"devise/sessions"` ビューは使われません。なので、忘れずに `"devise/sessions"` を `"admins/sessions"`にコピーして下さい。

    留意して欲しいのは、Deviseはフラッシュメッセージを使用してユーザのサインインが成功したか失敗したか伝えるということです。Deviseはアプリケーションが`"flash[:notice]"` と `"flash[:alert]"`を呼び出すことを前提としています。フラッシュの全体のハッシュを表示することはしないで下さい。特定のキーのみ表示するようにし、少なくとも`:timedout`キーをハッシュから取り除いて下さい。Deviseはこのキーを幾つかの状況でで追加しますが、このキーは表示されることを意図していません。

### ルートの設定

Deviseはデフォルトのルートが設定された状態でインストールされます。それらをカスタマイズしたい場合は、
大方 devise_for メソッドを通して行えるでしょう。そのメソッドは :class_name, :path_prefix、といったようないくつかのオプションを受け取ります。I18nのためのパス名の変更も含みます。：

Devise also ships with default routes. If you need to customize them, you should probably be able to do it through the devise_for method. It accepts several options like :class_name, :path_prefix and so on, including the possibility to change path names for I18n:

```ruby
devise_for :users, :path => "auth", :path_names => { :sign_in => 'login', :sign_out => 'logout', :password => 'secret', :confirmation => 'verification', :unlock => 'unblock', :registration => 'register', :sign_up => 'cmon_let_me_in' }
```
より詳細を知りたい場合は`devise_for` ドキュメンテーションを参照することを忘れないで下さい。
[http://rubydoc.info/github/plataformatec/devise/master/ActionDispatch/Routing/Mapper:devise_for:title]
Be sure to check `devise_for` documentation for details.

より深いカスタマイズが必要な場合、例えば "/users/sign_in"の他に"/sign_in" を許可したい場合、
やるべきことは、ルータ内で通常通りルートを定義して、`devise_scope` でラップすることだけです。
If you have the need for more deep customization, for instance to also allow "/sign_in" besides "/users/sign_in", all you need to do is to create your routes normally and wrap them in a `devise_scope` block in the router:

```ruby
devise_scope :user do
  get "sign_in", :to => "devise/sessions#new"
end
```

この方法で"/sign_in" にアクセスされたとき、deviseに :user スコープを使用するように指示できます。
ルータ内で、`devise_scope`には`as`というエイリアスがあることに注意して下さい。  
[http://rubydoc.info/github/plataformatec/devise/master/ActionDispatch/Routing/Mapper:devise_scope:title]

### I18n

Devise uses flash messages with I18n with the flash keys :notice and :alert. To customize your app, you can set up your locale file:

```yaml
en:
  devise:
    sessions:
      signed_in: 'Signed in successfully.'
```

You can also create distinct messages based on the resource you've configured using the singular name given in routes:

```yaml
en:
  devise:
    sessions:
      user:
        signed_in: 'Welcome user, you are signed in.'
      admin:
        signed_in: 'Hello admin!'
```

The Devise mailer uses a similar pattern to create subject messages:

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

Take a look at our locale file to check all available messages. You may also be interested in one of the many translations that are available on our wiki:

https://github.com/plataformatec/devise/wiki/I18n

### Test helpers

Devise includes some test helpers for functional specs. In order to use them, you need to include Devise in your functional tests by adding the following to the bottom of your `test/test_helper.rb` file:

```ruby
class ActionController::TestCase
  include Devise::TestHelpers
end
```

If you're using RSpec, you can put the following inside a file named `spec/support/devise.rb`:

```ruby
RSpec.configure do |config|
  config.include Devise::TestHelpers, :type => :controller
end
```

Now you are ready to use the `sign_in` and `sign_out` methods. Such methods have the same signature as in controllers:

```ruby
sign_in :user, @user   # sign_in(scope, resource)
sign_in @user          # sign_in(resource)

sign_out :user         # sign_out(scope)
sign_out @user         # sign_out(resource)
```

There are two things that is important to keep in mind:

1. These helpers are not going to work for integration tests driven by Capybara or Webrat. They are meant to be used with functional tests only. Instead, fill in the form or explicitly set the user in session;

2. If you are testing Devise internal controllers or a controller that inherits from Devise's, you need to tell Devise which mapping should be used before a request. This is necessary because Devise gets this information from router, but since functional tests do not pass through the router, it needs to be told explicitly. For example, if you are testing the user scope, simply do:

    ```ruby
    @request.env["devise.mapping"] = Devise.mappings[:user]
    get :new
    ```

### Omniauth

Devise comes with Omniauth support out of the box to authenticate with other providers. To use it, just specify your omniauth configuration in `config/initializers/devise.rb`:

```ruby
config.omniauth :github, 'APP_ID', 'APP_SECRET', :scope => 'user,public_repo'
```

You can read more about Omniauth support in the wiki:

* https://github.com/plataformatec/devise/wiki/OmniAuth:-Overview

### Configuring multiple models

Devise allows you to set up as many Devise models as you want. If you want to have an Admin model with just authentication and timeout features, in addition to the User model above, just run:

```ruby
# Create a migration with the required fields
create_table :admins do |t|
  t.string :email
  t.string :encrypted_password
  t.timestamps
end

# Inside your Admin model
devise :database_authenticatable, :timeoutable

# Inside your routes
devise_for :admins

# Inside your protected controller
before_filter :authenticate_admin!

# Inside your controllers and views
admin_signed_in?
current_admin
admin_session
```

Alternatively, you can simply run the Devise generator.

Keep in mind that those models will have completely different routes. They **do not** and **cannot** share the same controller for sign in, sign out and so on. In case you want to have different roles sharing the same actions, we recommend you to use a role-based approach, by either providing a role column or using [CanCan](https://github.com/ryanb/cancan).

### Other ORMs

Devise supports ActiveRecord (default) and Mongoid. To choose other ORM, you just need to require it in the initializer file.

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
