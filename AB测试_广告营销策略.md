### 一. 背景：  
最近在看AB测试的知识，没有机会实操这一块东西。刚好在天池找到可用的数据集：Audience Expansion Dataset。该数据集包含 3 张表，分别记录了支付宝两组营销策略的活动情况：
- (1) emb_tb_2.csv: 用户特征数据集
- (2) effect_tb.csv: 广告点击情况数据集
- (3) seed_cand_tb.csv: 用户类型数据集  
>
其中 (2) 表记录了商业定向广告的点击情况，包含列：   
  - dt：日志时间，1：第一天，2：第二天
  - dmp_id：营销策略编号 1：对照组，2：营销策略一，3：营销策略二
  - user_id：支付宝用户ID，唯一标识
  - label：用户当天是否点击活动广告 (0：未点击，1：点击)  
该篇用 (2) 表数据来研究哪种广告策略更好，以广告点击率作为评判广告效果好坏的指标。  

### 二. 数据下载地址：  
(google下不了，可能是我网络问题，最后换火狐下载到了)   
https://tianchi.aliyun.com/dataset/dataDetail?dataId=50893&lang=zh-cn  

### 三.数据清洗    
现在还不知道数据情况怎么样，初步想法是拿两组广告策略的点击率分别与对照组点击率进行比较。设点击率：对照组：p1，广告组1：p2，广告组3：p3，拿p1与p2对比、p1与p3对比。对比看哪组广告效果比较显著，又或者两组广告均没带来显著的效果。  
先看看数据。没有表头，先给各特征名补上，并且将没实际用处的日志列 dt 删除掉。    
```
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
plt.rcParams['font.sans-serif'] = ['SimHei']

# 读取广告点击情况数据集  
adv_df = pd.read_csv(r'E:\dataset\audience_expansion\effect_tb.csv', header=None, names=['dt','user_id','label','dmp_id'])
adv_df.drop(['dt'], axis=1, inplace=True)
print(adv_df.shape)
print(adv_df.sample(8))
print(adv_df.describe())
print(adv_df.info())
print(adv_df.isnull().sum())
print(adv_df.duplicated().sum())
adv_df.drop_duplicates(inplace=True)
print(adv_df.shape[0])
print(adv_df.pivot_table(index='dmp_id', columns='label', values='user_id', aggfunc='count', margins=True))            
```
观察:    
![](https://ftp.bmp.ovh/imgs/2020/11/70bf6b275a842503.png)
![](https://ftp.bmp.ovh/imgs/2020/11/0eda4afca8080db9.png)
![](https://ftp.bmp.ovh/imgs/2020/11/3e8cc28030b3df0e.png)
![](https://ftp.bmp.ovh/imgs/2020/11/a421f77a5193f197.png)   
a1: 可以看到源数据共有 2,645,958 行 3 列，数据量达到百万级别。总的来看，数据量还是可观的，后面再看各实验组的样本量够不够支撑实验结论       
a2: 各列都没有缺失值    
a3: 存在重复行 12,983，AB测试中不能有重复行。删除掉重复的行，保留一行即可     
a4: 没有异常值    
a5: 经过基本的清洗后，剩下 2,632,975 行记录     

### 四.假设检验
经过上述基础数据处理，下面可以开始做假设检验。   
先看看对照组的广告点击率：  
```
print(adv_df[adv_df['dmp_id']==1]['label'].mean())  # 对照组点击率
```
output: 0.012551012429794775
输出结果中，对照组广告点击率为1.26%。假设我们的目标是提升1%，看其他两组广告是否带来显著的好的影响。（提升什么指标、提升多少，看业务、产品等意见联合制定）  
在确定目标后，首先检查样本量是否达标。我们通过网站https://www.evanmiller.org/ab-testing/sample-size.html 来确定我们需要的最小样本量。
![](https://s3.ax1x.com/2020/11/25/DdsJYQ.png)  
由网站测算得，2167是最小的样本要求量。  
```
print(adv_df['dmp_id'].value_counts())
```
![](https://s3.ax1x.com/2020/11/25/Dd6QIS.png)  
两个实验组的样本量都是达标的。继续看两组的点击率。  
```
print(adv_df[adv_df['dmp_id']==2]['label'].mean())  # 广告组 1 点击率
print(adv_df[adv_df['dmp_id']==3]['label'].mean())  # 广告组 2 点击率
```
![](https://s3.ax1x.com/2020/11/25/Dd6zWQ.png)  
可以看到广告组1的点击率为1.53%，广告组2的点击率为2.62%。与对照组的1.26%相比，广告组1提升不足1%，广告组2提升1.36%。剔除广告1组，下面检验广告组2是否显著提高广告点击率。  

#### (4.1) 假设
原假设H0：p1 >= p3  
备择假设H1：p1 < p3(我想证明广告组2能提升一定的点击率，想收集证据去证明这一点，所以我在选备择假设的时候选了这个)  


