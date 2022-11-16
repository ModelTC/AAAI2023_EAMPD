# Practical AI Challenge at AAAI 2023
For full challenge details, please see the [AAAI2023 Practical AI challenge](https://practical-dl.sensecore.cn/#/intro) page.

For general questions regarding this challenge, you can join the group via Slack or WeChat.🤗

Slack: Join by clicking [this link](https://join.slack.com/t/w1666859556-fgw388026/shared_invite/zt-1iyl8cqq8-PnRCgvLEht9oVr9tA4KMmg).

WeChat: Join by scanning the following QR code.

![wechat](https://github.com/ModelTC/AAAI2023_EAMPD/blob/master/figs/wechat.png)

## Training and Submission

We recommend that participants use [United-Perception](https://github.com/ModelTC/United-Perception/tree/main/up) to train the model and provide two baseline models (resnet18 and resnet18c_x0_25). The results are as follows:

| model_id | backbone        | bs     | epoch | Bag of tricks | kd   | rep(type) | eql  | top1 (test1w) |
| -------- | --------------- | ------ | ----- | ------------- | ---- | --------- | ---- | ------------- |
| 1        | resnet18        | 4 * 64 | 100   | yes(strikes)  | no   | no        | yes  | 88.09         |
| 2        | resnet18c_x0_25 | 4 * 64 | 100   | yes(strikes)  | no   | no        | yes  | 87.01         |

You can reproduce the above results by following these steps:

**1. Prepare code base**

```python
git clone https://github.com/ModelTC/United-Perception
cd United-Perception
sh easy_setup.sh
```

**2. prepare your dataset**

You can download the training dataset of the challenge from [here](https://practical-dl.sensecore.cn/#/competitions). The training dataset is organized as follows:

```
your_data_path
├── data
│   ├── 0.png
│   ├── 1.png
│   ├── ...
│   └── 50002.png
└── train.txt
```

Change the data path in the provided configuration file to your data path:

```yaml
  train:
    dataset:
      type: cls
      kwargs:
        meta_file: your_data_path/train.txt
        image_reader:
          type: fs_pillow
          kwargs:
            image_dir: your_data_path/
            color_mode: RGB
        transformer: [*random_resized_crop, *random_horizontal_flip, *pil_color_jitter,*to_tensor, *normalize]
```

**3. Training the model**

You can easily train the model with the training scripts we provide:

```
sh scripts/dist_train.sh num_gpus your_config_path
```

**4. export the onnx file**

After training, you can easily export the onnx files using the scripts provided by UP:

```
sh scripts/to_onnx.sh num_gpus your_config_path
```

and the result will be saved in the `./toonnx/`.  It is worth noting that you need to modify the input_size before you run the to_onnx.sh

```python
#!/bin/bash

ROOT=..
T=`date +%m%d%H%M`
export ROOT=$ROOT
cfg=$2
export PYTHONPATH=$ROOT:$PYTHONPATH
CPUS_PER_TASK=${CPUS_PER_TASK:-2}

python -m up to_onnx \
  --ng=$1 \
  --launch=pytorch \
  --config=$cfg \
  --save_prefix=toonnx \
  --input_size=3x112x112 \  # change the input size to yours
  2>&1 | tee log.deploy.$(basename $cfg) 
```

**5. submission**

After exporting the onnxfile, submit your onnxfile to [online judge](https://practical-dl.sensecore.cn/#/onlineJudge) to get your scores.

<img src="https://github.com/ModelTC/AAAI2023_EAMPD/blob/master/figs/submit.png" alt="submit" style="zoom:25%;" />

## OpenI Platform

We provide training resources for participants on the OpenI platform where the participants can use two v100 or A100 to train their models.

**1. migrate the code from github**

You can first register your account in https://openi.org.cn/ and migrate the  [United-Perception](https://github.com/ModelTC/United-Perception/tree/main/up)  repo to openi as follows:

<img src="https://github.com/ModelTC/AAAI2023_EAMPD/blob/master/figs/home.png" alt="home" style="zoom:25%;" />

<img src="https://github.com/ModelTC/AAAI2023_EAMPD/blob/master/figs/migrate.png" alt="migrate" style="zoom:25%;" />

<img src="https://github.com/ModelTC/AAAI2023_EAMPD/blob/master/figs/repo.png" alt="repo" style="zoom:25%;" />

**2. build your dataset**

You can build your own datasets by associating datasets we provided.

<img src="https://github.com/ModelTC/AAAI2023_EAMPD/blob/master/figs/data1.png" alt="data1" style="zoom:25%;" />

<img src="https://github.com/ModelTC/AAAI2023_EAMPD/blob/master/figs/data2.png" alt="data2" style="zoom:25%;" />

**3. Debug and Train**

The OpenI provides convenient debugging and training functions. Participants can debug their code by following the steps below.

<img src="https://github.com/ModelTC/AAAI2023_EAMPD/blob/master/figs/debug.png" alt="debug" style="zoom:25%;" />

<img src="https://github.com/ModelTC/AAAI2023_EAMPD/blob/master/figs/debug2.png" alt="debug2" style="zoom:25%;" />

<img src="https://github.com/ModelTC/AAAI2023_EAMPD/blob/master/figs/debug3.png" alt="debug3" style="zoom:25%;" />

<img src="https://github.com/ModelTC/AAAI2023_EAMPD/blob/master/figs/debug4.png" alt="debug4" style="zoom:25%;" />

<img src="https://github.com/ModelTC/AAAI2023_EAMPD/blob/master/figs/debug5.png" alt="debug4" style="zoom:25%;" />

You can choose your most similar tools to debug your code.

<img src="https://github.com/ModelTC/AAAI2023_EAMPD/blob/master/figs/debug6.png" alt="debug4" style="zoom:25%;" />