tests.py
========

テストの書き方は、基本的にdjangoの規約に沿って書きます。TestCaseを継承して書きましょう

.. seealso:: 

	`はじめての Django アプリ作成、その 5 <https://docs.djangoproject.com/ja/3.1/intro/tutorial05/>`_

.. seealso::

    `Testing - Django Rest Framework <https://www.django-rest-framework.org/api-guide/testing/>`_

テストは、継続的に機能追加更新をするようなプロジェクトで重要な役割を持ちます。逆に言うと、
単発の規模が小さめのプロジェクトでは、テストを書きすぎると費用対効果が得られません。
その場合は、数時間くらい時間を取ってみんなで動作確認したり、REST APIならPostmanを使って手動で確認してあげるのが良いかと思います。

.. seealso::

    https://www.postman.com/