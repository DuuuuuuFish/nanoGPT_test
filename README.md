# nanoGPT_test
this is a project for myself to learn python
这个可以根据给定的文本生成文风相似的一两句话


以下均为个人理解：
这是一个简单的nanogpt的小模型，nanogpt是由一个个transformer模块堆叠起来的，每个模块都包含一个前馈神经网络和多头注意力

代码设计理由：
首先创造一个input.txt的文件，这个文件用于储存自己想生成的文章原版
代码会将文本中的字符进行排序和编码，为每一个字母标上对应的数字，用于后续计算机的阅读
随后代码将文本分为实验集和验证集，用户可以根据n = int(0.9*len(data))来调整两者之间的比例关系
get_batch是小批量处理函数，用于把文本分为小片段喂给模型进行训练
split：选择训练集还是验证集
batch_size：一次拿几段文本
block_size：每段文本的长度
ix 随机生成 batch_size 个起始位置，保证每段都不越界

单头注意力模块：
n_embd：每个字符被表示成一个向量的维度
head_size：这个注意力头输出的维度
key、query、value 是三个线性层。它们的作用是把输入的字符向量转换成三种不同的表示，用于计算注意力
tril 保证模型在预测时只能看到当前字符之前的字符
dropout 随机丢弃一些信息防止过拟合

向前传播：
B：一次性处理（）个句子
T：序列长度
C：每个词的向量维度
q 和 k 做点积得到注意力分数 wei，分数越高表示当前位置越应该关注另一个位置
除以 sqrt(C) 是为了让梯度更稳定
masked_fill 把未来位置的分数设成负无穷，这样 softmax 后这些位置的概率就是 0，模型看不到未来信息
softmax 把分数归一化成概率，然后对 v 加权求和，得到每个位置的新表示

多头注意力：
把每个注意力头拼接起来，再用线性层proj.进行合并

前馈神经网络：
一个简单的两层全连接网络，中间扩大 4 倍维度，用 ReLU 激活，再投影回原维度

nanogpt模块：
token_embedding：把每个字符的整数索引映射成一个稠密向量
position_embedding：因为 Transformer 本身不记录顺序，所以需要加上位置编码
blocks：堆叠 n_layer 个 Transformer 块
ln_f：最后的层归一化
lm_head：线性层，把最后的向量映射回词汇表大小的 logits（分数），用于预测下一个字符

使用方法：
首先创造一个input.txt的文件，这个文件用于储存自己想生成的文章原版
然后调整训练次数来训练模型（建议在1w以上），观察loss情况（建议结果在1.5以下）
max_new_tokens=（）续写词语个数
temperature=（）调整温度，小于1较保守，大于1较开放
调整 n_embd、n_head、n_layer、block_size 来适应数据和算力
