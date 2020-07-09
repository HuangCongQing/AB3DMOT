


# Lidar Detection Baseline 样例

进入 ModelArts 控制台，左侧栏选择 ”开发环境“，进入 Notebook

## 创建 Notebook：

工作环境选择 Python3；
GPU 选择 1×v100；
存储配置选择云硬盘，容量选择 100G；

然后点击下一步、创建；


进入 notebook ，通过右侧 "New" 按钮，新建 terminal，进入远程 terminal 环境：


## 下载数据和配置环境

### 从 obs 桶中下载代码和脚本

通过 ```vim data_download.py``` 创建 python 下载脚本，并 copy 以下内容到 py 文件中：

```
# download data, code
from modelarts.session import Session
import os

target_folder = ['/home/ma-user/work/workspace', '/home/ma-user/work/workspace/DeepCamp_Lidar', '/opt/conda/envs/']
for folder in target_folder:
    if not os.path.exists(folder):
        os.makedirs(folder)

session = Session()

# download code 
session.download_data(bucket_path="/deecamp-obs10/workspace/baseline.tar", path="/home/ma-user/work/workspace/baseline.tar")

# download data
data_file_list = ['kitti_infos_val.pkl', 'kitti_infos_train.pkl', 'labels_filer.tar', 'test_video_filter.tar', 'train_val_filter.tar', 'test_filter.tar']
for file_name in data_file_list:
	session.download_data(bucket_path="/deecamp-obs10/workspace/DeepCamp_Lidar/{}".format(file_name), path="/home/ma-user/work/workspace/DeepCamp_Lidar/{}".format(file_name))

# download environment
session.download_data(bucket_path="/deecamp-obs10/workspace/env/torch.tar", path="/opt/conda/envs/torch.tar")
```

运行脚本进行数据、代码及运行环境下载： 
```
# 每个文件下载成功后，会输出对应的 sucessfully 提示
python3 data_download.py
```

运行 shell 命令，对数据进行解压：
```
cd /home/ma-user/work/workspace
tar xvf baseline.tar

cd /home/ma-user/work/workspace/DeepCamp_Lidar
for tar in *.tar;  do tar xvf $tar; done

cd /opt/conda/envs/
tar xvf torch.tar
```

配置环境相应权限：
```
cd /opt/conda/envs/torch/bin
chmod 777 ./*
```


## 训练与评测：

### Train

进入 Det3D 文件夹，进行训练：
```
cd /home/ma-user/work/workspace/baseline/Det3D
source activate torch
mkdir res
python3.6 tools/train.py ./examples/second/configs/deepcamp_all_vfev3_spmiddlefhd_rpn1_mghead_syncbn.py --work_dir ./res
```

### Eval
训练 pipeline 默认每间隔 5 epoch 进行 eval 评测，并输出评测结果

### Note：
如果操作报错的话，需要检查数据是否下载成功