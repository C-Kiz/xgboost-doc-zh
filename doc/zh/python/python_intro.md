Python 软件包介绍
===========================

本文档给出了有关 xgboost python 软件包的基本演练.

***其他有用的链接列表***
* [Python 演练代码集合](https://github.com/tqchen/xgboost/blob/master/demo/guide-python)
* [Python API 参考](python_api.rst)

安装 XGBoost
---------------
要安装 XGBoost, 请执行以下步骤:

* 您需要在项目的根目录下运行 `make` 命令
* 在 `python-package` 目录下运行

```shell
python setup.py install
```

```python
import xgboost as xgb
```

数据接口
--------------
XGBoost python 模块能够使用以下方式加载数据:
- libsvm txt format file（libsvm 文本格式的文件）
- Numpy 2D array, and（Numpy 2维数组, 以及）
- xgboost binary buffer file. （xgboost 二进制缓冲文件）

这些数据将会被存在一个名为 ```DMatrix``` 的对象中.

* 要加载 ligbsvm 文本格式或者 XGBoost 二进制文件到 ```DMatrix``` 对象中. 代码如下:
```python
dtrain = xgb.DMatrix('train.svm.txt')
dtest = xgb.DMatrix('test.svm.buffer')
```
* 要加载 numpy 的数组到  ```DMatrix``` 对象中, 代码如下:
```python
data = np.random.rand(5,10) # 5 entities, each contains 10 features
label = np.random.randint(2, size=5) # binary target
dtrain = xgb.DMatrix( data, label=label)
```
* 要加载 scpiy.sparse 数组到 ```DMatrix``` 对象中, 代码如下:
```python
csr = scipy.sparse.csr_matrix((dat, (row, col)))
dtrain = xgb.DMatrix(csr)
```
* 保存 ```DMatrix``` 到 XGBoost 二进制文件中后, 会在下次加载时更快:
```python
dtrain = xgb.DMatrix('train.svm.txt')
dtrain.save_binary("train.buffer")
```
* 要处理 ```DMatrix``` 中的缺失值, 您可以通过指定缺失值的参数来初始化 ```DMatrix```:
```python
dtrain = xgb.DMatrix(data, label=label, missing = -999.0)
```
* 在需要时可以设置权重:
```python
w = np.random.rand(5, 1)
dtrain = xgb.DMatrix(data, label=label, missing = -999.0, weight=w)
```

设置参数
------------------
XGBoost 使用 pair 格式的 list 来保存 [参数](../parameter.md). 例如:
* Booster（提升）参数

```python
param = {'bst:max_depth':2, 'bst:eta':1, 'silent':1, 'objective':'binary:logistic' }
param['nthread'] = 4
param['eval_metric'] = 'auc'
```

* 您也可以指定多个评估的指标:

```python
param['eval_metric'] = ['auc', 'ams@0'] 

# alternativly:
# plst = param.items()
# plst += [('eval_metric', 'ams@0')]
```

* 指定验证集以观察性能
```python
evallist  = [(dtest,'eval'), (dtrain,'train')]
```

训练
-------- 
有用参数列表和数据以后, 您现在可以训练一个模型了.

* 训练
```python
num_round = 10
bst = xgb.train( plst, dtrain, num_round, evallist )
```

* 保存模型
训练之后，您可以保存模型并将其转储出去.

```python
bst.save_model('0001.model')
```

* 转储模型和特征映射
您可以将模型转储到 txt 文件并查看模型的含义

```python
# 转存模型
bst.dump_model('dump.raw.txt')
# 转储模型和特征映射
bst.dump_model('dump.raw.txt','featmap.txt')
```

* 加载模型
当您保存模型后, 您可以使用如下方式在任何时候加载模型文件
```python
bst = xgb.Booster({'nthread':4}) #init model
bst.load_model("model.bin") # load data
```

提前停止
--------------
如果您有一个验证集, 你可以使用提前停止找到最佳数量的 boosting rounds（梯度次数）.
提前停止至少需要一个 `evals` 集合.
如果有多个, 它将使用最后一个.

`train(..., evals=evals, early_stopping_rounds=10)`

该模型将开始训练, 直到验证得分停止提高为止.
验证错误需要至少每个 `early_stopping_rounds` 减少以继续训练.

如果提前停止，模型将有三个额外的字段: `bst.best_score`, `bst.best_iteration` 和 `bst.best_ntree_limit`.
请注意 `train()` 将从上一次迭代中返回一个模型, 而不是最好的一个.

这与两个度量标准一起使用以达到最小化（RMSE, 对数损失等）和最大化（MAP, NDCG, AUC）.
请注意, 如果您指定多个评估指标, 则 `param ['eval_metric']` 中的最后一个用于提前停止.

预测
----------
当您 训练/加载 一个模型并且准备好数据之后, 即可以开始做预测了.

```python
# 7 个样本, 每一个包含 10 个特征
data = np.random.rand(7, 10)
dtest = xgb.DMatrix(data)
ypred = bst.predict(xgmat)
```

如果在训练过程中提前停止, 可以用 `bst.best_ntree_limit` 从最佳迭代中获得预测结果:

```python
ypred = bst.predict(xgmat,ntree_limit=bst.best_ntree_limit)
```

绘图
--------
您可以使用 plotting（绘图）模块来绘制出 importance（重要性）以及输出的 tree（树）.

要绘制出 importance（重要性）, 可以使用 ``plot_importance``. 该函数需要安装 ``matplotlib``.

```python
xgb.plot_importance(bst)
```

输出的 tree（树）会通过 ``matplotlib`` 来展示, 使用 ``plot_tree`` 指定 target tree（目标树）的序号.
该函数需要 ``graphviz`` 和 ``matplotlib``.

```python
xgb.plot_tree(bst, num_trees=2)
```

当您使用 ``IPython`` 时, 你可以使用 ``to_graphviz`` 函数, 它可以将 target tree（目标树）转换成 ``graphviz`` 实例.
``graphviz`` 实例会自动的在 ``IPython`` 上呈现.

```python
xgb.to_graphviz(bst, num_trees=2)
```
