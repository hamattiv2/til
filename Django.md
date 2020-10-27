# 各ファイルの説明

## settings.py

### DEBUG
デバッグ結果を表示するかしないかの設定を行う。
デフォルトはTrueでデバッグ結果を表示する。
開発するときはTrue, 公開するときはFalse

### ALLOWED_HOSTS = []
公開するとき(DEBUG = False)にアクセスを許可するホスト(IP)を指定する。

### INSTALLED_APPS
Djangoにインストールしたアプリを記載する。

### ROOT_URLCONF
アクセスしたときにDjangoがまずアクセスする先

### 言語設定
```Python
LANGUAGE_CODE = 'ja'
```

### タイムゾーン設定
```
TIME_ZONE = 'Asia/Tokyo'
```

## models.py
実行ファイル。

## urls.py
リクエストを各処理に引き渡すファイル

## views.py
urls.pyから来たリクエストを色々な処理、加工をしてtemplate(thmlファイル)に返す。
関数ベースの書き方とクラスベースな書き方がある。


# startproject
プロジェクトを作成する。

```python
django-admin startproject プロジェクト名

"""
プロジェクト名(フォルダ)
　--プロジェクト名(フォルダ)
　　--各ファイル
"""
```

```python
django-admin startproject プロジェクト名 .

"""
プロジェクト名(フォルダ)
　--各ファイル
"""
```
ドットを付けると上位フォルダを生成しなくなる。

# startapp

```
python manage.py startapp アプリ名

"""
プロジェクトの中にアプリが生成される
"""

```

# 作成したモデルを管理画面で確認できるようにする

作成したアプリのadmin.pyを開き以下を記載する

```
from django.contrib import admin
from .models import *
# Register your models here.
admin.site.register('作成したモデル名')
```

このままだと管理画面に表示されるのはオブジェクト名(TodoModel object (1) ← こんなん)なので、表示名を変更するにはmodels.pyを開き以下のように記載する。

```
class TodoModel(models.Model):
    titile = models.CharField(max_length=100)
    memo = models.TextField()

    # 以下のように記載すれば管理画面でtitileが表示される
    def __str__(self):
        return self.titile
```

# クラスビューを使用したCRUD

C：ListView
R：DetailView
U：UpdateView
D：DelteView

### C:ListView

```Python:views.py
from django.views.generic import ListView
 
classs クラス名(ListView):
    # 遷移先テンプレート名
    template_name = "template.html"
    # 読み込んでくるモデル名(単一の場合？)
    # templateでは「object_list」という名前で利用可能
    model = モデル名
```

### R:DetailView

```Python:views.py
from django.views.generic import DetailView,

class Detailクラス名(DetailView):
    # 遷移先テンプレート名
    template_name = "template.html"
    # 読み込んでくるモデル名(単一の場合？)
    # templateでは「object」という名前で利用可能
    model = モデル名
```

```Python:urls.py
urlpatterns = [
    #必ずpkをパラメーターに持ってこさせる
    path('detail/<int:pk>', Detailクラス名.as_view())
]
```
### C:CreateView

```Python
from django.views.generic import CreateView
from django.urls import reverse_lazy

class TodoCreate(CreateView):
    template_name = 'create.html'
    # 保存先のmodelを指定する
    model = モデル名
    # 入力フォームを生成するときに指定モデルの、どのカラムを入力してもらうのかを指定する。
    fields = ("titile", "memo", "priority", "duedate")
    # データ保存が成功したときにどのページに遷移するかを指定する。
    # クラスの中で返すページを指定する時はreverse_lazy
    # 関数の中で返すページを指定するならreverse
    success_url = reverse_lazy("list")
```

```Html
<form action="" method="POST">
    {% csrf_token %}
    <!-- CreateViewで指定したモデルの入力フォームが展開される -->
    {{ form.as_p }}
    <input type="submit" value="作成する">
</form>
```

### D:DeleteView
deleteは比較的簡単。

```Python
class TodoDelete(DeleteView):
    template_name = "delete.html"
    model = TodoModel
    success_url = reverse_lazy("list")
```

```HTML:delete.html
<form action="" method="POST">
    {% csrf_token %}
    <input type="submit" value="削除する">
</form>
```

```Python:urls.py
urlpatterns = [
    path("delete/<int:pk>", TodoDelete.as_view(), name="delete"),
]
```

### U:UpdateView

```Python:views.py
class TodoUpdate(UpdateView):
    template_name = "update.html"
    model = TodoModel
    fields = ("titile", "memo", "priority", "duedate")
    success_url = reverse_lazy("list")
```

```HTML:update.html
<form action="" method="post">
  {% csrf_token %}
  {{ form.titile }}
  {{ form.memo }}
  {{ form.priority }}
  {{ form.duedate }}
  <input type="submit" value="更新する">
</form>
```

```Python:urls.py

urlpatterns = [
    path("update/<int:pk>", TodoUpdate.as_view(), name="update")
]
```



# migrateの挙動

### null-ableなカラムを設定しようとした時

makemigrations時に以下のアラートが出現する

```
You are trying to add a non-nullable field 'カラム名' to todomodel without a default; we can't do that (the database needs something to populate existing rows).
Please select a fix:
 1) Provide a one-off default now (will be set on all existing rows with a null value for this column)
 2) Quit, and let me add a default in models.py
```
1番は設定時に何らかの値を設定する。
2番はmodels.pyの修正からやり直す
ことを意味する。

日時系のカラムでこの質問が出たとき(null=True)を指定していないとき
timezone.nowを入れておけばカラム作成時に
既存のデータには現在日時を入れてくれる。

