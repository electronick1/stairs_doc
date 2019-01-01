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

Stairs - It's a simple tool which allows you to juggle with data. 

You can build data pipelines and make parallel/async/distributed calculations for most of your data related tasks.

Stairs available on Python3, but you can handle data calculation on any languages. And even use any of Streaming/Queue services which you want. 

It's easy to start, test all your ideas/hypotheses in rocket way and it will be ready for production in any moment of time without special magic. 

Get started with Installation and then get an overview with "Get started". 

# Installation


```python
# install redis https://redis.io/topics/quickstart
# sudo apt-get install redis
# brew install redis

pip install stairs-project
```



> It's recommend to use latest python 3 version

Just make `pip install stairs-project` to install all python dependenses.

Stairs require redis, for storing statistic and some meta information, 
even tho you will use different streaming/queue service. 

<aside class="notice">
You must install <code>redis</code> for stairs.
</aside>

# Mission

## Data Pipelines

> ![image](images/data_pipeline.png)

The main Stairs focus is data pipelines. It's a framework which helps you 
build and manipulation of data through the data flow graph. 

In the same way you can associate it with MVP framework (like django) but for data pipelines.
Different layears of abstractions allows you to build any kind of data flow graph,
and easily understand what's going on in your system. 



## Parallel/Async/Distributed

> ![parallel](images/parallel.png)

Each component of stairs data pipeline could be a "worker" - separate process which allows you
to process data in a parallel way. 

