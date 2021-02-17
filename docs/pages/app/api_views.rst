api_view.py
=================


基本的にviewでは、どのクエリを実行し、どのシリアライザーとサービスを組み合わせて実行するかしか書きません。
したがって、viewの内部に何行もロジックを書くのはナンセンスで、ほぼ機械的に定義してあげましょう。

以下はユーザー一覧、ユーザーの新規追加を行うビューになります。

GET /users でユーザー一覧
POST /users でユーザーの登録を行うようなREST APIを想定しています

.. code-block::python

    from django.core.paginator import Paginator

    from rest_framework.views import APIView

    from lib import mixins
    from lib.responses import SuccessResponse, ErrorResponse
    from . import models, serializers, services



    class UserListView(mixins.TsunapaBaseViewMixin, APIView):
        # ここでGETのクエリ呼び出しやページネーションの設定を行う
        def get_querysets(self):
            # 複雑なクエリは、Managerクラスに定義して呼び出す
            querysets = models.User.objects.search(**self.request.query_params)
            paginator = Paginator(querysets, 10)
            page_number = self.request.GET.get('page')
            page_obj = paginator.get_page(page_number)
            return page_obj
            
        # ユーザー一覧・検索
        def get(self, request):
            querysets = self.get_querysets()
            serializer = serializers.UserSerializer(querysets, many=True)
            return SuccessResponse({'users': serializer.data})

        # ユーザーの新規登録
        def post(self, request):
            serializer = serializers.UserSerializer(data=request.data, context={'request': request})
            if not serializer.is_valid():
                return ErrorResponse(serializer.errors)
            user = serializer.save()
            # 例えばユーザーにメール通知するなど、ビジネスロジックを定義したサービスを呼び出す
            service = services.UserService()
            service.send_welcome(user)
            return SuccessResponse()


以上のように、viewのなかではmodelを使ったクエリの呼び出し、シリアライザーへデータを渡し、サービスからビジネスロジックを呼び出すなど、
具体的な処理は個別のクラスに任せて、実行の組み合わせや順序を定義しているだけになります。
したがって、このviewクラスでは(タイプミスを除いて)バグが含まれてエラーになることはほぼ発生しないです。
もしviewクラス内でデバッグに時間をかけてしまっていたら、そのロジックはサービスクラスなど別のクラスに分けるように設計をし直すことをおススメします。