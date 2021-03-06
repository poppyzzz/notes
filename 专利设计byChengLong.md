# 专利设计

`Author:DC`

`Date:2021-08-18`





## 1 一种基于深度对比学习的流量异常检测方法

- 搜索关键词

  深度学习 对比学习 异常检测 攻击检测

- 解决问题

  现有方法：

  做流量异常检测时，在**有标注数据**的情况下，一、比较先进的方法都是使用自编码器等神经网络做自监督，重构正常样本来进行训练。模型预测时再通过loss来检测样本是否存在异常。二、也有一些是通过专家知识做一些特征工程，再通过神经网络分类器来进行训练。三、还有一些使用正常样本集，通过提取样本模板，通过向神经网络输入序列模板编码，预测序列下一条样本的方法来训练模型。模型通过是否能够正确预测序列下一条样本的方法来检测样本是否存在异常。在**无标注数据**的情况下，一、通过一些专家知识做一些特征工程，在再通过一些异常检测算法来检测异常。二、通过自编码器等神经网络自监督，提取隐层表示，在通过机器学习异常检测算法来检测异常。或是使用自编码器对特征工程生成的特征矩阵做降维，再使用一些机器学习异常检测算法来检测异常。
  这些方法在无标注数据条件下，**无法提取出一个紧凑的特征空间**。

  现有方法在无标注数据的情况下，提取流量文本的语义特征的能力是提升效果的重点，有一些文献表明BERT等一些**预训练模型**的隐层提取，在**特征空间上聚集不明显**。SimCSE 的方法提取的隐层表示刚好可以克服这些弱点。现有专利文献中只在中国科技大学的一个专利中有看到原理类似的改进损失函数，加入空间聚集程度因素的方法来做自监督，也是在有标注数据的情况下实现的神经网络。

- 设计思路

  框架

  ```mermaid
  graph TD;
  A[提取url数据做泛化]-->B[数据集切分为 evaluate/train集合]
  B-->C[对evaluate数据使用SimCSE相同tokenizer分词, 训练TFIDF模型]
  C-->D[将evaluate数据编码为TFIDF BOW vec, 并通过余弦相似度计算相似性]
  D-->E[剔除相似度小于0.9且高于0.1的相似对, 生成evaluation pairs]
  E-->F[使用train数据, SimCSE训练模型, 并使用部<br>分evaluation pairs来评估SimCSE的训练效<br>果, 当相似度准率较高, 且epoch内step评估<br>结果比较平稳时, 可提前停止训练]
  F-->G[使用剩下evaluation pairs对训练好的SimCSE模型做测试, 若过拟合则返回上一步重新训练]
  G-->H[使用训练好的SimCSE模型隐层来生成url数据的特征空间, 并使用IForest做异常检测]
  ```

  

## 2 一种流量异常检测可解释方法

- 搜索关键词

  异常检测 解释

- 解决问题

  现有方法（查新量暂时较少）有：

  1. 直接使用偏度等统计量对单个特征直接做异常值评分，判断单个特征是否异常。
  2. 使用多个不同异常检测算法对同一特征空间做异常检测，使用单个特征和异常结果之间的关联程度（如互信息判断），再结合异常算法得分加权，取得单个样本的异常程度。

  这些方法都是对特征的全局异常关联程度的判断，**没有考虑到单个样本与各特征之间的关联**，也没有考虑到样本**局部异常分布**情况。

- 设计思路

  框架

  ```mermaid
  graph TD;
  A[流量数据]-->B[做特征工程提取特征矩阵]
  B-->C[异常检测算法训练模型并预测异常样本]
  C-->D[使用训练正常样本做聚类, 找到正常簇质心]
  D-->E[计算所有正常簇质心簇间距离最大值]
  E-->F{当前异常样本是否与<br>各正常簇质心距离之<br>和超过正常簇质心簇<br>间距离最大值}
  F--是-->G[样本全局异常, 计算当前异常样<br>本与训练正常样本各特征的偏<br>离程度]
  F--否-->H[样本局部异常, 计算当前异常样<br>本与距离top 3最近正常簇各特<br>征的偏离程度]
  G-->I[使用互信息启发式搜索算法搜索高<br>贡献度特征集合, 对高贡献度特征做<br>权重提升, 低贡献度特征做权重抑制]
  H-->I
  I-->J[若特征偏离程度超过阈值, 则该特征认为被激发]
  J-->K[若阈值调整较小,仍找不到激<br>发特征,可以选取top 3偏离<br>程度的特征来填充]
  K-->L[对激发特征通过解释话术映射表, 映射为相关业务解释]
  ```

  

