## Scaling Celery to handle workflows and multiple queues

Celery is an asynchronous task queue that integrates nicely with [django](https://www.djangoproject.com/). In this post, I am not going to write a tutorial on how to set up and use celery, there are many articles for it already. I am going to discuss some of the advanced features of celery I used in some of the projects that I worked on.

### Grouping and Chaining of tasks

Consider a scenario, you're working on an e-commerce project, you want to write a task to update product details and make an API call to update the status only when all are updated. One way is to write a cron job without celery, but that'll be synchronous. Each product blocks the thread until it completes. But, with celery group primitives, it'll be asynchronous i.e. a new task will be created for each product and they run asynchronously without blocking each other.

Below is our function to update product details

```python
@app.task(name="update_product_details" )
def update_product_details(product_id):
    try:
        product_info = make_http_call(product_id)
        obj = Product.objects.get(id=product_id)
        for key, val in product_info.items():
            setattr(obj, key, val)
            obj.save()
    except Exception:
        return {"status": False, "message" : "error in updating"}
    return {"status": True, "message": "successfully updated"}
```

Below is the cron that runs every day to update the product details.

```python
from celery import group, chain

def cron_to_update_product(products):
    group_tasks = []
    for product in products:
        group_tasks.append(update_product_details.s(product.product_id))
    async_result = chain(
        group(group_tasks), update_status_through_callback.s()
    ).apply_async()
    print(
        "a task with id %s is created to update product details" % async_result.task_id
    )
```

#### Code Breakdown

`.s` - added to the task is called `signature`. 

`group(group_tasks)` - celery creates `n` number of products where `n` is the number of products. All these tasks will be executed concurrently without blocking each other.

`chain(group(group_tasks), update_status_through_callback.s())` - as the name says, tasks are executed sequentially. Once all the tasks in the group are finished, then `update_status_through_callback` runs

`apply_async` - runs the task

There's one key point to note here, the function `update_status_through_callback` should take `grouped_result` as a first argument. `grouped_result` will be a list of return values of all the grouped tasks. 
For example, there are 5 group tasks ran and 3 of them are failed. Then the `grouped_result` will look like this

```python
[
    {"status": False, "message": "error in updating"},
    {"status": True, "message": "successfully updated"},
    {"status": False, "message": "error in updating"},
    {"status": True, "message": "successfully updated"},
    {"status": False, "message": "error in updating"},
]
```

Finally, our `update_status_through_callback` look like this

```python
def update_status_through_callback(grouped_result):
    if not all([result["status"] for result in grouped_result]):
        return {"status": False,
                "message": "not all products are updated"
        }
    response = make_http_call()
    if not is_valid_response(rseponse):
        return {"status": False,
                "message": "error in making callback"
               }
    return {"status": True, "message": "updated status"}
```

In the first line of the function, we're checking if all the group tasks are successfully executed because we should only update the status of all products are updated.

### Task Routing

We all run celery with a simple command like this `celery worker -A proj_name`. Running only one worker scales when the project has less number of tasks. But, consider the same scenario you're working on an e-commerce project, you want to run different types of reports. Assume you are running only one queue and few reports take a lot of time (name them `long_running_tasks`) and few take less time (name them `short_running_tasks`). Assume when you get a lot of `long_running_tasks` which makes queue fill up and `short_running_tasks` has to wait till they finish. This may not scale well. So, the scalable solution for it is to create separate queues for each report type. But this approach also has a problem. If there are no tasks for a specific report type running those queues is a waste of resources. So, it's a trade-off whether to go with the first or second approach depending upon the business use case.

For running multiple queues based on the report type, you need to use this celery configuration

```python
CELERY_BROKER_URL = "redis://localhost:6379" # if your broker
# is different change this
CELERY_RESULT_BACKEND = "redis://localhost:6379" # change this
# if this is different for you
CELERY_ACCEPT_CONTENT = ["application/json"]
CELERY_TASK_SERIALIZER = "json"
CELERY_RESULT_SERIALIZER = "json"
CELERY_TASK_DEFAULT_QUEUE = "default"
CELERY_TASK_DEFAULT_EXCHANGE = "default"
CELERY_TASK_DEFAULT_EXCHANGE_TYPE = "topic"
CELERY_TASK_DEFAULT_ROUTING_KEY = "task.default"

CELERY_TASK_ROUTES = {  
  "foo.tasks.report_type1_aggregator": {
      "queue": "report_type1_aggregator_queue"
  },
  "foo.tasks.report_type2_aggregator": {
      "queue": "report_type2_aggregator_queue"
  },
  "foo.tasks.report_type3_aggregator": {
      "queue": "report_type3_aggregator_queue"
  },
  "foo.tasks.report_type1_report_queue": {
      "queue": "report_type1_report_queue"
  },
  "foo.tasks.report_type2_report_queue": {
      "queue": "report_type2_report_queue"
  },
  "foo.tasks.report_type3_report_queue": {
      "queue": "report_type3_report_queue"
  }
}
```

We defined routes for each task and assigned a queue for it. But, we haven't created workers in celery. We can create workers using the below commands

```bash
celery worker -A proj_name -O fair -Q {queue_name}
 -P gevent --autoscale=32,16 --loglevel=INFO 
 --logfile={queue_name}_celery.log
```

repeat the above command for all the queues we defined.

#### Tip :

instead of running many commands, use the `celery multi` utility. Examples are given [here](https://docs.celeryproject.org/en/stable/reference/celery.bin.multi.html)

Now, let's create a default worker

```bash
celery worker -A proj_name -O fair -Q default 
-P gevent --loglevel=INFO --logfile=celery.log
```

That's it! When you're running tasks they'll be routed to the respective queues.

### Additional Notes

1. Use [flower](https://github.com/mher/flower) to monitor workers and tasks.
2. Use `gevent` or `eventlet` if your tasks are I/O bound and use `prefork` if it is CPU bound.
3. Use `-O fair` for better scheduling.

### References

Celery Routing Tasks: <https://docs.celeryproject.org/en/stable/userguide/routing.html>

Running Workers: <https://docs.celeryproject.org/en/stable/userguide/workers.html>

Celery Worker Pools: <https://www.distributedpython.com/2018/10/26/celery-execution-pool/>