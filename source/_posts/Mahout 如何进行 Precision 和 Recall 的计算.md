---
title: Mahout 如何进行 Precision 和 Recall 的计算
date: 2016-08-18 17:30:00
tags: Mahout
categories: 推荐系统

---


> 当为某一个用户做推荐评估时，选择一个临界值，以该临界值为参照为该用户构造一个目标最大击中集（即相关项RelevantItemsIDs），然后将该用户数据中的包含在最大击中集中的物品去除，这样形成一个新的训练集。这个新的训练集中，只是去除了部分数据。然后，使用该训练集为该用户进行推荐。如果推荐的物品包含在最大击中集中，则说明击中。依照这个办法为该用户计算 Precision 和 Recall 值。依照这个办法为所有的用户计算 Precision 和 Recall 值。然后，Precision 和 Recall 的平均值作为模型的 Precision 和 Recall 值。

RecommenderIRStatsEvaluator是一个接口，用于得到推荐系统的准确率，召回率等统计指标。 
它定义的函数如下
```
public IRStatistics evaluate(RecommenderBuilder recommenderBuilder, DataModelBuilder dataModelBuilder, DataModel dataModel, IDRescorer rescorer, int at, double relevanceThreshold, doubleevaluationPercentage) 
/**
   * @param recommenderBuilder
   * 它通过public Recommender buildRecommender(DataModel model)定义推荐系统创建的方式；       

   * @param dataModelBuilder
   * 数据模型创建的方式，如果已经创建好了数据模型，一般这个参数可以为null

   * @param dataModel
   * 推荐系统使用的数据模型
   * 
   * @param rescorer
   * 推荐排序的方式，一般情况下可以为null
   * 
   * @param at
   * 推荐几个物品（TOPN),，它用来定义计算准确率的时候，一个user可以拥有的相关项items的最大个数，相关项item定义为user对这个item的评价超过了relevanceThreshold的项

   * @param relevanceThreshold
   * 相关临界值 ,和at一起使用定义一个user的相关项
```
### 计算临界值
```
//computeThreshold(prefs)这个方法主要是计算得到当前用户对物品 打分的平均值加上标准差的值
double theRelevanceThreshold = Double.isNaN(relevanceThreshold) ? computeThreshold(prefs)
					: relevanceThreshold;
```
### getRelevantItemsIDs
按照用户对物品的打分从大到小排序，将大于设定的的relevanceTheshold的值存入relevantItemIDs中，relevantItemIDs最大值为at
```
/**
 * getRelevantItemsIDs的实现，
 *1.首先得到userID的Preferences 
 *2.创建一个FastIDSet用来保存相关项的id，大小为at 
 *3.prefs根据值大小排序 
 *4.遍历pref，如果user对这个item的评价prefs.getValue(i)不小于相关阈值，则将这个item的加入相关项，最多取at个满足条件的item
 */
FastIDSet relevantItemIDs = dataSplitter.getRelevantItemsIDs(userID, at, theRelevanceThreshold, dataModel);
```
```
public FastIDSet getRelevantItemsIDs(long userID,
                                       int at,
                                       double relevanceThreshold,
                                       DataModel dataModel) throws TasteException {
    PreferenceArray prefs = dataModel.getPreferencesFromUser(userID);
	//定义最大理论击中物品集合 
    FastIDSet relevantItemIDs = new FastIDSet(at);
	//将用户的打分排序 
    prefs.sortByValueReversed();
    for (int i = 0; i < prefs.length() && relevantItemIDs.size() < at; i++) {
      if (prefs.getValue(i) >= relevanceThreshold) {
        relevantItemIDs.add(prefs.getItemID(i));
      }
    }
    return relevantItemIDs;
  }
```

### 获取训练集
这个训练集的构造规则是： 
1. 对于其它的用户，将他们所有的（item，preference）都加入训练集； 
2. 对于这个用户user，将它的除了相关项之外的其它项的喜好加入训练集；

然后，我们使用推荐算法进行推荐。推荐的时候，我们就可能给UserID 推荐移除的物品或者其他的物品。
```
FastByIDMap<PreferenceArray> trainingUsers = new FastByIDMap<PreferenceArray>(dataModel.getNumUsers());
			//对所有用户进行处理
			//processOtherUser :Adds a single user and all their preferences to the training model.
			LongPrimitiveIterator it2 = dataModel.getUserIDs();
			while (it2.hasNext()) {
				dataSplitter.processOtherUser(userID, relevantItemIDs, trainingUsers, it2.nextLong(), dataModel);
			}
```
### 构造训练模型
```
DataModel trainingModel = dataModelBuilder == null ? new GenericDataModel(trainingUsers): dataModelBuilder.buildDataModel(trainingUsers);
```
### 进行推荐
```
Recommender recommender = recommenderBuilder.buildRecommender(trainingModel);

int intersectionSize = 0;

List<RecommendedItem> recommendedItems = recommender.recommend(userID, at, rescorer);
```

### 计算推荐结果包含在相关项中的个数

```
List<RecommendedItem> recommendedItems = recommender.recommend(userID, at, rescorer);

			for (RecommendedItem recommendedItem : recommendedItems) {

				if (relevantItemIDs.contains(recommendedItem.getItemID())) {
					intersectionSize++;
				}
			}
```

### 计算查全率、查准率
```
int numRecommendedItems = recommendedItems.size();

			// Precision
			if (numRecommendedItems > 0) {
				precision.addDatum((double) intersectionSize / (double) numRecommendedItems);
			}

			// Recall
			recall.addDatum((double) intersectionSize / (double) numRelevantItems);
```

参考
> http://pan.baidu.com/s/1pKE97wJ
> http://www.myexception.cn/cloud/1983215.html