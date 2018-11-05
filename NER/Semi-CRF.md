### 基于MCNN-SCRF的新能源汽车命名实体识别
整个思路是 用神经网络获得特征表示，然后semi-crf负责片段级别的实体识别。

对Semi-crf同神经网络相结合做了深入的研究。

#### Semi-CRF

有个好处，semi-crf run fastter.

#### 特征表示
##### input unnit特征
1. blstm
2. cnn(因为有max pool 位置不敏感)  
**尽管使用CNN可以获取很好地字符级别特征，但是因为max pool的存在，对位置信息并不敏感，包含相同的字的片段，会有相同的特征向量**
3. concatentation（位置敏感，计算量小)
##### segment entire特征
1. 预训练的segment Embedding
2. 随机初始化的，有严重的overfitting  


#### 对比实验

  BLSTM_CRF  
  CNN_CRF  
  MCNN_CRF  
  concatenation_SCRF 速度比较快
  BLSTM_SCRF  
  CNN_SCRF CNN对位置不敏感  
  MCNN_SCRF  


SEGMENTAL RECURRENT NEURAL NETWORKS
