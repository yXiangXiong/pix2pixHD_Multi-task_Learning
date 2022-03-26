# Pix2pixHD Multi-task Generative Learning
To reduce the exposure of Gadolinium-based Contrast Agents (GBCAs) in brainstem glioma detection and provide high-resolution contrast information,
we propose a novel multi-task generative network for contrast-enhanced T1-weight MR synthesis on brainstem glioma images. The proposed network
can simultaneously synthesize the high-resolution contrast-enhanced image and the segmentation mask of brainstem glioma lesions. <br><br>

A multi-task generative network for simultaneous post-contrast MR image synthesis and tumor
segmentation: application to brainstem glioma

[Yajing Zhang]<sup>1</sup>, [Xiangyu Xiong]<sup>1</sup>, [Yaou Zhu]<sup>2</sup>
 
<sup>1</sup>MR Clinical Science, Philips Healthcare, Suzhou, China, <sup>2</sup> Department of Radiology, Beijing Tiantan Hospital, Capital Medical University, Beijing, China

## Image-to-image translation at 512x512 resolution

- {T1, T2, ASL}-to-{T1ce, tumor mask}
<p align='center'>
  <img src='imgs/BSG001_T1_1012.png' width='150'/>
  <img src='imgs/BSG001_T2_1012.png' width='150'/>
  <img src='imgs/BSG001_ASL_1012.png' width='150'/>
  <img src='imgs/BSG001_T1_1012_synthesized_image.jpg' width='150'/>
  <img src='imgs/BSG001_T1_1012_synthesized_mask.jpg' width='150'/>
</p>
<p align='center'>
  <img src='imgs/BSG043_T1_1007.png' width='150'/>
  <img src='imgs/BSG043_T2_1007.png' width='150'/>
  <img src='imgs/BSG043_ASL_1007.png' width='150'/>
  <img src='imgs/BSG043_T1_1007_synthesized_image.jpg' width='150'/>
  <img src='imgs/BSG043_T1_1007_synthesized_mask.jpg' width='150'/>
</p>

## Prerequisites
- Linux or Windows
- Python 3
- NVIDIA GPU (11G memory or larger) + CUDA cuDNN

## Getting Started
### Installation
- Install PyTorch and dependencies from http://pytorch.org
- Install python libraries [dominate](https://github.com/Knio/dominate).
```bash
pip install dominate
```
- Clone this repo:
```bash
git clone https://github.com/yXiangXiong/pix2pixHD_Multi-task_Learning
cd pix2pixHD_Multi-task_Learning
```


### Testing
- A few example Cityscapes test images are included in the `datasets` folder.
- Please download the pre-trained Cityscapes model from [here](https://drive.google.com/file/d/1h9SykUnuZul7J3Nbms2QGH1wa85nbN2-/view?usp=sharing) (google drive link), and put it under `./checkpoints/label2city_1024p/`
- Test the model (`bash ./scripts/test_1024p.sh`):
```bash
#!./scripts/test.sh
python test.py --dataroot F:\xiongxiangyu\pix2pixHD_Mask_Data --name NC2C --label_nc 0 --input_nc 9 --output_nc 6 --resize_or_crop none --gpu_ids 0 --which_epoch 200 --no_instance --how_many 144
```
The test results will be saved to a html file here: `./results/label2city_1024p/test_latest/index.html`.

More example scripts can be found in the `scripts` directory.


### Dataset
- We use the Cityscapes dataset. To train a model on the full dataset, please download it from the [official website](https://www.cityscapes-dataset.com/) (registration required).
After downloading, please put it under the `datasets` folder in the same way the example images are provided.


### Training
- Train a model at 1024 x 512 resolution (`bash ./scripts/train_512p.sh`):
```bash
#!./scripts/train.sh
python train.py --dataroot F:\xiongxiangyu\pix2pixHD_Mask_Data --name NC2C --label_nc 0 --input_nc 9 --output_nc 6 --netG global --resize_or_crop none --gpu_ids 0 --batchSize 1 --no_instance
```
- To view training results, please checkout intermediate results in `./checkpoints/label2city_512p/web/index.html`.
If you have tensorflow installed, you can see tensorboard logs in `./checkpoints/label2city_512p/logs` by adding `--tf_log` to the training scripts.

### Multi-GPU training
- Train a model using multiple GPUs (`bash ./scripts/train_512p_multigpu.sh`):
```bash
#!./scripts/train_512p_multigpu.sh
python train.py --name label2city_512p --batchSize 8 --gpu_ids 0,1,2,3,4,5,6,7
```
Note: this is not tested and we trained our model using single GPU only. Please use at your own discretion.

### Training with Automatic Mixed Precision (AMP) for faster speed
- To train with mixed precision support, please first install apex from: https://github.com/NVIDIA/apex
- You can then train the model by adding `--fp16`. For example,
```bash
#!./scripts/train_512p_fp16.sh
python -m torch.distributed.launch train.py --name label2city_512p --fp16
```
In our test case, it trains about 80% faster with AMP on a Volta machine.

### Training at full resolution
- To train the images at full resolution (2048 x 1024) requires a GPU with 24G memory (`bash ./scripts/train_1024p_24G.sh`), or 16G memory if using mixed precision (AMP).
- If only GPUs with 12G memory are available, please use the 12G script (`bash ./scripts/train_1024p_12G.sh`), which will crop the images during training. Performance is not guaranteed using this script.

### Training with your own dataset
- If you want to train with your own dataset, please generate label maps which are one-channel whose pixel values correspond to the object labels (i.e. 0,1,...,N-1, where N is the number of labels). This is because we need to generate one-hot vectors from the label maps. Please also specity `--label_nc N` during both training and testing.
- If your input is not a label map, please just specify `--label_nc 0` which will directly use the RGB colors as input. The folders should then be named `train_A`, `train_B` instead of `train_label`, `train_img`, where the goal is to translate images from A to B.
- If you don't have instance maps or don't want to use them, please specify `--no_instance`.
- The default setting for preprocessing is `scale_width`, which will scale the width of all training images to `opt.loadSize` (1024) while keeping the aspect ratio. If you want a different setting, please change it by using the `--resize_or_crop` option. For example, `scale_width_and_crop` first resizes the image to have width `opt.loadSize` and then does random cropping of size `(opt.fineSize, opt.fineSize)`. `crop` skips the resizing step and only performs random cropping. If you don't want any preprocessing, please specify `none`, which will do nothing other than making sure the image is divisible by 32.

## More Training/Test Details
- Flags: see `options/train_options.py` and `options/base_options.py` for all the training flags; see `options/test_options.py` and `options/base_options.py` for all the test flags.
- Instance map: we take in both label maps and instance maps as input. If you don't want to use instance maps, please specify the flag `--no_instance`.


## Citation

If you find this useful for your research, please use the following.

```
@inproceedings{wang2018pix2pixHD,
  title={High-Resolution Image Synthesis and Semantic Manipulation with Conditional GANs},
  author={Ting-Chun Wang and Ming-Yu Liu and Jun-Yan Zhu and Andrew Tao and Jan Kautz and Bryan Catanzaro},  
  booktitle={Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition},
  year={2018}
}
```

## Acknowledgments
This code borrows heavily from [pytorch-CycleGAN-and-pix2pix](https://github.com/junyanz/pytorch-CycleGAN-and-pix2pix).
