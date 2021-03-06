# ChineseNER
遍历整个训练集，统计每个字出现的出现次数，并按照降序排列，创建一个字典，键为汉字，值为该汉字对应的编号，
最后在字典末尾加入'pad'的键和其对应的值。然后按照键值的大小从前向后访问整个字典，如该字在对应的训练好的词向量
文件中，则将该字对应的100维字向量添加到一个空列表中，若该字不在训练好的词向量文件中，则随机产生100维的字向量
加入列表中。将每个字对应的编号也建立一个字典，键为tag名，值为对应的编号。然后再次遍历整个训练集，将每个字符都
用对应的编号代替，每个字符对应的tag也用对应的编号代替。将测试集也做同样处理。每次训练时，先将训练集混杂一下。
然后每次从训练集中选择batch_size个数据，求出这batch_size个数据最长句子的长度，其他的句子也都用pad对应的编号
填充到此长度。对应的tag则都用'O'对应的编号填充。然后构造BLSTM网络。神经网络由embedding层，blstm,映射层，CRF
层组成。选择将每一句话中每个汉字的字向量(此为取训练好的字向量，100维)，每个词语的组成的数字((一个字组成的词编码为0;
两个字组成的词为1,3;三个字组成的词为1,2,3))编码(此为取随机生成的数据，20维)作为embedding，然后加入dropout,
然后作为Blstm的输入，经过BLSTM得到output形状为[batch_size,max_seq_len, 2*lstm_dim]将其reshape为
[batch_size*max_seq_len, 2*lstm_dim],然后经过第一层projection,乘以对应的权值
(即[2*lstm_size,lstm_size]的随机数)并加上[lstm_size]的偏置,经过tanh函数得到hidden,然后再经过一层
projection层，乘以对应的权值(即[lstm_size,num_tags]的随机数)并加上[num_tags]的偏置得到self.logits
将其reshape为[batch_size,max_seq_len,num_tags]，最后考虑到各个字的状态的相互关系，接入CRF而不是
softmax作为self.loss。然后使用Adam优化器进行求解。
本程序训练结束后，accuracy:  98.59%,precision:  92.17%; recall:  91.11%; FB1:  91.64
