# 实体识别

**公众号文章：[命名实体识别常用算法及工程实现](https://mp.weixin.qq.com/s/KNNw9JUZxXljE87vVgW5Yg)**  
**公众号文章：[命名实体识别开源项目V4.0版本](https://mp.weixin.qq.com/s/oWHFdcBdVsifvuEyr_ruPQ)**  

此仓库是基于Tensorflow2.3的NER任务项目，支持BiLSTM-Crf、Bert-BiLSTM-Crf、Bert-Crf，可对Bert进行微调，提供可配置文档，配置完可直接运行。

## 更新历史
日期|版本|描述
:---|:---|---
2020-01-12|v1.0.0|初始仓库
2020-04-08|v1.1.0|重构项目代码，添加必要的注释
2020-04-13|v1.2.0|分别打印出每一个实体类别的指标
2020-09-09|v2.0.0|更新到tensorflow2.3版本
2020-09-10|v2.1.0|取消批量测试方法，简化预测的逻辑
2020-09-13|v3.0.0|增加Bert做embedding，通过配置支持BiLSTM-Crf和Bert-BiLSTM-Crf两种模型的训练与预测
2021-04-21|v3.0.1|添加中断之后再训练逻辑、通过配置可选GPU和CPU、bug-fix
2021-04-25|v3.1.0|使用tf.data.Dataset包装数据，合并数据处理类
2021-04-25|v3.1.1|bug-fix:读取token出现KeyError
2021-06-29|v4.0.0|重构项目代码，增加对Bert-CRF的支持以及其和Bert-Bilstm-CRF中对Bert的微调的支持

## 环境
* python 3.6.7
* **CPU:** tensorflow==2.3.0
* **GPU:** tensorflow-gpu==2.3.0
* tensorflow-addons==0.11.2
* transformers==3.0.2
* jieba==0.41
* tqdm==4.48.2

推荐使用GPU加速训练，其他环境见requirements.txt

## 数据集
人民日报语料

## 原理 
### Bilstm-CRF

![bilstm-crf-model](https://img-blog.csdnimg.cn/20210629194609507.png) 

### Finetune-Bert-CRF

![bert-crf-model](https://img-blog.csdnimg.cn/20210629194710746.png) 

### (Finetune)Bert-Bilstm-CRF

![bert-bilstm-crf-model](https://img-blog.csdnimg.cn/20210629194719983.png) 

 
### CRF层
[最通俗易懂的BiLSTM-CRF模型中的CRF层介绍](https://zhuanlan.zhihu.com/p/44042528)  
[CRF Layer on the Top of BiLSTM - 1](https://createmomo.github.io/2017/09/12/CRF_Layer_on_the_Top_of_BiLSTM_1/)  
CRF层需要使用viterbi译码法，知乎上[这个答案](https://www.zhihu.com/question/20136144)比较容易理解    

## 使用
### 训练
将已经标注好的数据切割好训练、验证集放入data目录下，如果只提供训练集将会有程序自己按照9:1切割训练集与验证集。  
在system.config的Datasets(Input/Output)下配置好数据集的路径、分隔符、模型保存地址等。  
在system.config的Labeling Scheme配置标注模式。  
在system.config的Model Configuration/Training Settings下配置模型参数和训练参数。  

设定system.config的Status中的为train:
```
################ Status ################
mode=train
# string: train/interactive_predict
```

是否使用Bert做embedding(选择True/False):
```
use_bert=False
```

当使用Bert的时候是否对Bert进行微调(选择True/False):
```
finetune=False
```

这个项目支持Finetune-Bert+Crf、Finetune-Bert+BiLstm+Crf、Bert+BiLstm+Crf、BiLstm+Crf四类模型的训练，配置组合如下：  

模型|use_bert|use_bilstm|finetune|
:---|:---|:---|---
BiLstm+Crf|False|True|False
Bert+BiLstm+Crf|True|True|False
Finetune-Bert+Crf|True|False|True
Finetune-Bert+BiLstm+Crf|True|True|True

  
运行main.py开始训练。  

* Bilstm-CRF模型下效果

![bilstm-crf-train](https://img-blog.csdnimg.cn/2020091319580672.png)  

* Finetune-Bert-CRF模型下效果

![bert-crf-train](https://img-blog.csdnimg.cn/20210701175300657.png)  

* Bert-Blism-CRF模型下效果

![bert-bilstm-crf-train](https://img-blog.csdnimg.cn/20200913200450351.png)  

***注(1):这里使用的[transformers](https://github.com/huggingface/transformers)包加载Bert，初次使用的时候会自动下载Bert的模型***  
***注(2):当重新训练的时候，Bert-Bilstm-CRF和Bilstm-CRF各自自动生成自己vocabs/label2id文件，不能混用，如果需要共用，你可以手动的定义标签***   
***注(3):使用Bert-Bilstm-CRF时候max_sequence_length不能超过512并且embedding_dim默认为768***

### 在线预测
仓库中已经训练好了Bilstm-CRF和Bert-Bilstm-CRF两个模型在同一份数据集上的参数，可直接进行试验，两者位于checkpoints/目录下  
* 使用Bilstm-CRF模型时使用bilstm-crf/里的system.config配置
* 使用Bert-Bilstm-CRF模型时使用bert-bilsm-crf/里的system.config配置   
将对应的配置替换掉当前的配置。  
最后，运行main.py开始在线预测。   
下图为在线预测结果，你可以移植到自己项目里面做成对外接口。    

![online_predict](https://img-blog.csdnimg.cn/202009131958050.png)  

## 参考
+ NER相关的论文整理在[papers](papers)下
+ [https://github.com/scofield7419/sequence-labeling-BiLSTM-CRF](https://github.com/scofield7419/sequence-labeling-BiLSTM-CRF)

## 公众号
相关问题欢迎在公众号反馈：  

![小贤算法屋](https://img-blog.csdnimg.cn/20210427094903895.jpg)

