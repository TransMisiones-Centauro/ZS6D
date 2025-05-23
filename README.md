# ZS6D
<img src="./assets/overview.png" width="500" alt="teaser"/>

We demonstrate the effectiveness of deep features extracted from self-supervised, pre-trained Vision Transformer (ViT) for Zero-shot 6D pose estimation.
For more detailed information check out the corresponding [[paper](https://ieeexplore.ieee.org/document/10611464)].

## Overview of the Pipeline:

![pipeline](./assets/ZS6D_pipeline.png)

Note that this repo only deals with 6D pose estimation, you need segmentation masks as input. 
These can be obtained with supervised trained methods or zero-shot methods. 
For zero-shot we refer to [CNOS](https://github.com/nv-nguyen/cnos).

## Installation:
2025/05/07 - Tested with:
- 11th Gen Intel(R) Core(TM) i7-11800H @ 2.30GHz - 1 socket, 8 cores per socket, 2 threads per core
- 32GiB RAM - 2 x 16GiB SODIMM DDR4 Synchronous 3200 MHz
- NVIDIA GeForce RTX 3080 Mobile 16GB
- Ubuntu 22.04.5
- NVIDIA Driver Version: 535.247.01
- CUDA Version: 12.2.
- Conda 25.3.1

To setup the environment to run the code locally follow these steps:

### Install Miniconda:
[documentation](https://www.anaconda.com/docs/getting-started/miniconda/install#quickstart-install-instructions)
```
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh \
  -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm ~/miniconda3/miniconda.sh
source ~/miniconda3/bin/activate
```

### Create a Conda environment:
```
conda create --name zs6d python=3.9
conda activate zs6d
conda install pytorch==2.0.1 torchvision==0.15.2 torchaudio==2.0.2 pytorch-cuda=11.7 -c pytorch -c nvidia
pip install tqdm==4.65.0
pip install timm==0.9.16
pip install matplotlib==3.8.3
pip install scikit-learn==1.4.1.post1
pip install opencv-python==4.9.0.80
pip install git+https://github.com/lucasb-eyer/pydensecrf.git@dd070546eda51e21ab772ee6f14807c7f5b1548b
pip install transforms3d==0.4.1
pip install pillow==9.4.0
pip install plyfile==1.0.3
pip install trimesh==4.1.4
pip install imageio==2.34.0
pip install pypng==0.20220715.0
pip install vispy==0.12.2
pip install pyopengl==3.1.1a1
pip install pyglet==2.0.10
pip install numba==0.59.0
pip install jupyter==1.0.0
```

### Clone this repository:
```
git clone https://github.com/TransMisiones-Centauro/ZS6D.git
cd ZS6D && git submodule update --init --recursive
```

## Testing:

### Download templates:
Download ycbv templates (zs6d_templates_ycbv.zip) from [this link](https://drive.google.com/file/d/1t0nk_1R931LYl-F9nrYLfk-jKxoki-0A/view?usp=sharing) into ./templates
```
cd ./templates && unzip zs6d_templates_ycbv.zip
```

### Render templates:
To generate templates from a object model to perform inference, we refer to the [ZS6D_template_rendering](https://github.com/haberger/ZS6D_template_rendering) repository.

### Prepare templates:
Run the preparation script with your config_file to generate your_template_gt_file.json and prepare the template descriptors and template uv maps.
```
python3 prepare_templates_and_gt.py --config_file \
  zs6d_configs/template_gt_preparation_configs/cfg_template_gt_generation_ycbv.json
```

### Download dataset for testing:
```
mkdir -p dataset/ycbv_test_bop19
cd dataset/ycbv_test_bop19
export DATASET_NAME=ycbv
export SRC=https://huggingface.co/datasets/bop-benchmark/${DATASET_NAME}/resolve/main
wget $SRC/${DATASET_NAME}_test_bop19.zip
unzip ycbv_test_bop19.zip
```

### Launch Jupyter server:
```
jupyter notebook test_zs6d.ipynb
```

![test results](./assets/test_bop19.png)

## Docker setup:

Build the Docker image:
```
docker build -t zs6d .
```

Launch Jupyter in a Docker container:
```
docker run -it --rm --network host --runtime nvidia \
  -v .:/code -v $HOME/.cache/torch:/root/.cache/torch \
  zs6d jupyter notebook --no-browser --allow-root
```
Open in your browser one of the shown URLs:
```
    	http://localhost:8888/tree?token=...
    	http://127.0.0.1:8888/tree?token=...
```

<!--

### Docker setup:

### ROS integration:
To run the ros wrapper do the following:

- set up the NVIDIA container toolkit
- download ycbv templates from [this link](https://drive.google.com/file/d/1t0nk_1R931LYl-F9nrYLfk-jKxoki-0A/view?usp=sharing) and put the ycbv folder into ./templates
- edit camera intrinsics and obj name mappings in ```zs6d_configs/bop_eval_configs/cfg_ros_ycbv_inference_bop.json```. The keys in the object name mapping are the names that are passed to the pose estimator and the values are the bop object ids (as strings).
- Set the ROS_IP and ROS_MASTER_URI in ```ros_entrypoint.sh```.
- update the submodules with ```git submodule update --init --recursive```
- Build the docker image with ```docker build -t zs6d .```
- Allow the docker container to access the display by running ```xhost local:docker```.
- Run the docker container with the following command:

```bash
docker run -it --rm --runtime nvidia --privileged \
  -e DISPLAY=${DISPLAY} -e NVIDIA_DRIVER_CAPABILITIES=all \
  -v `pwd`:/code -v `pwd`/torch_cache:/root/.cache/torch \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
 --net host zs6d /bin/bash
```
Once you are inside the docker container, run the following command to check if the docker container has access to the GPU:

```bash
glxinfo | grep "OpenGL version string"
```
The output should be ```OpenGL version string: > 2.xx``` and show the nvidia driver version.

Afterwards, run the following command to calculate the descriptors for the templates and prepare the ground truth:
```bash
python prepare_templates_and_gt.py
```

**All of the previous step only have to be done once per machine.**



The following commands have to be run **every time** you want to start the ros container:

```bash
xhost local:docker
docker run -it --rm --runtime nvidia --privileged \
  -e DISPLAY=${DISPLAY}  -e NVIDIA_DRIVER_CAPABILITIES=all \
  -v `pwd`:/code -v `pwd`/torch_cache:/root/.cache/torch \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  --net host zs6d
```

## Template rendering:
To generate templates from a object model to perform inference, we refer to the [ZS6D_template_rendering](https://github.com/haberger/ZS6D_template_rendering) repository.

## Template preparation:

1. set up a config file for template preparation

```zs6d_configs/template_gt_preparation_configs/your_template_config.json```

2. run the preparation script with your config_file to generate your_template_gt_file.json and prepare the template descriptors and template uv maps

```python3 prepare_templates_and_gt.py --config_file zs6d_configs/template_gt_preparation_configs/your_template_config.json```


## Inference:
After setting up your_template_config.json you can instantiate your ZS6D module and perform inference. An example is provided in:

```test_zs6d.ipynb```


## Evaluation on BOP Datasets:

1. set up a config file for BOP evaluation

```zs6d_configs/bop_eval_configs/your_eval_config.json```

2. Create a ground truth file for testing, the files for BOP'19-23 test images are provided for lmo, tless and ycbv. For example for lmo:

```gts/test_gts/lmo_bop_test_gt_sam.json```

Additionally, you have to download the corresponding [BOP test images](https://bop.felk.cvut.cz/datasets/#LM-O). If you want to test another dataset as the provided, you have to generate a ground truth file with the following structure:

```json
{
  "object_id": [
    {
      "scene_id": "00001", 
      "img_name": "relative_path_to_image/image_name.png", 
      "obj_id": "..", 
      "bbox_obj": [], 
      "cam_t_m2c": [], 
      "cam_R_m2c": [], 
      "cam_K":[],
      "mask_sam": [] // mask in RLE encoding
    }
    ,...
  ]
}
```

3. run the evaluation script with your_eval_config.json

```python3 prepare_templates_and_gt.py --config_file zs6d_configs/template_gt_preparation_configs/your_eval_config.json```

-->

## Acknowledgements
This project is built upon [dino-vit-features](https://github.com/ShirAmir/dino-vit-features), which performed a very comprehensive study about features of self-supervised pretrained Vision Transformers and their applications, including local correspondence matching. Here is a link to their [paper](https://arxiv.org/abs/2112.05814). We thank the authors for their great work and repo.

## Citation
If you found this repository useful please consider starring ⭐ and citing :

```
@inproceedings{ausserlechner2024zs6d,
  title={Zs6d: Zero-shot 6d object pose estimation using vision transformers},
  author={Ausserlechner, Philipp and Haberger, David and Thalhammer, Stefan and Weibel, Jean-Baptiste and Vincze, Markus},
  booktitle={2024 IEEE International Conference on Robotics and Automation (ICRA)},
  pages={463--469},
  year={2024},
  organization={IEEE}
}
```
