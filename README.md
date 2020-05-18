# EverydayRailsRSpecによるRailsテスト入門

* Rails 5.1
* Ruby 2.4
* RSpec 3.6

## 2. RSpec セットアップ 

RSpec をアプリにセットアップする手順

1. `rspec-rails` Gem をBundler でインストール
2. config/database.yml に追記 > test 用DB 作成
3. `rails g rspec:install` でRSpec インストール
4. binstub テストランナー インストール(spring-commands-rspec Gem)
5. `rails g` でspec ファイルも生成するように設定(config/application.rb)
6. minitest で使用するtest ディレクトリは使用しない場合は削除してok

### view, UI 周りのテスト

* view のテスト作成は工数が掛かる為行わない
* UI 周りのテストは基本的に統合テストで行う

## 3. model spec

model spec(単体テスト) の書き方

### spec ファイル内の用語

* describe: テストの対象
* context: 条件
* it, example: 期待する出力
* pending: test 内容が書かれていない保留中のtest

### matcher(マッチャ)

* 期待値とアウトプットを比較し一致, 不一致を返すオブジェクト
* `to`, `eq` など複数有り
 
### should

* 古いRSpec の構文
* expect に変更になった
* 現在は非推奨

### expectation(エクスペクテーション) 

* example ブロックに記述される期待するアウトプット
* 判定を反転する`_not` 付きのmatcher オブジェクトが用意されている
* `not_` と`_not` ２つの記述がある(標準的なのは`_not`)

### テストの機能チェック

テストが正常にパスしたらテスト自体が機能しているかテスト毎に毎回確認する

* テストがパスしたら一部をコメントアウトしてテスト実行
* expectation を反転(`_not`) を付けてテスト実行

### spec ファイルのDRY 化

* describe とcontext を使い分けシンプルに記述
  * describe: 外枠, テストの要件
  * context: 内枠, 条件毎のテストを囲むイメージ
* before フックを使いテストデータを自動セットアップ 

### spec 作成時の必須事項

* 期待する結果は能動系で記述
* 起きてほしい事, 起きてほしくない事どちらもテストする
* 境界値(文字数など)をテストする
* DRY化

## 4. テストデータ作成

FactoryBot Gem を使ったテストデータ作成方法

### FactoryBot

* テストデータ作成サポートGem
* Rails 標準のFixture の代替で使用する
* Fixture に比べ複雑な制御が可能

### FactoryBot 準備

1. `factory_bot_rails` をBundler でインストール
2. config/application.rb に自動的にファクトリを作成するように設定
3. `rails g factory_bot:model model_name` でmodel にファクトリを追加 
4. `FactoryBot.create(:model_name)` でspec ファイルからテストデータを作成, 利用する

### FactoryBot createメソッド, buildメソッド

* create: インスタンスをDB に保存する
* build: インスタンス作成するがDB に保存はしない, 動作が軽量 

### FactoryBot sequence

* ファクトリ作成時に属性に連番を振る場合などに使用する
* alias を作成する事でファクトリで作成されるテストデータの件数をコントロールする事ができる

### FactoryBot 継承

* factory 内でネストし、差分だけ記述する事で親factory の属性を継承できる
* ネストせずに分離して記述することもできる

### FactoryBot trait

属性値 を抽象化しメンテナンスしやすくなる

https://qiita.com/ogomr/items/935da1072301ddc1aeaf

### FactoryBot コールバック

ファクトリがオブジェクト生成後(create, build, stub)にアクションを追加できる

https://github.com/thoughtbot/factory_bot/blob/master/GETTING_STARTED.md#callbacks

### FactoryBot 注意点

* 大量, 予期しないテストデータが生成されてしまう
* コールバックはtrait の中でセットアップする(ファクトリ呼び出し時に毎回実行されないようにする為)
* 可能な限りbuild を使う(パフォーマンスセーブ)

## 5. controller spec

controller spec(機能テスト) の書き方

### contorller spec テストの流れ

1. `rails g rspec:contoroller contoroller_name` でspec生成
2. レスポンス, レスポンスコード(200, 404など)をテスト
3. テストがパスするようにコードを修正
4. エクスペクテーションの反転, 機能のコメントアウトなどでテストが機能しているか確認

