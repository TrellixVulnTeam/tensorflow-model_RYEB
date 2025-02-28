# Running Locally

This page walks through the steps required to train an object detection model
on a local machine. It assumes the reader has completed the
following prerequisites:

1. The Tensorflow Object Detection API has been installed as documented in the
[installation instructions](installation.md). This includes installing library
dependencies, compiling the configuration protobufs and setting up the Python
environment.
2. A valid data set has been created. See [this page](preparing_inputs.md) for
instructions on how to generate a dataset for the PASCAL VOC challenge or the
Oxford-IIIT Pet dataset.
3. A Object Detection pipeline configuration has been written. See
[this page](configuring_jobs.md) for details on how to write a pipeline configuration.

## Recommended Directory Structure for Training and Evaluation

```
+data
  -label_map file
  -train TFRecord file
  -eval TFRecord file
+models
  + model
    -pipeline config file
    +train
    +eval
```

## Running the Training Job

A local training job can be run with the following command:

```bash
# From the tensorflow/models/research/ directory
python object_detection/train.py \
    --logtostderr \
    --pipeline_config_path=${PATH_TO_YOUR_PIPELINE_CONFIG} \
    --train_dir=${PATH_TO_TRAIN_DIR}
```

where `${PATH_TO_YOUR_PIPELINE_CONFIG}` points to the pipeline config and
`${PATH_TO_TRAIN_DIR}` points to the directory in which training checkpoints
and events will be written to. By default, the training job will
run indefinitely until the user kills it.

## Running the distributed Training Job

Wrapper for distributed training on different hosts, hosts seprated as "ps", "master" and "worker"
There is an example cluster master(192.168.1.52), ps(192.168.1.52), worker(192.168.1.29).
For the default port for master, ps, worker are 3000, 3001 and 3002
Here is example command for master. 
For different type of rule, your should just change "type" paramter and running it in each host.

```bash
# From the tensorflow/models/research/ directory
python object_detection/distributed_train.py --type=master --id=0 \
    --master=192.168.1.52 --ps=192.168.1.52 --worker=192.168.1.29 \
    --train_dir=/tmp/train2_log \
    --pipeline_config_path=object_detection/samples/configs/faster_rcnn_resnet101.config
```

It will automatic retry job after 60 second if any execption happened.

## Running the Evaluation Job

Evaluation is run as a separate job. The eval job will periodically poll the
train directory for new checkpoints and evaluate them on a test dataset. The
job can be run using the following command:

```bash
# From the tensorflow/models/research/ directory
python object_detection/eval.py \
    --logtostderr \
    --pipeline_config_path=${PATH_TO_YOUR_PIPELINE_CONFIG} \
    --checkpoint_dir=${PATH_TO_TRAIN_DIR} \
    --eval_dir=${PATH_TO_EVAL_DIR}
```

where `${PATH_TO_YOUR_PIPELINE_CONFIG}` points to the pipeline config,
`${PATH_TO_TRAIN_DIR}` points to the directory in which training checkpoints
were saved (same as the training job) and `${PATH_TO_EVAL_DIR}` points to the
directory in which evaluation events will be saved. As with the training job,
the eval job run until terminated by default.

## Running Tensorboard

Progress for training and eval jobs can be inspected using Tensorboard. If
using the recommended directory structure, Tensorboard can be run using the
following command:

```bash
tensorboard --logdir=${PATH_TO_MODEL_DIRECTORY}
```

where `${PATH_TO_MODEL_DIRECTORY}` points to the directory that contains the
train and eval directories. Please note it may take Tensorboard a couple minutes
to populate with data.
