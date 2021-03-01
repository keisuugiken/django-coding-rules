views.py
=================


viewsは、api_viewsと同様に、forms, models, servicesを呼び出す方法を定義しているだけです。
GETの場合はクエリを取得してテンプレートを選択する。
POSTの場合はフォームを検証してエラーページを表示したり、
正しいフォームデータであればデータを保存したりビジネスロジックを呼び出したりします。

こちらも、煩雑になった場合はロジックをserviceやformsに移すように変更しましょう。

.. code-block:: python

    from django.shortcuts import render, redirect
    from django.views.generic.base import View
    from django.contrib.auth import logout
    from django.contrib.auth.mixins import LoginRequiredMixin
    from django.utils import timezone

    from lib import mixins
    from . import models, forms, services


    class LoginView(View):
        template = 'users/login.html'

        def get(self, request):
            form = forms.LoginForm()
            context = dict(form=form)
            return render(request, self.template, context)

        def post(self, request):
            form = forms.LoginForm(request.POST, context={'request': request})
            if not form.is_valid():
                print(form.errors)
                context = dict(errors=form.errors)
                return render(request, self.template, context)
            form.save()
            service = services.UserLoginService()
            # リダイレクト先を選択する
            next_path = service.get_next_path(request)
            return redirect(next_path)


    class LogoutView(mixins.BaseMixin, View):
        def get(self, request):
            logout(request)
            return redirect('top_view')

    class UserDetailView(mixins.BaseMixin, View):
        template = 'users/detail.html'

        def get_queryset(self, user_id):
            try:
                reset_token = models.User.objects.get(user_id=user_id)
            except models.User.DoesNotExist:
                raise Http404
            return user

        def get(self, request, user_id):
            user = self.get_queryset(user_id)
            context = dict(user=user)
            return render(request, self.template, context)