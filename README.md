# SpixelFCN: Superpixel Segmentation with Fully Convolutional Network

This is is a PyTorch implementation of the superpixel segmentation network introduced in our CVPR-20 paper:

[Superpixel Segmentation with Fully Convolutional Network](http://personal.psu.edu/fuy34)

[Fengting Yang](http://personal.psu.edu/fuy34/), [Qian Sun](https://www.linkedin.com/in/qiansuun), [Hailin Jin](https://research.adobe.com/person/hailin-jin/), and [Zihan Zhou](https://faculty.ist.psu.edu/zzhou/Home.html)

Please contact Fengting Yang (fuy34@psu.edu) if you have any question.

## Prerequisites
The training code was mainly developed and tested with python 2.7, PyTorch 0.4.1, CUDA 9, and Ubuntu 16.04.

During test, we make use of the component connection method in [SSN](https://github.com/NVlabs/ssn_superpixels) to enforce the connectivity 
in superpixels. The code has been included in ```/third_paty/cython```. To compile it:
 ```
cd third_party/cython/
python setup.py install --user
cd ../..
```
## Demo
The demo script ```run_demo.py``` provides the superpixels with grid size ```16 x 16``` using our pre-trained model (in ```/pretrained_ckpt```).
Please feel free to provide your own images by copying them into ```/demo/inputs```, and run 
```
python run_demo.py --data_dir=./demo/inputs --data_suffix=jpg --output=./demo 
```
The results will be generate in a new folder under ```/demo``` called ```spixel_viz```.

 
## Data preparation 
To generate training and test dataset, please first download the data from the original [BSDS500 dataset](http://www.eecs.berkeley.edu/Research/Projects/CS/vision/grouping/BSR/BSR_full.tgz), 
and extract it to  ```<BSDS_DIR>```. Then, run 
```
cd data_preprocessing
python pre_processing_bsd500.py --dataset=<BSDS_DIR> --dump_root=<DUMP_DIR>
python pre_processing_bsd500_ori.py --dataset=<BSDS_DIR> --dump_root=<DUMP_DIR>
cd ..
```
The code will generate three folders under the ```<DUMP_DIR>```, named as ```/train```, ```/val```, and ```/test```, and three ```.txt``` files 
record the absolute path of the images, named as ```train.txt```, ```val.txt```, and ```test.txt```.


## Training
Once the data is prepared, we should be able to train the model by running the following command
```
python main.py --data=<DUMP_DIR> --savepath=<CKPT_LOG_DIR>
```

if we wish to continue a train process or fine-tune from a pre-trained model, we can run 
```
python main.py --data=<DUMP_DIR> --savepath=<CKPT_LOG_DIR> --pretrained=<PATH_TO_THE_CKPT> 
```
The code will start from the recorded status, which includes the optimizer status and epoch number. 

The training log can be viewed from the `tensorboard` session by running
```
tensorboard --logdir=<CKPT_LOG_DIR> --port=8888
```

If everything is set up properly, reasonable segmentation should be observed after 10 epochs.

## Testing
We provide test code to generate: 1) superpixel visualization and 2) the```.csv``` files  for evaluation. 

To test on BSDS500, run
```
python run_infer_bsds.py --data_dir=<DUMP_DIR> --output=<TEST_OUTPUT_DIR> --pretrained=<PATH_TO_THE_CKPT>
```

To test on NYUv2, please first extract our pre-processed dataset from ```/nyu_test_set/nyu_preprocess_tst.tar.gz``` 
to ```<NYU_TEST>``` , or follow the [intruction on the superpixel benchmark](https://github.com/davidstutz/superpixel-benchmark/blob/master/docs/DATASETS.md)
 to generate the test dataset, and then run
```
python run_infer_nyu.py --data_dir=<NYU_TEST> --output=<TEST_OUTPUT_DIR> --pretrained=<PATH_TO_THE_CKPT>
```

To test on other datasets,  please first collect all the images into one folder ```<CUSTOM_DIR>```, and then convert them into the same 
format (e.g. ```.png``` or ```.jpg```) if necessary, and run
```
python run_demo.py --data_dir=<CUSTOM_DIR> --data_suffix=<IMG_SUFFIX> --output=<TEST_OUTPUT_DIR> --pretrained=<PATH_TO_THE_CKPT>
```
Superpixels with grid size ```16 x 16``` will be generated by default. To generate the superpixel with a different grid size, we simply need to 
 resize the images into the approporate resolution before passing them through the code. Please refer to ```run_infer_nyu.py``` for the details. 

## Evaluation
We use the code from [superpixel benchmark](https://github.com/davidstutz/superpixel-benchmark) for superpixel evaluation. 
A detailed  [instruction](https://github.com/davidstutz/superpixel-benchmark/blob/master/docs/BUILDING.md) is available in the repository, please
 
(1) download the code and build it accordingly;

(2) edit the variables ```$SUPERPIXELS```, ```IMG_PATH``` and ```GT_PATH``` in ```/eval_spixel/my_eval.sh```,

(3) run 
 ```
cp /eval_spixel/my_eval.sh <path/to/the/benchmark>/examples/bash/
cd  <path/to/the/benchmark>/examples/
bash my_eval.sh
 ```
several files should be generated in the ```map_csv``` folders in the corresponding test outputs;

(4) run 
```
cd eval_spixel
python copy_resCSV.py --src=<TEST_OUTPUT_DIR> --dst=<PATH_TO_COLLECT_EVAL_RES>
```
(5) open ```/eval_spixel/plot_benchmark_curve.m``` , set the ```our1l_res_path``` as  ```<PATH_TO_COLLECT_EVAL_RES>``` and modify the ```num_list``` 
according to the test setting. The default setting is for our BSDS500 test set.;

(6) run the ```plot_benchmark_curve.m```, the ```ASA Score```, ```CO Score```, and ```BR-BP curve```  of our method should
 be shown on the screen. If you wish to compare our method with the others, you can first run the method and organize the data 
 as we state above, and uncomment the code in the  ```plot_benchmark_curve.m``` to generate a similar figure shown in our papers.  

 
## Acknowledgement
Our code is developed based on the training framework provided by [FlowNetPytorch](https://github.com/ClementPinard/FlowNetPytorch). 