### controller spec 種類

* ログインユーザ/ゲストユーザ 機能制限のテスト
* 期待する/期待しない レスポンスのテスト
* 期待する/期待しない フォーマットで出力されるのかテスト


### controller spec 注意点

* 現在ではあまり使われない(model, view に問題があっても発見できない)
* 対象となる機能の単体テストとして最低限で使用する

## 6. feature spec

* UI テスト, 統合テスト, 受け入れテスト, システムテスト
* model とcontoroller の動作を確認する
* model spec, controller spec の部分的なテストの穴を埋める
* 実際のユーザがリアルなやり取りを行うのをシュミレートしてアプリ全体の機能をテストする
* 単体テストに比べると処理に時間が掛かる 
* RSpec 3.7, Rails 5.1 以降はsystem spec に変更された(詳細は付録A)

### Capybara

* UI テストの為のRubyフレームワーク(Gem) 
* RSpec 以外のテストフレームワーク(Minitestなど)でも使用可
* ブラウザの動きをシュミレートする
* ヘッドレスブラウザ(UI を持たない)ブラウザを使ってテストを実行する

### Capybara インストール

* rails 5.1 以降は標準Gem(Gemfileのコメントアウトを外すだけ)
* test 環境のみに追加する(group: test)
* インストール後にrails_helper.rb にライブラリを追加(require)

### feature spec 流れ

1. `rails g rqpec:feature controller_name` でspecファイル生成
2. ユーザをFactoryBot で生成
3. Capybara でユーザの動きを明示的にシュミレート(シナリオ作成)
4. 期待する/期待しない 動作のテスト

### RSpec でCapybara 使用時のポイント

* RSpec はCapybara 使用を前提にしている
* `it` の代わりに`scenario` を使う

### Capybara DSL

* ブラウザ上の操作をシュミレートするCapybara のメソッド
* `visit`, `click_link`, `fill_in` など多数

https://github.com/teamcapybara/capybara#the-dsl

### save_and_open_page

* Capybara DSL
* ブラウザで生成されたHTML を生成できる
* 生成されたHTML の保存先はスタックトレースに表示される
* launchy Gem をインストールする事で保存されたHTML を自動で開ける(トリガーは`save_and_open_page` を呼び出した時)
* spec がパスしたら`save_and_open_page` を削除する(コミットしない)

### Launchy Gem

`save_and_open_page` で任意のページのHTML を自動でブラウザで開くGem

https://github.com/copiousfreetime/launchy

### JavaScript feature spec

* Capybara はJavaScript の実行はシュミレートできない
* `js: true` でCapybara にJavaScript 使用可能なドライバを使わせる
* ドライバはselenium-sebdriver を使用
* ドライバ, ブラウザの指定はspec_helper.rb, rails_helper.rb, または別の設定ファイルを作成する
* spec の実行に非常に時間が掛かる為、必要最低限の使用にする
* Capybara の設定ファイルで実行時間の待ち時間を設定できる

### JavaScript feature spec 流れ

1. webdrivers Gem でドライバ,ブラウザインターフェイスを用意(selenium-webdriver, ChromeDriver)
2. 設定ファイルで読み込み
3. spec 実行

### selenium-webdriver Gem

* JavaScript が実行可能なドライバ(Selenium) をインストールするGem 
* webdrivers Gem を使う場合はインストール不要(webdrivers に同梱されている)
* feature spec でJavaScript の動作テストをする際に使用
* spec_helper.rb 等で設定が必要

### webdrivers Gem

* ブラウザインターフェイス(ChromeDriver), ドライバ(selenium-webdriver) を一緒にインストールするGem
* 別途でselenium-webdriver をインストール必要がない

https://github.com/titusfortner/webdrivers

### PhantomJS

* ヘッドレスドライバのひとつ
* 現在は少なくなっている
* poltergeist Gem のインストールが必要
* Capybara への設定が必要

https://phantomjs.org/

### ヘッドレスドライバ

* spec 実行中にブラウザウィンドウを開きたくない環境(CI 環境) で使用
* Chrome 59.0 以上が必要
* Selenium 以外にもPhantomJS など選択肢有り

