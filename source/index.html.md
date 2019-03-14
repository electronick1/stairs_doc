---
title: Stairs documentation

language_tabs:
  - python

toc_footers:
  - <a href='https://github.com/electronick1/stairs'>Stairs on github</a>
  - <a href='https://github.com/electronick1/stepist'>Stepist on github</a>


search: true
---

# Welcome

Stairs is a simple tool which allows you to divide your data into parts and make it understandable. 

You can build data pipelines, draw parallels and make async or distributed calculations for most of your data related tasks.

Stairs is available in Python3, but you can write data processors in any language. 
You can also use any of streaming or queue services.

It's easy to start. Quickly test all your ideas or hypotheses and everything is 
ready for immediate use without any special magic. 

Start with "Installation" and then proceed on to "Getting started". 

# Installation


```python
# install redis https://redis.io/topics/quickstart
# sudo apt-get install redis
# brew install redis

pip install stairs-project
```



> It's recommended to use the latest python 3 version.

Just perform `pip install stairs-project` to install stairs along with all 
python dependencies.

Stairs requires Redis for storing statistics and certain meta-information, 
even if you use a different streaming or queue service. 

<aside class="notice">
A running Redis server is required for Stairs to work. 
</aside>

# Mission

## Data Pipelines

<!-- > ![image](images/data_pipeline.png) -->

The main focus of Stairs is data pipelines. It's a framework which helps you to
build and manipulate data with the help of a data flow graph. 

You can think of it as an MVP framework (like Django) for data pipelines.
Different layers of abstractions and components allow you to build any kind of 
data flow graphs and easily understand what's going on in your system. 


## Parallel/Async/Distributed

<!-- > ![parallel](images/parallel.png) -->

Each component of the data pipeline can be represented as a separate python 
process (worker). The components communicate with each other by using 
streaming/queue services, and together they can process your data in a parallel way.

Right now Stairs is using: <br>
- celery <br>
- self-implemented redis queue <br>
- kafka (under development) <br>

