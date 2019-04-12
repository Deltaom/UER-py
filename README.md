# UER-py

<img src="uer-logo.jpg" width="350" hegiht="350" align=left />

Pre-training has become an essential part for NLP tasks and has led to remarkable improvements.
UER-py is a toolkit for pre-training on general-domain corpus and fine-tuning on downstream task. UER-py maintains model modularity and supports research extensibility. It facilitates the implementations of different pre-training models (such as BERT), and provides interfaces for users to further extend upon. UER-py also incorporates a series of mechanisms for better pre-training performance and efficiency. Our works outperform Chinese model provided by Google BERT on a range of datasets.


<br/>

## Features
UER-py has the following features:
- UER-py is able to reproduce the results of existing pre-training models (such as [Google BERT](https://github.com/google-research/bert)). 
- UER-py supports CPU, single GPU, and distributed training modes. 
- Model modularity. UER-py consists of four components: subencoder, encoder, target, and fine-tuning. UER-py 
- Ample modules are implemented in UER-py. Clear and robust interface allows users to combine modules with as few restrictions as possible.
- UER-py incorporates many mechanisms in pre-processing, pre-training, and fine-tuning stages, which largely boosts the efficiency in both speed and memory.
- SOTA results. Our works outperform Google BERT's Chinese model, which provides a new strong baselines for a range of datasets.
- Chinese model zoo. We are training models with different corpora, encoders, and targets.


<br/>

## Quickstart
We use BERT model and book review classification dataset to demonstrate the way of using UER-py. There are three input files: book review corpus, book review dataset, and vocabulary. All files are encoded in UTF-8 and they are included in this project.

The format of the corpus for BERT is as follows：
```
doc1-sent1
doc1-sent2
doc1-sent3

doc2-sent1

doc3-sent1
doc3-sent2
```

The format of classification dataset is as follows (label and instance are separated by \t):
```
1 instance1
0 instance2
1 instance3
```

The format of vocabulary is as follows:
```
word-1
word-2
...
word-n
```

Suppose we have a machine with 8 GPUs.
First of all, we preprocess the book review corpus:
```
python3 preprocess.py --corpus_path corpora/book_review_bert.txt --vocab_path models/google_vocab.txt --dataset_path dataset \
                      --dataset_split_num 8 --target bert
```
Since we have 8 GPUs, we split the dataset into 8 parts. Each GPU processes one part.
以单机8GPU情形为例
这里我们使用半监督的微调策略，先让模型在下游任务数据集语料上进行无监督训练，使其熟悉下游任务数据集的内容
首先我们对语料进行预处理
```
python3 preprocess.py --corpus_path corpora/book_review_bert.txt --vocab_path models/google_vocab.txt --dataset_path dataset --dataset_split_num 8
```
由于有8个GPU，我们将数据集分成8份，每个GPU对应一个进程，处理一份数据
然后下载Google预训练模型google_model.bin，放到models文件夹之中
接着加载Google预训练模型，对语料进行预训练（8GPU大概需要5小时，BERT-PyTorch已经提供了训练好的模型book_review_model.bin）
```
python3 pretrain.py --dataset_path dataset --vocab_path models/google_vocab.txt --pretrained_model_path models/google_model.bin \
    --output_model_path models/book_review_model.bin  --world_size 8 --gpu_ranks 0 1 2 3 4 5 6 7 \
    --total_steps 20000 --save_checkpoint_steps 1000
```
最后进行分类任务，我们可以使用Google提供的预训练模型google_model.bin：
```
python3 classifier.py --pretrained_model_path models/google_model.bin --vocab_path models/google_vocab.txt \
    --train_path datasets/book_review/train.txt --dev_path datasets/book_review/dev.txt --test_path datasets/book_review/test.txt \
    --epochs_num 3 --batch_size 64
```
也可以使用我们自己预训练得到的模型book_review_model.bin：
```
python3 classifier.py --pretrained_model_path models/review_model.bin --vocab_path models/google_vocab.txt \
    --train_path datasets/book_review/train.txt --dev_path datasets/book_review/dev.txt --test_path datasets/book_review/test.txt \
    --epochs_num 3 --batch_size 64
```
实验结果表明Google预训练模型的结果是87.5;而使用预训练模型book_review_model.bin的结果是88.1

<br/>

## 中文预训练模型
BERT-PyTorch目前提供谷歌中文模型、人民日报模型、豆瓣书评模型，更多的中文预训练模型将陆续开放
模型下载链接：
<table>
<tr align="center"><th> 模型 <th> 链接 <th> 说明 
<tr align="center"><td> google_model.bin <td> https://share.weiyun.com/51tMpcr <td> 谷歌官方中文模型，字
<tr align="center"><td> rmrb_model.bin <td> https://share.weiyun.com/5w1lGV0 <td> 人民日报中文模型，字
<tr align="center"><td> book_review_model.bin <td> https://share.weiyun.com/59OoBes <td> 豆瓣书评中文模型，字
<tr align="center"><td> google_vocab.txt <td> https://share.weiyun.com/5iOrZxD <td> 谷歌字表，字
<tr align="center"><td> 敬请期待 <td> ~ <td> 百度百科
<tr align="center"><td> 敬请期待 <td> ~ <td> 搜狗新闻
<tr align="center"><td> 敬请期待 <td> ~ <td> 金融新闻
<tr align="center"><td> 敬请期待 <td> ~ <td> 知乎问答
<tr align="center"><td> 敬请期待 <td> ~ <td> 微博
<tr align="center"><td> 敬请期待 <td> ~ <td> 文学作品
<tr align="center"><td> 敬请期待 <td> ~ <td> 四库全书
<tr align="center"><td> 敬请期待 <td> ~ <td> 综合
</table>

<br/>

## Documentation
### UER-py's framework
BERT-PyTorch主要分成Embedding层、Encoder层、Target层
项目细节：
```
bert-pytorch/
    |--bert/
    
    |   |--encoders/：编码器模块
    |   |--layers/：神经网络模块，包括Embedding，LayerNorm，Multi-Head等
    |   |--models/：模型构造模块
    |   |--targets/：目标任务模块
    |   |--utils/：工具模块
    |   |--model_builder.py：模型构造
    |   |--model_saver.py：模型保存
    |   |--trainer.py：训练迭代执行，分布式功能实现
    |
    |--corpora/：预训练语料存放文件夹
    |--datasets/：下游任务数据集存放文件夹
    |--models/：模型，词表，配置文件存放文件夹
    |--scripts/：实用脚本存放文件夹
    |
    |--preprocess.py: 预处理入口脚本
    |--pretrain.py: 预训练入口脚本
    |--classifier.py: 下游任务文本分类示例脚本
    |--cloze.py: 下游任务完型填空示例脚本
    |--tagger.py: 下游任务序列标注示例脚本
    |--feature_extractor.py: 抽取文本表示示例脚本
    |--README.md
```


下面对BERT-PyTorch的使用方式进行详细介绍

### 数据预处理
入口脚本是preprocess.py，使用CPU运行
```
usage: preprocess.py [-h] --corpus_path CORPUS_PATH --vocab_path VOCAB_PATH
                     [--dataset_path DATASET_PATH]
                     [--tokenizer {bert,char,word,space}]
                     [--dataset_split_num DATASET_SPLIT_NUM]
                     [--docs_buffer_size DOCS_BUFFER_SIZE]
                     [--seq_length SEQ_LENGTH] [--dup_factor DUP_FACTOR]
                     [--short_seq_prob SHORT_SEQ_PROB] [--seed SEED]
```
单CPU和单机单GPU训练模式预处理示例：
```
python3 preprocess.py --corpus_path corpora/book_review_bert.txt --vocab_path models/google_vocab.txt --dataset_path dataset
```
单机多GPU训练模式预处理示例，--dataset_split_num n表示将整个语料分成n份，并启动n个进程进行预处理。预处理后得到n份数据，在预训练阶段每个进程会加载1份数据。以单机8GPU为例，dataset_split_num设置为8，使用google词表：
```
python3 preprocess.py --corpus_path corpora/book_review_bert.txt --vocab_path models/google_vocab.txt --dataset_path dataset --dataset_split_num 8
```
多机多GPU训练模式预处理示例，如果是2机每机8GPU的情况，总共16个进程，则执行命令如下：
```
python3 preprocess.py --corpus_path corpora/book_review_bert.txt --vocab_path models/google_vocab.txt --dataset_path dataset --dataset_split_num 16
```
最后会生成 dataset-0.pt到dataset-15.pt，总共16个dataset文件，其中后缀“-0.pt”是由代码自动生成，到时候需要手动把 dataset-8.pt到dataset-15.pt文件复制到第二台参与预训练的机器

### 预训练
入口脚本是pretrain.py。预训练计算量非常大，推荐使用多GPU模式。预训练时参数初始化有两种方案：（1）随机初始化；（2）加载已有的预训练模型
```
usage: pretrain.py [-h] [--dataset_path DATASET_PATH] --vocab_path VOCAB_PATH
                   [--pretrained_model_path PRETRAINED_MODEL_PATH]
                   --output_model_path OUTPUT_MODEL_PATH
                   [--config_path CONFIG_PATH] [--total_steps TOTAL_STEPS]
                   [--save_checkpoint_steps SAVE_CHECKPOINT_STEPS]
                   [--report_steps REPORT_STEPS] [--batch_size BATCH_SIZE]
                   [--instances_buffer_size INSTANCES_BUFFER_SIZE]
                   [--emb_size EMB_SIZE] [--hidden_size HIDDEN_SIZE]
                   [--feedforward_size FEEDFORWARD_SIZE]
                   [--heads_num HEADS_NUM] [--layers_num LAYERS_NUM]
                   [--dropout DROPOUT] [--seed SEED]
                   [--learning_rate LEARNING_RATE] [--warmup WARMUP]
                   [--world_size WORLD_SIZE]
                   [--gpu_ranks GPU_RANKS [GPU_RANKS ...]]
                   [--master_ip MASTER_IP] [--backend {nccl,gloo}]
```
#### 随机初始化
单机CPU预训练示例：
```
python3 pretrain.py --dataset_path dataset --vocab_path models/google_vocab.txt --output_model_path models/model.bin
```
单机单GPU预训练示例，使用id为3的GPU：
```
python3 pretrain.py --dataset_path dataset --vocab_path models/google_vocab.txt --output_model_path models/model.bin --gpu_ranks 3
```
单机8GPU预训练示例，IP设置为localhost或者127.0.0.1，端口设置为可用端口即可（默认设置）：
```
python3 pretrain.py --dataset_path dataset --vocab_path models/google_vocab.txt \
                    --output_model_path models/model.bin --world_size 8 --gpu_ranks 0 1 2 3 4 5 6 7 
```
2机每机8GPU预训练示例，总共启动16个进程。依次在不同机器上启动脚本，唯一不同的参数为gpu_ranks。不同的rank值对应不同的进程，跑在不同的GPU上
**注意点：**(1)通常机器有几个GPU就启动几个进程，preprocess.py阶段--dataset_split_num通常设置为启动的进程个数；（2）--master_ip参数设置为gpu_ranks包含0的机器的 IP:Port，rank值为0的进程是主节点；（3）--dataset_path 中的dataset不要加后缀“-n.pt”。启动示例：
```
Node-0 : python3 pretrain.py --dataset_path dataset --vocab_path models/google_vocab.txt \
            --output_model_path models/model.bin --world_size 16 --gpu_ranks 0 1 2 3 4 5 6 7 --master_ip tcp://node-0-addr:port
Node-1 : python3 pretrain.py --dataset_path dataset --vocab_path models/google_vocab.txt \
            --output_model_path models/model.bin --world_size 16 --gpu_ranks 8 9 10 11 12 13 14 15 --master_ip tcp://node-0-addr:port            
```

#### 加载已有的预训练模型
这种模式能够利用已有的预训练模型，我们推荐使用这种模式，通过参数 --pretrained_model_path 控制加载已有的预训练模型。单机单GPU、单机CPU预训练示例:
```
python3 pretrain.py --dataset_path dataset --vocab_path models/google_vocab.txt --pretrained_model_path models/google_model.bin \
                    --output_model_path models/model.bin
python3 pretrain.py --dataset_path dataset --vocab_path models/google_vocab.txt --pretrained_model_path models/google_model.bin \
                    --output_model_path models/model.bin --gpu_ranks 3
```
单机8GPU预训练示例：
```
python3 pretrain.py --dataset_path dataset --vocab_path models/google_vocab.txt --pretrained_model_path models/google_model.bin \
                    --output_model_path models/model.bin --world_size 8 --gpu_ranks 0 1 2 3 4 5 6 7 
```
2机每机8GPU预训练示例：
```
Node-0 : python3 pretrain.py --dataset_path dataset --vocab_path models/google_vocab.txt --output_model_path models/model.bin \
            --pretrained_model_path models/google_model.bin --world_size 16 --gpu_ranks 0 1 2 3 4 5 6 7 --master_ip tcp://node-0-addr:port
Node-1 : python3 pretrain.py --dataset_path dataset --vocab_path models/google_vocab.txt --output_model_path models/model.bin \
            --pretrained_model_path models/google_model.bin --world_size 16 --gpu_ranks 8 9 10 11 12 13 14 15 --master_ip tcp://node-0-addr:port
```

### 下游任务微调
BERT-PyTorch目前包含4类下游任务：分类（classification）、序列标注（sequence labeling）、完型填空（cloze test）、特征抽取（feature extractor）
#### 分类
classifier.py选择BERT-PyTorch的Encoder最后一层输出的第一个隐层状态，接前向神经网络+激活函数+前向神经网络分类器。后面的前向神经网络参数随机初始化
```
usage: classifier.py [-h] [--pretrained_model_path PRETRAINED_MODEL_PATH]
                     [--output_model_path OUTPUT_MODEL_PATH]
                     [--vocab_path VOCAB_PATH] --train_path TRAIN_PATH
                     --dev_path DEV_PATH --test_path TEST_PATH
                     [--config_path CONFIG_PATH] [--batch_size BATCH_SIZE]
                     [--seq_length SEQ_LENGTH]
                     [--tokenizer {bert,char,word,space}]
                     [--learning_rate LEARNING_RATE] [--warmup WARMUP]
                     [--dropout DROPOUT] [--epochs_num EPOCHS_NUM]
                     [--report_steps REPORT_STEPS] [--seed SEED]
```
使用示例：
```
python3 classifier.py --pretrained_model_path models/google_model.bin --vocab_path models/google_vocab.txt \
                      --train_path datasets/book_review/train.txt --dev_path datasets/book_review/dev.txt \
                      --test_path datasets/book_review/test.txt --epochs_num 3 --batch_size 64

```

#### 序列标注
tagger.py选择BERT-PyTorch的Encoder最后一层输出的隐层状态，每个隐层状态经过前向神经网络
```
usage: tagger.py [-h] [--pretrained_model_path PRETRAINED_MODEL_PATH]
                 [--output_model_path OUTPUT_MODEL_PATH]
                 [--vocab_path VOCAB_PATH] [--train_path TRAIN_PATH]
                 [--dev_path DEV_PATH] [--test_path TEST_PATH]
                 [--config_path CONFIG_PATH] [--batch_size BATCH_SIZE]
                 [--seq_length SEQ_LENGTH] [--learning_rate LEARNING_RATE]
                 [--warmup WARMUP] [--dropout DROPOUT]
                 [--epochs_num EPOCHS_NUM] [--report_steps REPORT_STEPS]
                 [--seed SEED]
```

使用示例：
```
python3 tagger.py --pretrained_model_path models/google_model.bin --vocab_path models/google_vocab.txt \
                  --train_path datasets/msra/train.txt --dev_path datasets/msra/dev.txt --test_path datasets/msra/test.txt \
                  --epochs_num 5 --batch_size 32
```

#### 完型填空
cloze.py基于BERT-PyTorch中的MLM任务，对遮住的词进行预测，返回前topn最有可能的词
```
usage: cloze.py [-h] [--model_path MODEL_PATH] [--vocab_path VOCAB_PATH]
                [--input_path INPUT_PATH] [--output_path OUTPUT_PATH]
                [--config_path CONFIG_PATH] [--batch_size BATCH_SIZE]
                [--seq_length SEQ_LENGTH] [--tokenizer {bert,char,word,space}]
                [--topn TOPN]
```
使用示例：
```
python3 cloze.py --pretrained_model_path models/google_model.bin --vocab_path models/google_vocab.txt \
                 --input_path ./datasets/cloze_input.txt --output_path output.txt

```

#### 特征抽取
输入文本文件，每行一个句子，输出每个句子的向量表示
```
usage: feature_extractor.py [-h] --input_path INPUT_PATH --model_path
                            MODEL_PATH --vocab_path VOCAB_PATH --output_path
                            OUTPUT_PATH [--seq_length SEQ_LENGTH]
                            [--batch_size BATCH_SIZE]
                            [--config_path CONFIG_PATH]
                            [--tokenizer {bert,char,word,space}]
```
使用示例：
```
python3 feature_extractor.py --input_path datasets/cloze_input.txt --pretrained_model_path models/google_model.bin \
                             --vocab_path models/google_vocab.txt --output_path output.npy
```

<br/>
## 实用脚本说明
<table>
<tr align="center"><th> 脚本名 <th> 功能描述
<tr align="center"><td> average_model.py <td> 对多个模型的参数取平均，深度学习中常用的ensemble策略
<tr align="center"><td> build_vocab.py <td> 根据给定的数据集构造词表
<tr align="center"><td> check_model.py <td> 查看模型是多GPU版本，还是单GPU版本，测试加载单GPU版本模型是否成功
<tr align="center"><td> diff_vocab.py <td> 比较两个词表的重合度 
<tr align="center"><td> dynamic_vocab_adapter.py <td> 动态调整模型词表
<tr align="center"><td> multi_single_convert.py <td> 模型的多GPU和单GPU版本转换
<tr align="center"><td> dedup.py+simhash.py <td> 基于Simhash的数据集去重脚本
<tr align="center"><td> word_polysemy.py <td> 多义词相似度计算脚本
<tr align="center"><td> longtail_word_2_char.py <td> 根据排序的词表，把长尾词切成字
</table>


<br/>
## 实验
### 速度评测
速度评测运行环境：docker容器、CUDA Version 9.0.176、CUDNN 7.0.5，容器评测速度仅供参考
```
GPU信息：
型号：Tesla P40

CPU信息：
CPU(s):                88
Thread(s) per core:    2
Core(s) per socket:    22
CPU 系列：          6
型号：              79
型号名称：        Intel(R) Xeon(R) CPU E5-2699 v4 @ 2.20GHz
CPU MHz：             2201.000
```

<table>
<tr align="center"><th> 机器数 <th> GPU数/机 <th> 速度：tokens/秒
<tr align="center"><td> 1 <td> 0 <td> 276
<tr align="center"><td> 1 <td> 1 <td> 7050
<tr align="center"><td> 1 <td> 2 <td> 13071
<tr align="center"><td> 1 <td> 4 <td> 24695
<tr align="center"><td> 1 <td> 8 <td> 44300
<tr align="center"><td> 3 <td> 8 <td> 84386
</table>

### 实验评测
这里使用多个公开的中文数据集去评估BERT-PyTorch的效果。这些数据集被包括在本项目之中。用户可以轻松的还原实验结果
让模型在下游任务数据集语料上进行无监督训练对结果有显著的提升。这种先在数据集上进行无监督训练，然后再根据标签进行有监督训练的过程，也被叫作semi-supervised微调策略
后续我们会加上更多的改进策略
<table>
<tr align="center"><th> 模型/数据集              <th> 豆瓣书评 <th> ChnSentiCorp <th> Shopping <th> MSRA-NER
<tr align="center"><td> BERT                    <td> 87.5    <td> 94.3         <td> 96.3     <td> 93.0/92.4/92.7
<tr align="center"><td> BERT+semi-supervision   <td> 88.1    <td> 95.6         <td> 97.0     <td> 94.3/92.6/93.4
</table>


<br/>
## 联系我们
陈辉 chenhuichen@tencent.com
赵哲 nlpzhezhao@tencent.com; helloworld@ruc.edu.cn
张晋斌 westonzhang@tencent.com

<br/>
## 致谢
感谢犀牛鸟计划对本项目的支持
赵哲和陈希是通过犀牛鸟计划进入腾讯的实习生。他们的企业导师是邓浩棠，鞠奇
赵哲的学校导师是刘桃，杜小勇；陈希的学校导师是邓志鸿
实现BERT-PyTorch过程中参考或引用了业界一些公开的代码，这里一并列出表示感谢
1. https://github.com/google-research/bert
2. https://github.com/huggingface/pytorch-pretrained-BERT
3. https://github.com/pytorch/examples/tree/master/imagenet

此外腾讯内部还有其它的高质量的BERT复现工作，大家可以一并参考
1. https://git.code.oa.com/jcykcai/BERT

<br/>
## 参考文献
1. Devlin J, Chang M W, Lee K, et al. Bert: Pre-training of deep bidirectional transformers for language understanding[J]. arXiv preprint arXiv:1810.04805, 2018.
