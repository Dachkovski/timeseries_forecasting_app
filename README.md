# AI Gold Price Predictor

![](https://www.artnet.de/WebServices/images/ll00357lldm1VJFgETeR3CfDrCWvaHBOcBubF/hajime-sorayama-sexy-robot-gold-be@rbrick-1000.jpg)

## Introduction

AI Gold Price Predictor is an web app that predicts the price of Gold for the next trading day. The prediction is made by a state-of-the-art machine learning model based on a transformer neural network. Its purpose is to help traders with their investment decisions with a trustworthy and free open source software.

The app delivers:
* [x] Free trading helper 
* [x] State-of-the-art AI with significant prediction accuracy
* [x] Easy to use and modify software architecture 
* [x] Based on tensorflow serving which supports serving and inference of multiple models with GPU acceleration

The main challenge as for all trading bots is to predict the direction of the price action movement. To find out how the transformer network performs in this field, I chose  "accuracy" as a metric. Here a success is defined that the direction of the prediction is the same with the actual value for the according trading day.

## Installation

The web app consists of two components: 
A frontend with the web user interface, and a backend running a model server with an administrator user interface called 'simple tensorflow serving' [source code](https://github.com/dachkovski/simple_tensorflow_serving)

Install the backend server with:

```bash
cd simple_tensorflow_serving

python ./setup.py install

python ./setup.py develop

bazel build simple_tensorflow_serving:server

```

The frontend app doesnt need an installation. But it needs some packages that are installed with:

```bash
pip install -r ../frontend/requirements.txt

```


## Quick Start

Start the backend server with the TensorFlow [SavedModel](https://www.tensorflow.org/programmers_guide/saved_model).

```bash
cd ../simple_tensorflow_serving
simple_tensorflow_serving --model_base_path="./models/transformer"
```

Check out the admin dashboard in [http://127.0.0.1:8500](http://127.0.0.1:8500) in web browser.
 
![dashboard](https://github.com/Dachkovski/simple_tensorflow_serving/blob/9064944828d35f1c30e2dcd82f409802ad5f59d3/images/dashboard.png)

Start the frontend web app with:

```bash
python ./frontend/predictor.py
```

Check out the web app in [http://127.0.0.1:3001](http://127.0.0.1:3001) in web browser.
 
![frontend](./frontend/static/images/frontend.png)

## Software Architecture
The app (client) uses Tensorflow serving to deploy, serve and query the model.

The following software architecture description stems from [Source](https://github.com/llSourcell/Make_Money_with_Tensorflow_2.0).
![serving_architecture](./frontend/static/images/serving_architecture.svg)

TensorFlow Serving is a flexible, high-performance serving system for machine learning models, designed for production environments.
TensorFlow Serving makes it easy to deploy new algorithms and experiments, while keeping the same server architecture and APIs.
TensorFlow Serving provides out of the box integration with TensorFlow models, but can be easily extended to serve other types of models.
![tf_serving](./frontend/static/images/tf_serving.jpg)

Wait, why use Serving instead of a regular web server framework like Flask or Django?
TensorFlow-Serving allows developers to integrate client requests and data with deep learning models served independently of client systems.
Benefits of this include clients being able to make inferences on data without actually having to install TensorFlow or even have any contact with the actual model, and the ability to serve multiple clients with one instance of a model.
Although we could wrap a simple model in an API endpoint written in a Python framework like Flask, Falcon or similar But there are good reasons we don’t want to do that:

#### Reason #1 - TF Serving is faster
If the model(s) are complex and run slowly on CPU, you would want to run your models on more accelerated hardware (like GPUs). Your API-microservice(s), on the other hand, usually run fine on CPU and they’re often running in “everything agnostic” Docker containers. In that case you may want to keep those two kinds of services on different hardware.
#### Reason #2 - TF Serving is more space efficient
If you start messing up your neat Docker images with heavy TensorFlow models, they grow in every possible direction (CPU usage, memory usage, container image size, and so on). You don’t want that.
#### Reason #3 - Its going a version control system built in
Let's say your service uses multiple models written in different versions of TensorFlow. Using all those TensorFlow versions in your Python API at the same time is going to be a total mess.

You could of course wrap one model into one API. Then you would have one service per model and you can run different services on different hardware. Perfect! Except, this is what TensorFlow Serving ModelServer is doing for you. So don’t go wrap an API around your Python code (where you’ve probably imported the entire tf library, tf.contrib, opencv, pandas, numpy, …). TensorFlow Serving ModelServer does that for you. 


## Model Architecture

As estimator a transformer neural net was taken from [Kaggle](https://www.kaggle.com/shujian/transformer-with-lstm) and adopted to predict Gold prices. It was trained for 300 epochs.

![transformer](./frontend/static/images/transformer.png)

## Data Analysis

As data source for obtaining the price data the yahoo finance API is implemented in the web app. With every call the app queries the API and gets current price data. 
```
                   Open         High          Low        Close  Volume  ...
Date                                                                     
2020-11-06  1940.800049  1958.800049  1940.800049  1950.300049     304   
2020-11-09  1955.599976  1963.199951  1847.099976  1853.199951     745   
2020-11-10  1879.300049  1885.300049  1871.199951  1875.400024     276   
2020-11-11  1878.800049  1878.800049  1855.500000  1860.699951     222   
2020-11-12  1869.000000  1878.500000  1866.599976  1872.599976     220  
```
Then the app calculates a daily Mid price, that is the mean of High and Low daily prices. 

![gold_chart](./frontend/static/images/gold_chart.png)

## Prediction Methodology
The transformer model expects the last 60 days of scaled data (between 0 and 1) and outputs the prediction for day 61. Therefore the data is rescaled with a moving window:

![scaled_chart](./frontend/static/images/scaled_chart.png)

Each moving window is saved in a new DataFrame as input data point for the model.


## Results Signal Prediction

The following chart shows the model performance on the held back test data set: 
![prediction_test](./frontend/static/images/prediction_test.png)

The accuracy of the prediction is: 0.55. This means, that the prediction accuracy of the model is 5% above coincidence.

## Conclusion

The prediction of stock prices is one of the most difficult tasks in economics and especially in machine learning. Furthermore, successful models tend to become inaccurate with time, so that temporary gains not a guarantee to succeed in the future. 

Thus one improvement could be the implementation of continouus learning model pipeline, which adopts the models to new incoming data. 
Besides, the model uses only the time series of gold prices. The accuracy may be improved by including more features (as the stocequity and stock market is a strongly interdependent system) and fundamental macro economics data (inflation, FED balance sheet, etc.) 

