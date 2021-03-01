scheduler.py
============

scheduler.pyは、django-apschedulerを使用することを前提としています。「毎月1日に支払の請求を実行」や「5分ごとにファイルの更新があるか確認する」といった定期的な処理をしたい場合は、
schedulerを定義します。

schedulerには、apschedulerを使って、どのようなタイミングで処理を実行するかを定義します。しかし、どのような処理を行うかについては、serviceクラスに記述してください。
例えば、以下は毎日1日に支払いの請求を実行する場合のサンプルコードです。

.. code-block:: python

    import logging

    from apscheduler.schedulers.background import BackgroundScheduler
    from django_apscheduler.jobstores import DjangoJobStore
    from apscheduler.triggers.cron import CronTrigger
    from django_apscheduler.jobstores import register_events
    from django.conf import settings

    from invoices import services

    # Create scheduler to run in a thread inside the application process
    scheduler = BackgroundScheduler(timezone=settings.TIME_ZONE)
    scheduler.add_jobstore(DjangoJobStore(), "default")

    def start():
        if settings.DEBUG:
            # Hook into the apscheduler logger
            logging.basicConfig()
            logging.getLogger('apscheduler').setLevel(logging.DEBUG)

        billing_service = services.BillingService()

        scheduler.add_job(
            billing_service.request_payment,
            trigger=CronTrigger(day=1),  # 毎月1日に実行
            id="request_payment",
            max_instances=1,
            replace_existing=True,
        )
        # Add the scheduled jobs to the Django admin interface
        register_events(scheduler)

        scheduler.start()