## 7. request spec

* API のテストで使用
* spec/requests ディレクトリに配置
* Capybara は使わない
* HTTPメソッド(get, post, delete等) を使う
* controller spec よりもrequest spec を使うようにする

### resquest spec 流れ

1. `rails g rspec:request api_names` でspec を生成
2. ファイル名を複数形から単数形に変更する(api_namesからapi_name)
3. FactoryBot でAPI のインスタンスを生成
4. HTTP メソッド(ルーティング名)別にリクエストのレスポンスをテストする
5. Devise などのAPI をテストする際にはrails_helper.rb に別途設定が必要

https://github.com/heartcombo/devise/wiki/How-To:-sign-in-and-out-a-user-in-Request-type-specs-(specs-tagged-with-type:-:request)

### request spec と controller spec の違い

* controller spec は`get '/users'` のようにメソッドを呼ぶ, request spec は`users_url` のようにルーティング名を呼びエンドポイントを指定する
* request spec はエンドポイントを呼ぶ事でcontoller に依存しない
* request spec はAPI のテストが可能
* request spec は複数のcontoller, 複数のセッションを通したリクエストのテストが可能

### エンドポイント

API の各機能に割り当てられたルーティング, URI を指す言葉

|エンドポイント|機能|
|---|---|
|POST /siginup|Signup|
|GET /posts|List all posts |
|DELETE /posts/:id|Delete a post|

### エンティティ

実体を指す言葉

https://wa3.i-3-i.info/word11594.html

## 8. DRY

spec ファイルをDRY 化する方法について

### DRY 化する方法

