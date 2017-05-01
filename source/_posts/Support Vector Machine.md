---
title: 支持向量机 Support Vector Machine
date: 2017/3/13 18:43:15  
tags: Machine learning
categories: Machine learning

---

支持向量机（SVM）是90年代中期发展起来的基于统计学习理论的一种机器学习方法，通过寻求结构化风险最小来提高学习机泛化能力，实现经验风险和置信范围的最小化，从而达到在统计样本量较少的情况下，亦能获得良好统计规律的目的。

通俗来讲，它是一种二类分类模型，其基本模型定义为特征空间上的间隔最大的线性分类器，即支持向量机的学习策略便是间隔最大化，最终可转化为一个凸二次规划问题的求解。

- 线性分割
- kernel trick 核技巧，多特征，将x,y映射到多维空间进行特征分割，之后再返回二维空间形成非线性分割线

![](http://ojpgmz933.bkt.clouddn.com/17-3-13/56429398-file_1489401775120_16d40.png)

## 使用 sklearn 实战

<!-- more -->

###调参

在 rbf 核下，分别使用10、100、1000、10000的C参数进行调整输出准确率。
使用1%的数据集。

**C参数**： 低C使决策表面平滑，而高C旨在通过给予模型自由选择更多样本作为支持向量来正确地分类所有训练样本。

```
features_train = features_train[:len(features_train)/100]
labels_train = labels_train[:len(labels_train)/100]
```
结果如下
```
C=10.0 accuracy=0.616040955631
C=100.0 accuracy=0.616040955631
C=1000. accuracy=0.821387940842
C=10000. accuracy=0.892491467577
```

### 结果计算

在全数据集下，进行准确度计算，并计算预测为Chris邮件的个数

```
#!/usr/bin/python

""" 
    This is the code to accompany the Lesson 2 (SVM) mini-project.

    Use a SVM to identify emails from the Enron corpus by their authors:    
    Sara has label 0
    Chris has label 1
"""
    
import sys
from time import time
sys.path.append("../tools/")
from email_preprocess import preprocess


### features_train and features_test are the features for the training
### and testing datasets, respectively
### labels_train and labels_test are the corresponding item labels
features_train, features_test, labels_train, labels_test = preprocess()

# reduce training data
#features_train = features_train[:len(features_train)/100]
#labels_train = labels_train[:len(labels_train)/100]


#########################################################
### your code goes here ###
from sklearn.svm import SVC
clf = SVC(kernel='rbf',C=10000.)


#### now your job is to fit the classifier
#### using the training features/labels, and to
#### make a set of predictions on the test data
t0 = time()
clf.fit(features_train,labels_train)
print "training time : " ,round(time()-t0,3),"s"

#### store your predictions in a list named pred
t1 = time()
pred = clf.predict(features_test)
print "predicting time : " ,round(time()-t1,3),"s"

from sklearn.metrics import accuracy_score
acc = accuracy_score(pred, labels_test)
print "accuracy: ", acc

# answer1=pred[10]
# answer2=pred[26]
# answer3=pred[50]

num = 0
for n in pred:
    if n == 1:
        num = num + 1

print "total Chris(1): ", num
#########################################################



```

结果如下
```
no. of Chris training emails: 7936
no. of Sara training emails: 7884
training time :  159.096 s
predicting time :  19.393 s
accuracy:  0.990898748578
total Chris(1):  877
```