# 电商用户行为分析
## 一. 项目背景  
### 项目概览  
UserBehavior是阿里巴巴提供的一个淘宝用户行为数据集，用于隐式反馈推荐问题的研究。该数据集包含了2017年11月25日至2017年12月3日之间，有行为的约一百万随机用户的所有行为（行为包括点击、购买、加购、喜欢）  

**注：隐式反馈推荐问题**（推荐系统中用户对物品的反馈分为显式和隐式反馈，显式反馈 (如评分、评级) 或单一的隐式反馈 (如浏览、点击、加入购物车)。 隐式反馈推荐是推荐系统通过对内容和用户行为的分析,建立适当的模型,帮助用户从海量的数据中找到自己感兴趣的内容。推荐系统中用户的行为反馈包括显式反馈和隐式反馈,隐式反馈信息在推荐系统算法中被广泛应用。隐式反馈体现着用户的兴趣爱好,对隐式反馈信息的挖掘有助于提高推荐系统的效果,以更好地设计推荐系统）。

### 数据来源与理解  
数据来源：[阿里巴巴天池](https://tianchi.aliyun.com/dataset/dataDetail?dataId=649&userId=1)  
数据集介绍：  
 
  文件名称  | 说明  | 包含特征  
  ---- | ----- | ------  
  UserBehavior.csv  | 包含所有的用户行为数据 | 用户ID，商品ID，商品类目ID，行为类型，时间戳  
 
字段说明：  
 
  列名称  | 说明  
  ---- | -----  
  用户ID  | 整数类型，序列化后的用户ID  
  商品ID  | 整数类型，序列化后的商品ID  
  商品类目ID  | 整数类型，序列化后的商品所属类目ID  
  行为类型  | 字符串，枚举类型，包括('pv'--商品详情页pv，等价于点击；'buy'--商品购买；'cart'--将商品加入购物车；'fav'--收藏商品)
  时间戳  | 行为发生的时间戳  
 
## 二. 分析目的  
通过对2017年11月25日至2017年12月3日之间的用户行为数据分析，为客户提供更精准的隐式反馈推荐，提高用户忠诚度，提高商家的成交转化率。  

## 三. 分析思路  
![分析思路](./image/框架.jpg)  

## 四. 数据清洗  
### 1. 选择子集  
源数据超过1亿，使用python截取100万数据进行分析  
```
import pandas as pd
DATA_PATH = 'C:/Users/项目/UserBehavior/UserBehavior.csv'
df = pd.read_csv(DATA_PATH, nrows=1000000)
target_name = 'C:/Users/项目/UserBehavior/UserBehavior.csv'
df.to_csv(target_name)
```  
### 2. 列名重命名  
源数据无列名，对CSV文件进行列名重命名  
![列名重命名](./image/列名重命名.jpg)  

### 3. 重复值处理  
```
SELECT user_id, item_id,timestamp FROM ubhsmall
GROUP BY user_id, item_id, timestamp
HAVING COUNT(user_id) > 1;-- 查询无记录
```
![重复值处理](./image/重复值处理.jpg)  
```
CREATE TABLE ubhsc
SELECT * FROM ubhsmall
GROUP BY user_id, item_id, cate_id, behavior_type, timestamp;-- 备份表
```

### 4. 缺失值处理  
```
SELECT COUNT(user_id), COUNT(item_id), 
COUNT(cate_id), COUNT(behavior_type), COUNT(timestamp)
FROM ubhsc;
```
查询无缺失值  
![缺失值查询](./image/缺失值查询.jpg)  

### 5. 一致化处理  
```
ALTER TABLE ubhsc ADD ID int unsigned primary key auto_increment;-- 新增字段ID作为主键
ALTER TABLE ubhsc ADD(longdate VARCHAR(255), date VARCHAR(255), time VARCHAR(255));-- 新增字段longdate, date, time用于存放时间

UPDATE ubhsc
SET longdate=FROM_UNIXTIME(timestamp,'%Y-%m-%d %k:%i:%s'),
date=FROM_UNIXTIME(timestamp,'%Y-%m-%d'),
time=FROM_UNIXTIME(timestamp,'%k:%i:%s')
WHERE ID BETWEEN 1 and 1000000;-- 将timestamp字段进行格式化，存放于longdate, date, time
```
```
ALTER TABLE ubhsc ADD hour INT(30);
UPDATE ubhsc SET hour = HOUR(time);-- 新增字段hour, 用于存储小时
```
![一致化处理](./image/一致化处理.jpg)  

### 6. 异常值处理  
```
SELECT COUNT(longdate)
FROM ubhsc
WHERE longdate<'2017-11-25 00:00:00' or longdate >'2017-12-03 24:00:00';
```
![异常值处理1](./image/异常值处理1.jpg)  
```
DELETE FROM ubhsmallcopy
WHERE longdate<'2017-11-25 00:00:00' or longdate >'2017-12-03 24:00:00';
```
![异常值处理2](./image/异常值处理2.jpg)  

### 7. 查看数据情况  
```
SELECT 
COUNT(distinct user_id) AS 用户数,
COUNT(distinct item_id) AS 商品数量,
COUNT(distinct cate_id) AS 商品类型数量,
SUM(case when behavior_type = 'pv' then 1 else 0 end) AS 浏览次数,
SUM(case when behavior_type = 'fav' then 1 else 0 end) AS 收藏次数,
SUM(case when behavior_type = 'cart' then 1 else 0 end) AS 加入购物车次数,
SUM(case when behavior_type = 'buy' then 1 else 0 end) AS 购买次数,
COUNT(longdate) AS 数据总量
from ubhsc;
```
![数据情况](./image/数据情况.jpg)  

## 五. 数据分析  
### （一）用户角度  
#### 1. 总体运营分析  
##### 1.1 用户流量分析
###### ①总访问量PV、总访客数UV、平均访问量PV/UV  
```
SELECT COUNT(DISTINCT user_id) AS '总访客数UV',
sum(case when behavior_type='pv' then 1 else 0 END) as '总访问量PV',
sum(case when behavior_type='pv' then 1 else 0 END)/COUNT(DISTINCT user_id) as '人均访问次数'
FROM ubhsc;
```
2017年11月25日至2017年12月3日，PV为9739，UV为876539，人均访问次数为90次  
![pvuv](./image/pvuv.jpg)  

###### ②PV、UV、平均访问量与日期/时间的关系  
（备注：可使用处理过后的源数据在tableau中直接进行可视化，以下SQL可提取对应数据）  
```
#以日期为维度
SELECT Date,
COUNT(DISTINCT user_ID) AS '总访客数UV',
sum(case when behavior_type='pv' then 1 else 0 END) as '总访问量PV',
sum(case when behavior_type='pv' then 1 else 0 END)/COUNT(DISTINCT user_id) as '人均访问次数'
FROM ubhsc
GROUP BY Date
ORDER BY Date;

#以（天）小时为维度（备注，以小时为维度，可计算日均UV、PV）
SELECT `hour`,
COUNT(DISTINCT User_ID) AS '总访客数UV',
sum(case when behavior_type='pv' then 1 else 0 END) as '总访问量PV',
sum(case when behavior_type='pv' then 1 else 0 END)/COUNT(DISTINCT user_id) as '人均访问次数'
FROM ubhsc
GROUP BY `hour`
ORDER BY `hour`;
```
使用tableau连接mysql数据库，进行可视化，结果显示在2017/11/25-2017/12/3期间，PV与UV随日期的变化趋势相似，11/25-12/1保持稳定的水平，12/2开始较为明显的增长，增长率约为33%，而人均访问次数则相对平稳，自12/1有缓慢下降趋势。  
![pvuv日期关系](./image/pvuv日期关系.jpg)  
以一天为维度，10点-18点之间，UV、PV均无较大波动（UV在6000上下波动，PV在45000上下波动），18点-23点，UV、PV出现明显增长，PV波动明显（用户频繁访问）  
![pvuv时间关系](./image/pvuv时间关系.jpg)  

##### 1.2 总体销售情况  
由于数据源中未提供金额字段，此处仅分析成交量变化  
