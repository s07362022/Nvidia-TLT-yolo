# Nvidia-TLT-yolo

CUDA install; CUDA10.2
-------------
Update Note: When you install CUDA and get error, please use it.
```Bash
sudo apt-get purge nvidia*
sudo apt-get autoremove
sudo apt-get autoclean
sudo rm -rf /usr/local/cuda*
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-ubuntu1804.pin
sudo mv cuda-ubuntu1804.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/10.2/Prod/local_installers/cuda-repo-ubuntu1804-10-2-local-10.2.89-440.33.01_1.0-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1804-10-2-local-10.2.89-440.33.01_1.0-1_amd64.deb
sudo apt-key add /var/cuda-repo-10-2-local-10.2.89-440.33.01/7fa2af80.pub
sudo apt-get update
sudo apt-get -y install cuda
sudo reboot
```

OR 

using apt install
-----------------

first, search deviece version
```Bash 
sudo add-apt-repository ppa:graphics-drivers
ubuntu-drivers list
```

Install Nvidia-driver
```Bash
sudo apt install nvidia-driver-<version>
sudo reboot
```
eg : sudo apt install nvidia-driver-440 ; #for  RTX2060

add cudnn 7.6.5
---------
download : https://developer.nvidia.com/compute/machine-learning/cudnn/secure/7.6.5.32/Production/10.2_20191118/cudnn-10.2-linux-x64-v7.6.5.32.tgz

after unzip ; copy to cuda dir (include) , (lib), (bin)

```Bash
sudo cp  cuda/lib/* /usr/local/cuda/lib/ 
sudo cp cuda/include/* /usr/local/cuda/include/
sudo cp cuda/bin/* /usr/local/cuda/bin/
```

add cudnn path to env

```Bash
echo 'export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/cuda/lib:/usr/local/cuda/extras/CUPTI/lib"' >> ~/.bashrc
echo 'export CUDA_HOME=/usr/local/cuda' >> ~/.bashrc
echo 'export PATH="/usr/local/cuda/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```


