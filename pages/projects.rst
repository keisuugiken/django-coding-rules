コーディングスタイル
==================

主にDjangoをWebブラウザアプリケーション、もしくはREST APIを開発するために使用することを想定しています

* :ref:`projects-app`
* :ref:`projects-lib`
.. * :ref:`projects-admin`
.. * :ref:`projects-api_views`
.. * :ref:`projects-apps`
.. * :ref:`projects-forms`
.. * :ref:`projects-management`
.. * :ref:`projects-migrations`
.. * :ref:`projects-models`
.. * :ref:`projects-scheduler`
.. * :ref:`projects-serializers`
.. * :ref:`projects-services`
.. * :ref:`projects-signals`
.. * :ref:`projects-templates`
.. * :ref:`projects-tests`
.. * :ref:`projects-urls`
.. * :ref:`projects-views`

.. _projects-app:

    ディレクトリ構造
--------------------

Djangoの`startapp`コマンドで自動生成されるファイル以外にも、ビジネスロジックを書くときはどのファイルに記述すればいいのか、REST APIのJSONシリアライザーをどこに書けばいいのかなど、
拡張するときのルールを様々な機能開発に対して網羅可能なディレクトリ構造として、以下の構成をおススメしています。

    <appname>/
        __init__.py
        admin.py              # models.pyにモデルを作成したら、必ず管理画面で編集できるように追加しましょう
		api_views.py          # REST APIのようなJSONデータのやり取りを行うエンドポイントを書きます。ファイルの意義はviews.pyと同じです
        apps.py               # 管理画面での表示に関わります。必ず書きましょう。また、スケジューラーの実行にも使用されます
		forms.py              # HTMLから送信されるフォームの定義。送信データの検証はここに書く
		management/           # 管理ユーザーを作る。シードデータのファイルをDBに取り込むみたいなコマンドを定義します
			__init__.py
			commands/
				__init__.py
				<name>.py     # python manage.py <name>で実行できるようになります
        migrations/           # このままの構造で作成します。自動生成されたファイルは自分で編集したり削除しないようにしましょう
            __init__.py
        models.py             # データベースのテーブルを作成する場合は必要になります
		scheduler.py          # 毎日1回実行するといったスケジューラの定義をここに書きます
		serializers.py        # REST APIのJSONシリアライザーやデータ検証のロジックを書きます。検証ロジックはすべてここに書きます
		services.py           # 次項以降で定義された各ファイルの定義に入らない、例えば外部APIの呼び出しや統計データの算出などビジネスロジックに係ることはここに書きます
		signals.py            # モデルのトリガー関数を書きます
		templates/            # HTMLファイルのテンプレート
			<appname>/
				email/ 		  # 送信するメールのテンプレートを定義する場合
					<mail_name>.html
					<mail_name>.txt
				page.html
        tests.py              # テスト
		urls.py               # パス名とviews.pyもしくはapi_views.pyで定義したビュークラスとのマッピングを定義します
        views.py              # フォームデータを受け取り、HTMLを返すような機能はここに定義します

appのフォルダを作成するときに考えないといけないのは、Webアプリケーションのパスです。
例えばブログアプリケーションを作成するとしたら、以下のようなエンドポイントを用意するでしょう
- /posts : 記事一覧
- /posts/<post_id> : 記事ページ
- /users : ユーザー一覧
- /users/<user_id> : ユーザーの投稿した記事一覧
- /users/login : ログイン
- /users/register : 登録
- /mypage : ログイン後に投稿したりするための個人ページ

エンドポイントの設計手法についてはREST APIの仕様に準拠してもらうとして、もし上記のようなエンドポイントである場合、
- posts
- users
- mypage

の3つのappを作成することで、特にurls.pyが統一されます。したがって、このエンドポイント自体を設計するときに、
機能としてまとまりのあるエンドポイントの設計になっているのかを意識するのが良いでしょう。

.. seealso:: 

	`Django's Split Settings Wiki <https://docs.djangoproject.com/ja/3.1/intro/tautorial01/#creating-the-polls-app>`_ 
		startappコマンドで作成されるディレクトリ構造が紹介されています


.. _projects-lib:

    libフォルダ
--------------

もしすべてのappに共通して使用するクラスなどを記述したり、外部ライブラリをプロジェクトに直接追加したい場合は、プロジェクト直下にlibフォルダを作って、libフォルダ内に配置しましょう