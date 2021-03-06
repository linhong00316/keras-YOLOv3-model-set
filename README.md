# TF Keras YOLOv3/v2 Modelset

[![license](https://img.shields.io/github/license/mashape/apistatus.svg)](LICENSE)

## Introduction

A general YOLOv3/v2 object detection pipeline inherited from [keras-yolo3-Mobilenet](https://github.com/Adamdad/keras-YOLOv3-mobilenet)/[keras-yolo3](https://github.com/qqwweee/keras-yolo3) and [YAD2K](https://github.com/allanzelener/YAD2K). Implement with tf.keras, including data collection/annotation, model training/tuning, model evaluation and on device deployment. Support different architecture and different technologies:

#### Backbone
- [x] Darknet53/Tiny Darknet
- [x] Darknet19
- [x] MobilenetV1
- [x] MobilenetV2
- [x] EfficientNet
- [x] Xception
- [x] VGG16

#### Head
- [x] YOLOv3
- [x] YOLOv3 Lite
- [x] YOLOv3 spp
- [x] YOLOv3 Lite spp
- [x] YOLOv3 Nano ([YOLO nano](https://arxiv.org/abs/1910.01271)) (unofficially)
- [x] Tiny YOLOv3
- [x] Tiny YOLOv3 Lite
- [x] YOLOv2
- [x] YOLOv2 Lite
- [x] Tiny YOLOv2
- [x] Tiny YOLOv2 Lite

#### Loss
- [x] Standard YOLOv3 loss
- [x] Standard YOLOv2 loss
- [x] Binary focal classification loss
- [x] Softmax focal classification loss
- [x] GIoU localization loss
- [x] DIoU localization loss ([Distance-IoU Loss](https://arxiv.org/abs/1911.08287))
- [x] Binary focal loss for objectness (experimental)
- [x] Label smoothing for classification loss

#### Postprocess
- [x] Numpy YOLOv3/v2 postprocess implementation
- [x] TFLite/MNN C++ YOLOv3/v2 postprocess implementation
- [x] TF YOLOv3/v2 postprocess model
- [x] tf.keras batch-wise YOLOv3/v2 postprocess Lambda layer
- [x] SoftNMS bounding box postprocess (numpy)
- [x] DIoU-NMS bounding box postprocess (numpy)

#### Train tech
- [x] Transfer training from imagenet
- [x] Singlescale image input training
- [x] Multiscale image input training
- [x] Dynamic learning rate decay (Cosine/Exponential/Polynomial)
- [x] Pruned model training (only valid for TF 1.x)

#### On-device deployment
- [x] Tensorflow-Lite Float32/UInt8 model inference
- [x] MNN Float32/UInt8 model inference


## Quick Start

1. Install requirements on Ubuntu 16.04/18.04:

```
# apt install python3-opencv
# pip install -r requirements.txt
```

2. Download Related Darknet/YOLOv2/YOLOv3 weights from [YOLO website](http://pjreddie.com/darknet/yolo/).
3. Convert the Darknet YOLO model to a Keras model.
4. Run YOLO detection on your image or video, default using Tiny YOLOv3 model.

```
# wget -O weights/darknet53.conv.74.weights https://pjreddie.com/media/files/darknet53.conv.74
# wget -O weights/darknet19_448.conv.23.weights https://pjreddie.com/media/files/darknet19_448.conv.23
# wget -O weights/yolov3.weights https://pjreddie.com/media/files/yolov3.weights
# wget -O weights/yolov3-tiny.weights https://pjreddie.com/media/files/yolov3-tiny.weights
# wget -O weights/yolov3-spp.weights https://pjreddie.com/media/files/yolov3-spp.weights
# wget -O weights/yolov2.weights http://pjreddie.com/media/files/yolo.weights
# wget -O weights/yolov2-voc.weights http://pjreddie.com/media/files/yolo-voc.weights
# wget -O weights/yolov2-tiny.weights https://pjreddie.com/media/files/yolov2-tiny.weights
# wget -O weights/yolov2-tiny-voc.weights https://pjreddie.com/media/files/yolov2-tiny-voc.weights

# python tools/convert.py cfg/yolov3.cfg weights/yolov3.weights weights/yolov3.h5
# python tools/convert.py cfg/yolov3-tiny.cfg weights/yolov3-tiny.weights weights/yolov3-tiny.h5
# python tools/convert.py cfg/yolov3-spp.cfg weights/yolov3-spp.weights weights/yolov3-spp.h5
# python tools/convert.py cfg/yolov2.cfg weights/yolov2.weights weights/yolov2.h5
# python tools/convert.py cfg/yolov2-voc.cfg weights/yolov2-voc.weights weights/yolov2-voc.h5
# python tools/convert.py cfg/yolov2-tiny.cfg weights/yolov2-tiny.weights weights/yolov2-tiny.h5
# python tools/convert.py cfg/yolov2-tiny-voc.cfg weights/yolov2-tiny-voc.weights weights/yolov2-tiny-voc.h5
# python tools/convert.py cfg/darknet53.cfg weights/darknet53.conv.74.weights weights/darknet53.h5
# python tools/convert.py cfg/darknet19_448_body.cfg weights/darknet19_448.conv.23.weights weights/darknet19.h5

# python yolo.py --image
# python yolo.py --input=<your video file>
```
For other model, just do in a similar way, but specify different model type, weights path and anchor path with `--model_type`, `--weights_path` and `--anchors_path`.

Image detection sample:

<p align="center">
  <img src="assets/dog_inference.jpg">
  <img src="assets/kite_inference.jpg">
</p>

## Guide of train/evaluate/demo

### Train
1. Generate train/val/test annotation file and class names file.

    Data annotation file format:
    * One row for one image in annotation file;
    * Row format: `image_file_path box1 box2 ... boxN`;
    * Box format: `x_min,y_min,x_max,y_max,class_id` (no space).
    * Here is an example:
    ```
    path/to/img1.jpg 50,100,150,200,0 30,50,200,120,3
    path/to/img2.jpg 120,300,250,600,2
    ...
    ```
    1. For VOC style dataset, you can use [voc_annotation.py](https://github.com/david8862/keras-YOLOv3-model-set/blob/master/tools/voc_annotation.py) to convert original dataset to our annotation file:
       ```
       # cd tools && python voc_annotation.py -h
       usage: voc_annotation.py [-h] [--dataset_path DATASET_PATH]
                                [--output_path OUTPUT_PATH]
                                [--classes_path CLASSES_PATH] [--include_difficult]
                                [--include_no_obj]

       convert PascalVOC dataset annotation to txt annotation file

       optional arguments:
         -h, --help            show this help message and exit
         --dataset_path DATASET_PATH
                               path to PascalVOC dataset, default is ../VOCdevkit
         --output_path OUTPUT_PATH
                               output path for generated annotation txt files,
                               default is ./
         --classes_path CLASSES_PATH
                               path to class definitions
         --include_difficult   to include difficult object
         --include_no_obj      to include no object image
       ```
       By default, the VOC convert script will try to go through both VOC2007/VOC2012 dataset dir under the dataset_path and generate train/val/test annotation file separately, like:
       ```
       2007_test.txt  2007_train.txt  2007_val.txt  2012_train.txt  2012_val.txt
       ```
       You can merge these train & val annotation file as your need. For example, following cmd will creat 07/12 combined trainval dataset:
       ```
       # cp 2007_train.txt trainval.txt
       # cat 2007_val.txt >> trainval.txt
       # cat 2012_train.txt >> trainval.txt
       # cat 2012_val.txt >> trainval.txt
       ```
       P.S. You can use [LabelImg](https://github.com/tzutalin/labelImg) to annotate your object detection dataset with Pascal VOC XML format

    2. For COCO style dataset, you can use [coco_annotation.py](https://github.com/david8862/keras-YOLOv3-model-set/blob/master/tools/coco_annotation.py) to convert original dataset to our annotation file:
       ```
       # cd tools && python coco_annotation.py -h
       usage: coco_annotation.py [-h] [--dataset_path DATASET_PATH]
                                 [--output_path OUTPUT_PATH] [--include_no_obj]

       convert COCO dataset annotation to txt annotation file

       optional arguments:
         -h, --help            show this help message and exit
         --dataset_path DATASET_PATH
                               path to MSCOCO dataset, default is ../mscoco2017
         --output_path OUTPUT_PATH
                               output path for generated annotation txt files,
                               default is ./
         --include_no_obj      to include no object image
       ```
       This script will try to convert COCO instances_train2017 and instances_val2017 under dataset_path. You can change the code for your dataset

   If you want to download PascalVOC or COCO dataset, refer to [Dockerfile](https://github.com/david8862/keras-YOLOv3-model-set/blob/master/Dockerfile) for cmd

   For class names file format, refer to  [coco_classes.txt](https://github.com/david8862/keras-YOLOv3-model-set/blob/master/configs/coco_classes.txt)

2. If you're training Darknet YOLOv3/Tiny YOLOv3/Darknet YOLOv2, make sure you have converted pretrain model weights as in [Quick Start](https://github.com/david8862/keras-YOLOv3-model-set#quick-start)

3. [train.py](https://github.com/david8862/keras-YOLOv3-model-set/blob/master/train.py)
```
# python train.py -h
usage: train.py [-h] [--model_type MODEL_TYPE] [--anchors_path ANCHORS_PATH]
                [--model_image_size MODEL_IMAGE_SIZE]
                [--weights_path WEIGHTS_PATH]
                [--annotation_file ANNOTATION_FILE]
                [--val_annotation_file VAL_ANNOTATION_FILE]
                [--val_split VAL_SPLIT] [--classes_path CLASSES_PATH]
                [--batch_size BATCH_SIZE] [--optimizer OPTIMIZER]
                [--learning_rate LEARNING_RATE] [--decay_type DECAY_TYPE]
                [--transfer_epoch TRANSFER_EPOCH]
                [--freeze_level FREEZE_LEVEL] [--init_epoch INIT_EPOCH]
                [--total_epoch TOTAL_EPOCH] [--multiscale]
                [--rescale_interval RESCALE_INTERVAL] [--model_pruning]
                [--label_smoothing LABEL_SMOOTHING] [--data_shuffle]
                [--gpu_num GPU_NUM] [--eval_online]
                [--eval_epoch_interval EVAL_EPOCH_INTERVAL]
                [--save_eval_checkpoint]

optional arguments:
  -h, --help            show this help message and exit
  --model_type MODEL_TYPE
                        YOLO model type: yolo3_mobilenet_lite/tiny_yolo3_mobil
                        enet/yolo3_darknet/..., default=yolo3_mobilenet_lite
  --anchors_path ANCHORS_PATH
                        path to anchor definitions,
                        default=configs/yolo3_anchors.txt
  --model_image_size MODEL_IMAGE_SIZE
                        Initial model image input size as <num>x<num>, default
                        416x416
  --weights_path WEIGHTS_PATH
                        Pretrained model/weights file for fine tune
  --annotation_file ANNOTATION_FILE
                        train annotation txt file, default=trainval.txt
  --val_annotation_file VAL_ANNOTATION_FILE
                        val annotation txt file, default=None
  --val_split VAL_SPLIT
                        validation data persentage in dataset if no val
                        dataset provide, default=0.1
  --classes_path CLASSES_PATH
                        path to class definitions,
                        default=configs/voc_classes.txt
  --batch_size BATCH_SIZE
                        Batch size for train, default=16
  --optimizer OPTIMIZER
                        optimizer for training (adam/rmsprop/sgd),
                        default=adam
  --learning_rate LEARNING_RATE
                        Initial learning rate, default=0.001
  --decay_type DECAY_TYPE
                        Learning rate decay type
                        (None/Cosine/Exponential/Polynomial), default=None
  --transfer_epoch TRANSFER_EPOCH
                        Transfer training (from Imagenet) stage epochs,
                        default=20
  --freeze_level FREEZE_LEVEL
                        Freeze level of the model in transfer training stage.
                        0:NA/1:backbone/2:only open prediction layer
  --init_epoch INIT_EPOCH
                        Initial training epochs for fine tune training,
                        default=0
  --total_epoch TOTAL_EPOCH
                        Total training epochs, default=250
  --multiscale          Whether to use multiscale training
  --rescale_interval RESCALE_INTERVAL
                        Number of iteration(batches) interval to rescale input
                        size, default=10
  --model_pruning       Use model pruning for optimization, only for TF 1.x
  --label_smoothing LABEL_SMOOTHING
                        Label smoothing factor (between 0 and 1) for
                        classification loss, default=0
  --data_shuffle        Whether to shuffle train/val data for cross-validation
  --gpu_num GPU_NUM     Number of GPU to use, default=1
  --eval_online         Whether to do evaluation on validation dataset during
                        training
  --eval_epoch_interval EVAL_EPOCH_INTERVAL
                        Number of iteration(epochs) interval to do evaluation,
                        default=10
  --save_eval_checkpoint
                        Whether to save checkpoint with best evaluation result
```

Following is a reference training config cmd:
```
# python train.py --model_type=yolo3_mobilenet_lite --anchors_path=configs/yolo3_anchors.txt --annotation_file=trainval.txt --classes_path=configs/voc_classes.txt --eval_online --save_eval_checkpoint
```

Checkpoints during training could be found at `logs/000/`. Choose a best one as result

You can also use Tensorboard to monitor the loss trend during train:
```
# tensorboard --logdir=logs/000
```

MultiGPU usage: use `--gpu_num N` to use N GPUs. It is passed to the [Keras multi_gpu_model()](https://keras.io/utils/#multi_gpu_model).

Loss type couldn't be changed from CLI options. You can try them by changing params in [loss.py(v3)](https://github.com/david8862/keras-YOLOv3-model-set/blob/master/yolo3/loss.py) or [loss.py(v2)](https://github.com/david8862/keras-YOLOv3-model-set/blob/master/yolo2/loss.py)

Postprocess type (SoftNMS, DIoU-NMS) could be configured in [postprocess_np.py](https://github.com/david8862/keras-YOLOv3-model-set/blob/master/yolo3/postprocess_np.py)

### Model dump
We need to dump out inference model from training checkpoint for eval or demo. Following script cmd work for that.

```
# python yolo.py --model_type=yolo3_mobilenet_lite --weights_path=logs/000/<checkpoint>.h5 --anchors_path=configs/yolo3_anchors.txt --classes_path=configs/voc_classes.txt --model_image_size=416x416 --dump_model --output_model_file=model.h5
```

Change model_type, anchors file & class file for different training mode. If "--model_pruning" was added in training, you also need to use "--pruning_model" here for dumping out the pruned model.

### Evaluation
Use [eval.py](https://github.com/david8862/keras-YOLOv3-model-set/blob/master/eval.py) to do evaluation on the inference model with your test data. It support following metrics:

1. Pascal VOC mAP: draw rec/pre curve for each class and AP/mAP result chart in "result" dir with default 0.5 IOU or specified IOU, and optionally save all the detection result on evaluation dataset as images

2. MS COCO AP evaluation. Will draw overall AP chart and AP on different scale (small, medium, large) as COCO standard. It can also optionally save all the detection result

```
# python eval.py --model_path=model.h5 --anchors_path=configs/yolo3_anchors.txt --classes_path=configs/voc_classes.txt --model_image_size=416x416 --eval_type=VOC --iou_threshold=0.5 --conf_threshold=0.001 --annotation_file=2007_test.txt --save_result
```


If you enable "--eval_online" option in train.py, a default Pascal VOC mAP evaluation on validation dataset will be executed during training. But that may cost more time for train process.


Following is a sample result trained on Mobilenet YOLOv3 Lite model with PascalVOC dataset (using a reasonable score threshold=0.1):
<p align="center">
  <img src="assets/mAP.jpg">
  <img src="assets/COCO_AP.jpg">
</p>

Some experiment on MSCOCO dataset and comparison:

| Model name | InputSize | TrainSet | TestSet | COCO AP | Size | Speed | Ps |
| ----- | ------ | ------ | ------ | ----- | ----- | ----- | ----- |
| [YOLOv3 Lite-Mobilenet](https://github.com/david8862/keras-YOLOv3-model-set/releases/download/v1.1.0/yolo3_mobilenet_lite_320_coco.tar.gz) | 320x320 | train2017 | val2017 | 24.43 | 32MB | 17ms | Keras on Titan XP |
| [YOLOv3 Lite-Mobilenet](https://github.com/david8862/keras-YOLOv3-model-set/releases/download/v1.1.0/yolo3_mobilenet_lite_416_coco.tar.gz) | 416x416 | train2017 | val2017 | 27.93 | 32MB| 20ms | Keras on Titan XP |
| [Tiny YOLOv3 Lite-Mobilenet](https://github.com/david8862/keras-YOLOv3-model-set/releases/download/v1.1.0/tiny_yolo3_mobilenet_lite_320_coco.tar.gz) | 320x320 | train2017 | val2017 | 21.29 | 21MB | 9ms | Keras on Titan XP |
| [Tiny YOLOv3 Lite-Mobilenet](https://github.com/david8862/keras-YOLOv3-model-set/releases/download/v1.1.0/tiny_yolo3_mobilenet_lite_416_coco.tar.gz) | 416x416 | train2017 | val2017 | 24.29 | 21MB | 11ms | Keras on Titan XP |
| [ssd_mobilenet_v1_coco](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md) | 600x600 | COCO train | COCO val | 21 | 28MB | 30ms | TF on Titan X |
| [ssdlite_mobilenet_v2_coco](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md) | 600x600 | COCO train | COCO val | 22 | 19MB | 27ms | TF on Titan X |

Some experiment on PascalVOC dataset and comparison:

| Model name | InputSize | TrainSet | TestSet | mAP | Size | Speed | Ps |
| ----- | ------ | ------ | ------ | ----- | ----- | ----- | ----- |
| [YOLOv3 Lite-Mobilenet](https://github.com/david8862/keras-YOLOv3-model-set/releases/download/v1.0.0/yolo3_mobilnet_lite_320_voc.tar.gz) | 320x320 | VOC07+12 | VOC07 | 73.31% | 31.8MB | 17ms | Keras on Titan XP |
| [YOLOv3 Lite-Mobilenet](https://github.com/david8862/keras-YOLOv3-model-set/releases/download/v1.0.0/yolo3_mobilnet_lite_416_voc.tar.gz) | 416x416 | VOC07+12 | VOC07 | 75.93% | 31.8MB| 20ms | Keras on Titan XP |
| [YOLOv3 Lite-SPP-Mobilenet](https://github.com/david8862/keras-YOLOv3-model-set/releases/download/v1.0.0/yolo3_mobilnet_lite_spp_416_voc.tar.gz) | 416x416 | VOC07+12 | VOC07 | 75.93% | 34MB | 22ms | Keras on Titan XP |
| [Tiny YOLOv3 Lite-Mobilenet](https://github.com/david8862/keras-YOLOv3-model-set/releases/download/v1.0.0/tiny_yolo3_mobilnet_lite_320_voc.tar.gz) | 320x320 | VOC07+12 | VOC07 | 69.12% | 20.1MB | 9ms | Keras on Titan XP |
| [Tiny YOLOv3 Lite-Mobilenet](https://github.com/david8862/keras-YOLOv3-model-set/releases/download/v1.0.0/tiny_yolo3_mobilnet_lite_416_voc.tar.gz) | 416x416 | VOC07+12 | VOC07 | 72.60% | 20.1MB | 11ms | Keras on Titan XP |
| [Tiny YOLOv3 Lite-Mobilenet with GIoU loss](https://github.com/david8862/keras-YOLOv3-model-set/releases/download/v1.0.0/tiny_yolo3_mobilnet_lite_giou_416_voc.tar.gz) | 416x416 | VOC07+12 | VOC07 | 72.73% | 20.1MB | 11ms | Keras on Titan XP |
| [YOLOv3 Nano](https://github.com/david8862/keras-YOLOv3-model-set/releases/download/v1.0.1/yolo3_nano_weights_416_voc.tar.gz) | 416x416 | VOC07+12 | VOC07 | 69.40% | 19MB | 29ms | Keras on Titan XP |
| [YOLOv3-Xception](https://github.com/david8862/keras-YOLOv3-model-set/releases/download/v1.0.0/yolo3_xception_512_voc.tar.gz) | 512x512 | VOC07+12 | VOC07 | 78.89% | 419.8MB | 48ms | Keras on Titan XP |
| [YOLOv3-Mobilenet](https://github.com/Adamdad/keras-YOLOv3-mobilenet) | 320x320 | VOC07 | VOC07 | 64.22% || 29fps | Keras on 1080Ti |
| [YOLOv3-Mobilenet](https://github.com/Adamdad/keras-YOLOv3-mobilenet) | 320x320 | VOC07+12 | VOC07 | 74.56% || 29fps | Keras on 1080Ti |
| [YOLOv3-Mobilenet](https://github.com/Adamdad/keras-YOLOv3-mobilenet) | 416x416 | VOC07+12 | VOC07 | 76.82% || 25fps | Keras on 1080Ti |
| [MobileNet-SSD](https://github.com/chuanqi305/MobileNet-SSD) | 300x300 | VOC07+12+coco | VOC07 | 72.7% | 22MB |||
| [MobileNet-SSD](https://github.com/chuanqi305/MobileNet-SSD) | 300x300 | VOC07+12 | VOC07 | 68% | 22MB |||
| [Faster RCNN, VGG-16](https://github.com/ShaoqingRen/faster_rcnn) | ~1000x600 | VOC07+12 | VOC07 | 73.2% || 151ms | Caffe on Titan X |
|[SSD,VGG-16](https://github.com/pierluigiferrari/ssd_keras) | 300x300 | VOC07+12 | VOC07	| 77.5% | 201MB | 39fps | Keras on Titan X |


### Demo
1. [yolo.py](https://github.com/david8862/keras-YOLOv3-model-set/blob/master/yolo.py)
> * Demo script for trained model

image detection mode
```
# python yolo.py --model_type=yolo3_mobilenet_lite --weights_path=model.h5 --anchors_path=configs/yolo3_anchors.txt --classes_path=configs/voc_classes.txt --model_image_size=416x416 --image
```
video detection mode
```
# python yolo.py --model_type=yolo3_mobilenet_lite --weights_path=model.h5 --anchors_path=configs/yolo3_anchors.txt --classes_path=configs/voc_classes.txt --model_image_size=416x416 --input=test.mp4
```
For video detection mode, you can use "input=0" to capture live video from web camera and "output=<video name>" to dump out detection result to another video

### Tensorflow model convert
Using [keras_to_tensorflow.py](https://github.com/david8862/keras-YOLOv3-model-set/tree/master/tools/keras_to_tensorflow.py) to convert the keras .h5 model to tensorflow frozen pb model (only for TF 1.x):
```
# python keras_to_tensorflow.py
    --input_model="path/to/keras/model.h5"
    --output_model="path/to/save/model.pb"
```

You can also use [eval.py](https://github.com/david8862/keras-YOLOv3-model-set/blob/master/eval.py) to do evaluation on the pb inference model

### Inference model deployment
See [on-device inference](https://github.com/david8862/keras-YOLOv3-model-set/tree/master/inference) for TFLite & MNN model deployment


### TODO
- [ ] enhance data augmentation ([Mixup](https://arxiv.org/abs/1710.09412)/[AutoAugment](https://arxiv.org/abs/1805.09501))
- [ ] provide more imagenet pretrained backbone (e.g. shufflenet, shufflenetv2), see [Training backbone](https://github.com/david8862/keras-YOLOv3-model-set/tree/master/common/backbones/imagenet_training)


## Some issues to know

1. The test environment is
    - Ubuntu 16.04/18.04
    - Python 3.6.8
    - tensorflow 2.0.0/tensorflow 1.15.0
    - tf.keras 2.2.4-tf

2. Default YOLOv3/v2 anchors are used. If you want to use your own anchors, probably some changes are needed. [kmeans.py](https://github.com/david8862/keras-YOLOv3-model-set/blob/master/tools/kmeans.py) could be used to do K-Means anchor clustering on your dataset

3. Always load pretrained weights and freeze layers in the first stage of training.

4. Training strategy is for reference only. Adjust it according to your dataset and your goal. And add further strategy if needed.


## Contribution guidelines
New features, improvements and any other kind of contributions are warmly welcome via pull request :)


# Citation
Please cite keras-YOLOv3-model-set in your publications if it helps your research:
```
@article{MobileNet-Yolov3,
     Author = {Adam Yang},
     Year = {2018}
}
@article{keras-yolo3,
     Author = {qqwweee},
     Year = {2018}
}
@article{YAD2K,
     title={YAD2K: Yet Another Darknet 2 Keras},
     Author = {allanzelener},
     Year = {2017}
}
@article{yolov3,
     title={YOLOv3: An Incremental Improvement},
     author={Redmon, Joseph and Farhadi, Ali},
     journal = {arXiv},
     year={2018}
}
@article{redmon2016yolo9000,
  title={YOLO9000: Better, Faster, Stronger},
  author={Redmon, Joseph and Farhadi, Ali},
  journal={arXiv preprint arXiv:1612.08242},
  year={2016}
}
@article{Focal Loss,
     title={Focal Loss for Dense Object Detection},
     author={Tsung-Yi Lin, Priya Goyal, Ross Girshick, Kaiming He, Piotr Dollár},
     journal = {arXiv},
     year={2017}
}
@article{GIoU,
     title={Generalized Intersection over Union: A Metric and A Loss for Bounding Box Regression},
     author={Hamid Rezatofighi, Nathan Tsoi1, JunYoung Gwak1, Amir Sadeghian, Ian Reid, Silvio Savarese},
     journal = {arXiv},
     year={2019}
}
@article{DIoU Loss,
     title={Distance-IoU Loss: Faster and Better Learning for Bounding Box Regression},
     author={Zhaohui Zheng, Ping Wang, Wei Liu, Jinze Li, Rongguang Ye, Dongwei Ren},
     journal = {arXiv},
     year={2020}
}
```
