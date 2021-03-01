management
=================

managementを追加するケースとして、特に多いのはシードデータをデータベースに取り込むような、
一度だけ実行する機能を実装する場合です。例えばExcelにマスターとなるデータを書いておいて、
import_excelのようなコマンドを作成してください。

ここで注意しないといけないのは、コマンドにコアロジックを書かないことです。代わりにサービスクラスに
ビジネスロジックを書いて、コマンド定義から呼び出すように強いましょう

例えばシードデータをExcelからインポートするコマンドの場合は以下のように書きます

.. code-block:: python

    from django.core.management.base import BaseCommand
    from django.conf import settings
    from appname import services


    class Command(BaseCommand):
        help = 'import seed data from excel'

        def add_arguments(self, parser):
            parser.add_argument('--file', type=str)


        def handle(self, *args, **options):
            filename = options['file']
            service = services.DataImportService()
            service.import(filename)
            self.stdout.write(self.style.SUCCESS('all data were imported successfully'))


また、ほとんどのプロジェクトでは管理者ユーザーを自動で作成したいケースが発生します。
そこで、createsuというコマンドを定義します。

.. code-block:: python

    from django.core.management.base import BaseCommand
    from django.conf import settings
    from users import models


    class Command(BaseCommand):
        help = 'Create new superuser to login admin console'

        def add_arguments(self, parser):
            parser.add_argument('--email', type=str, default=settings.SUPERUSER_EMAIL)
            parser.add_argument('--password', type=str, default=settings.SUPERUSER_PASSWORD)


        def handle(self, *args, **options):
            email = options['email']
            password = options['password']

            try:
                user = models.User.objects.get(email=email)
            except models.User.DoesNotExist:
                user = models.User.objects.create_user(email, email, password)

            user.is_superuser = True
            user.is_staff = True
            user.set_password(password)
            user.save()
            self.stdout.write(self.style.SUCCESS('super user was created successfully'))

環境変数に管理ユーザーのメールアドレスとパスワードを設定しておいて、
python manage.py createsuで管理者ユーザーを作成します