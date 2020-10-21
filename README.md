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


 
