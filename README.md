# Kaggle-Titanic

Titanic生存率分思路


# (1) 了解数据
data_train.description()

1.1 乘客属性分布

1.1 幸存人数

1.2 乘客等级分布

1.3 按年龄看获救分布
1.4 各等级的乘客年龄分布
1.5 各登陆船口上船人数

1.6 属性与获救结果的关联统计
性别，乘客，登船港口和堂兄弟/妹，孩子/父母与获救结果的关系


# (2) 数据预处理


1. 缺值的情况，我们会有几种常见的处理方式

如果缺值的样本占总数比例极高，我们可能就直接舍弃了，作为特征加入的话，可能反倒带入noise，影响最后的结果了


如果缺值的样本适中，而该属性非连续值特征属性(比如说类目属性)，那就把NaN作为一个新类别，加到类别特征中


如果缺值的样本适中，而该属性为连续值特征属性，有时候我们会考虑给定一个step(比如这里的age，我们可以考虑每隔2/3岁为一个步长)，然后把它离散化，之后把NaN作为一个type加到属性类目中。


有些情况下，缺失的值个数并不是特别多，那我们也可以试着根据已有的值，拟合一下数据，补充上。

2. 对类目型的特征因子化

3. 标准化

# (3)逻辑回归建模，baseline


把需要的feature字段取出来，转成numpy格式，使用scikit-learn中的LogisticRegression建模


test_data”也要做和”train_data”一样的预处理

# (4) 逻辑回归系统优化

1 模型系数关联分析

分析model系数和feature关联：

权重绝对值非常大的feature，在我们的模型上：

Sex属性，如果是female会极大提高最后获救的概率，而male会很大程度拉低这个概率。


Pclass属性，1等舱乘客最后获救的概率会上升，而乘客等级为3会极大地拉低这个概率。


有Cabin值会很大程度拉升最后获救概率(这里似乎能看到了一点端倪，事实上从最上面的有无Cabin记录的Survived分布图上看出，即使有Cabin记录的乘客也有一部分遇难了，估计这个属性上我们挖掘还不够)
Age是一个负相关，意味着在我们的模型里，年龄越小，越有获救的优先权(还得回原数据看看这个是否合理）


有一个登船港口S会很大程度拉低获救的概率，另外俩港口压根就没啥作用(这个实际上非常奇怪，因为我们从之前的统计图上并没有看到S港口的获救率非常低，所以也许可以考虑把登船港口这个feature去掉试试)。
船票Fare有小幅度的正相关(并不意味着这个feature作用不大，有可能是我们细化的程度还不够，举个例子，说不定我们得对它离散化，再分至各个乘客等级上？)

# 交叉验证验证算法的效果

优化操作：

Age属性不使用现在的拟合方式，而是根据名称中的『Mr』『Mrs』『Miss』等的平均值进行填充。
Age不做成一个连续值属性，而是使用一个步长进行离散化，变成离散的类目feature。


Cabin再细化一些，对于有记录的Cabin属性，我们将其分为前面的字母部分(我猜是位置和船层之类的信息) 和 后面的数字部分(应该是房间号，有意思的事情是，如果你仔细看看原始数据，你会发现，这个值大的情况下，似乎获救的可能性高一些)。


Pclass和Sex俩太重要了，我们试着用它们去组出一个组合属性来试试，这也是另外一种程度的细化。


单加一个Child字段，Age<=12的，设为1，其余为0(你去看看数据，确实小盆友优先程度很高啊)


如果名字里面有『Mrs』，而Parch>1的，我们猜测她可能是一个母亲，应该获救的概率也会提高，因此可以多加一个Mother字段，此种情况下设为1，其余情况下设为0
登船港口可以考虑先去掉试试(Q和C本来就没权重，S有点诡异)


把堂兄弟/兄妹 和 Parch 还有自己 个数加在一起组一个Family_size字段(考虑到大家族可能对最后的结果有影响)


Name是一个我们一直没有触碰的属性，我们可以做一些简单的处理，比如说男性中带某些字眼的(‘Capt’, ‘Don’, ‘Major’, ‘Sir’)可以统一到一个Title，女性也一样。


# learning curves


learning curve可以帮我们判定我们的模型现在所处的状态。我们以样本数为横坐标，

训练和交叉验证集上的错误率作为纵坐标，两种状态分别如下两张图所示：过拟合(overfitting/high variace)，欠拟合(underfittin


过拟合而言，通常以下策略对结果优化是有用的：

做一下feature selection，挑出较好的feature的subset来做training


提供更多的数据，从而弥补原始数据的bias问题，学习到的model也会更准确

# 模型融合




















