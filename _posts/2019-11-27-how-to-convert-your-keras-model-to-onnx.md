---
layout: base
title: "How to Convert Your Keras Model to ONNX"
categories: [programming]
tags: [Keras, machine learning, ONNX]
---

# How to Convert Your Keras Model to ONNX

## Intuition
I love Keras for its simplicity. With about 10 minutes, I can build a
deep learning model with its sequential or functional API with elegant
code. However, Keras always loads its model in a very slow speed. Moreover,
I have to use another deep learning framework because of system constrains
or solely my boss tells me to use that framework. Though I can convert
my Keras model to other frameworks with other guy's script, I still convert
my model to ONNX for trying its claimed interoprability in the AI tools.

## What is ONNX?
[ONNX](https://onnx.ai/) is an abbreviation of "Open Neural Network Exchange".
The goal of ONNX is to become an open format to represent deep learning models
so that we can move model between frameworks in ease, and it is created by 
Facebook and Microsoft. 

## Converting Your Keras Model to ONNX
1. Download the example code from [my GitHub](https://github.com/onnx/keras-onnx)
2. Download the pre-trained weight from [here](https://drive.google.com/file/d/1ouJ8xZzi6x2cEkojS3DC1Wy77zjBGP1c/view)
3. Type the following commands to set up
```
$ conda create -n keras2onnx-example python=3.6 pip
$ conda activate keras2onnx-example
$ pip install -r requirements.txt
``` 

4. Run this command to convert the pre-trained Keras model to ONNX
```
$ python convert_keras_to_onnx.py
```

5. Run this command to inference with ONNX runtime
```
$ python main.py 3_001_0.bmp
```

It should output the following messages in the end:
```
...
3_001_0.bmp
    1.000  3
    0.000  37
    0.000  42
    0.000  14
    0.000  17
```

## Some Explanations
`convert_keras_to_onnx.py` converts a Keras `.h5` model to ONNX format, i.e., `.onnx`.
The code of it is shown below:
```python=
from tensorflow.python.keras import backend as K
from tensorflow.python.keras.models import load_model
import onnx
import keras2onnx

onnx_model_name = 'fish-resnet50.onnx'

model = load_model('model-resnet50-final.h5')
onnx_model = keras2onnx.convert_keras(model, model.name)
onnx.save_model(onnx_model, onnx_model_name)
```

There are some points for converting Keras model to ONNX:
1. Remember to import `onnx` and `keras2onnx` packages.
2. `keras2onnx.convert_keras()` function converts the keras model to ONNX object.
3. `onnx.save_model()` function is to save the ONNX object into `.onnx` file.

`main.py` inferences fish image using ONNX model. 
And I paste the code in here:
```python=
from tensorflow.python.keras import backend as K
from tensorflow.python.keras.models import load_model
from tensorflow.python.keras.preprocessing import image
import sys
import numpy as np
import keras2onnx
import onnxruntime

IMAGE_SIZE = 224

img_path = sys.argv[1]

#net = load_model('model-resnet50-final.h5')
#onnx_net = keras2onnx.convert_keras(net, net.name)
onnx_model = 'fish-resnet50.onnx'

# 41 + 1 classes
cls_list = ['10', '11', '12', '13', '14', '15', '16', '17', '18', '19',
    '1', '20', '21', '22', '23', '24', '25', '26', '27', '28', '29',
    '2', '30', '31', '32', '33', '34', '35', '36', '37', '38', '39',
    '3', '40', '41', '42', '4', '5', '6', '7', '8', '9']

# image preprocessing
img = image.load_img(img_path, target_size=(IMAGE_SIZE, IMAGE_SIZE))
x = image.img_to_array(img)
x = np.expand_dims(x, axis=0)

# runtime prediction
#content = onnx_net.SerializeToString()
#sess = onnxruntime.InferenceSession(content)
sess = onnxruntime.InferenceSession(onnx_model)
x = x if isinstance(x, list) else [x]
feed = dict([(input.name, x[n]) for n, input in enumerate(sess.get_inputs())])
pred_onnx = sess.run(None, feed)[0]
pred = np.squeeze(pred_onnx)
top_inds = pred.argsort()[::-1][:5]
print(img_path)
for i in top_inds:
    print('    {:.3f}  {}'.format(pred[i], cls_list[i]))
```
and there are some outlines when inferencing:
1. Remember import `onnxruntime` package.
2. `onnxruntime.InferenceSession()` function loads ONNX model.
3. `run()` function in line 34 predicts the image and return the predicted result.
What's more, `sess.run(None, feed)[0]` returns the 0-th elemnent as a numpy matrix.
4. `np.squeeze(pred_onnx)` squeezes the numpy matrix to numpy vector, i.e., remove
the 0-th dimension, so that we can get the probabilities of each class.

## Inference Time
### Total Inference Time (load model + inference)
Some reports say ONNX run faster both mode loading time and inferencing time.
Hence, I make an inference time experiment on my laptop in this section.

The hardware for this experiment is shown below:
- CPU: Core i5-3230M
- RAM: 16GB

The software and packages are shown here:
- OS: CentOS 7.6
- Programming language: Python 3.6
- Packages
    - keras version 2.2.4
    - tensorflow version 1.13.1
    - onnxruntime 
    
Each inferencing runs three times to eliminate the erro of other factors, e.g.,
context switching.

For inferencing with Keras, my computer runs with the following results:
```
$ time python resnet50_predict.py 3_001_0.bmp
# run for three times
...
real    0m37.801s
user    0m37.254s
sys 0m1.590s
...
real    0m35.558s
user    0m35.838s
sys 0m1.362s
...
real    0m36.444s
user    0m36.542s
sys 0m1.418s
```
The inferencing time is about (37.081+35.58+36.444)/3 = 36.37 seconds (round to the 
second digit after decimal point).

As a contrary, inferencing with ONNX runtime shows:
```
$ time python main.py
# run three times
...
real    0m2.576s
user    0m2.919s
sys 0m0.759s
...
real    0m2.530s
user    0m2.931s
sys 0m0.700s
...
real    0m2.560s
user    0m2.944s
sys 0m0.710s

```
Whoa! What a huge improvement! The inferencing tims is about (2.576+2.530+2.560)/3 = 
2.56 seconds.

> The code infercing with Keras can be found [on my GitHub repo](https://github.com/Cuda-Chen/fish-classifier/tree/master/cnn).
>

### Inference Time (only inference)
> Edit: One of my friend said I should test only inference time between Keras and ONNX
> because we load model once in practice. As a result, I will test only inference
> time between Keras and ONNX, and I will split to two parts: 
> 1. Keras (with TensorFlow installed by `pip`) v.s. ONNX
> 2. Keras (with TensorFlow installed by `conda`) v.s. ONNX

Of course, I write `comparison.py` in order to make comparison test which is 
shown below:
```pyton=
from tensorflow.python.keras import backend as K
from tensorflow.python.keras.models import load_model
from tensorflow.python.keras.preprocessing import image
import sys
import numpy as np
import onnxruntime
import time

IMAGE_SIZE = 224
loop_count = 10
img_path = sys.argv[1]

# load Keras and ONNX model
net = load_model('model-resnet50-final.h5')
onnx_model = 'fish-resnet50.onnx'
sess = onnxruntime.InferenceSession(onnx_model)

# 41 + 1 classes
cls_list = ['10', '11', '12', '13', '14', '15', '16', '17', '18', '19',
    '1', '20', '21', '22', '23', '24', '25', '26', '27', '28', '29',
    '2', '30', '31', '32', '33', '34', '35', '36', '37', '38', '39',
    '3', '40', '41', '42', '4', '5', '6', '7', '8', '9']

# image preprocessing
img = image.load_img(img_path, target_size=(IMAGE_SIZE, IMAGE_SIZE))
x = image.img_to_array(img)
x = np.expand_dims(x, axis=0)

# Keras predtion
start_time = time.time()
for i in range(loop_count):
    pred = net.predict(x)[0]
print("Keras inferences with %s second in average" %((time.time() - start_time) / loop_count))

# ONNX prediction
x = x if isinstance(x, list) else [x]
feed = dict([(input.name, x[n]) for n, input in enumerate(sess.get_inputs())])

start_time = time.time()
for i in range(loop_count):
    pred_onnx = sess.run(None, feed)[0]
print("ONNX inferences with %s second in average" %((time.time() - start_time) / loop_count))

```

You can see I run each inference method with 10 times and take the average time,
and I run `comparison.py` three times for easing the error.

#### Keras (with TensorFlow installed by `pip`) v.s. ONNX
The comparison is shown below:
```
$ python comparison.py
...
Keras inferences with 0.8759469270706177 second in average
ONNX inferences with 0.3100883007049561 second in average
...
Keras inferences with 0.8891681671142578 second in average
ONNX inferences with 0.313812255859375 second in average
...
Keras inferences with 0.9052883148193359 second in average
ONNX inferences with 0.3306725025177002 second in average
```

We find that Keras inference needs (0.88+0.87+0.91)/3 = 0.87 seconds, while ONNX
inference need (0.31+0.31+0.33)/3 = 0.32 seconds. That's a speedup ratio of 
0.87/0.32 = 2.72x between ONNX and Keras.

#### Keras (with TensorFlow installed by `conda`) v.s. ONNX
Wait a minute! `pip install tensorflow` installs TensorFlow without optimization
of Intel's processor. So let's remove TensorFlow first then install it via `conda`
(the version I install is `1.13.1`).

Then run `comparison.py` again:
```
$ python comparison.py
...
Keras inferences with 0.9810404300689697 second in average
ONNX inferences with 0.604683232307434 second in average
...
Keras inferences with 0.8862279415130615 second in average
ONNX inferences with 0.6059059381484986 second in average
...
Keras inferences with 0.9496192932128906 second in average
ONNX inferences with 0.5927849292755127 second in average
```

We find Keras takes (0.98+0.89+0.95)/3 = 0.94 seconds to inference. Compared to ONNX,
it spend (0.60+0.61+0.59)/3 = 0.6 seconds for inferencing. That's a speedup of 0.94/0.6
= 1.57x. Interestingly, both Keras and ONNX become slower after install TensorFlow via
`conda`.

## Conclusion
In this post, I make an introduction of ONNX and show how to convert your Keras 
model to ONNX model. I also demonstrate how to make prediction with ONNX model.
Hope you enjoy this post!
