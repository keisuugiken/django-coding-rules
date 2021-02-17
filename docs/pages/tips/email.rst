メール送信
=============

AWSの場合、django-sesを使用します。ローカルでテストしたい場合は、Gmailを設定することも可能です。

https://github.com/django-ses/django-ses

https://qiita.com/nnsnodnb/items/e771364e53ce37b38059

どちらも、共通の設定ファイルを書きます。

.. code-block::python

FROM_EMAIL = env('FROM_EMAIL')

if STAGE == 'local':
    EMAIL_USE_TLS = True
    EMAIL_HOST = 'smtp.gmail.com'
    EMAIL_HOST_USER = 'your.address@gmail.com'
    EMAIL_HOST_PASSWORD = 'passowrd'
    EMAIL_PORT = 587
    EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
else:
    EMAIL_BACKEND = 'django_ses.SESBackend'
    AWS_ACCESS_KEY_ID = env('AWS_ACCESS_KEY_ID')
    AWS_SECRET_ACCESS_KEY = env('AWS_SECRET_ACCESS_KEY')
    AWS_SES_REGION_NAME = 'ap-northeast-1'
    AWS_SES_REGION_ENDPOINT = 'email.ap-northeast-1.amazonaws.com'


メールの文章は、テンプレートエンジンを使用しましょう。
<project_name>/templates/emailというフォルダーを作成して、html/txtテンプレートを配置します。