There is interesting wiki article about workers/jobs -> [Wiki](https://en.wikipedia.org/wiki/Job_(computing))

Stairs framework focusing on speed and light, and speed of your "workers" mostly limited by your streaming/queues service.


## For data-science and data-engineering with love

> ![ds_en](images/ds_en.svg)

Data-science and data-engineering growing  fast, and it's  hard to be expert on everything
in the same time. 

For example for train ML models you should spend about 80% of time to process data -- how fast 
you able to do that and test all hypotheses in your head directly influence your final result.

Stairs allows data scientist  "scalable" solutions without high-level data engineering skills.

- Data scientist could focus only on data processing
- Data engeneer could focus only on storing and moving data (beetween pipeline components)

Data pipeline paradigm allows you to separate this two subjects, and speed up working with data.


#Get started


##Project

```shell
stairs-admin project:new name
```

> ![project](images/project.svg)



When you done with installation let's try to kick start your first stairs project.

For creating default project template just use following command:

`stairs-admin project:new name`


This command will generate basic project stucture with one app inside.<br>
It's similar to django maner, but you a free to change this structure in any way you like. 

Project has config file and "manager.py".

"manager.py" - in django maner allows you to read config, detect apps and execute shell commands.

<br><br><br><br><br><br>



##App

```shell
stairs-admin app:new name
```


> ![app](images/app.svg)


App - it's a way to generalize different approaches to one similar form - why? Because right now
data-science approaches too scattered and it's hard to understand what's going on when you have
a ton of maths and algorithms around. 


Each app has following components:

- pipeline - represents a data flow graph and determines how the data will be processed. Each pipeline consists of multiple smaller components like "Flow" (Data flow).  

- flow (Data Flow) - set of functions (called [steps](https://en.wikipedia.org/wiki/Job_(computing))) which could change/filter/populate your data.

- producer - function which helps you to read the source (file, database ...) and then follow it to data pipeline.

- consumer - function which writes data to data store or change "global state".


To create new "default" app use following command:

`stairs-admin app:new name`



```python
from stairs import App

app = App(name="my_app")

```

To define your app you should initialize App object with name and config (More about app config in "App components" section).

If you want to add new app to the project, populate `apps` variable in config file or use `StairsProject.add_app(app)`

<br>

![image](images/app_2.svg)

<br>

#App components

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

Flow - It's a low level component which actually define's data pipeline. 

Flow represent's data flow graph as a chain of functions called "steps". You can connect those steps simply define "next step" in decorator:

`@step(next_step, next_step ... )`

The last step in your graph should be define with next step set to None.

`@step(None)`


All steps executed in one "worker" (process).

---

```python
class MyFlow(Flow)
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

The input for next step is an output from current. 

Result of each step is accumulating, this means that from any low-level steps you will be able to get values from high-level steps:

<br><br><br><br><br><br><br><br><br><br>

---

```python
class MyFlow(Flow)
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

You can define mutiple "next" steps and this will allow you to build complex branched out pipelines, like on example bellow ->

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


You can call any step from your flow. Then whole chain (pipeline) of steps will be executed. 

`self.mystep(**kwargs_for_highest_step)`

or

`self.start_from(self.mystep, **kwargs_for_highest_step)`


As a result you will get data from last step in your pipeline (with next_step set to None).


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

It's also possible to customize steps which should return back result data. 
Just set `save_result` flag to True.

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


Flow - it's a class, this means we could use inheritance to redefine some steps logic.

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
        self.second_step.save_result = True

    @step(None)
    def third_step(self, value):
        return dict(power4=value ** 4)
```

Inheritance also allows you to reconnect some steps and change Flow structure.

It's possible to add new step to the top, insert in the middle or add "save_result" flag.

<br><br>

![image](images/flow4.svg)

<br><br>

## Pipeline

```python

@app.pipeline()
def my_pipeline(pipeline, value):
    return value.subscribe_flow(MyFlow(), as_worker=True) \
                .subscribe_flow(MySecondFlow())\
                .subscribe_consumer(save_result)

```



Pipeline - it's a way to connect multiple flows (or others pipelines) into one big graph/pipeline. 

Each pipeline component - could be a worker, which communicates with anothers components throught streaming/queue service.

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
    my_flow_result = all_at_once.subscribe(MyFlow(), as_worker=True)

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
             .subscribe(MyFlow(), as_worker=True)\
             .get("result1")\
             .subscribe_flow(MyFlow2())\
             .make(result='result1')\

    return result

```

### Manipulating data inside pipeline

Input of stairs pipeline is a ["mock"](https://en.wikipedia.org/wiki/Mock_object) values called "DataPoint" - it's representation of the data which will be executed inside pipeline components. 

You can subscribe DataPoint by some Flow component and result of this subscribtion will be new object called "DataFrame" 
- it's represent result of your flow.

You could subscribe both DataPoint or DataFrame. But if you want to extract some values from DataFrame (result of your flow) you can use
`get('value')` method. Result of "get" method will be DataPoint.

If you want to modify your DataFrame you can use `make(value=new_value)` method and result will be new DataFrame.

Now one of the most interesting part: If you want to combine multiple DataPoints and DataFrame into one DataFrame you can use 
`concatinate(value1=data_point, value2=data_point2)` function - which return DataFrame with defined arguments. 


Here an example of pipeline -> 

As you can see It's quite simple to difine such complex and hard architecture just with 6 lines of code.
And it's a bit similar to how we define Neural Networks using [Keras](https://keras.io/)

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

### How flow change data

Unlike Flow, Pipeline components could accomulate data or completely change/redefine it. 

For this stairs has two defenitions: <br>
- subscribe_smths <br>
- apply_smths <br>

subscribe - accomulate/update data <br>
apply - completely redefine data based on pipeline component result. 

<br><br>

![image](images/pipeline_3.svg)

<br>



---

```python

@app.pipeline()
def base_pipeline(pipeline, value):
    return value.subscribe_flow(BaseFlow())

@app.pipeline()
def my_pipeline(pipeline, value):
    return value.subscribe_pipiline(base_pipeline)\
                .subscribe_consumer(save_result)

```

### Call another pipeline

Inside your pipeline you can use any other pipelines. 

Note: that all pipelines - is a workers, and you can't set worker=False to pipeline. 

<br><br><br><br><br><br><br><br><br><br><br>



## Producer

Producer - it's a set of components for reading any typos of data and then call pipeline to handle your data.

This component will populate your pipeline by "real" data which you can read from any source you want. 



So far, We have two types of producer components: <br>

- simple iterator <br>
- worker iterator - the way to sefly read your data <br>


When you definied producer you can call it from shell using manager.py:
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

It's a iterator function which yields data to pipeline. You can run this "producer" from console:<br>
`python manager.py producer:process`

It's simply goes by all items which yields producer, and send them to streaming/queue service which then goes to pipeline. 

To prevent overfitting of your streaming service you can set "limit" and at the moment when producer reach this limit it will sleep for a while. 

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

It's a parallel/distributed way to populate your pipeline with data. It's almost 99% safe - it has smart features to prevent data lose or duplication.

The way how it works a bit complex then simple producer, but if you know something about "batch" processing - everything will be simple for you.

The idea to split your data by batches and read each batch independally. If whole batch was read successfuly it goes to pipeline. Internally Worker producer
using bitmaps to track all your batches status, and only way when you can lose your data is a fail with redis.

Worker producer has two states: first initializing - it's just checking amount of batches and creates all needed meta information. 
To initilaze worker_producer run: <br>
`python manager.py producer:init`

It should be executed only once.

<br>

Then to start reading process you can run: <br>
`python manager.py producer:process`

As it was mentioned before worker_producer - it's a parrallel way to read your data. So if you want more processes just run command above multiple times. 
It will read batches from `producer:init` command.

In the same way like simple_producer it will prevent queue from overfitting. 

<br><br><br>

## Consumer

```python
@app.consumer()
def save_to_redis(**data):
    redis.set(json.dumps(data))
```

Consumer - it's a set of components for writing/saving your data to any type of store or change global state of your system.

You are free to write/save your data inside Flow component, but consumer it's not only about saving - it's a way to accumulate all
data to one place, and stairs has 3 types of consumers:

- "simple consumer" - just a simple function which should not return any data. Useful for saving data to data store.
- "standalone_consumer" - function which could be called as a separate process. Useful for writing data to a file or accumulating it inside one process for something. 
- "consumer_iter" - function which yields data from pipeline. Useful when you want to train neural network and needs data generator. 


Here on the right example of "simple consumer" -> 
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


Standalone consumer it's type of consumer which will NOT execute automatically.

To execute it and process data inside, you need to run special command:

`python manager.py consumer:standalone app.write_to_file`


It useful when you need to write data using one process only (for example in case of file writing).
 

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


Consumer as a separe process.

If you want to train neural network, and use output of your pipeline as a train set, you can use:<br>
`@consumer_iter()` component which allows you to read data from streaming/queue service directly to your function.



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

It's a place when you can setup your app. 

App config allows to defene app settings which useful when you want to share your app with the world.

Here it's also possible to define pipeiline config like on example -> 

<br>



#Examples

<br>

#FAQ