There is an interesting article about workers/jobs on wiki
-> [Wiki](https://en.wikipedia.org/wiki/Job_(computing))

The Stairs framework focuses on speed and light, and the speed 
of your "workers" is limited mostly by your streaming/queue service.


## For data-science and data-engineering with love

<!-- > ![ds_en](images/ds_en.svg) -->

Data-science and data-engineering are growing fast, and it's hard 
to be an expert in everything at the same time. 

For example, to train ML models, you should spend about 80% of your time 
to process data -- your fast ability to process  data and test 
all hypotheses will influence your final result.

Stairs allows a data scientist to build "scalable" solutions without 
a high level of data-engineering skills.

- A data-scientist can focus only on data processing
- A data-engineer can focus only on storing and moving data 
(between the components of the pipeline)


#Getting started


##Project

```shell
stairs-admin project:new name
```

> ![project](images/project.svg)


When you are done with installation, let's try kick-starting your first stairs project.

The Stairs project is similar to the django approach (when you can create a default 
project template). To have a better overview of your components, you can create a
similar project with Stairs. It will consist of apps with all basic layers inside. <bt>
But you are completely free to use any other structure you want. The default 
project template is just a way to kick-start your idea quickly. 


To create a default project template, just use the following command:

`stairs-admin project:new name`


This command will generate a basic project structure with one app inside.<br>

The project has a config file and "manager.py".

"manager.py"  allows you to read a config, detect apps and 
execute shell commands in the Django manner.


<br><br><br><br><br><br>



##App

```shell
stairs-admin app:new name
```


> ![app](images/app.svg)


The app is a way to generalize different approaches to one similar form. Why? 
Because right now data-science approaches are too scattered, and it's hard to 
understand what's going on, when there are tons of maths and algorithms around. 


Each app has the following components:

- a pipeline - represents a data flow graph and shows how data 
will be processed. Each pipeline consists of multiple small components 
like "Flow" (Data flow).  

- a producer - a function which helps you to read a source 
(a file, a database ...) and then directs it to the data pipeline.

- a consumer - a function which writes data to the data store 
or changes "the global state".

- a flow (Data Flow) - a set of functions 
(called [steps](https://en.wikipedia.org/wiki/Job_(computing))
which can change/filter/populate your data.



To create a new "default app" structure (with a package and modules), 
type the following command:

`stairs-admin app:new name`



```python
from stairs import App

app = App(name="my_app")

```

To define your app, you should initialize an App object with a name and a config 
(More about the app config in the section "App components").
You can find this app object in the "app_config.py" file inside the default app. 
Don't forget to change its name. 

If you want to add a new app to the project, populate 
`apps` variable in the config file or use `StairsProject().add_app(app)`

<br>

![image](images/app_2.svg)

<br>

#App components


## Pipeline

```python

@app.pipeline()
def my_pipeline(pipeline, value):
    return value.subscribe_func(my_function_which_process_data, as_worker=True) \
                .subscribe_flow(MySecondFlow())\
                .subscribe_consumer(save_result)

```



The pipeline is a way to combine multiple objects (functions/classes/other pipelines) in one big graph. 

The way it works is a bit tricky, but it's quite simple to understand. The input of each pipeline can be any data you want, then
you can subscribe some objects to these data and add more and more objects to one big graph, which is super easy to understand
and manipulate.

Each component of the pipeline can be a worker, which communicates with other components through a streaming/queue service.

To run a pipeline (and let data go through the components of the pipeline ), use: <br>
`python manager.py pipelines:run`

It will run all the workers and start to process your queue (using a streaming/queue service).

If you want to run a particular pipeline, use the following command: <br>
`python manager.py pipelines:run app_name.pipeline_name` <br>

Let's dive a bit deeper into the structure of pipelines:


<br>

![image](images/pipeline_1.svg)

<br><br>

---

```python

@app.pipeline()
def full_pipeline(pipeline, value1, value2, value3):

    # DataFrame
    all_at_once = concatinate(data_point1=value1,
                              data_point2=value2,
                              data_point3=value3)

    # DataFrame
    my_flow_result = all_at_once.subscribe_func(my_function, as_worker=True)

    # DataPoint
    flow_data_point = my_flow_result.get("result_data_point")

    # DataFrame
    result = flow_data_point.subscribe_flow(MyFlow2())\
                            .make(result=flow2_result)

    return result

@app.pipeline
def short_pipeline(pipeline, value1, value2, value3):

    # DataFrame
    result = concatinate(data_point1=value1, 
                         data_point2=value2, 
                         data_point3=value3)\
             .subscribe_func(my_function, as_worker=True)\
             .get("result1")\
             .subscribe_flow(MyFlow2())\
             .make(result='result1')\

    return result

```

### Manipulating data inside the pipeline

The input of the stairs pipeline is ["mock"](https://en.wikipedia.org/wiki/Mock_object) values called "DataPoint". It's a representation of  ANY data which will be performed inside the components of the pipeline. 

The mock data will be converted into "real" data as soon as you call the pipeline: <br>

`short_pipeline(value1=1, value2=2, value3=3)` <br>

But this "real" data will be accessible only inside the functions and flows, which you have used in the subscribing methods.
(you can't use "real" values directly inside the pipeline function - this function is just for building pipelines, not for data manipulation)

You can subscribe to DataPoint with the help of a certain function or a Flow component, and the result of this subscription will be a new object called "DataFrame" 
(a kind of the dict object with a key: the DataPoint structure) - it represents the result of your flow.

You can subscribe to both DataPoint or DataFrame. But if you want to extract some values from the DataFrame (the result of your flow) you can use the
`get('value')` method. The result of the "get" method will be DataPoint.

If you want to modify your DataFrame, you can use the `make(value=new_value)` method, and the result will be a new DataFrame.

Now the most interesting part: if you want to combine multiple DataPoints and DataFrames into one DataFrame, you can use 
the `concatenate(value1=data_point, value2=data_point2)` function, which returns the DataFrame with defined arguments. 


Here is an example of a pipeline -> 

As you can see, it's quite simple to define such complex architecture just with 6 code lines.
And it's a bit similar to how we define Neural Networks using [Keras](https://keras.io/).

<br>

![image](images/pipeline_2.svg)
<br><br>

---

```python

@app.pipeline()
def my_pipeline(pipeline, value):
    return value.subscribe_flow(MyFlow(), as_worker=True) \
                .apply_flow(MySecondFlow())\
                .subscribe_consumer(save_result)

```

### How the pipeline flow changes data

The components of the pipeline can accumulate data or completely change/redefine them. 

For this stairs has two definitions: <br>
- subscribe_smths <br>
- apply_smths <br>

to subscribe - to accumulate/update data <br>
to apply - to completely redefine data based on the pipeline component result. 

Take into consideration that the result of each component is a dict object, which
"accumulates" by updating dict keys and values.

<br><br>

![image](images/pipeline_3.svg)

<br>



---

```python

@app.pipeline()
def base_pipeline(pipeline, value):
    return value.subscribe_flow(BaseFlow(**pipeline.config))

@app.pipeline()
def my_pipeline(pipeline, value):
    return value.subscribe_pipeline(base_pipeline)\
                .subscribe_consumer(save_result)
                
@app.pipeline()
def my_pipeline_with_config(pipeline, value):
    config = dict(use_lower=True)
    return value.subscribe_pipeline(base_pipeline, config=config)\
                .subscribe_consumer(save_result)

```

### Call another pipeline

Each stairs pipeline is a worker which handles jobs in a separate process.  
You can use other pipelines inside the pipeline and send data between
them using a queue/streaming service. 

Note: you can't set the worker=False to the pipeline. 

One of the core features inside pipelines is a scalable way to configure them. 
The structure of the app and pipelines is quite friendly to configuration, 
you can set new config values and then call a new pipeline.

`value.subscribe_pipeline(base_pipeline, config=dict(path='/home'))`

These values will be available inside `base_pipeline` as:

`pipeline.config.get('path')`


<br><br><br><br><br><br><br><br><br><br><br>


---

```python

def custom_function(new_value):
    return dict(new_value=new_value*2)


@app.pipeline()
def base_pipeline(pipeline, value):
    return value\
        .subscribe_func(lambda value: dict(new_value=value+1), name='plus_one')\
        .subscribe_func(custom_function, as_worker=True)

```

### Subscribe to any function you want

It's possible to add any function to your data pipeline. 

If you are using the lambda function, it's quite important to set a name, otherwise this function will be a worker, and it will be impossible to recognize it. 

Note: all these functions must return the `dict` object.

<br><br><br><br><br><br><br><br><br><br><br>

---

```python

def custom_function(value):
    return dict(new_value=new_value*2)


@app.pipeline()
def base_pipeline(pipeline, value):
    return value\
        .subscribe_func(custom_function)\
        .add_value(file_path='/tmp/save_data.txt')\
        .subscribe_consumer(save_to_file, as_worker=True)

```

### Custom values

It's possible to add extra values (with real data) into your pipeline. 

It is useful if you want to configure something with constant variables or use the
pipeline config:

`data.add_value(pipeline.config.get('url'))`

<br><br><br><br><br><br><br><br><br><br><br>


## Flow


```python
class MyFlow(Flow)
    @step(None)
    def first_step(self, value):
        return dict(first_step_result=value)

    @step(first_step)
    def second_step(self, first_step_result):
        return dict(second_step_result=first_step_result)
```

The Flow is a low-level component which actually defines the data pipeline. 

The problem with data pipelines builders is that it's not quite easy to 
change/redefine something, also, a great amount of functions
make pipelines like hell of dependencies (luigi is a good example of it). <br>
To solve these problems, we have the FLOW component which can be used to: <br>

- change/redefine/extend your pipeline easily (just use python inheritance)
- configure easily
- understand what's going on
- each Flow can be a worker - the Flow has steps which should be run inside another worker

The Flow represents a data flow graph as a chain of functions called "steps". You can connect these steps simply by defining the "next step" in the decorator:

`@step(next_step, next_step ... )`

The last step in your graph should be defined with the next step set to None.

`@step(None)`


All steps are executed in one "worker" (process).

The structure of the `Flow` class was actually inspired by [stepist](https://github.com/electoronick1/stepist)

---

```python
class MyFlow(Flow):
    @step(None)
    def third_step(self, value, first_step_result, second_step_result):
        # which actually means value * 3
        return dict(flow_result=value+first_step_result+second_step_result)

    @step(third_step)
    def second_step(self, first_step_result):
        return dict(second_step_result=first_step_result)

    @step(second_step)
    def first_step(self, value):
        return dict(first_step_result=value)    
   
```

The input of the next step is the output from the current. 

This idea is based on [stepist](https://github.com/electronick1/stepist)

The result of each step is accumulation, which means that from any low-level steps 
you will be able to get values from high-level steps.

<br><br><br><br><br><br><br><br><br><br>

---

```python
class MyFlow(Flow):
    @step(None)
    def second_step_2(self):
        pass

    @step(None)
    def second_step_1(self):
        pass

    @step(second_step_1, second_step_2):
    def first_step(self):
        # this step will be executed right after
        # root1 and root2
        # data from root1 and root2 will be merge into current step
        pass
```

You can define multiple "next" steps, and this will allow you to build complex 
branchy pipelines, like in the example below ->

![image](images/flow1.svg)
<br><br><br><br><br>

---

```python
from stairs import FLow, step

class MyFlow(Flow):
    def __call__(self):
        result_for_2 = self.calculate_stats(value=2)
        result_for_4 = self.start_from(self.calculate_stats, value=2)

        return result_for_2.validate_data.value + result_for_4.validate_data.value
    
    @step(None)
    def validate_data(self, value):
        value = max(value, 100)

        return dict(value=value)
    
    @step(validate_data)
    def calculate_stats(value):
        return value ** 2
            
```

Now, to execute your flow class, you should define the `__call__` method. 

Inside the `__call__`, you can execute any step from your flow. Then the whole chain 
(a pipeline) of steps will be executed. 

`self.mystep(**kwargs_for_highest_step)`

or

`self.start_from(self.mystep, **kwargs_for_highest_step)`


As a result, you will get data from the last step in your pipeline (with the next_step set to None).


<br><br><br><br><br><br><br><br><br><br><br>

---

```python
from stairs import FLow, step

class MyFlow(Flow):
    def __call__(self, value):
        result = self.start_from(first_step, value=value)
        return {**result.first_step, **result.second_step}
    
    @step(None)
    def second_step(self, value):
        return dict(power3=value ** 3)
    
    @step(second_step, save_result=True)
    def first_step(self, value):
        return dict(power2=value ** 2)
```

It's also possible to customize steps, which should return the data result back. 
Just set the `save_result` flag to True. 

<br><br><br>

![image](images/flow2.svg)

<br><br>

---

```python
from stairs import FLow, step

class MyFlow(Flow):
    def __call__(self, value):
        result = self.start_from(first_step, value=value)
        return {**result.first_step, **result.second_step}
    
    @step(None)
    def second_step(self, value):
        return dict(power3=value ** 3)
    
    @step(second_step, save_result=True)
    def first_step(self, value):
        return dict(power2=value ** 2)


class MyFlow2(MyFlow):

    @step(second_step, save_result=True)
    def first_step(self, value):
        return dict(power4=value ** 4)
```


The flow is a class, which means that we can use inheritance to redefine the logic of certain steps.

It's a very powerful way to extend/change your data pipeline.

<br><br>

![image](images/flow3.svg)

<br><br><br><br><br><br>

---

```python
from stairs import FLow, step

class MyFlow(Flow):
    def __call__(self, value):
        result = self.start_from(first_step, value=value)
        return {**result.first_step, **result.second_step}

    @step(None)
    def second_step(self, value):
        return dict(power3=value ** 3)
    
    @step(second_step, save_result=True)
    def first_step(self, value):
        return dict(power2=value ** 2)


class MyFlow2(MyFlow):

    def __reconect__(self):
        self.second_step.set_next(self.third_step)
        self.second_step.set(save_result=True)
        
    @step(None)
    def third_step(self, value):
        return dict(power4=value ** 4)
```

The inheritance also allows you to reconnect certain steps and change the Flow structure.

It's possible to add a new step to the top, insert it in the middle or add the "save_result" flag.

<br><br>

![image](images/flow4.svg)

<br><br>




## Producer

The producer is a set of components for reading any type of data and then for calling a pipeline to handle your data.

This component will populate your pipeline with "real" data which you can read from any source.



So far, we have two types of the producer components: <br>

- a simple iterator <br>
- a worker iterator - a way to read your data safely <br>


When you defined the producer, you can call it from the shell using manager.py:
` python manager.py producer:process `

<br><br>

---


```python

@app.producer(pipeline.my_pipeline)
def read_file():
    with open(FILE_PATH, "w") as f:
        for row in f:
            yield row

```

### Simple Producer

It's a function of the iterator which yields data to the pipeline. You can run this "producer" from the console:<br>
`python manager.py producer:process`

It simply goes through all the items that the producer yields and sends them to the streaming/queue service, which then goes to the pipeline. 

To prevent overfitting of your streaming service, you can set a "limit". When the producer reaches this limit, it will sleep for a while. 

<br><br><br>

---


```python

@app.worker_producer(pipeline.my_pipeline)
def read_database():
    for i in range(AMOUNT_OF_ROWS / BATCH_SIZE):
        yield read_batch(i)

def read_batch(batch_id):
    interval = (batch_id*BATCH_SIZE, (batch_id+1)*BATCH_SIZE)
    cursor.execute("SELECT * FROM table where id>%s and id<%s" % interval)
    for row in cursor:
        yield row

```

### Worker Producer

It's a parallel/distributed way to populate your pipeline with data. It's almost 99% safe, as it has smart features to prevent data loss or duplication.

The way it works is a bit more complicated then the way the simple producer works, but if you know something about "batch" processing, everything will be simple to you.

The idea is to split your data into batches and read each batch independently. If the whole batch is read successfully, it goes to the pipeline. Internally, the worker producer
uses bitmaps to track the status of all your batches, and the only way you can lose your data is a fail with the redis.

The worker producer has two states: the first  is initializing. It checks the number of batches and creates all necessary meta information. 
To initilaze the worker_producer, run: <br>
`python manager.py producer:init`

It must be executed only once.

<br>

To start reading the process, you can run: <br>
`python manager.py producer:process`

As it was mentioned earlier, the worker_producer is a parallel way to read your data. So, if you want more processes, just run the command above multiple times. 
It will read batches from the `producer:init` command.

It will prevent the queue from overfitting, similar to the simple_producer. 

<br><br><br>

## Consumer

```python
@app.consumer()
def save_to_redis(**data):
    redis.set(json.dumps(data))
```

The consumer is a set of components for writing/saving your data to any type of store or for changing the global state of your system.

You are free to write/save your data inside the Flow component, but the consumer is not only about saving. It is also a way to accumulate all
data to one place, and Stairs has 3 types of consumers:

- "a simple consumer" is a simple function which should not return any data. It's useful for saving data to the data store.
- "standalone_consumer" is a function which can be called as a separate process. It's useful for writing data to a file or for accumulating them inside one process. 
- "consumer_iter" is a function which yields data from the pipeline. It's useful when you want to train the neural network. It also needs data generator. 


Here, on the right, is an example of "a simple consumer" -> 
<br><br>


--- 

```python
import json

f = open("file.txt", "w")

@app.standalone_consumer()
def write_to_file(**data):
    f.write(json.dumps(data))
    f.write("\n")

```


The Standalone consumer is a type of a consumer which will NOT execute automatically.

To execute it and process data inside, you need to run a special command:

`python manager.py consumer:standalone app.write_to_file`


It is useful if you need to write data using one process only (for example, in the case of file writing).
 

--- 


```python

@app.consumer_iter()
def x_data_for_nn(**data):
    return data

@app.consumer_iter()
def y_data_for_nn(**data):
    return data


# Example of pipeline
@app.pipeline()
def prepare_data_for_nn(pipeline, data)
    result_data = data.subscribe_flow(Flow())

    x_consumer = result_data.get('x')\
                            .subscribe_consumer(x_data_for_nn)
    y_consumer = result_data.get('y')\
                            .subscribe_consumer(y_data_for_nn)

    return concatinate(x=x_consumer, y=y_consumer)


if __name__ == "__main__"

keras.Model().fit(x=x_data_for_nn(),
                  y=y_data_for_nn())


```


The Consumer as a separe process.

If you want to train the neural network and use the output from your pipeline as a train set, you can use:<br>
The `@consumer_iter()` component allows you to read data from the streaming/queue service directly to your function.



## APP Config

```python

# app_config.py

from stairs import App

app = App(name="myapp")

app.config = {"use_validation": False}

my_pipeline_config = {"validation_flow": ValidationFLow()}

# pipelines.py

@app.pipeline(config=my_pipeline_config)
def my_pipeline(pipeline, value):
    result = value.subscribe_flow(pipeline.config.validation_flow)

    if app.config.use_validation:
        result.subscribe_flow(ValidationFLow())

    return result


# another project

app.config.use_validation = True

```

It's a place where you can set up your app. 

The App config allows defining the app settings, which are useful if you want to share your app with the world.

It's also possible to define the pipeline config as in the example -> 

<br>



#Examples

##ETL example: hacker news

[github hacker_news](https://github.com/electronick1/stairs_examples/tree/master/hacker_news)<br>

The idea here is to extract data from a certain source (in this case it's google cloud), to change them somehow and save them in a elegant format 
(for example, for creating charts later or for building neural networks).

You can start exploring this project from `producers.py` inside the hacker_new app. 
It's quite insignificant where we read data in this module, and what format we use as a result.

Then each producer of `pipelines.py` will send data to the pipelines, in our case we have two of them:<br>
- `cleanup_and_save_localy` - makes a basic text cleanup and filtering
- `calculate_stats` -  based on "clean" data, it calculates stats we need

The next (and the last) `consumers.py` - a place where all data come at the end of the pipeline.

<br>

##ML example: bag of words

[github bag of words](https://github.com/electronick1/stairs_examples/tree/master/bag_of_words)<br>

Here, we try teaching the neural network to solve [kaggle task "Bag of Words Meets Bags of Popcorn"](https://www.kaggle.com/c/word2vec-nlp-tutorial)

This example is based on [this repo](https://github.com/wendykan/DeepLearningMovies/), and it's a kind of the copy-paste solution, but for much better representation.

What does "better representation" mean? 

If you look inside this repo, it's just a plain code. 
If you want to make calculations in a parallel way, it's not very trivial to do.
Also, if you want to change something, it's not easy to undestand all the changes of the data flow.

Stairs solves all these problems:

- It makes calculations in the parallel by default
- You can easily understand what's going on inside the `pipelines.py`
- It is super easy to change something (just redefine certain methods in the FLow classes).


#Features 

## Inspect the status of your queues

```bash
python manager.py inspect:status app_name

# Queue: cleanup_and_save_localy
# Amount of jobs: 10000
# Queue decreasing by 101.0 tasks per/sec


python manager.py inspect:monitor app_name

# Queue: cleanup_and_save_localy
# Amount of jobs: 3812
# New jobs per/sec: 380.4
# Jobs processed per/sec: 10.0


```

There are two types of inspection:

- inspect:status - returns the current amount of jobs/tasks in your queue and basic information about the speed (not very accurate)
- inspect:monitor - returns the amount of jobs added and processed per sec. It's accurate, but works only for the redis (so far)


## Shell



```bash
python manager.py shell
```

```python

In [1]: from stairs import get_project

In [2]: get_project().get_app("hacker_news")
Out[2]: <stairs.core.app.App at 0x105fa7d30>

In [3]: get_project().get_app("hacker_news").components.producers
Out[3]:
{'read_google_big_table': <stairs.core.producer.Producer at 0x1257c4828>}

In [4]: producer = get_project().get_app("hacker_news").components.producers.get("read_google_big_table")

In [5]: producer.process()

```

It's possible to run all producers, pipelines, consumers using ipython.


## Change the queue/streaming server

```python
# in manager.py 

from stepist import App
from stairs.services.management import init_cli
from stairs.core.project import StairsProject

if __name__ == "__main__":
    stepist_app = App()
    celery = Celery(broker="redis://localhost:6379/0")
    app.worker_engine = CeleryAdapter(app, celery)

    stairs_project = StairsProject(stepist_app=stepist_app)
    stairs_project.load_config_from_file("config.py")
    init_cli()
```


Stairs is based completely on stepist. You can just define a new stepist app with a new "broken" engine, 
and your stairs project is ready to go. 

[Stepist](https://github.com/electronick1/stepist)


## Admin panel

```bash
python manager.py admin
```

It's a way to visualize all your pipelines, to see the status of queues and information about each component of the pipeline.


![image](images/admin.png)

<aside class="notice">
Under development
</aside>

#FAQ


## What is the reason behind apps?

```python

# example of app config

app = App("myapp")
app.config.update(
    train_data_path='/home/train.data'
)


# example of pipeline config

pipeline_config = dict(cleanup_flow=CleanUpText())

@app.pieline(config)
def external_pipeline(pipeline, value):
    return value.subscribe_flow(pipeline.config.cleanup_flow)

# in some other app, you can now make like this:
def my_pipeline(pipeline, value):
    config = dict(cleanup_flow=MYCleanup())
    return value.subscribe_pipeline(external_pipeline, config=config)

# And it ^ will be executed with your "clean up" flow 

```

The main idea is to simplify reusing external solutions.

The Data-Science world is non-standardized right now, and stairs is trying to create the enviroment
where reusing someone's approach will be easy and scalable for you. 

For example, each app has a config. It allows you to set different config variables to external apps (inside your app/project).

Each pipeline has a config. It allows you to redefine certain components of the pipeline or change any logic you want. 

A good example of configs is [here](https://github.com/electronick1/stairs_examples/blob/master/bag_of_words/word2vec/app_config.py) 
or [here](https://github.com/electronick1/stairs_examples/blob/master/bag_of_words/word2vec/pipelines.py#L13)



## Why does the pipeline builder use "mocked" data ?

The pipeline builder `app.pipeline()` exists only to create a pipeline, to configure it, 
and return the "Worker" object which then will be executed by using a streaming/queue service. 

At the moment we are building a pipeline, we know nothing about real data. Due to this fact, we use certain mock objects. When you run the producer, it will populate these 'mock' objects, and the components of the pipeline will work with real data.


## What should data return to each component of the pipeline?

Except the "flow_generator", all components must return `dict` as a result. Where we have key:value defined.


It's used for combining "real" data with "mock" values. 





