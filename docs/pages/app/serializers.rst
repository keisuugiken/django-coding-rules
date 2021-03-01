serializers.py
=====================

django-restframeworkを使用したときに定義するファイルです。シリアライザーで実装する機能は主に以下の3つがあります

* クエリで取得したモデルインスタンスを、要件に合うJSONに変換する (GETで使用)
* 送信されたデータを検証する (POSTで使用)
* 検証したデータをモデルインスタンスに変換して保存/更新する (POSTで使用)

views.pyではクラスの呼び出しだけを担当し、models.pyではデータベースの定義を行うだけです。
そのためAPIとしての機能を決めるのは、managers.py、services.py、そしてserializers.pyになります。
ただmanagers.pyやservices.pyは、データベースの設計を上手く行うことで定義する必要がない場面が多いです。しかし、
serializers.pyはほとんどのAPI実装で必要となるクラスであり、djangoの実装において一番重要なポイントになります。
(HTMLページにおいて対応するforms.pyも同様です)

以下に、Post(貴意投稿)モデルを例として参考

JSONに変換する
-----------------

シリアライザーの呼び出され方は、views.pyを参照してください。HTTP GETメソッドで呼び出されるときは、クエリセットを引数として初期化され、
受け取ったクエリセットをJSONに変換する方法をシリアライザ内で定義します。

基本的には、定義はMetaクラスに書きましょう。そして不要な読み書き権限を与えないように、read_only_fieldsとして定義します。
もし書き込みも許可するフィールドの場合は、fieldsとして定義します

.. code-block:: python

    from rest_framework import serializers

    from . import models

    class PostSerializer(serializers.ModelSerializer):
        publisher = models.CharField(source='user__username', read_only=True)

        class Meta:
            model = models.Post
            read_only_fiels = ('post_id', 'published_at', 'publisher', 'comments_count',)
            fields = ('title', 'body',)

        def get_comments_count(self, instance):
            comments = models.Comment.objects.filter(post=instance)
            return len(comments)

もしモデルに定義されていないフィールドを追加したい場合は、get_{フィールド名} メソッドを実装しましょう。
詳細はdjango-restframeworkのドキュメントを確認してください。


送信されたデータを検証する
----------------------------------

次にPOSTメソッドで呼び出されたときに、JSONデータを検証するためのデータ定義をします。

ここで、検証する際にカスタム定義を記述し、その中でデータ変換を行うことが出来ます。例えば以下の例を見てください。

.. code-block:: python

    from rest_framework import serializers

    from . import models

    class PostSerializer(serializers.ModelSerializer):
        co_publisher_name = models.CharField(source='co_publisher__name')

        class Meta:
            model = models.Post
            write_only_field = ('co_publisher_id',)
            read_only_fiels = ('co_publisher_name')

        def validate_co_publisher_id(self, value):
            try:
                co_publisher = models.User.objects.get(user_id=value)
            except models.User.DoesNotExist:
                raise serializers.ValidationError('このユーザーIDは存在しません')
            return co_publisher

co_publisher_idというフィールドを定義しています。POSTで送信されたco_publisher_idを使ってユーザーを検索し、最終的にUserインスタンスを返しています。
結果、validated_dataにはユーザーインスタンスが保存されるため、この後にcreateメソッドでインスタンスを永続化するときに役立ちます。

また、検証に失敗したら、必ずValidationErrorを出しましょう。ValidationErrorの引数に渡した文章が、
エラー説明分としてJSONに入ります。


検証したデータをモデルインスタンスに変換して保存/更新する
--------------------------------------------------------------------

2で検証が完了したら、create/updateメソッドを定義してインスタンスの永続化の方法を定義しましょう。
例えばデータを作成したユーザーを保存しておきたい場合は、contextからリクエストのインスタンスを取得して、
ログインユーザーのインスタンスを取得出来ます。以下保存の例です。

.. code-block:: python

    class PostSerializer(serializers.ModelSerializer):
        ...

        def create(self, validated_data):
            publisher = self.context['request'].user
            co_publisher = validated_data['co_publisher_id']

            post = models.Post.objects.create(
                publisher=publisher,
                title=validated_data['title'],
                body=validated_data['body'],
                co_publisher=co_publisher,
            )

            return post

        def update(self, instance, validated_data):
            co_publisher = validated_data['co_publisher_id']

            instance.title = validated_data['title']
            instance.body = validated_data['body']
            instance.co_publisher = co_publisher

            return instance        