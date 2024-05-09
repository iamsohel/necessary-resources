## Run celery, celery beat and flower in python project 

- pip install celery redis flower gevent [for windows]


- celery.py
```
from celery import Celery
from celery.schedules import crontab
from datetime import timedelta

app = Celery('test-celery',
             broker='redis://localhost:6379/10',
             include=['task_queue.tasks'])

app.conf.beat_schedule = {
    'add-every-30-seconds': {
        'task': 'task_queue.tasks.add',
        'schedule': timedelta(seconds=30)
    },
}
app.conf.broker_connection_retry_on_startup = True
app.conf.timezone = 'UTC'

if __name__ == '__main__':
    app.start()
```

- tasks.py
  
```
from celery import shared_task
from .utils import added


@shared_task()
def add():
    added()
```

### the the below command

```
celery -A task_queue worker --concurrency=8 -l info -P gevent
celery -A task_queue beat 
celery -A task_queue flower 
```
