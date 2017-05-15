# CNN-Basic

### Back Propagation

卷积层$m_i$, 输入是$x_i$，输出是$y_i$,该卷积层的损失的倒数是
$$
{\partial loss\over\partial m_i} = {{\partial loss\over\partial y_i}} \cdot {\partial y_i\over\partial m_i} = \Delta y_i \cdot  {\partial y_i\over\partial m_i}
$$

$$
\Delta y_i = \Delta x_{i+1} {\partial x_{i+1}\over \partial y_i} =  \Delta y_{i+1} {\partial y_{i+1}\over \partial x_{i+1}} {\partial x_{i+1}\over \partial y_i}
$$

* x + y的反向是将梯度分别传递到x, y
* max(x, y)的反向是将梯度传递到大的数上

### Convolution Layer

卷积核的大小：一般选择奇数满足对称，大小的话用（3*3的效果加多层的效果更好）

1 * 1卷积：没有办法进行空间运算，好处，卷积核参数量非常少（m个神经元，n层，有mn个参数，而3*3有9mn的参数。第一层100 * 100 * 512，通过128个1 * 1卷积，最后得到100 * 100 * 128的矩阵，也就是说非常高效的降维（Google Net）

pad(边界扩充)，实际上没什么影响

卷积层为什么是64， 128，256，GPU并行更高效

特征图中的第t张，i行，j列数值表达？

反向传播中的第t张，i行，j列梯度表达？

反向传播中的第t卷积核，第s层i行j列梯度表达

### Functional Layer

池化：每张特征图都进行降维

归一化(normalization layer): 原因是特征数scale不一致，加速训练，提高精度，有可能尽可能归一化

batch normalization: mini-batch mean -> mini-batch variance -> normalize -> scale and shift

Local Response normalization

Slice layer: 对某些层菲苾进行处理，多套参数，更强的特征描述能力 (AlexNet)

Merge layer: Inception Concatenation (ResNet, GoogleNet)