Nvidia-TLT-v2.0_py3
------------------
![nvidia-smi](https://i.ibb.co/f852NGV/Screenshot-from-2021-09-05-01-40-16.png)
    **Base Components		Version
    Ubuntu 			18.04.2
    NVIDIA CUDA® 		10.2.89
    NVIDIA cuDNN 		7.6.5
    NVIDIA NCCL 			2.6.4
    NVIDIA Docker 		2.2.2
    Docker			19.03.8
    NVIDIA Driver		440.82
    DCGM (NV SWITCH)		1.7.2
   

Step1 : Install Nvidia-Docker :
------------------------------

Note : your local CUDA Version have to correspond Nvidia-docker CUDA Version.


eg : like me, my local machiene is cuda10.2 , you have to install nvidia-docker cuda10.2


*reference : [install nvidia-docker to ubuntu](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#installing-on-ubuntu-and-debian)
```Bash
curl https://get.docker.com | sh \
  && sudo systemctl --now enable docker
```

*setup :

```Bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
   && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
   && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
```

*install nividia-docker2

```Bash
sudo apt-get install -y nvidia-docker2
```

*start Docker:

```Bash
sudo systemctl stop docker
sudo systemctl start docker
sudo systemctl restart docker
```

*Run & Check Nvidia-Docker Version:

```Bash
sudo docker run --rm --gpus all nvidia/cuda:10.2-base nvidia-smi
```

![nvdia-docker/nvidia-smi](https://i.ibb.co/fDJPRJX/Screenshot-from-2021-09-05-02-03-01.png)

    *Note: if your local cuda version is 10.2 then nvidia-docker also cuda 10.2 too.
    can't update version or low than local version.
    if you wnat to change version, you have to change local machine&cuda <version>*


Step2 : get NGC Key 
-----------------

please logo in (https://ngc.nvidia.com/setup/api-key)

*AMD64 Linux Install

```Bash
wget -O ngccli_linux.zip https://ngc.nvidia.com/downloads/ngccli_linux.zip && unzip -o ngccli_linux.zip && chmod u+x ngc
```

```Bash
md5sum -c ngc.md5
```

```Bash
echo "export PATH=\"\$PATH:$(pwd)\"" >> ~/.bash_profile && source ~/.bash_profile
```

```Bash
ngc config set
```

Logo in NGC : 

```Bash
docker login nvcr.io
```

*right-up "Generate API Key"
  you can get 
  Username : $ ....
  Password : ......
  (API Key)*


Step3 : download TLT 
-------------

*Note : Now TLT version is V3.0 for CUDA11.1 up, but we just CUDA10.2. So we donwloda TLT version is V2.0

downloda TLT <version> = v2.0_py3:

  ```Bash
  docker pull nvcr.io/nvidia/tlt-streamanalytics:<version>
  ```
  
  
  
  Run TLT (docker env):
  ```Bash
  docker run --runtime=nvidia -it nvcr.io/nvidia/tlt-streamanalytics:<version> /bin/bash
  ```
  
  
  
  Use net connect port to port:
  ```Bash
  docker run --runtime=nvidia -it -v /home/<username>/tlt-experiments:/workspace/tlt-experiments -p 8888:8888 nvcr.io/nvidia/tlt-streamanalytics:<version>
  ```
  
  
  Run Jupyer to TLT :
  ```Bash
  jupyter notebook --ip 0.0.0.0 --allow-root
  ```

  
*Note the code are in dir "examples"*
  
  

Train Model
================
  
Step1 : build Dataset
-----------------------
  
![tree](https://i.ibb.co/zxPd2f8/Screenshot-from-2021-09-05-02-28-28.png)
  
under the <training>  dir, we make two dir <images>,<labels>
  
Note : the objection Label is YOLO format, we have to converse to KITTI format.
  
  Use Python code to converse "YOLO" to "KITTI"
  **code in file "yolo_to_kitti.py"
  
  check the code and fix:
  ``` python
  class_name = ' ' #change here
  restore_results('./images','./labels') #change here
  
  
```Bash
  python3 yolo_to_kitti.py
```

![kitti-label](https://i.ibb.co/b385M7f/Screenshot-from-2021-09-05-02-41-42.png)
  
Step2 : Open yolo.ipynb
------------------------
generate the best anchor shape
  
![config info](https://i.ibb.co/3mVZ67R/Screenshot-from-2021-09-05-02-45-11.png)

*Note : The anchor shape generated by this script is sorted. Write the first 3 into small_anchor_shape in the config file. Write middle 3 into mid_anchor_shape. Write last 3 into big_anchor_shape.*

  
Prepare tfrecords from kitti format dataset

  PNG -> JPG
![](https://i.ibb.co/qrPtYp6/Screenshot-from-2021-09-05-02-50-22.png)

Step3 : modify pre-model-config
-------------------------------
  
*the code in github file, you can download to use*

we have to modify < big_anchor_shape>  <mid_anchor_shape>  <small_anchor_shape> from Step2. (prefix 3 tuple) (infix 3 tupel) (postfix 3 tuple) 
  
```python
yolo_config {
  big_anchor_shape: "[(62.00, 33.00), (56.00, 37.00), (62.00, 39.00)]"
  mid_anchor_shape: "[(58.00, 42.00), (67.50, 39.00), (64.00, 42.00)]"
  small_anchor_shape: "[(61.00, 46.00), (68.00, 43.00), (74.00, 48.00)]"
  matching_neutral_box_iou: 0.5
```
  
we have to modify label classes "key","value"

```python
    image_extension: "jpg"
  target_class_mapping {
      key: "cap"
      value: "cap"
  }
```
  
Train TLT - yolo 

```Bash
print("To run with multigpu, please change --gpus based on the number of available GPUs in your machine.")
!tlt-train yolo -e $SPECS_DIR/user_resnet.txt \
                -r $USER_EXPERIMENT_DIR/experiment_dir_unpruned \
                -k $KEY \
                -m $USER_EXPERIMENT_DIR/pretrained_resnet18/tlt_pretrained_object_detection_vresnet18/resnet_18.hdf5 \
                --gpus 1
```
  

  
![TLT train](https://i.ibb.co/GxNF6zd/Screenshot-from-2021-09-05-03-08-22.png)

DONE!!!!!!!!!!!!!!!!!
----------------------

   
if meet root apt-get install erro:
```bash
   sudo rm /var/lib/dpkg/lock-frontend
   sudo rm /var/lib/dpkg/lock
   sudo rm /var/cache/apt/archives/lock
   ````
