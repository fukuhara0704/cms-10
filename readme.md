# Windows10 home + Docker + VScode + Python + Django 環境構築 手順

* Dockerを起動
* VScodeを起動
* ターミナルを起動
* 開発用のディレクトリを作成
* Dockerfile, docker-compose.yml, requirements.txtを作成
* イメージとコンテナをビルド
```
docker-compose build
```
* 1. Djangoプロジェクトを作成
```
docker-compose run web django-admin.py startproject [任意のプロジェクト名] .
```
* 例 docker-compose run web django-admin.py startproject config .

```
django-admin startproject config
```

* 2. コンテナを起動
```
docker-compose up -d
```
* [任意のプロジェクト名]/setting.pyのALLOWED_HOSTS = []をALLOWED_HOSTS = ['*']に変更
* 3. dockerのIPでブラウザ確認

# アプリケーションの作成

* docker exec -it [コンテナ名] bashでコンテナ内に入って下のコマンドを実行
```
python manage.py startapp  [任意アプリケーション名]
```

# テンプレート呼び出し
*  [任意アプリケーション名]/views.pyを開く
```
from django.shortcuts import render

def index(request):
    return render(request, 'index.html'))
```
*  [任意アプリケーション名]/urls.pyを作成
*  [任意アプリケーション名]/urls.pyを開く
```
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index),
]
```
*  [任意アプリケーション名]/templates/index.htmlを作成
```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>web_app</title>
</head>

<body>
    <p>test</p>
</body>

</html>
```
* [任意のプロジェクト名]/urls.pyを修正
```
from django.contrib import admin
from django.urls import include, path #追加 inculde

urlpatterns = [
    path('[任意アプリケーション名]/', include('[任意アプリケーション名].urls')), #追加
    path('admin/', admin.site.urls),
]
```
* [任意のプロジェクト名]/setting.pyを修正
```
INSTALLED_APPS = [
    '[任意アプリケーション名]', #追加
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```
* dockerのIP/[任意アプリケーション名]/でブラウザ確認
# CSSを読み込み
* [任意のプロジェクト名]/setting.pyを修正
```
STATIC_URL = '/static/'
STATICFILES_DIRS = [                    #追加
    os.path.join(BASE_DIR, "static"),
]
```
* templateファイルの修正
```
{% load static %}   #追加
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="{% static '[staticフォルダ内のファイル名を指定]' %}"> #追加
    <title>web_app</title>
</head>

<body>
    <p>test</p>
</body>

</html
```
# 「browser-sync」の導入
* browser-syncはnpmを使って導入するので、事前に「node.js」をインストールしておきます。
* browser-syncのインストール
* CLIにて、インストールしたいプロジェクトのディレクトリまで移動します。
```
cd プロジェクトディレクトリ
```
* npmでbrowser-syncを開発用でローカルインストール。
```
npm install browser-sync --save-dev
```
* インストールが完了した後にbrowser-syncのコンフィグファイルを作成します。
```
npx browser-sync init
```
* 「bs-config.js」を編集。
```
"files": ["code/*.html", "code/static/*.css"],
"proxy": 'http://192.168.99.100:8000',
```
* npmの「package.json」を作成
```
npm init
```
* npmの「package.json」を編集します。”scripts“の項目に以下を追記します。
```
"bs": "browser-sync start --config bs-config.js"
```
* これで以下コマンドを実行することでbrowser-syncが起動します。
```
npm run bs
```

# DB操作
* modelにカラムの作成または変更を行った際は
```
python manage.py makemigrations

python manage.py migrate
```

# Herokuにデプロイ
* codeフォルダ内にProcfile, requirements.txt, runtime.txtを作成

* Procfile
```
web: gunicorn config.wsgi --log-file -
```

* requirements.txt
```
appdirs==1.4.4
certifi==2020.6.20
chardet==3.0.4
defusedxml==0.6.0
distlib==0.3.1
dj-database-url==0.5.0
Django==2.2.15
django-allauth==0.41.0
django-heroku==0.3.1
django-widget-tweaks==1.4.8
filelock==3.0.12
gunicorn==20.0.4
idna==2.10
oauthlib==3.1.0
pipenv==2020.6.2
psycopg2==2.8.6
python3-openid==3.2.0
pytz==2020.1
requests==2.24.0
requests-oauthlib==1.3.0
six==1.15.0
sqlparse==0.3.1
urllib3==1.25.10
virtualenv==20.0.25
virtualenv-clone==0.5.4
whitenoise==5.2.0
```

* runtime.txt
```
python-3.8.3
```

* setting.py 内を変更
```
 #追加
import django_heroku

# 変更
ALLOWED_HOSTS = ['*']
↓
ALLOWED_HOSTS = ['127.0.0.1','.herokuapp.com']


# 追加
MIDDLEWARE = [
    'whitenoise.middleware.WhiteNoiseMiddleware',
]

# 追加
django_heroku.settings(locals())
STATIC_ROOT='staticfiles'
```

*