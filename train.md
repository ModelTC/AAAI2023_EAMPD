## Baseline Results

We recommend that participants use [United-Perception](https://github.com/ModelTC/United-Perception/tree/main/up) to train the model and provide two baseline models (resnet18 and resnet18c_x0_25). The results are as follows:

[EQL loss paper](https://arxiv.org/abs/2210.05566)

| model_id | backbone        | bs     | epoch | Bag of tricks | eql  | top1 (test1w) |
| -------- | --------------- | ------ | ----- | ------------- | ---- | ------------- |
| 1        | [resnet18](https://github.com/ModelTC/AAAI2023_EAMPD/blob/master/configs/res18_strikes_100e_bce.yaml)        | 4 * 64 | 100   | yes(strikes)  | no   | 86.88         |
| 2        | [resnet18](https://github.com/ModelTC/AAAI2023_EAMPD/blob/master/configs/res18_strikes_100e_bce_eql.yaml)        | 4 * 64 | 100   | yes(strikes)  | yes  | 88.09         |
| 3        | [resnet18c_x0_25](https://github.com/ModelTC/AAAI2023_EAMPD/blob/master/configs/res18_0.25_strikes_100e_bce.yaml) | 4 * 64 | 100   | yes(strikes)  | no   | 84.52         |
| 4        | [resnet18c_x0_25](res18_0.25_strikes_100e_bce_eql.yaml) | 4 * 64 | 100   | yes(strikes)  | yes  | 87.01         |

You can reproduce the above results by following these steps:

### 1. Prepare code base

```python
git clone https://github.com/ModelTC/United-Perception
cd United-Perception
# in this challenge, we only need python requirements
pip install --user -r requirements.txt 
```
### 2. prepare your dataset

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

### 3. Training and Eval the model

You can easily train the model with the training scripts we provide:

**Specially, in this challenge, we only need cls tasks.**
**export DEFAULT_TASKS=cls**

* train
```shell
sh scripts/dist_train.sh num_gpus your_config_path

#!/bin/bash

ROOT=./  # your up path root
T=`date +%m%d%H%M`
export ROOT=$ROOT
cfg=$2
export PYTHONPATH=$ROOT:$PYTHONPATH
# in this challenge, we only need cls tasks
export DEFAULT_TASKS=cls
python -m up train \
  --ng=$1 \
  --launch=pytorch \
  --config=$cfg \
  --display=10 \
  2>&1 | tee log.train.$T.$(basename $cfg) 
```

* eval or test
```shell
sh scripts/dist_test.sh num_gpus your_config_path

#!/bin/bash

ROOT=./  # your up path root
T=`date +%m%d%H%M`
export ROOT=$ROOT
cfg=$2
export PYTHONPATH=$ROOT:$PYTHONPATH
# in this challenge, we only need cls tasks
export DEFAULT_TASKS=cls
python -m up train \
  -e \
  --ng=$1 \
  --launch=pytorch \
  --config=$cfg \
  --display=10 \
  2>&1 | tee log.test.$T.$(basename $cfg) 
```

### 4. export the onnx file

After training, you can easily export the onnx files using the scripts provided by UP:

```
sh scripts/to_onnx.sh num_gpus your_config_path
```

and the result will be saved in the `./toonnx/`.  It is worth noting that you need to modify the input_size before you run the to_onnx.sh

```shell
#!/bin/bash

ROOT=./  # your up path root
T=`date +%m%d%H%M`
export ROOT=$ROOT
cfg=$2
export PYTHONPATH=$ROOT:$PYTHONPATH
# in this challenge, we only need cls tasks
export DEFAULT_TASKS=cls
CPUS_PER_TASK=${CPUS_PER_TASK:-2}

python -m up to_onnx \
  --ng=$1 \
  --launch=pytorch \
  --config=$cfg \
  --save_prefix=toonnx \
  --input_size=3x112x112 \  # change the input size to yours
  2>&1 | tee log.deploy.$(basename $cfg) 
```


### 5. More Detail

Please refer [United-Perception](https://github.com/ModelTC/United-Perception/tree/main/up) docs.