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
(1) 先看看数据。没有表头，先给各特征名补上，并且将没实际用处的日志列 dt 删除掉。    
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
```
观察:    
![](https://ftp.bmp.ovh/imgs/2020/11/70bf6b275a842503.png)
![](https://ftp.bmp.ovh/imgs/2020/11/0eda4afca8080db9.png)
![](https://ftp.bmp.ovh/imgs/2020/11/3e8cc28030b3df0e.png)   
a1: 可以看到数据共有 2,645,958 行 3 列，数据量达到百万级别。总的来看，数据量还是可观的，后面再看各实验组的样本量够不够支撑实验结论。  
a2: 各列都没有缺失值  
a3: 存在重复行 12,983，AB测试中不能有重复行。删除掉重复的行，保留一行即可。  
