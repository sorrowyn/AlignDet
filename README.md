# Aligning Pre-training and Fine-tuning in Object Detection [[Project Page]](https://liming-ai.github.io/AlignDet/) [[arXiv]](https://arxiv.org/abs/2307.11077) [[Paper]](https://arxiv.org/pdf/2307.11077.pdf)
Official PyTorch Implementation of [AlignDet: Aligning Pre-training and Fine-tuning in Object Detection (ICCV 2023)](https://arxiv.org/abs/2307.11077)
* Existing detection algorithms are constrained by the data, model, and task discrepancies between pre-training and fine-tuning.
* AlignDet aligns these discrepancies in an efficient and unsupervised paradigm, leading to significant performance improvements across different settings.

![](./images/motivation.png)

Comparison with other self-supervised pre-training methods on data, models and tasks aspects. AlignDet achieves more efficient, adequate and detection-oriented pre-training

![](./images/comparison.png)

Our pipeline takes full advantage of the existing pre-trained backbones to efficiently pre-train other modules. By incorporating self-supervised pre-trained backbones, we make the first attempt to fully pre-train various detectors using a completely unsupervised paradigm.

![](./images/pipeline.png)


## Data Download
Please download the [COCO 2017 dataset](https://cocodataset.org/), and the folder structure should be:
```
data
├── coco
│   ├── annotations
│   ├── filtered_proposals
│   ├── semi_supervised_annotations
│   ├── test2017
│   ├── train2017
│   └── val2017
```

The folder `filtered_proposals` for self-supervised pre-training can be downloaded in [Google Drive](https://drive.google.com/file/d/1HnDNzax2p1ZES5AS1xUOgPMNfhvyAcjA/view?usp=sharing).

The folder `semi_supervised_annotations` for semi-supervised fine-tuning can be downloaded in [Google Drive](https://drive.google.com/file/d/1CyXw412wuXJvqrXF0VDRkp6k5SZPP_up/view?usp=sharing), or generated by `tools/generate_semi_coco.py`


## Environments
```bash
# Sorry our code is not based on latest mmdet 3.0+
pip3 install -r requirements.txt
```

## Pre-training and Fine-tuning Instructions
### Pre-training
```bash
bash tools/dist_train.sh configs/selfsup/mask_rcnn.py 8 --work-dir work_dirs/selfsup_mask-rcnn
```

### Fine-tuning
1. Using `tools/model_converters/extract_detector_weights.py` to extract the weights.
```bash
python3 tools/model_converters/extract_detector_weights.py \
work_dirs/selfsup_mask-rcnn/epoch_12.pth  \ # pretrain weights
work_dirs/selfsup_mask-rcnn/final_model.pth  # finetune weights
```

2. Fine-tuning models like normal mmdet training process, usually the learning rate is increased by 1.5 times, and the weight decay is reduced to half of the original setting. Please refer to the released logs for more details.
```bash
bash tools/dist_train.sh configs/coco/mask_rcnn_r50_fpn_1x_coco.py 8 \
--cfg-options load_from=work_dirs/selfsup_mask-rcnn/final_model.pth \ # load pre-trained weights
optimizer.lr=3e-2 optimizer.weight_decay=5e-5  \ # adjust lr and wd
--work-dir work_dirs/finetune_mask-rcnn_1x_coco_lr3e-2_wd5e-5
```

## Checkpoints and Logs
We show part of the results here, and **all the checkpoints & logs can be found in [the HuggingFace Space](https://huggingface.co/spaces/limingcv/AlignDet/tree/main).**


### Different Methods
| Method (Backbone) | Pre-training | Fine-tuning |
|:------------------------:|:-----------------------:|:----------------:|
| FCOS (R50)               |  [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/pretrain/selfsup_fcos_mstrain-soft-teacher_sampler-2048_temp0.5)  |   [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_fcos_1x_coco_lr1.5e-2_wd5e.5)    |
| RetinaNet (R50)          |  [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/pretrain/selfsup_retinanet_mstrain-soft-teacher_sampler-2048_temp0.5) | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_retinanet_1x_coco_lr1.5e-2_wd5e-5) |
| Faster R-CNN (R50)       |  [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/pretrain/selfsup_mask-rcnn_mstrain-soft-teacher_sampler-4096_temp0.5) | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_faster-rcnn_1x_coco_lr3e-2_wd5e-5) |
| Mask R-CNN (R50)         |  [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/pretrain/selfsup_mask-rcnn_mstrain-soft-teacher_sampler-4096_temp0.5) | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_mask-rcnn_1x_coco_lr3e-2_wd5e-5) |
| DETR  (R50)              |  [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/pretrain/selfsup_detr_cluster-id-as-class_contrastive) | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_detr_150e_coco_lr-mult-0.1_selfsup-clusters-as-classes_add-contrastive-temp0.5-weight1.0) |
| SimMIM (Swin-B)          |  [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/pretrain/selfsup_mask-rcnn_swin-b_simmim-800e) | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_mask-rcnn_swin-b_lsj-3x-coco_simmim-800e_lr9e-5_wd2.5e-3) |
| CBNet v2 (Swin-L)        |  [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/pretrain/selfsup_cbv2_swin-L_1x_coco) | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_cbnetv2_swin-L_bs128_1x_coco_lr8e-4_wd2.5e-3) |


