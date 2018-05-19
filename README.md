# 豆瓣电影推荐系统

## 数据：

### MySQL数据表：

#### Movie 表
| Field     | Type         | Null | Key  | Default | Extra          |
| --------- | ------------ | ---- | ---- | ------- | -------------- |
| number    | int(11)      | NO   | PRI  | NULL    | auto_increment |
| rate      | varchar(11)  | NO   |      | NULL    |                |
| title     | varchar(255) | NO   |      | NULL    |                |
| url       | varchar(255) | NO   |      | NULL    |                |
| id        | varchar(32)  | YES  |      | NULL    |                |
| directors | varchar(32)  | YES  |      | NULL    |                |
| year      | varchar(32)  | YES  |      | NULL    |                |
| actors    | varchar(255) | YES  |      | NULL    |                |
| type      | varchar(64)  | YES  |      | NULL    |                |
| countries | varchar(32)  | YES  |      | NULL    |                |
| summary   | text         | YES  |      | NULL    |                |
#### User 表

| Field        | Type        | Null | Key  | Default | Extra          |
| ------------ | ----------- | ---- | ---- | ------- | -------------- |
| user_id      | int(11)     | NO   | PRI  | NULL    | auto_increment |
| name         | varchar(64) | YES  |      | NULL    |                |
| rates        | longtext    | YES  |      | NULL    |                |
| following_id | mediumtext  | YES  |      | NULL    |                |
| comments     | longtext    | YES  |      | NULL    |                |

### 使用方式

```python
import pandas as pd
from sqlalchemy import create_engine
engine = create_engine('mysql://root:qwert12345@localhost:3306/douban', convert_unicode=True, encoding='utf-8', connect_args={"charset":"utf8"})
df_movie = pd.read_sql('movie', engine)
df_user = pd.read_sql('user', engine)
```

PS.  使用Python3。建议在Pycharm上进行操作，在iterm2的terminal上有中文显示问题。

### 数据预处理：

#### 特征内容：

先不考虑文本数据挖掘和社交关系，所需特征如下：（2018.05.14）

| =user one hot= | =item one hot= | =user's history rating + item's average rating= | =year= | =type= |
- **user one hot:** user ID 的 one hot 形式，1000维
- **item one hot:** item ID 的 one hot 形式，1000维
- **user's history rating + item's average rating::** 该 user 的历史评分，分数从0-5星(0是没看过)。其中对于该item的评分用item的平均分代替,1000维
- **year:** 该电影放映的年份（如果能够弄到用户看片的年份就最好了）
- **type:** 该 item 所属类别，形式为one hot形式，需要先写个type2index的映射，???维


#### 输出格式：

每行为：$y_i$(空格)$j_0:x_{i,j_0}$(空格)$j_1:x_{i,j_1}$(空格)$j_2:x_{i,j_2}$...(空格)$j_n:x_{i,j_n}$

参考libFM的格式：http://www.libfm.org/libfm-1.42.manual.pdf

输出文件为douban.data(所有记录)、douban.train(训练集)、douban.test(测试集)、douban.validation(验证集)


### 模型初步实现：

#### ItemPop 模型

#### SVM 模型

#### Neural FM模型

##### Factorization Machine模型使用方式

- 模型原理：

- 模型使用：

  - 进入 neural_factorization_machine/文件夹
  - 输入`python FM.py` 即可，详细参数可见 `python FM.py -h`

- 参数调优：

  - 调优方式：

    - 进入 neural_factorization_machine/bash，新建 .sh 文件，命名规则为 **模型名\_参数名1_参数名2.sh**

    - 举例：新建 FM_lr_bs.sh，输入

      ```shell
      for lr in 0.005 0.01 0.05 0.1 0.5
      do
          for bs in 16 32 64 128 256 512
          do
              CUDA_VISIBLE_DEVICES="-1" python FM.py --lr $lr --batch_size $bs --log_path zhankui --log_on [\'lr\',\'batch_size\'];
          done
      done
      ```

      其中 log_path 是记录文件夹路径，log_on是记录文件夹的命名规则，for循环代表Grid-search范围。

