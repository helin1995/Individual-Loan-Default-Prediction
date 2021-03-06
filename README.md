参考代码：[https://github.com/jiguang123/Credit-Loans-of-Data-Analysis](https://github.com/jiguang123/Credit-Loans-of-Data-Analysis)

# 实验记录

项目分为探索性数据分析（EDA）、数据预处理和模型
***

# 一、探索性数据分析（EDA）
1. 样本不平衡：经过统计，提供给我们的数据中，is_default=0的样本占83.17%，is_default=1
的样本占16.83%。所以，正负样本不均衡，当然这也是符合常理的，因为实际上违约的贷款人并不会很多。
那么，对于样本不平衡，通常就是过采样和欠采样来平衡正负样本的比例。当前的话，我是采用过采样的
方法：SMOTE，增加正样本。
2. 共线性分析：通过皮尔森相关性图谱找出冗余特征并将其剔除。
***

# 二、数据预处理

1. 时间格式转换：将原始数据中的时间属性，包括：issue_date、earlies_credit_mon，转换
成pandas时间类型%Y-%m-%d。然后对时间提取了多尺度信息：将时间中的年份、月份提取出来作为
单独的属性，然后设置了初始时间base_time，用于提取时间差。另外，对于earlies_credit_mon
这个属性目前是去掉了，暂时没有使用。

2. Object类型数据处理：这类数据在给的数据中有：class、employer_type、industry、
work_year，也包括1中所说的时间类型数据。在这里就介绍一下class、employer_type、
industry、 work_year这四个属性。对于这几个属性，一般都是用字符串表示。所以，我们就要对
这些属性进行编码。我们知道编码方式常用的包括：序号编码、one-hot 编码和二进制编码。序号编码
一般是用于那些属性值之间具有大小关系的数据，比如等级：A，B，C，D这类，属性值之间是具有大小 
关系的。所以可以使用序号编码，序号编码也保持了属性值之间的大小关系。那么，对于那些属性之间
没有大小关系的数据，可以尝试使用one-hot编码。所以，对于class、work_year我们使用编码，
employer_type、industry使用one-hot编码。
3. 缺失值处理：对于数值型数据缺失，在这次比赛中目前使用mean值补全。


# 三、特征选择
特征的选择优先选取与预测目标相关性较高的特征，不相关特征可能会降低分类的准确率，因此为了
增强模型的泛化能力，我们需要从原有特征集合中挑选出最佳的部分特征，并且降低学习的难度，能
够简化分类器的计算，同时帮助了解分类问题的因果关系。

一般来说，根据特征选择的思路将特征选择分为3种方法：嵌入方法（embedded approach）、过滤方法（filter approach）、包装方法（wrapper approacch）。

* 过滤式方法（filter approach）: 通过自变量之间或自变量与目标变量之间的关联关系选择特征。
* 嵌入式方法（embedded approach）: 通过学习器自身自动选择特征。
* 包裹式方法（wrapper approacch）: 通过目标函数（AUC/MSE）来决定是否加入一个变量。


本次项目采用Filter、Embedded和Wrapper三种方法组合进行特征选择。


# 四、模型
目前只采用了Logistic Regression


# 2021.10.14
实验1：构建baseline，模型采用Logistic Regression，对数据的处理包括一、二、三中描述的所有
处理。将原始数据分成7：3，70%数据用于训练，30%数据用于测试。

| 编号 | 预处理手段 | 线下表现（AUC） | 线上分数 | 提交时间 | 数据版本 |
| :---: | :----: | :-------: | :----: | :----: | :----: |
|   1  | 一、二、三 | 0.819098 | 0.805626 | 2021.10.14 | v1.0 |

# 2021.10.15
实验1：比较了未经过过采样的数据与经过过采样的数据的效果，如下：采用5-fold交叉验证，取5次的平均值

| 数据 | 预处理手段 | 线下表现（AUC） | CV |
| :---: | :----: | :-------: | :----: |
|   原始数据  | 一、二、三 | 0.8561 | 5 |
|   上采样  | 一、二、三 | 0.8720 | 5 |

经过上采样后，模型的平均性能有所提升，所以，上采样对这个比赛是有效的。

实验2：使用GridSearch调参，然后在训练集上训练，线上测试。

| 数据 | 预处理手段 | 线上分数 |
| :---: | :----: | :-------: |
|   上采样  | 一、二、三 | 0.80705194 |

相比较未经过调参的模型，经过GridSearch调参之后，模型性能略有提升。但是模型线上和线下分数相差较大，过拟合了。


# 2021.10.20

实验1：采用集成模型：随机深林

| 编号 | 预处理手段 | 线下表现（AUC） | 线上分数 | 提交时间 | 数据版本 |
| :---: | :----: | :-------: | :----: | :----: | :----: |
|   1  | 一、二、三 | 0.975578 | 0.782730 | 2021.10.20 | v1.0 |

模型过拟合了，尝试不采用过采样进行试验

实验1：采用集成模型：随机深林、不采用过采样

| 编号 | 预处理手段 | 线下表现（AUC） | 线上分数 | 提交时间 | 数据版本 |
| :---: | :----: | :-------: | :----: | :----: | :----: |
|   1  | 一、二、三 | 0.866925 | 0.618062 | 2021.10.20 | v1.0 |

由于正样本数据量非常少，样本不平衡，采用了过采样之后模型肯定会过拟合。但是线上效果要比不做过采样要好，所以，
采用过拟合是可以提升模型性能的。