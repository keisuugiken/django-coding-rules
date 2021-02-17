services.py
==================

ビジネスロジックを記述します。ビジネスロジックというと定義が難しそうですが、基本的には、managers.py, forms.py, serializers.pyなど、もしくはlibフォルダで共通して定義されていないことを記述してください。
例えば、外部の決済代行サービスAPIにデータを送信する、メール送信サービスを呼び出す、CSVのデータを読み込む、などです。

以下はユーザーが新規登録されたときにメールを送信する一例です

.. code-block::python

    from django.conf import settings

    from lib.email import send_simple_mail
    from . import models


    class UserService(object):
        def send_welcome(self):
            # welcomeメール送信
            subject = 'ようこそ！'
            template = 'welcome'
            login_url = 'https://{}{}'.format(settings.HOST_NAME, settings.LOGIN_URL)
            context = {'login_url': login_url}
            send_simple_mail(email, subject, template, context)