* サポートモジュールに切り出す(spec/support/ にファイル作成)
* Devise などのAPIのヘルパーメソッドが用意されている場合は優先的に使う(rails_helper.rb に設定)
* インスタンスを生成する際の`before` を`let` に変更する(インスタンス変数を使わない事に注意)
* shared_context に切り出す(spec/support/contexts/ にファイル作成)
* カスタムマッチャを作成する(spec/support/matchers/ にファイル作成)
* `aggregate_failures` を使い複数のテストをまとめる
* 「シングルレベルの抽象化を施したテスト」を参考に抽象化する(https://thoughtbot.com/blog/acceptance-tests-at-a-single-level-of-abstraction)

### let, let!

* インスタンスが呼ばれた後に処理を読み込む(遅延読み込み)
* before に比べて必要な分だけ処理するので動作が軽量になる
* let で生成したインスタンスをインスタンス変数に代入する必要がない
* `let!` は遅延読み込みされない
* `let` と`let!` は見分けにくいのでbefore を使う事も見当する

### aggregate_failures

* 複数のエクスペクテーションを続けて実行できる
* RSpec 3.3 から追加された
* 失敗するエクスペクテーションにだけ有効に働く(一般的な実行環境には働かない)

## 9. 高速なspec, 速く書けるspec

高速に処理できるspec と簡素的なspec の書き方について

### spec リファクタリングのヒント

* `subject` を使いテスト対象物をまとめる(再利用でき)
* `is_expected` でワンライナーで記述
* Shulda Machers Gem を使い効率的に記述
* モック, スタブを使い処理を速くする
* タグを使い特定のテストだけを実行させる
* `skip`を使い不必要なテストを実行させない(以前は`pendeng`が使われてた)
* ParallelTests Gem を使い並列に処理する
* binstub 経由でSpring を使いRails を切り離して実行する

### Shulda Machers Gem

* 豊富なマッチャを使えるようになるShulda から切り出されたGem
* shulda 自体は開発は終わっている
* 使用するRails のバージョンによって使うブランチを使用する必要がある
* `bundle install` 後にrails_helper.rb に設定が必要

https://github.com/thoughtbot/shoulda-matchers

### モック(mock)

* 本物のオブジェクトのふりをするオブジェクト
* test doubles とも呼ばれる
* Factory, PORO などで作成される
* 外部API のテスト等で使用される
* `double` を使って生成される 
* 基本的には「自分で管理していないコードをモック化するな(Don't mock what you don't own)」に従う
* 基本的にはオブジェクトやFactory を使うだけでテストは十分なので無理に使う必要はない


https://qiita.com/jnchito/items/640f17e124ab263a54dd

### スタブ(stub)

* 呼び出されるとテスト用に本物の結果を返すダミーメソッド
* DB やネットワークを使う処理が対象になる事が多い
* 広い意味でモックと同意義
* `allow` を使って生成される
 
### PORO

* Plan Old Ruby Object
* ActiveRecord などを継承していないすっぴんのRuby オブジェクト

### Mocha

モック, スタブを作成するテストフレームワーク Gem

https://github.com/mochajs/mocha

### テストダブル

テストダブルとは、テスト対象が依存する他のクラスやオブジェクトの代替品として振る舞ってくれるオブジェクトのこと(Double=影武者)

https://qiita.com/k5trismegistus/items/94599074a961074d9237

### タグ

* 特定のテストの実行, スキップなどを行う
* テストスピードを上げたい場合に使用する

### skip, pending の違い

`skip`: spec 実行時にスキップしたことが表示される
`pending`: spec 実行時にそのまま実行される, テスト失敗時にpending(保留中)が表示される

### ParallelTests Gem

* テストを分割し並列に実行する事で処理を高速にする
* 遅い処理が見えなくなってしまう可能性があるので基本的なリファクタリングを行った後で使用する

https://github.com/grosser/parallel_tests

## 10. その他の機能に関するテスト

以下の内容に対するテスト

* ファイルアップロード
* バックグラウンド処理
* メール送信機能
* 外部webサービス

### VCR Gem

* テスト中に投げたHTTP リクエストをカセット(cassete) に記録して2回目以降のテスト実行時間を短縮するGem
* Video Cassette Recorder の略
* API の呼び出しを最小限に抑える
* テスト後にリクエストから返ってきたパラメータはspec/cassettes ディレクトリに保存されたYAML ファイルを使いHTTP リクエストとレスポンスをスタブ化する
* API の使用変更されていた場合に古いにcassetes を使っているかもしれないのでGit から追跡を除外するか自動的にcassetes を再記録する方法をとる必要がある
* API に個人情報等の機密情報をcassetes に含めないように注意する

https://github.com/vcr/vcr

### WebMock Gem

* HTTP をスタブ化するライブラリ
* VCR 使用時にはこのライブラリを使っている 

https://github.com/bblimke/webmock

## 11. テスト駆動開発

RSpec を使用したテスト駆動開発の導入方法

### テスト駆動開発の導入の流れ

1. feature spec シナリオのみを追記, 機能追加前に全てパスするか確認
2. feature spec アウトライン作成 新しいシナリオのスタブを追加
3. feature spec シナリオにステップをコメントのみ追加
4. feature spec コメントの機能テストを追加(実装中のテストのみを実行させたい場合はタグを追加する, commit 時にはタグを外す)
5. feature spec spec 実行し全て失敗する事を確認
6. feature spec spec がパスするように機能を実装(テストを前進させる必要最小限のコードのみを書く事を意識する)
7. controller spec 異常系の操作のテストを追加(正常系はfeature spec で確認済みの為)

### テスト駆動開発の考え方

* 正常系は実装時に確認できる, 異常系は機能実装後にテストを追加し精度を高める
* 最小限のコードのみを実行させる事で次にどんなコードを書けばいいのかspec が教えてくれる
* 高レベルのテスト(API, view): request spec(場合によってfeature spec)
* 低レベルのテスト(機能): model spec(場合によってcontoller spec)
* レッド, グリーン, リファクタのサイクルで進める
* リファクタリングは小さく変更してテストは常にパスするようにする(レッドになっても一時的なものにする)


## 12. アドバイス

アドバイスや参考URL

### RSpec のアドバイス

* コードを書く前にspec を書く
* 作業中のタスクを意識して進める(チェックリストを使い漏れがないように)
* 仮のコード(スパイク)を作る際はテスト用ブランチを切って試すと理解が深まる
* テストと実装はできるだけセットで行う
* feature spec(統合テスト) > controller spec(機能テスト) > model spec(モデルテスト) の順で作れるようにする(outside-in)

### 参考URL

* [Relish](https://relishapp.com/rspec)
* [Better Specs](http://www.betterspecs.org/)
