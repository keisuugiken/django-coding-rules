models.py
=================

モデルはデータベースの設計をそのままDjangoの形式に転記するだけです。ここで何かロジックを考えたりプログラミングすることはありません。
以下のようなフォーマットで記述しましょう

.. code-block:: python

    class Post(BaseModel, models.Model):
        post_id = models.UUIDField('投稿ID', default=uuid.uuid4, unique=True, db_index=True)
        publisher = models.ForeignKey('users.User', db_index=True, on_delete=models.CASCADE, related_name="posts")
        title = models.CharField('タイトル', max_length=250)
        body = models.TextField('本文')
        co_publisher = models.ForeignKey('users.User', db_index=True, on_delete=models.CASCADE, related_name="co_publishing_posts")
        
        objects = managers.PostManager()

        class Meta:
            verbose_name = 'posts'
            verbose_name_plural = 'Post'

        def __str__(self):
            return self.title


BaseModelでは、created_atとupdated_atのタイムスタンプフィールドに加え、
liveというソフトデリートのためのフィールドを定義しています。
どのモデルにも共有するフィールドになるため、ベースクラスにまとめています