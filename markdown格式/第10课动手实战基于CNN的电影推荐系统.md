# 本资源由微信公众号：光明顶一号，分享
本文从深度学习卷积神经网络入手，基于 Github 的开源项目来完成 MovieLens 数据集的电影推荐系统。

### 什么是推荐系统呢？

什么是推荐系统呢？首先我们来看看几个常见的推荐场景。

如果你经常通过豆瓣电影评分来找电影，你会发现下图所示的推荐：

![enter image description
here](https://images.gitbook.cn/ae7b9690-84cc-11e8-b31a-735b9fbd81d4)

如果你喜欢购物，根据你的选择和购物行为，平台会给你推荐相似商品：

![enter image description
here](https://images.gitbook.cn/452fa580-84ce-11e8-bd72-e72df50fbc8a)

在互联网的很多场景下都可以看到推荐的影子。因为推荐可以帮助用户和商家满足不同的需求：

  * 对用户而言：找到感兴趣的东西，帮助发现新鲜、有趣的事物。

  * 对商家而言：提供个性化服务，提高信任度和粘性，增加营收。

常见的推荐系统主要包含两个方面的内容，基于用户的推荐系统（UserCF）和基于物品的推荐系统（ItemCF）。两者的区别在于，UserCF
给用户推荐那些和他有共同兴趣爱好的用户喜欢的商品，而 ItemCF 给用户推荐那些和他之前喜欢的商品类似的商品。这两种方式都会遭遇冷启动问题。

下面是 UserCF 和 ItemCF 的对比：

![enter image description
here](https://images.gitbook.cn/f5728ff0-84d0-11e8-9658-97176c275e7c)

### CNN 是如何应用在文本处理上的？

提到卷积神经网络（CNN），相信大部分人首先想到的是图像分类，比如 MNIST 手写体识别，CAFRI10 图像分类。CNN
已经在图像识别方面取得了较大的成果，随着近几年的不断发展，在文本处理领域，基于文本挖掘的文本卷积神经网络被证明是有效的。

首先，来看看 CNN 是如何应用到 NLP 中的，下面是一个简单的过程图：

![enter image description
here](https://images.gitbook.cn/5ccd6290-84d3-11e8-815d-196036bcf3c7)

和图像像素处理不一样，自然语言通常是一段文字，那么在特征矩阵中，矩阵的每一个行向量（比如 word2vec 或者 doc2vec）代表一个
Token，包括词或者字符。如果一段文字包含有 n 个词，每个词有 m 维的词向量，那么我们可以构造出一个 `n*m` 的词向量矩阵，在 NLP
处理过程中，让过滤器宽度和矩阵宽度保持一致整行滑动。

### 动手实战基于 CNN 的电影推荐系统

将 CNN 的技术应用到自然语言处理中并与电影推荐相结合，来训练一个基于文本的卷积神经网络，实现电影个性化推荐系统。

首先感谢作者 chengstone 的分享，源码请访问下面网址：

  * [Github](https://github.com/chengstone/movie_recommender)

在验证了 CNN 应用在自然语言处理上是有效的之后，从推荐系统的个性化推荐入手，在文本上，把 CNN
成果应用到电影的个性化推荐上。并在特征工程中，对训练集和测试集做了相应的特征处理，其中有部分字段是类型性变量，特征工程上可以采用 `one-hot`
编码，但是对于 UserID、MovieID 这样非常稀疏的变量，如果使用 `one-hot`，那么数据的维度会急剧膨胀，对于这份数据集来说是不合适的。

具体算法设计如下：

**1.** 定义用户嵌入矩阵。

用户的特征矩阵主要是通过用户信息嵌入网络来生成的，在预处理数据的时候，我们将
UserID、MovieID、性别、年龄、职业特征全部转成了数字类型，然后把这个数字当作嵌入矩阵的索引，在网络的第一层就使用嵌入层，这样数据输入的维度保持在（N，32）和（N，16）。然后进行全连接层，转成（N，128）的大小，再进行全连接层，转成（N，200）的大小，这样最后输出的用户特征维度相对比较高，也保证了能把每个用户所带有的特征充分携带并通过特征表达。

具体流程如下：

![enter image description
here](https://images.gitbook.cn/3983eae0-88cd-11e8-a341-85988e434d41)

**2.** 生成用户特征。

生成用户特征是在用户嵌入矩阵网络输出结果的基础上，通过2层全连接层实现的。第一个全连接层把特征矩阵转成（N，128）的大小，再进行第二次全连接层，转成（N，200）的大小，这样最后输出的用户特征维度相对比较高，也保证了能把每个用户所带有的特征充分携带并通过特征表达。

具体流程如下：

![enter image description
here](https://images.gitbook.cn/747e2f20-88cd-11e8-9944-4f7451c47515)

**3.** 定义电影 ID 嵌入矩阵。

通过电影 ID 和电影类型分别生成电影 ID 和电影类型特征，电影类型的多个嵌入向量做加和输出。电影 ID
的实现过程和上面一样，但是对于电影类型的处理相较于上面，稍微复杂一点。因为电影类型有重叠性，一个电影可以属于多个类别，当把电影类型从嵌入矩阵索引出来之后是一个（N，32）形状的矩阵，因为有多个类别，这里采用的处理方式是矩阵求和，把类别加上去，变成（1，32）形状，这样使得电影的类别信息不会丢失。

具体流程如下：

![enter image description
here](https://images.gitbook.cn/9483d0e0-88cd-11e8-8555-ffb97409ea91)

**4.** 文本卷积神经网络设计。

文本卷积神经网络和单纯的 CNN
网络结构有点不同，因为自然语言通常是一段文字与图片像素组成的矩阵是不一样的。在电影文本特征矩阵中，矩阵的每一个行构成的行向量代表一个
Token，包括词或者字符。如果一段文字有 n 个词，每个词有 m 维的词向量，那么我们可以构造出一个 `n*m` 的矩阵。而且 NLP
处理过程中，会有多个不同大小的过滤器串行执行，且过滤器宽度和矩阵宽度保持一致，是整行滑动。在执行完卷积操作之后采用了 ReLU
激活函数，然后采用最大池化操作，最后通过全连接并 Dropout 操作和 Softmax
输出。这里电影名称的处理比较特殊，并没有采用循环神经网络，而采用的是文本在 CNN 网络上的应用。

对于电影数据集，我们对电影名称做 CNN
处理，其大致流程，从嵌入矩阵中得到电影名对应的各个单词的嵌入向量，由于电影名称比较特殊一点，名称长度有一定限制，这里过滤器大小使用时，就选择2、3、4、5长度。然后对文本嵌入层使用滑动2、3、4、5个单词尺寸的卷积核做卷积和最大池化，然后
Dropout 操作，全连接层输出。

具体流程如下：

![enter image description
here](https://images.gitbook.cn/49e32370-8e3a-11e8-8ee0-a17ea463076e)

具体过程描述：

（1）首先输入一个 `32*32` 的矩阵；

（2）第一次卷积核大小为 `2*2`，得到 `31*31` 的矩阵，然后通过 `[1,14,1,1]` 的 `max-pooling` 操作，得到的矩阵为
`18*31`；

（3）第二次卷积核大小为 `3*3`，得到 `16*29的矩阵，然后通过[1,13,1,1]` 的 `max-pooling` 操作，得到的矩阵为
`4*29`；

（4）第三次卷积核大小 `4*4`，得到 `1*26` 的矩阵，然后通过 `[1,12,1,1]` 的 `max-pooling` 操作，得到的矩阵为
`1*26`；

（5）第四次卷积核大小 `5*5`，得到 `1*22` 的矩阵，然后通过 `[1,11,1,1]` 的 `max-pooling` 操作，得到的矩阵为
`1*22`；

（6）最后通过 Dropout 和全连接层，`len(window_sizes) * filter_num =32`，得到 `1*32`的矩阵。

**5.** 电影各层做一个全连接层。

将上面几步生成的特征向量，通过2个全连接层连接在一起，第一个全连接层是电影 ID 特征和电影类型特征先全连接，之后再和 CNN
生成的电影名称特征全连接，生成最后的特征集。

具体流程如下：

![enter image description
here](https://images.gitbook.cn/336d6d10-88ce-11e8-bee1-4fab1cb0896f)

**6.** 完整的基于 CNN 的电影推荐流程。

把以上实现的模块组合成整个算法，将网络模型作为回归问题进行训练，得到训练好的用户特征矩阵和电影特征矩阵进行推荐。

![enter image description
here](https://images.gitbook.cn/abffaae0-88ce-11e8-950d-3f19ed380d25)

### 基于 CNN 的电影推荐系统代码调参过程

在训练过程中，我们需要对算法预先设置一些超参数，这里给出的最终的设置结果：

    
    
        # 设置迭代次数
        num_epochs = 5
        # 设置BatchSize大小
        batch_size = 256
        #设置dropout保留比例
        dropout_keep = 0.5
        # 设置学习率
        learning_rate = 0.0001
        # 设置每轮显示的batches大小
        show_every_n_batches = 20
    

首先对数据集进行划分，按照 4:1 的比例划分为训练集和测试集，下面给出的是算法模型最终训练集合测试集使用的划分结果：

    
    
        #将数据集分成训练集和测试集，随机种子不固定
        train_X,test_X, train_y, test_y = train_test_split(features,  
                                                     targets_values,  
                                                     test_size = 0.3,  
                                                     random_state = 0) 
    

接下来是具体模型训练过程。训练过程，要不断调参，根据经验调参粒度可以选择从粗到细分阶段进行。

调参过程对比：

（1）第一步，先固定，`learning_rate=0.01` 和 `num_epochs=10`，测试 `batch_size=128` 对迭代时间和
Loss 的影响；

（2）第二步，先固定，`learning_rate=0.01` 和 `num_epochs=10`，测试 `batch_size=256` 对迭代时间和
Loss 的影响；

（3）第三步，先固定，`learning_rate=0.01` 和 `num_epochs=10`，测试 `batch_size=512` 对迭代时间和
Loss 的影响；

（4）第四步，先固定，`learning_rate=0.01` 和 `num_epochs=5`，测试 `batch_size=128` 对迭代时间和
Loss 的影响；

（5）第五步，先固定，`learning_rate=0.01` 和 `num_epochs=5`，测试 `batch_size=256` 对迭代时间和
Loss 的影响；

（6）第六步，先固定，`learning_rate=0.01` 和 `num_epochs=5`，测试 `batch_size=512` 对迭代时间和
Loss 的影响；

（7）第七步，先固定，`batch_size=256` 和 `num_epochs=5`，测试 `learning_rate=0.001` 对 Loss
的影响；

（8）第八步，先固定，`batch_size=256` 和 `num_epochs=5`，测试 `learning_rate=0.0005` 对 Loss
的影响；

（9）第九步，先固定，`batch_size=256` 和 `num_epochs=5`，测试 `learning_rate=0.0001` 对 Loss
的影响；

（10）第十步，先固定，`batch_size=256` 和 `num_epochs=5`，测试 `learning_rate=0.00005` 对
Loss 的影响。

得到的调参结果对比表如下：

![enter image description
here](https://images.gitbook.cn/64b2fe20-88cf-11e8-81e9-19f90a93f5ff)

通过上面（1）-（6）步调参比较，在 `learning_rate`、`batch_size` 相同的情况下，`num_epochs`
对于训练时间影响较大；而在 `learning_rate`、`num_epochs` 相同情况下，`batch_size` 对 Loss
的影响较大，`batch_size` 选择512，Loss 有抖动情况，权衡之下，最终确定后续调参固定采用
`batch_size=256`、`num_epochs=5` 的超参数值，后续（7）-（10）步，随着 `learning_rate` 逐渐减小，发现
Loss 是先逐渐减小，而在 `learning_rate=0.00005` 时反而增大，最终选择出学习率为 `learning_rate=0.0001`
的超参数值。

### 基于 CNN 的电影推荐系统电影推荐

在上面，完成模型训练验证之后，实际来进行推荐电影，这里使用生产的用户特征矩阵和电影特征矩阵做电影推荐，主要有三种方式的推荐。

**1.** 推荐同类型的电影。

思路是：计算当前看的电影特征向量与整个电影特征矩阵的余弦相似度，取相似度最大的 `top_k` 个，这里加了些随机选择在里面，保证每次的推荐稍稍有些不同。

    
    
        def recommend_same_type_movie(movie_id_val, top_k = 20):
    
            loaded_graph = tf.Graph()  #
            with tf.Session(graph=loaded_graph) as sess:  #
                # Load saved model
                loader = tf.train.import_meta_graph(load_dir + '.meta')
                loader.restore(sess, load_dir)
    
                norm_movie_matrics = tf.sqrt(tf.reduce_sum(tf.square(movie_matrics), 1, keep_dims=True))
                normalized_movie_matrics = movie_matrics / norm_movie_matrics
    
                #推荐同类型的电影
                probs_embeddings = (movie_matrics[movieid2idx[movie_id_val]]).reshape([1, 200])
                probs_similarity = tf.matmul(probs_embeddings, tf.transpose(normalized_movie_matrics))
                sim = (probs_similarity.eval())
                print("您看的电影是：{}".format(movies_orig[movieid2idx[movie_id_val]]))
                print("以下是给您的推荐：")
                p = np.squeeze(sim)
                p[np.argsort(p)[:-top_k]] = 0
                p = p / np.sum(p)
                results = set()
                while len(results) != 5:
                    c = np.random.choice(3883, 1, p=p)[0]
                    results.add(c)
                for val in (results):
                    print(val)
                    print(movies_orig[val])
                return result
    

**2.** 推荐您喜欢的电影。

思路是：使用用户特征向量与电影特征矩阵计算所有电影的评分，取评分最高的 `top_k` 个，同样加了些随机选择部分。

    
    
        def recommend_your_favorite_movie(user_id_val, top_k = 10):
    
            loaded_graph = tf.Graph()  #
            with tf.Session(graph=loaded_graph) as sess:  #
                # Load saved model
                loader = tf.train.import_meta_graph(load_dir + '.meta')
                loader.restore(sess, load_dir)
    
                #推荐您喜欢的电影
                probs_embeddings = (users_matrics[user_id_val-1]).reshape([1, 200])
                probs_similarity = tf.matmul(probs_embeddings, tf.transpose(movie_matrics))
                sim = (probs_similarity.eval())
    
                print("以下是给您的推荐：")
                p = np.squeeze(sim)
                p[np.argsort(p)[:-top_k]] = 0
                p = p / np.sum(p)
                results = set()
                while len(results) != 5:
                    c = np.random.choice(3883, 1, p=p)[0]
                    results.add(c)
                for val in (results):
                    print(val)
                    print(movies_orig[val])
    
                return results
    

**3.** 看过这个电影的人还看了（喜欢）哪些电影。

（1）首先选出喜欢某个电影的 `top_k` 个人，得到这几个人的用户特征向量；

（2）然后计算这几个人对所有电影的评分 ；

（3）选择每个人评分最高的电影作为推荐；

（4）同样加入了随机选择。

    
    
        def recommend_other_favorite_movie(movie_id_val, top_k = 20):
            loaded_graph = tf.Graph()  #
            with tf.Session(graph=loaded_graph) as sess:  #
                # Load saved model
                loader = tf.train.import_meta_graph(load_dir + '.meta')
                loader.restore(sess, load_dir)
                probs_movie_embeddings = (movie_matrics[movieid2idx[movie_id_val]]).reshape([1, 200])
                probs_user_favorite_similarity = tf.matmul(probs_movie_embeddings, tf.transpose(users_matrics))
                favorite_user_id = np.argsort(probs_user_favorite_similarity.eval())[0][-top_k:]
    
                print("您看的电影是：{}".format(movies_orig[movieid2idx[movie_id_val]]))
    
                print("喜欢看这个电影的人是：{}".format(users_orig[favorite_user_id-1]))
                probs_users_embeddings = (users_matrics[favorite_user_id-1]).reshape([-1, 200])
                probs_similarity = tf.matmul(probs_users_embeddings, tf.transpose(movie_matrics))
                sim = (probs_similarity.eval())
                p = np.argmax(sim, 1)
                print("喜欢看这个电影的人还喜欢看：")
                results = set()
                while len(results) != 5:
                    c = p[random.randrange(top_k)]
                    results.add(c)
                for val in (results):
                    print(val)
                    print(movies_orig[val])
                return results
    

### 基于 CNN 的电影推荐系统不足

这里讨论一下基于上述方法所带来的不足：

  1. 由于一个新的用户在刚开始的时候并没有任何行为记录，所以系统会出现冷启动的问题；

  2. 由于神经网络是一个黑盒子过程，我们并不清楚在反向传播的过程中的具体细节，也不知道每一个卷积层抽取的特征细节，所以此算法缺乏一定的可解释性；

  3. 一般来说，在工业界，用户的数据量是海量的，而卷积神经网络又要耗费大量的计算资源，所以进行集群计算是非常重要的。但是由于本课程所做实验环境有限，还是在单机上运行，所以后期可以考虑在服务器集群上全量跑数据，这样获得的结果也更准确。

### 总结

上面通过 [Github](https://github.com/chengstone/movie_recommender) 上一个开源的项目，梳理了
CNN 在文本推荐上的应用，并通过模型训练调参，给出一般的模型调参思路，最后建议大家自己把源码下载下来跑跑模型，效果更好。

### 参考文献及推荐阅读

  1. [推荐系统](https://blog.csdn.net/dream_catcher_10/article/details/50733172)

  2. Deep Convolutional Neural Networks for Sentiment Analysis of ShortTexts,CND Santos ,M Gattit ,2014.

  3. 推荐系统实践，p50-60，p120-130，项亮。


## 更多资源下载交流请加微信：Morstrong
# 本资源由微信公众号：光明顶一号，分享,一个用技术共享精品付费资源的公众号！