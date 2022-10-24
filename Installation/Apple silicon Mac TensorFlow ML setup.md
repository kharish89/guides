---
aliases: [m1_ml_setup]
tags:
 - M1
 - M2
 - apple
 - machine_learning
 - GPU
 - tensorflow 
---

This installtion guide will capture setups to install tensorflow for mac m1 (apple silicon) that will leverage metal GPU enabled learning with a sample python notebook.

## Prerequisite
- HomeBrew (Recommended)

## Steps
1. [[Apple silicon Mac TensorFlow ML setup#Install Xcode Command Line Tools|Install Xcode Command Line Tools]]
2. [[Apple silicon Mac TensorFlow ML setup#Install Miniforge|Install Miniforge]]
3. [[Apple silicon Mac TensorFlow ML setup#Create a virtual environment|Create a virtual environment with Python >= 3.8]]
4. [[Apple silicon Mac TensorFlow ML setup#Install Tensorflow-MacOS|Install TensorFlow >= 2.5]]
5. [[Apple silicon Mac TensorFlow ML setup#Install Jupyter Notebook & Pandas|Install Jupyter Notebook, Pandas]]
6. [[Apple silicon Mac TensorFlow ML setup#Run benchmark by training MNIST dataset|Run a benchmark by training MNIST dataset]]

---
### Install Xcode Command Line Tools

```shell
xcode-select --install
```

---
### Install Miniforge

```shell
brew install miniforge
```

After installing miniforge, we can turn off the default base environment with the below command
```shell
conda config --set auto_activate_base false
```

---
### Create a virtual environment

```shell
conda create --name mlp python=3.8
```

and activate the same with
```shell
conda activate mlp
```

---
### Install Tensorflow-MacOS

```shell
conda install -c apple tensorflow-deps
```

Install base TensorFlow
```shell
pip install tensorflow-macos
```

Install metall plugin
```shell
pip install tensorflow-metal
```

---
### Install Jupyter Notebook & Pandas

```shell
conda install -c conda-forge -y pandas jupyter
```

---
### Run benchmark by training MNIST dataset

Install tensorflow dataset
```shell
pip install tensorflow_datasets
```

Open Jupyter Notebook
```shell
jupyter notebook
```

create a new python3 notebook and test tensorflow and hardware
```python
import tensorflow as tf

print("Num GPUs Available: ", len(tf.config.experimental.list_physcaldevices('GPU')))
```

Run the bemchmark with below sample code
```python
%%time

import tensorflow as tf

import tensorflow_datasets as tfds

print("TensorFlow version:", tf.__version__)  
print("Num GPUs Available: ", len(tf.config.experimental.list_physical_devices('GPU')))  
tf.config.list_physical_devices('GPU')

(ds_train, ds_test), ds_info = tfds.load(  
    'mnist',  
    split=['train', 'test'],  
    shuffle_files=True,  
    as_supervised=True,  
    with_info=True,  
)

def normalize_img(image, label):  
  """Normalizes images: `uint8` -> `float32`."""  
  return tf.cast(image, tf.float32) / 255., label
  
batch_size = 128

ds_train = ds_train.map(  
    normalize_img, num_parallel_calls=tf.data.experimental.AUTOTUNE)  
ds_train = ds_train.cache()  
ds_train = ds_train.shuffle(ds_info.splits['train'].num_examples)  
ds_train = ds_train.batch(batch_size)  
ds_train = ds_train.prefetch(tf.data.experimental.AUTOTUNE)

ds_test = ds_test.map(  
    normalize_img, num_parallel_calls=tf.data.experimental.AUTOTUNE)  
ds_test = ds_test.batch(batch_size)  
ds_test = ds_test.cache()  
ds_test = ds_test.prefetch(tf.data.experimental.AUTOTUNE)

model = tf.keras.models.Sequential([  
  tf.keras.layers.Conv2D(32, kernel_size=(3, 3),  
                 activation='relu'),  
  tf.keras.layers.Conv2D(64, kernel_size=(3, 3),  
                 activation='relu'),  
  tf.keras.layers.MaxPooling2D(pool_size=(2, 2)),  
#   tf.keras.layers.Dropout(0.25),  
  tf.keras.layers.Flatten(),  
  tf.keras.layers.Dense(128, activation='relu'),  
#   tf.keras.layers.Dropout(0.5),  
  tf.keras.layers.Dense(10, activation='softmax')  
])  

model.compile(  
    loss='sparse_categorical_crossentropy',  
    optimizer=tf.keras.optimizers.Adam(0.001),  
    metrics=['accuracy'],  
)

model.fit(  
    ds_train,  
    epochs=12,  
    validation_data=ds_test,  
)
```