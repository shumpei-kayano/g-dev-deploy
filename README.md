## フォルダ構造
```
- project_root/
  |- Dockerfile
  |- docker-compose.yaml
  |- nginx/
  |  |- nginx.conf
  |- src/
  |  |- manage.py
  |  |- ...
```

## 学生用プロジェクト作成手順
1. ローカルにgit cloneする
2. srcディレクトリ内のサンプルを削除し、チームのプロジェクトを作成する
    
    ```bash
    django-admin statproject TeamA
    ```
    
3. Dockerfileの設定を変更する（2箇所）
プロジェクト名に変更する
4. settings.pyの編集
    
    ```python
    import os
    # debug_toolbarの設定
    import mimetypes
    mimetypes.add_type("application/javascript", ".js", True)
    ```
    
    ```python
    INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'debug_toolbar', # 追加
        'sass_processor', # 追加
        'django_extensions', # 追加
        'django_cleanup' # 追加
        'django.contrib.sites', # 追加
        'allauth', # 追加
        'allauth.account', # 追加
    ]
    ```
    
    ```python
    MIDDLEWARE = [
        'django.middleware.security.SecurityMiddleware',
        'django.contrib.sessions.middleware.SessionMiddleware',
        'django.middleware.common.CommonMiddleware',
        'django.middleware.csrf.CsrfViewMiddleware',
        'django.contrib.auth.middleware.AuthenticationMiddleware',
        'django.contrib.messages.middleware.MessageMiddleware',
        'django.middleware.clickjacking.XFrameOptionsMiddleware',
        'debug_toolbar.middleware.DebugToolbarMiddleware', # 追加
    ]
    
    INTERNAL_IPS = ['127.0.0.1', '::1', 'localhost', '0.0.0.0'] # 追加
    ```
    
    ```bash
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': 'o-hara_db',
            'USER': 'admin',
            'PASSWORD': 'o-hara',
            'HOST': 'mysql_db', # dbのコンテナ名
            'PORT': '3306',
        }
    }
    ```
    
    ```python
    # 言語設定
    LANGUAGE_CODE = 'ja'
    
    TIME_ZONE = 'Asia/Tokyo'
    ```
    
    ```python
    # 静的ファイルの設定
    STATIC_URL = '/static/'
    STATIC_ROOT = '/usr/share/nginx/html/static/'
    ```
    
    ```python
    # 画像ファイルのアップロード先の設定
    MEDIA_URL = '/media/'
    MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
    ```
    
    ```python
    # debug_toolbarの設定
    def show_debug_toolbar(request):
        return request.META.get('REMOTE_ADDR') in settings.INTERNAL_IPS
    
    DEBUG_TOOLBAR_CALLBACK = show_debug_toolbar
    
    DEBUG_TOOLBAR_CONFIG = {
        "SHOW_TOOLBAR_CALLBACK": lambda request: True,
        'STATIC_URL': '/debug_toolbar/',
    }
    
    # allauth用
    SITE_ID = 1
    ```
    
5. urls.pyの編集
    
    ```python
    from django.contrib import admin
    from django.urls import path, include
    from django.conf import settings
    from django.conf.urls.static import static
    
    urlpatterns = [
        path('admin/', admin.site.urls),
    ]
    
    # 画像のURLを追加
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
    
    # debug_toolbarの設定
    if settings.DEBUG:
        import debug_toolbar
        urlpatterns += [
            path('__debug__/', include(debug_toolbar.urls)),
        ]
    ```
    
6. コンテナの作成して検証
    
    ```python
    docker compose up --build -d
    ```
    
7. githubへ公開
8. メンバーの招待

アプリケーションは http://localhost/ でアクセス可能
phpMyAdminは http://localhost:8080/ でアクセス可能

## コンテナ作成手順（学生）
docker desktopをインストールしていることが前提です。

[Windows に Docker Desktop をインストール — Docker-docs-ja 19.03 ドキュメント](https://docs.docker.jp/docker-for-windows/install.html)

1. リポジトリからurlをコピーする    
2. 任意のディレクトリ（ドキュメントなど）をVSCodeで開く
3. ターミナルを起動し、以下を入力
    
    ```bash
    git clone GitHubからコピーしたURL
    ```
    
4. クローンされたディレクトリを開き直す（Dockerfileがあるディレクトリ）
5. DockerDesktop（Dockerエンジン）が起動した状態で以下をターミナルで入力
    
    ```bash
    docker compose up --build -d
    ```
    
6. localhostでアクセスできます。

**コンテナ起動後**

srcディレクトリとコンテナ内がマウントされて同期しています。

mainブランチではソースが全くないので進捗のあるブランチにチェックアウトしてください。

WEBサーバーにNginx、アプリケーションサーバーにGunicorn、DBにMysqlで構成しています。

**静的ファイルについて**

Nginxに静的ファイルを送らないと反映されません。

gunicornコンテナに入ります。

```bash
docker exec -it django_gunicorn /bin/bash
```

静的ファイルをNginxに送ります。

```bash
python manage.py collectstatic --noinput
```

**サンプルデータについて**

シーダーファイルがないのでサンプルデータがありません。

ログイン時はサンプルユーザーを作成してください。

gunicornコンテナに入ります。

```bash
docker exec -it django_gunicorn /bin/bash
```

スーパーユーザーを作成します。

```bash
python manage.py createsuperuser
```

## gitingnoreの作成
キャッシュファイルをgit管理から外します。
1. ルートディレクトリ（Dockerfileがあるディレクトリ）に.gitignoreファイルを作成します。
2. 以下を記述して保存する。 
```
.gitignore
*.pyc
__pycache__/
```

## Sassの設定方法
1. プロジェクト直下にstatic/アプリ名/cssディレクトリを作成する
2. VSCodeの拡張でLive Sass Compilerをインストール
3. LiveSassCompilerの設定画面からFormatsをsettings.jsonで開く
4. savePathのディレクトリでcssファイルの出力先を指定する。（ルートは現在開いているディレクトリが基点）
```json
"savePath": "/src/【プロジェクト名】/static/【アプリ名】/css",
例）
"savePath": "/src/TeamA/static/dialy/css",
```
5. cssディレクトリやscssディレクトリにstyle.scssを作成し、cssを記述する
6. 画面右下のwatchsassボタンをクリックするとリアルタイムでコンパイルして指定フォルダへ出力される。

## サーバーサイドの編集について
※サーバーを再起動しないと反映されません。
```bash
docker compose restart web
```
3秒かからず再起動が完了し、反映されます。

## フロントエンドの編集について
※nginxに静的ファイルを送らないと反映されません。
1. コンテナ内に入ります。
```bash
docker exec -it django_gunicorn /bin/bash
```
2. collectstaticコマンドでnginxに静的ファイルを送ります。
```bash
python manage.py collectstatic --noinput
```
※即時反映されます。shift + F5でスーパーリロードしてください。