# How to train an object detection classifier for multiple objects using tensorflow (GPU) on EC2 instance
This repository is test and tutorial documents for how to use set up tensorflow object detection environment on AWS EC2 instance. 

## Brief Summary

* Last upadated: 11/13/2018  

This readme describes every step required to get going with running Tensorflow object detection classifier on EC2 instance;
libraries

Create an EC2 instance on AWS with Deep learning AMI (Amazon Linux) Version 17.0 - ami 0d3348bce8bdde5a6

Insall required libaries

Type these commands line by line.

```bash
sudo su
yum update 
yum upgrade 
pip install Cython
yum install -y protobuf-compiler python-pil python-lxml python-tk 
mkdir -p tensorflow
cd tensorflow
# Download tensorflow models 
git clone https://github.com/tensorflow/models.git
cd models/research
```

## Protobuf Compilation

The Tensorflow Object Detection API uses Protobufs to configure model and
training parameters. Before the framework can be used, the Protobuf libraries
must be compiled. This should be done by running the following command from
the tensorflow/models/research/ directory:

Download and install the 3.0 release of protoc, then unzip the file.

```bash
# From tensorflow/models/research/
wget -O protobuf.zip https://github.com/google/protobuf/releases/download/v3.0.0/protoc-3.0.0-linux-x86_64.zip
unzip protobuf.zip
```

Run the compilation process again, but use the downloaded version of protoc

```bash
# From tensorflow/models/research/
./bin/protoc object_detection/protos/*.proto --python_out=.
```

```bash
# From tensorflow/models/research/
export PYTHONPATH=$PYTHONPATH:`pwd`:`pwd`/slim
```

Now the environment is set up, we can exit sudo mode and then activate conda environment to run the testing code. 

```bash
# From tensorflow/models/research/
exit
source activate tensorflow_p27
cd tensorflow/models/research/
```

Note: This command needs to run from every new terminal you start. If you wish
to avoid running this manually, you can add it as a new line to the end of your
~/.bashrc file, replacing \`pwd\` with the absolute path of
tensorflow/models/research on your system.


## Testing the Installation

You can test that you have correctly installed the Tensorflow Object Detection\
API by running the following command:


```bash
# From tensorflow/models/research/
export PYTHONPATH=$PYTHONPATH:`pwd`:`pwd`/slim
python object_detection/builders/model_builder_test.py
cd object_detection
```

if you see "OK" at the end of running, that means the environment installation for Tensorflow API is ready

## Download pre-trained model

```bash
# From tensorflow/models/research/object_detection
sudo su
wget http://download.tensorflow.org/models/object_detection/faster_rcnn_inception_v2_coco_2018_01_28.tar.gz
tar -xvzf faster_rcnn_inception_v2_coco_2018_01_28.tar.gz
rm faster_rcnn_inception_v2_coco_2018_01_28.tar.gz
```

## Download public training data
```bash
# From tensorflow/models/rsearch/object_detection
git clone https://github.com/larui529/tensowflow-od-aws-ec2.git
cp -r tensowflow-od-aws-ec2/* .
sudo rm -f -R tensorflow-od-aws-ec2
```

## Convert csv to tfrecord files
```bash
# From tensorflow/models/research/object_detection
exit # exit sudo mode
sudo chmod -R 777 ~/tensorflow/*
python generate_tfrecord.py --csv_input=images/train_labels.csv --image_dir=images/train/ --output_path=train.record

python generate_tfrecord.py --csv_input=images/test_labels.csv --image_dir=images/test --output_path=test.record
```

## Change mode of folder and start training

```bash
# From tensorflow/models/research/object_detection
#sudo chmod -R 777 ~/tensorflow/*
python legacy/train.py --logtostderr --train_dir=training/ --pipeline_config_path=training/faster_rcnn_inception_v2_pets.config
```

## Quit training 

When the loss is converged we can quit training by pressing "Q" and the result will be saved 

## Export inference graph
The post-trained model is saved in tensorflow/models/research/object_detection/training. You could see the last 5 checkpoints.
There are two ways of export inference graph. The first one is to export frozen inference graph. The command is below:
```bash
# From tensorflow/models/research/object_detection
python export_inference_graph.py --input_type image_tensor --pipeline_config_path training/faster_rcnn_inception_v2_pets.config --trained_checkpoint_prefix training/model.ckpt-XXXX --output_directory inference_graph # you have to change "XXXX" to the step number from the checkpoint file.
```

The second method could export unfrozen inference graph for serving. What we need to do is overwrite `exporter.py` file in object_detection folder. 
I have include the modified `exporter.py` file in the folder /unfrozen_exporter/ and the following command could overwrite the current file and export inference graph
```bash
# From tensorflow/models/research/object_detection
cp unfrozen_exporter/exporter.py .
python export_inference_graph.py --input_type image_tensor --pipeline_config_path training/faster_rcnn_inception_v2_pets.config --trained_checkpoint_prefix training/model.ckpt-XXXX --output_directory inference_graph # you have to change "XXXX" to the step number from the checkpoint file.
```

## 