forms.py
=================


こちらはお問い合わせフォームのサンプルです

.. code-block:: python

    from django import forms
    from django.conf import settings

    from . import models


    class ContactForm(forms.Form):
        subject = forms.CharField(min_length=1, required=True)
        message = forms.CharField(min_length=1, required=True)

        def __init__(self, *args, **kwargs):
            self.context = kwargs.pop('context', {})
            super(ContactForm, self).__init__(*args, **kwargs)
        
        def save(self):
            user = self.context['request'].user

            subject = self.cleaned_data['subject']
            message = self.cleaned_data['message']

            contact = models.Contact.objects.create(user=user,
                                                    subject=subject,
                                                    message=message)
            return contact
        
        
注意すべき点は2つあります。

1つは__init__でcontextを引数にとれるように修正することです。これはserializersのクラス定義と同じように、
リクエスト呼び出し元のリクエストオブジェクトを使ってログインユーザーの取得をフォーム内で行うためです。

2つ目は、フォームフィールドの定義を複雑にしないことです。特にウィジェットを定義するのは控えましょう。
ウィジェットを使うことで、HTMLテンプレートにおいてinputフィールドを同じコードで展開できますが、
基本的にdjangoテンプレートは、フロントエンドのコードを繰り返しや条件分岐によって効率よく展開するだけに機能をとどめておかないと、
フロントエンドのコード変更時にバックエンドの内部ロジックまで影響を及ぼすことになります。
そのため、formの内部までにHTMLテンプレートに関わるロジックを記述するのは危険です。
POSTするフォームのフィールド名と型を決めておいて、出来るだけフロントとバックエンドが分担して並列して作業するめられるようにしましょう。