# models.py

### choices
optionタグなどで値を選択させたいカラムに使える。

```python
PRIORITY = ({"danger", "high"}, {"info", "normal"}, {"success", "low"})

priority = models.CharField(
    max_length=50,
    choices = PRIORITY
)
```

PRIORITYにタプル内辞書で設定をしている。
右側が実際に表示される文字列で、左側が右側を選択したときに選択される値。


# imageファイルの取り扱い

```Python:models.py
class BoardModel(models.Model):
    images = models.ImageField(upload_to="")
```

```Python:settings.py
MEDIA_ROOT = os.path.join(BASE_DIR, "madia")
# URL上に表示される名前(/medi/ファイル名 になる)
MEDIA_URL = "/medi/"
```

```Python:urls.py
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path("", include("boardapp.urls"))
# urlpatternsのお尻にmediaの情報を追加する
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

### templateに表示する場合

```HTML:template.html
<!-- urlを必ずつける  -->
<p><img src="{{ object.images.url }}" width="300"></p>
```

# css(js)ファイルの取り扱い

```Python:settings.py
# 以下を指定する。
STATIC_URL = '/static/'
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, "static")
]

# collectstatic時に集約させるフォルダを指定する
STATIC_ROOT = "/static/"
```

```Python:urls.py
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path("", include("boardapp.urls"))
] + static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```

# ログイン判定
関数ベースでログイン判定をするには`login_required`を使用する。
[ログインデコレーター](https://docs.djangoproject.com/ja/3.0/topics/auth/default/#the-login-required-decorator)

```Python:views.py
from django.contrib.auth.decorators import login_required

@login_required
# 実行したい関数の直前にデコレーターを設置することでログインしているかどうかの判定を行ってくれる。
# 未ログインの場合のリダイレクト先はsettings.pyに記載する(デコレーターに指定することも可能)
def my_view(request):
    ...
```

```Python:settings.py
LOGIN_URL = "login"
# urls.pyの名前でOK

'''
urlpatterns = [
    path("login/", loginfunc, name="login"),
]
'''
```

# ログアウト処理
関数ベースでログアウト処理を行うには`logout`を読み込む
[ログアウト](https://docs.djangoproject.com/ja/3.0/topics/auth/default/#how-to-log-a-user-out)

```Python:views.py
def logoutfunc(request):
    logout(request)
    return redirect("login")
```

```Python:urls.py
urlpatterns = [
    path("logout/", logoutfunc, name="logout")
]

```

# 対象ページのキャッシュを保存させない。
[キャッシュを使う](https://narito.ninja/blog/detail/75/#_7)

# 本番環境ではrunserverを使わない
[DjangoのrunserverとGunicorn](https://scrapbox.io/shimizukawa/Django%E3%81%AErunserver%E3%81%A8Gunicorn)


# Djangoのforms.PasswordInputとか
Djangoにはformsを実装する時PasswordInputとかを定義することがある。

```project/forms.py
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User

class SignUpForm(UserCreationForm):
    password = forms.CharField(widget=forms.PasswordInput)
    class Meta:
        model = User
        fields = ('username', 'email', 'password')
```


上記で登場するforms.PasswordInputは`django/forms/widgets.py`に定義されたPasswordInputクラスを呼び出して実行されている。

```django/forms/widgets.py
class PasswordInput(Input):
    input_type = 'password'
    template_name = 'django/forms/widgets/password.html'

    def __init__(self, attrs=None, render_value=False):
        super().__init__(attrs)
        self.render_value = render_value

    def get_context(self, name, value, attrs):
        if not self.render_value:
            value = None
        return super().get_context(name, value, attrs)
```

このクラス(ほかにもTextInputやEmailInputクラスが存在する)が呼び出されると、template_nameに記載のあるhtmlファイルが呼ばれ、そのクラスに合った(PasswordInputの場合)inputフィールドが展開される。

# 管理画面のアプリケーションの名称、モデルの名称を変更する。
### アプリケーションの名称
`アプリケーション/apps.py`内に自動的にDjangoに読み込ませるアプリケーション情報が格納されているが、
ここでAdmin画面上でどう表示するか指定できる。
```apps.py
from django.apps import AppConfig


class ShopConfig(AppConfig):
    name = 'shop'
    verbose_name = 'ショップ'
```

### モデル名称
```models.py
class Book(models.Model):

    class Meta:
        db_table = 'book'
        verbose_name = verbose_name_plural = '本'

    title = models.CharField(verbose_name='タイトル', max_length=255)

    def __str__(self):
        return self.title
```
Djangoではデフォルトでモデル名にsがつく(複数形になる)ためsが不要な場合verbose_name_pluralで名称を定義してあげることでsが削除される。

# ログアウト後の遷移画面を指定する
```settings.py
LOGOUT_REDIRECT_URL = "urls"
```

# パスワード再設定メール送信
https://docs.djangoproject.com/ja/3.1/ref/contrib/admin/#adding-a-password-reset-feature
上記を参考にurls.pyにパスを設定することで、パスワード再設定用のメール送信画面、パスワード再設定画面が構築される。
メール送信時に送られるリンクの有効期限はデフォルト3日間。
PASSWORD_RESET_TIMEOUT_DAYSの設定を変更することで有効期限を変更することが可能。
また、メールのテンプレは以下に格納されている。

```
django/contrib/auth/templates/registration/password_reset_subject.txt
django/contrib/admin/templates/registration/password_reset_email.html
```
