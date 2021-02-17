urls.py
============

URLの定義は機械的に行います。例えば以下はユーザーの一覧、ログイン、登録ビューへのURLを定義しています。

.. code-block::python

    from django.urls import path

    from . import views


    app_name = 'users'


    urlpatterns = [
        path('', views.UserListView.as_view(), name='list'),
        path('login/', views.UserLoginView.as_view(), name='login'),
        path('signup/', views.UserSignupView.as_view(), name='signup'),
    ]


パスの名称を定義するときは、出来るだけViewの命名規則に合わせてあげましょう。例えばloginの場合、
テンプレートからURLを呼び出すと、`{% url 'users:login' %}`と書けます。
ここから、どのViewにマッピングされているかをurls.pyを見なくてもすぐに把握することが出来ますし、
またURLの呼び出しを見ただけですぐに何をするビューへのリンクになっているかが理解できます。