admin.py
===============

models.pyにモデルを作成したら、もれなくadmin.pyに追加しましょう。他のファイルと同様に、adminの定義もクラスで定義します。

例えば以下のようなPollモデルがあった場合、


.. code-block::python

    class User(models.Model):
        username = models.CharField(max_length=100)

    class Poll(models.Model):
        user = ForeignKey(User)
        created_at = models.DateTimeField(default=datetime.datetime.now)


admin.pyには以下を記述します

.. code-block::python

    from django.contrib import admin

    from . import models


    @admin.register(models.User)
    class UserAdmin(admin.ModelAdmin):
        list_display = (
            'username',
        )

    @admin.register(models.Poll)
    class PollAdmin(admin.ModelAdmin):
        date_heirarchy = (
            'created_at',
        )
        list_display = (
            'user__username',
            'created_at'
        )

ModelAdminを継承するクラスは、必ず{model名}Adminという名前で定義します。
また、管理画面の一覧に表示させる項目を定義し、もし外部キーが含まれる場合は、外部キーの接続先モデルのうち、どのフィールドを表示するかを指定します。
もしモデルで__repr__をきちんと定義していて、これを表示したい場合は外部キーのフィールド名は特に指定する必要はありません。
さらに、タイムスタンプモデルを継承するモデルの場合はcreated_atとupdated_atをdate_heirarchyに追加します。これで作成されたデータの順番で管理画面上に表示されます