# Custom Object Detection with TensorFlow

![](screenshots/starwars_small.gif)

## Installation Instructions

First, with python and pip installed, install the scripts requirements:

```
pip install -r requirements.txt
```
Then you must compile the Protobuf libraries:

```
protoc object_detection/protos/*.proto --python_out=.
```

Add `models` and `models/slim` to your `PYTHONPATH`:

```
export PYTHONPATH=$PYTHONPATH:`pwd`:`pwd`/slim
```

**IMPORTANT NOTE:** This must be ran every time you open terminal, or added to your `~/.bashrc` file.


## Creating TensorFlow Records
Run the script:

```
python object_detection/create_tf_record.py
```

Once the script finishes running, you will end up with a `train.record` and a `val.record` file. This is what we will use to train the model.

## Downloading a Base Model
Training an object detector from scratch can take days, even when using multiple GPUs! In order to speed up training, we’ll take an object detector trained on a different dataset, and reuse some of it’s parameters to initialize our new model.

You can find models to download from this [model zoo](https://github.com/bourdakos1/Custom-Object-Detection/blob/master/object_detection/g3doc/detection_model_zoo.md). Each model varies in accuracy and speed. I used `faster_rcnn_resnet101_coco` for the demo.

Extract the files and move all the `model.ckpt` to our models directory.

**Note:** If you don't use `faster_rcnn_resnet101_coco`, replace `faster_rcnn_resnet101.config` with the corresponding [config file](https://github.com/bourdakos1/Custom-Object-Detection/tree/master/object_detection/samples/configs).

## Train the Model
Run the following script to train the model:

```
python object_detection/train.py \
        --logtostderr \
        --train_dir=train \
        --pipeline_config_path=faster_rcnn_resnet101.config
```

## Export the Inference Graph
When you model is ready depends on your training data, the more data, the more steps you’ll need. My model was pretty solid at ~4.5k steps. Then, at about ~20k steps, it peaked. I even went on and trained it for 200k steps, but it didn’t get any better.

**Note:** If training takes way to long, [read this](https://medium.com/@nickbourdakos/object-detection-isnt-scary-how-to-build-a-custom-detector-with-tensorflow-c8c86419225e).

I recommend testing your model every ~5k steps to make sure you’re on the right path.

You can find checkpoints for your model in `Custom-Object-Detection/train`.

Move the model.ckpt files with the highest number to the root of the repo:
*. `model.ckpt-STEP_NUMBER.data-00000-of-00001`
*. `model.ckpt-STEP_NUMBER.index`
*. `model.ckpt-STEP_NUMBER.meta`

In order to use the model, you first need to convert the checkpoint files (`model.ckpt-STEP_NUMBER.*`) into a frozen inference graph by running this command:

```
python object_detection/export_inference_graph.py \
        --input_type image_tensor \
        --pipeline_config_path faster_rcnn_resnet101.config \
        --trained_checkpoint_prefix model.ckpt-STEP_NUMBER \
        --output_directory output_inference_graph
```

You should see a new `output_inference_graph` directory with a `frozen_inference_graph.pb` file.

## Test the Model
Just run the following command:

```
python object_detection/object_detection_runner.py
```

It will run your object detection model found at `output_inference_graph/frozen_inference_graph.pb` on all the images in the `test_images` directory and output the results in the `output/test_images` directory.

## Results
Here’s what I got from running my model over all the frames in this clip from Star Wars: The Force Awakens.

[![Watch the video](screenshots/youtube.png)](https://www.youtube.com/watch?v=xW2hpkoaIiM)
