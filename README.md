# Django フレームワークについて
## まずはプロジェクトを立てる
以下のコマンドを実行
```bash
django-admin startproject $name
```
`$name`の名前でプロジェクトが作られる。

```
$name/
├── manage.py
└── $name/
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    ├── asgi.py
    └── wsgi.py
```

point!!!

`test`や`Django`という名前のプロジェクト名は避けるべき。（built-in functionとコンフリクトする恐れがあるため）
さらに、htmlやcss,javascript、画像などの静的ファイルをドキュメントルートに配置するが、スクリプトファイルは別の場所に配置して、Apacheやnginxの設定にその旨を記述する。これはセキュリティ面で重要で、仮にドキュメントルートにあるとウェブ上からそのドキュメントルートに容易にアクセスができるため、脆弱性の問題が生じる

* `__init__.py`はパッケージ(モジュール（サブモジュール）が複数含まれるディレクトリ)をインポートしたタイミングで実行される。なお、サブモジュールは、サブモジュールをインポートしたタイミングで実行される。例えば、サブモジュールを直接インポートする場合は、そのサブモジュール自体と、そのサブモジュールが属するパッケージの`__init__.py`が実行される。

=====これ以下はよくわかってないので、おいおい=======

* `setting.py`
    * settings.py に含まれる主な要素は以下の通りです。
DATABASES：Django が使用するデータベースに関する設定を定義します。
INSTALLED_APPS：Django で使用するアプリケーションを定義します。各アプリケーションは、Python パッケージとしてインストールされている必要があります。
MIDDLEWARE：Django で使用するミドルウェアを定義します。ミドルウェアは、リクエストとレスポンスの間で動作する処理を指します。
STATIC_URL：静的ファイルの URL を定義します。これは、CSS、JavaScript、画像などの静的ファイルを指します。
MEDIA_URL：ユーザーがアップロードしたファイルの URL を定義します。
TEMPLATES：Django で使用するテンプレートエンジンを定義します。これは、HTML テンプレートを使用して動的なコンテンツを生成する際に使用されます。
SECRET_KEY：Django で使用される暗号化のためのキーを定義します。
なお、settings.py ファイルに直接記述することもできますが、機密性の高い情報や環境に依存する設定については、環境変数から取得するなど、外部から取得することが推奨されます。

* urls.py：DjangoアプリケーションのURLパターンを定義するファイル。
* asgi.py：ASGI(Webサーバーとアプリケーション間のインターフェース)サーバーを実行するためのエントリーポイント。
* wsgi.py：WSGI(Webサーバーとアプリケーション間のインターフェース)サーバーを実行するためのエントリーポイント。

## ローカルサーバー起動

プロジェクトディレクトリ内で以下を実行
```
python manage.py runserver
```

`http://127.0.0.1:8000/`がデフォルト。

最初なんで、
```
You have 18 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
```
こんなのでても無視でおk

```
python manage.py runserver 8080
```
でポート番号を変えれる。

```
python manage.py runserver 0.0.0.0:8000
```
でIPをしていできる。
この辺はよくわからん。笑
ちゃんと勉強せんと。。。

## Appを実装する

### AppとProjectの違い
Appはブログシステムや、簡単な投票アプリ？のことを指す。一方でプロジェクトは複数の設定ファイルやアプリケーションによって構成される特定のウェブサイトを指す。

```
python manage.py startapp $APP_NAME
```

で作成する。

### view.pyについて

`view.py`で実装されるビュー関数はHTTPリクエスト（POST,GET,PUT,DELETEなど）を受け取って、HTTPレスポンスを返す関数。
HTTPレスポンスはHTMLページ、JSONデータ、ファイルダウンロードなどを返すことができる。
例えば、`GET "/"`すると`index.html`を返すようにビュー関数を組んだりする。

### urls.pyについて

デフォルトではアプリケーションの作成時に`urls.py`は作成されない（プロジェクトには作成される）
アプリケーション特有の`urls.py`を処理する必要に応じて、作成する。

`（上について。必要に応じて、とはどんな時だろうか？）`

URLディスパッチャがURLパターンとビュー関数をマッピングするためにつかう。

URLディスパッチャはMVCモデル（モデル・ビュー・コントローラーモデル）のルーターの役割を担う。

URLディスパッチャは、以下の手順に従ってHTTPリクエストを処理する。

1. HTTPリクエストが受信される。
2. URLパターンがプロジェクトのurls.pyファイルで定義されたパターンに一致するかを確認する。
3. 一致する場合、対応するビュー関数を呼び出す。
4. 一致しない場合、アプリケーションのurls.pyファイルのパターンに一致するかを確認する。
5. 一致する場合、対応するビュー関数を呼び出す。
6. 一致しない場合、404エラーを返す。

### `include()`について
この関数は他のパッケージのURLconfを参照する時につかう。
プロジェクトのurls.pyは以下の通り。

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("polls/", include("polls.urls"))
    path('admin/', admin.site.urls),
]
```

`localhost:8000/polls/`にアクセスがあった時に、`polls.urls`の名前空間で指定されたURLパターンを参照する。すなわち、下記の`polls`appのurlpatternを参照する。その結果、views.pyの`view.index`関数が働き、HTTPresponseが返る。

#### path関数について
これは、URLconfでURLパターンとビュー関数を結びつけるために使用される。
つまり、特定のURL（第一引数で指定する）に対して、どのビューを呼び出すかを定義する。

チュートリアルのスクリプトを確認しよう。

```python
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name="index")
]
```
のようになっている。

ここではpath()の第一引数に""（＝"/"）を指定している。すなわち、"/"というURLにリクエストが送信された時に、`view.index`が参照される。（デフォルトでは`GET`メソッドでリクエストが送信される。指定したい場合は`method=["POST"]`のように指定する）

