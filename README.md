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

===============================終了================================
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

## データベースについて

`$PROJECT_NAME/settings.py`
にデータベース設定を書き込む。デフォルトではPythonはSQLiteをデータベースとして用いる。
その他のデータベースを用いることもできるが、その場合は`settings.py`の`ENGINE`にその旨を記述する。

`INSTALLED_APPS`はデフォルトでインストールされているものが存在する。
以下を参照。（よくわからん）
`admin`は管理画面を生成するAPPなのかな？
```
django.contrib.admin – The admin site. You’ll use it shortly.
django.contrib.auth – An authentication system.
django.contrib.contenttypes – A framework for content types.
django.contrib.sessions – A session framework.
django.contrib.messages – A messaging framework.
django.contrib.staticfiles – A framework for managing static files.
```

`python manage.py migrate`

を実行する。マイグレーションが実行される、、、？
なぜ。笑

ひとまず、データベースのテーブル定義は`model`によって行う。`model`とは、pythonのクラスのことであり、これによってテーブルの内容を定義する。

モデルをデータベースのテーブルにマッピングする。
モデル中で定義した、フィールドと呼ばれる属性がテーブルのカラムに対応する

以下にその対応関係を記す。

| Database | python |
|----------|--------|
|テーブル|モデル（クラス）|
|カラム|フィールド（属性）|

このモデルの変更をした場合、マイグレーションによってこのモデルの変更を適用する。
マイグレーションにはマイグレーションファイルが必要で、

`python manage.py makemigrations`
によって作成できる。


以下の例を見てみよう。

```python
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```
モデルとして、`Question`や`Choice`が定義されている。
これらのモデルは`models.Model`を継承している。
これはなんだろうか。

これは`django.db.models.Model`のサブクラスとして定義するという意味である。

このクラスを継承することで、Djangoは自動的に、テーブルの作成、更新、削除などのデータベース操作を行うために必要なコードを生成する。

上記の例では、`Question`や`Choice`は`models.Model`を継承しているため、Djangoがこれらのクラスをデータベースのテーブルとして認識する。そして、`question_text`や`pub_date`といったフィールド名に応じてカラムをせいせいします。

### modelをActivateする
以下を追記します。

```python
INSTALLED_APPS = [
    'polls.apps.PollsConfig', #追記
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

`python manage.py makemigrations polls`

コマンドで、マイグレーションファイルを作成します。

すると、マイグレーションファイルが生成されます。

続いて以下のコマンドを実行します。

`python manage.py sqlmigrate polls 0001`

これは0001に対応するマイグレーションが実行された時に、実行されるSQL文をコンソールに吐き出します。
これで、実際に行われているものが確認できます。

まとめると、

* Change your models (in models.py).
* Run python manage.py makemigrations to create migrations for those changes
* Run python manage.py migrate to apply those changes to the database.

という手順で、migrateを実行します。

### インタラクティブシェルでデータベース登録を試す

`python manage.py shell`
を実行。

`python`のみで実行するinteractive shellでなく、なぜこれを実行するのかというと、

`DJANGO_SETTINGS_MODULE`環境で実行できるようにするため。
これにより、Djangoアプリケーションのモデルやビュー、テンプレートなどの構成要素を操作することができます
Djangoでの開発を行う上では、このようにすると良いでしょう。(?)

以下で確認してみましょう。

```python
>>> from polls.models import Choice, Question  
# Import the model classes we just wrote.

# No questions are in the system yet.
>>> Question.objects.all()
<QuerySet []>

# Create a new Question.
# Support for time zones is enabled in the default settings file, so
# Django expects a datetime with tzinfo for pub_date. Use timezone.now()
# instead of datetime.datetime.now() and it will do the right thing.
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

# Save the object into the database. You have to call save() explicitly.
>>> q.save()

# Now it has an ID.
>>> q.id
1

# Access model field values via Python attributes.
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=datetime.timezone.utc)

# Change values by changing the attributes, then calling save().
>>> q.question_text = "What's up?"
>>> q.save()

# objects.all() displays all the questions in the database.
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>]>
```



さらにmodels.pyを次のように書き換える。

```python

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

    def __str__(self):
        return self.question_text

    # pub_dateはpublishされた日付、今の時点から１日前の時間をさっぴく。これよりも新しい日付ならば、Trueを、古い日付ならばFalseを返す。
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

    def __str__(self):
        return self.choice_text
    
```

もう一度、`python manage.py shell`を立ち上げる

下記を実行。（よくわからん）
```python
>>> from polls.models import Choice, Question

# Make sure our __str__() addition worked.
>>> Question.objects.all()
<QuerySet [<Question: What's up?>]>

# Django provides a rich database lookup API that's entirely driven by
# keyword arguments.
>>> Question.objects.filter(id=1)
<QuerySet [<Question: What's up?>]>
>>> Question.objects.filter(question_text__startswith='What')
<QuerySet [<Question: What's up?>]>

# Get the question that was published this year.
>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
<Question: What's up?>

# Request an ID that doesn't exist, this will raise an exception.
>>> Question.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Question matching query does not exist.

# Lookup by a primary key is the most common case, so Django provides a
# shortcut for primary-key exact lookups.
# The following is identical to Question.objects.get(id=1).
>>> Question.objects.get(pk=1)
<Question: What's up?>

# Make sure our custom method worked.
>>> q = Question.objects.get(pk=1)
>>> q.was_published_recently()
True

# Give the Question a couple of Choices. The create call constructs a new
# Choice object, does the INSERT statement, adds the choice to the set
# of available choices and returns the new Choice object. Django creates
# a set to hold the "other side" of a ForeignKey relation
# (e.g. a question's choice) which can be accessed via the API.
>>> q = Question.objects.get(pk=1)

# Display any choices from the related object set -- none so far.
>>> q.choice_set.all()
<QuerySet []>

# Create three choices.
>>> q.choice_set.create(choice_text='Not much', votes=0)
<Choice: Not much>
>>> q.choice_set.create(choice_text='The sky', votes=0)
<Choice: The sky>
>>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)

# Choice objects have API access to their related Question objects.
>>> c.question
<Question: What's up?>

# And vice versa: Question objects get access to Choice objects.
>>> q.choice_set.all()
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
>>> q.choice_set.count()
3

# The API automatically follows relationships as far as you need.
# Use double underscores to separate relationships.
# This works as many levels deep as you want; there's no limit.
# Find all Choices for any question whose pub_date is in this year
# (reusing the 'current_year' variable we created above).
>>> Choice.objects.filter(question__pub_date__year=current_year)
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

# Let's delete one of the choices. Use delete() for that.
>>> c = q.choice_set.filter(choice_text__startswith='Just hacking')
>>> c.delete()
```

## admin画面を作成する

`python manage.py createsuperuser`を実行し、

```
Username: admin
Email address: admin@example.com
Password: **********
Password (again): *********
```
を入力。

`python manage.py runserver`を実行

admin.pyに以下を追記.
```

```
