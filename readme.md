# From Bias To Fairness

分为两个文件，一个是训练模型的，一个是测试模型的

请分别创建两个文件夹来保存，分别下载对应的依赖

### train model: CDA + LoRA

请先安装LMFLOW框架，具体安装请参考文件：https://github.com/OptimalScale/LMFlow

安装完成后接下来进行以下操作：

##### 1、插入文件

将下面两个文件加入到LMFLOW的scripts中：

**"train_model/scripts/create_cda_data.sh" "train_model/scripts/run_finetune_with_lora_save_aggregated_weights_cda.sh"**



将下面的文件加入到LMFLOW中的examples中：
**"train_model/examples/cda_augments_data.py"**



将下面的文件加入到LMFLOW中去，具体地址可以加入到lmflow/data/text/

**"train_model/data\text/wikipedia-10.txt.zip"**

##### 



##### 2、运行脚本

###### 解压原始的用于cda的数据集

`unzip lmflow/data/text/wikipedia-10.txt.zip`

###### cda

先进行反事实数据生成，在LMflow目录下运行以下指令：
`bash scripts/create_cda_data.sh`

该脚本有两个参数，分别是生成的数据集的保存地址和原始的用于cda的数据集地址

```
--save-path "data/cda_data/" 
--ori-data-path "data/text/wikipedia-10.txt"
```

###### LoRA

接下来是使用cda数据集来微调模型，在LMflow目录下运行以下指令

`bash scripts/run_finetune_with_lora_save_aggregated_weights_cda.sh`

该脚本有以下参数，请根据实际情况进行修改

```
bias_types=('gender' 'race' 'religion')  
model_name_or_path="/date1/zwx/bias-bench-main/gpt2"  微调的模型的地址
deepspeed_args="--master_port=10004"                  通信端口号
steps                                                 训练数据的步长
dataset_path                                          cda生成数据集的地址
output_dir                                            训练好后的模型输出地址

其他的参数请根据具体情况进行修改
```



### test model: bias-bench-main

依赖下载【若缺少依赖，直接下载最新的即可】：

```
torch==2.0.0
transformers==4.31.0
scipy==1.7.3
scikit-learn==1.0.2
nltk==3.7.0
datasets==1.18.3
accelerate==0.24.1
deepspeed==0.10.0
numpy==1.22.4
tqdm==4.66.1
```



测试模型前请分别修改batch_jobs中各个脚本的参数为自己的实际情况

##### bath_jobs中用到的各个脚本介绍

各个脚本的参数都是需要进行实际修改的，请根据自己的微调模型和原始模型的等的实际地址进行修改

 _experiment_configuration.sh   包含偏见类别，主文件地址**【需要修改为自己的文件目录】**，模型名称映射等

 seat.sh                                            测试原始模型在SEAT数据集的偏见程度

 seat_debias.sh                               测试微调后的模型在SEAT数据集的偏见程度

 crows.sh                                         测试原始模型在SEAT数据集的偏见程度

 crows_debias.sh                            测试微调后的模型在Crows-Pairs数据集的偏见程度

 stereoset.sh                                   测试原始模型在stereoset数据集的偏见程度

 stereoset_debias.sh                     测试微调后的模型在stereoset数据集的偏见程度

 stereoset_to_json.sh                    将模型在stereoset数据集上测试的结果转化为json格式

##### 脚本运行顺序：

先修改 _experiment_configuration.sh中的参数为自己的参数，比如persistent_dir等

接下来就可以执行各个测试集脚本了[**各个脚本的参数请根据实际情况进行修改**],例如

`bash batch_jobs/seat.sh`

`bash batch_jobs/seat_debias.sh`

`bash batch_jobs/crows.sh`

`bash batch_jobs/crows_debias.sh`

`bash batch_jobs/stereoset.sh`

`bash batch_jobs/stereoset_debias.sh`

`bash batch_jobs/stereoset_to_json.sh`

**注意事项**

关于测试原始模型的脚本，模型加载主要是通过参数**model_name_or_path** 

而对于测试微调模型的脚本，模型加载则主要是通过参数**load_path**





