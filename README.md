# HOGraspNet
This repository contains instructions on getting the data and code of the work `Dense Hand-Object(HO) GraspNet with Full Grasping Taxonomy and Dynamics` presented at ECCV 2024.

Project page : [HOGraspNet](https://hograspnet2024.github.io/)


## Overview
HOGraspNet provides the following data and models:
- `data/source_data`: Full 1920*1080 size RGB & Depth images ("Source_data/Object Pose" is unnecessary data. It will be removed soon.)
- `data/labeling_data`: Json files for annotations.
- `data/extra_data`: Binary hand & object mask data for cropped image. (Bounding box is provided through the dataloader module.)
- `data/source_augmented`: Cropped images around the hand and background augmented RGB images.
- `data/obj_scanned_models`: Manually scanned 3D models for 30 objects utilized in the dataset.

<!-- See [`data_structure.md`](./docs/data_structure.md) for an explanation of the data you will download. -->

## Installation

- This code is tested with PyTorch 2.0.0, 2.3.1 and Python 3.10 on Linux and Windows 11.
- Clone and install the following main packages.
    ```bash
    git clone https://github.com/UVR-WJCHO/HOGraspNet.git
    cd HOGraspNet
    pip install -r requirements.txt
    ```
- (TBD, for visualization) Install pytorch3d following [here](https://github.com/facebookresearch/pytorch3d) (our code uses version 0.7.3)

- Set the environment variable for dataset path
    ```bash
    (linux) export HOG_DIR=/path/to/HOGGraspNet 
    (windows) set HOG_DIR=/path/to/HOGGraspNet
    ```

## Download HOGraspNet

1. Please fill this [form](https://forms.gle/UqH15zN2PiBGQDUs7) to download the dataset after reading the [terms and conditions](#terms).

2. Copy the data URL from the form, download it and unzip.
    ```bash
    cd assets
    wget -O urls.zip "[URL]"
    unzip urls.zip
    cd ..
    ```


After running the above, you should expect:
<!-- HOGraspNet/assets/checksum.json -->
```
HOGraspNet/assets/urls/
        images.txt: Full RGB & Depth images
        annotations.txt: annotations
        extra.txt: hand & object segmentation masks(pseudo)
        images_augmented.txt: Cropped & background augmented RGB images

```

**Download procedure**

⚠️ [24.08.14] The connection to Dropbox shared links might be unstable. We are testing alternative methods to ensure a stable download environment.

1. Download the dataset 
	1. with default option: 
		- Cropped/Background augmented Images + Annotations + Masks
		- All subject (S1~S99)
		- Scanned object 3D models

		```bash
		python scripts/download_data.py
		```

	2. or with maual option (example): 
		```bash
		python scripts/download_data.py --type [TYPE] --subject [SUBJECT] --objModel [OBJMODEL]
		```

2. Unzip them all:
	```bash
	python scripts/unzip_data.py
	```

The raw downloaded data can be found under `data/zipped/`. The unzipped data and models can be found under `data/`. See [`visualization.md`](./docs/visualization.md) for the explanation of how the files can be visualized.


**Options**

Depending on your usage of the dataset, we suggest different download options. 


* [TYPE] (type: int, default: 0): 
    * 0 : source_augmented(cropped) + labeling_data + extra_data(mask)
    * 1 : 0 + source_data
    * 2 : source_augmented
    * 3 : labeling_data
    * 4 : extra_data
    * 5 : source_data

* [SUBJECT] (type: string, default: all): 
    * all : subject 1~99
    * small : pre-defined 5 subjects
    * 1 : subject 1
    * 1,2 : subject 1 and 2
    * 1-3 : subject 1 to 3
        
* [OBJMODEL] (type: bool, default : True): 
    * True : Download the scanned object 3D models
    * False : Skip

⚠️ If the full dataset is not downloaded (e.g., setting the subject option to "small" or a specific subject index), only the s0 split is fully available in the dataloader.


**Subject info**

Here, we provide a summary of each subject's information included in the dataset. [`HOGraspNet_subject_info`](./assets/HOGraspNet_subject_info.csv)
Please check it if you need data on a specific type of subject.



## Dataloader

* Set the environment variable for dataset path
    ```bash
    export HOG_DIR=/path/to/HOGGraspNet
    ```

* Utilize the dataloader as below
    ```bash
    from scripts.HOG_dataloader import HOGDataset

    setup = 's2'
    split = 'test'
    db_path = os.path.join(os.environ['HOG_DIR'], "data")
    dataloader = HOGDataset(setup, split, db_path)
    ```

* See [`data_structure.md`](./docs/data_structure.md) for detailed structures of the sample from dataloader (WIP)



## Data visualization

- This code is tested with Python 3.10, cuda 11.8, PyTorch 2.0.0 on Linux and Window 11.

- Set MANO code and models in `thirdparty` from the MANO project page(https://mano.is.tue.mpg.de).
    - You need to comment out the `print 'FINITO'` (line 137) statement in `thirdparty/mano_v1_2/webuser/smpl_handpca_wrapper_HAND_only.py`.
    - If you set your MANO folder in another path, modify the `mano_path` parameter in `scripts/config.py`
- We utilized a differentiable MANO layer for PyTorch from https://github.com/hassony2/manopth. `thirdparty/manopth`


<!-- ```bash
conda create -n HOG_vis python=3.10
conda activate HOG_vis
pip install -r requirements_vis.txt 
``` -->

- Set required CUDA, torch, pytorch3d environments.
    ```bash
    pip install torch==2.0.0 torchvision==0.15.1 torchaudio==2.0.1 --index-url https://download.pytorch.org/whl/cu118
    pip install --no-index --no-cache-dir pytorch3d -f https://dl.fbaipublicfiles.com/pytorch3d/packaging/wheels/py310_cu118_pyt200/download.html
    ```

- Run the code; this will produce rendered results of 10 random samples from downloaded data.
    ```bash
    python scripts/visualization.py
    ```

- Rendered images will be saved in `[HOG_DIR]/vis/`.

## Manual background augmentation
 
- As HOGraspNet has captured on a monotonic environment, background augmentations are available with random images. We utilized samples from [HanCo](https://github.com/lmb-freiburg/contra-hand) dataset as background sources.

- Here, we provide example codes for manual background augmentation on source data. This will require `source_augmented` and `extra_data` of the dataset. (`[TYPE] 2 & 4` in download options)

- [Download](https://www.dropbox.com/scl/fi/ql79p3d3tu5npdl9g8jpp/bg_samples.zip?rlkey=sb1y5w443ale0l2udn0nrq64j&st=scr7jr9q&dl=1) or manually prepare background sample images in `[HOG_DIR]/data/bg_samples`.

- Run the code; this will produce augmented rgb images in `[HOG_DIR]/data/manual_augmented`.
    ```bash
    python scripts/manual_augmentation.py
    ```




## TODO ##

<!-- - [ ] An uncompleted task
- [x] A completed task -->
- [x] Update data server protocol as HTTP to HTTPS (24/07/24)
- [x] Support the windows platform
- [x] Update docs
- [ ] HALO model annotation (ETA: 24/10)
- [ ] Full continuous video sequence (ETA: 24/10)
- [ ] HOGraspNet v2 (ETA: 24/10)
	- Object pose/contact map quality will be enhanced.
	- Images and annotations for articulated objects will be added.


## Terms and conditions
<a name="terms"></a>
The download and use of the dataset is released for academic research only and it is free to researchers from educational or research institutes for non-commercial purposes. When downloading the dataset you agree to (unless with expressed permission of the authors): not redistribute, modificate, or commercial usage of this dataset in any way or form, either partially or entirely.

If using this dataset, please cite the following paper:

```
@inproceedings{2024graspnet,
        title={Dense Hand-Object(HO) GraspNet with Full Grasping Taxonomy and Dynamics},
        author={Cho, Woojin and Lee, Jihyun and Yi, Minjae and Kim, Minje and Woo, Taeyun and Kim, Donghwan and Ha, Taewook and Lee, Hyokeun and Ryu, Je-Hwan and Woo, Woontack and Kim, Tae-Kyun},
        booktitle={ECCV},
        year={2024}
}
```

## Acknowledgments
이 연구는 과학기술정보통신부의 재원으로 정보통신기획평가원 및 한국지능정보사회진흥원의 지원을 받아 구축된 "물체 조작 손 동작 3D 데이터"을 활용하여 수행된 연구입니다. (CRC 21011, No.2019-0-01270 and RS-2023-00228996, MSIT).
본 연구에 활용된 데이터는 AI 허브([aihub.or.kr](http://aihub.or.kr/))에서 다운로드 받으실 수 있습니다.
This research (paper) used datasets from 'The Open AI Dataset Project (AI-Hub, S. Korea)'.
All data information can be accessed through 'AI-Hub ([www.aihub.or.kr](http://www.aihub.or.kr/))'.