### Mask R-CNN with Different Backbones Sizes
| Backbone | Pre-training | Fine-tuning |
|:--------------------:|:-----------------------:|:----------------:|
| MobileNet v2         | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/pretrain/selfsup_mask-rcnn_mbv2_mstrain-soft-teacher_1x_coco_sampler-4096_temp0.5) | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_mask-rcnn_mbv2_1x_coco) |
| ResNet-18            | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/pretrain/selfsup_mask-rcnn_r18_mstrain-soft-teacher_sampler-4096_temp0.5_1x_coco) | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_mask-rcnn_r18_1x_coco_lr3e-2_wd5e-5) |
| Swin-Small           | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/pretrain/selfsup_mask-rcnn_swin-s_mstrain-soft-teacher_sampler-4096_temp0.5) | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_mask-rcnn_swin-s_1x_coco) |
| Swin-Base            | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/pretrain/selfsup_mask-rcnn_swin-b_mstrain-soft-teacher_sampler-4096_temp0.5) | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_mask-rcnn_swin-b_1x_coco_lr1e-4_wd1e-2) |

### Models with Self-supervised Backbones (ResNet-50)
| Mask R-CNN | Pre-training | Fine-tuning |
|:--------------------:|:-----------------------:|:----------------:|
| MoCo v2    | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_mask-rcnn_1x_coco_mocov2-init_moco-setting) | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_mask-rcnn_1x_coco_mocov2-init_moco-setting) |
| PixPro     | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_mask-rcnn_1x_coco_pixpro-init_moco-setting) | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_mask-rcnn_1x_coco_pixpro-init_moco-setting) |
| SwAV       | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/pretrain/selfsup_mask-rcnn_1x_coco_swav) | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_mask-rcnn_1x_coco_swav_lr3e-2_wd5e-6_warmup1k) |
| **RetinaNet** | **Pre-training** | **Fine-tuning** |
| MoCo v2 | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/pretrain/selfsup_retinanet_1x_coco_mocov2) | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_retinanet_1x_coco_mocov2_moco-setting_lr1.5e-2_wd5e-5) |
| PixPro  | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/pretrain/selfsup_retinanet_1x_coco_pixpro) | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_retinanet_1x_coco_pixpro_lr1.5e-2_wd5e-5) |
| SwAV    | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/pretrain/selfsup_retinanet_1x_coco_swav) | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_retinanet_1x_coco_swav_moco-setting_lr1.5e-2_wd5e-5) |


### Transfer Learning on VOC Dataset
| Method | Fine-tuning |
|:--------------------:|:-----------------------:|
| FCOS | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_fcos_12k_voc0712_lr1.5e-2_wd5e-5) |
| RetinaNet | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_retinanet_12k_voc0712_lr1.5e-2_wd5e-5) |
| Faster R-CNN | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_mask-rcnn_12k_voc0712_lr3e-2_wd5e-5) |
| DETR | [link](https://huggingface.co/spaces/limingcv/AlignDet/tree/main/finetune/finetune_detr_100e_voc0712) |

## Citation
If you find our work to be useful for your research, please consider citing.
```
@inproceedings{aligndet,
  title={AlignDet: Aligning Pre-training and Fine-tuning in Object Detection},
  author={Li, Ming and Wu, Jie and Wang, Xionghui and Chen, Chen and Qin, Jie and Xiao, Xuefeng and Wang, Rui and Zheng, Min and Pan, Xin},
  booktitle={Proceedings of the IEEE/CVF International Conference on Computer Vision},
  year={2023}
}
```