##### Factorization Machine模型使用方式

- 模型原理：

- 模型使用：

  - 进入 neural_factorization_machine/文件夹
  - 输入`python NeuralFM.py` 即可，详细参数可见 `python NeuralFM.py -h`

- 参数调优：

  - 调优方式：

    - 进入 neural_factorization_machine/bash，新建 .sh 文件，命名规则为 **模型名\_参数名1_参数名2.sh**

    - 举例：新建 NeuralFM_lr_bs.sh，输入

      ```shell
      for lr in 0.005 0.01 0.05 0.1 0.5
      do
          for bs in 16 32 64 128 256 512
          do
              CUDA_VISIBLE_DEVICES="-1" python NeuralFM.py --lr $lr --batch_size $bs --log_path zhankui --log_on [\'lr\',\'batch_size\'];
          done
      done
      ```

      其中 log_path 是记录文件夹路径，log_on是记录文件夹的命名规则，for循环代表Grid-search范围。

##### 各自任务：

- 占魁：learning rate, batch size, hidden factor, layers 调优
- 小花：feature的组成、表征探索
- 冬皓：dropout, batch_norm, regularization 调优

PS. 启动程序脚本非常简单，主要是想多用几台服务器节约时间。

### 社交网络实现：

- 用户关注的graph可视化
- 数据：user表的user_id和following_id字段



### 任务安排：

#### 目标：

- **Baseline:** ItemPopularity、SVM、FM
- **Main Model:** NeuFM
- **Graph visualization**

#### 冬皓： 

完成了社交关系的可视化，接下来可以探索一下pyecharts的graph可视化。我们把1000人之外的用户叫做“其他用户”，最后建议把结果整理成这种格式：

- 不包括其他用户：
  - 网络可视化图
  - 统计信息（如最大、最小度数统计等）
- 包括其他用户的depth=1的广度优先搜索结果：
  - 网络可视化图
  - 统计信息（如最大、最小度数统计等）
- 完成自己部分的PPT

#### 小花：

完成了feature的提取，接下来完成代码的整理、负样本采集和数据集划分的算法。弄好之后，周末可以尝试SVM的搭建。任务如下：

- 负样本采集要求：
  - 用一个单独的文件，名为douban.neg
  - 每行输出格式同上
  - 输出顺序为逐用户从上自下排列（即先写user 0的所有负样本，再写user 1的所有负样本，依次往下）
- 数据集划分：
  - 生成一个所有正样本的文件，名为douban.pos
  - 测试集：douban.pos中每个user抽取一个正样本，和douban.neg中所有的负样本合并，成为一个测试集，名为douban.test
  - 验证集：douban.pos中每个user抽取一个正样本，和douban.neg中所有的负样本合并，成为一个测试集，名为douban.validation，正样本和douban.test不得有重复
  - 训练集：除douban.test和douban.validation之外的所有正样本，另外每个user从douban.neg中抽取4个负样本加入训练集，训练集名为douban.train
- 统计信息计算：
  - 正样本、负样本的条数
  - 用户与电影交互数量的直方图


- 完成自己部分的PPT

#### 占魁：

完成了数据导入和FM、NeuFM模型启动（在其他数据集上），接下来等待我们的数据集，以及添加HR和NDCG等ranking的metric，以及用ItemPopularity算法作为简单的Baseline。具体任务为：

- 实现ItemPopularity算法，计算HR、NDCG metric结果。
- 在FM、NeuFM、SVM上添加HR、NDCG的metric


- 跑出ItemPop、SVM、FM、NeuFM的初步实验结果，预期为：
  - RSME: NeuFM < FM < SVM
  - HR、NDCG: ItemPop < SVM < FM < NeuFM
- 完成自己部分的PPT



Update 2018.05.14



