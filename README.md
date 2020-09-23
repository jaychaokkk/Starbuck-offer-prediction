# Strabuck
* 这个数据集是一些模拟 Starbucks rewards 移动 app 上用户行为的数据。每隔几天，星巴克会向 app 的用户发送一些推送。这个推送可能仅仅是一条饮品的广告或者是折扣券或 BOGO（买一送一）。一些顾客可能一连几周都收不到任何推送。我的任务是将交易数据、人口统计数据和推送数据结合起来判断哪一类人群会受到某种推送的影响。
* 用到的库 pandas，numpy，math，json, seaborn, matplotlib, tqdm, datetime, pickle, sklearn,warnings
* 文件 Starbucks_Capstone_notebook.ipynb

* 步骤：
  * 1. 读入3个数据
    * portfolio.json – 包括推送的 id 和每个推送的元数据（持续时间、种类等等），
    * profile.json – 每个顾客的人口统计数据，
    * transcript.json – 交易、收到的推送、查看的推送和完成的推送的记录
  * 2. 对类型数据进行onehot编码，把时间数据转化为datetime，分析异常值和缺失值，删除对数据分布没有影响的异常和缺失值，把dict类数据拆成多列，把相似的字符进行统一
  * 3. 将3个处理好的数据集 通过用户id 和 推送offer id，进行整合，得到每个用户的用户信息+交易信息+推送活动信息
  * 4. 增加用户特征，比如总交易量、平均交易量、交易次数、不同推送活动的完成情况、无效完成情况等等
  * 5. 根据每个用户的推送活动id 和活动信息，判断该推送有效时间以内是否成功交易，如果成功交易，则对应的offer recieve推送记为 validrec =1，如果没有交易或者在推送时间以外交易，则 validrec =0。
  * 6. 进行数据分析，观察用户特征 如性别，年龄和交易行为之间的关系
  * 7. 将不同的推送offer特征，和不同的用户特征整合到一起，作为数据集，预测这些offer对于特定用户的validrec，即用户是否会完成推送offer的交易活动
  * 8. 评估了一个朴素模型（假设所有推送活动对于所有用户均为有效完成）的准确性和F1得分，又比较了逻辑回归，随机森林和梯度提升模型的性能。
  
* 数据清洗和分析结果：
  * 对于 profile数据，发现118岁的用户有2175名，占总数据的14%，年龄值异常，并且在income和gender上都缺失了信息，在通过特征工程拓展特征以后，缺失和无缺失的分布基本一致，可以删除缺失行。
  * 对于 transcript数据，value 列有两种情况 'offer id' 或者'offer_id' 字典，值为offerid，'amount'字典，值为交易量，因此这一列需要拆成2列，分别用'offer_id'和'amount'作为列名，值为对应的字典值，注意需要把'offer id'变为'offer_id'
  * 对于bogo_5_7_5和discount_20_10_5这两种推送，complete 竟然比 view还多 说明有的人 complete 和推送无关, 属于无效记录。排除无效记录后，对所有推送均有 complete 比 view 少，情况合理。
  * age和income 基本符合正态分布，became_member人数 每年逐渐增多，女性平均收入较高 其次是无性别的 
  * 用户单次交易的平均值是16元，大部分都在20元以下；用户交易量的平均值是125元，大部分都在150元以下；用户交易次数的平均值是8次，大部分都在11次以下。
  * 女性的平均交易量最多, 其次是其他性别和男性。男性的交易次数最多, 其次是其他性别和女性。男性女性的总交易量比较接近,可能是因为男性虽然单次交易量少,但是人数多.其他性别和unknown的总交易量都很少。
  * 除了 income 和 avg_spending 以外，这些特征之间没有很强的关联性。
  
* 预测模型结果：
  * 结果表明，随机森林模型具有最佳的训练数据准确性和F1分数。
  * 随机森林模型在训练集上准确性为 0.865，F1分数为 0.873。在测试集上准确性为 0.781，F1分数为 0.798，表明建立的随机森林模型没有过拟合。
  * 特征重要性前5名，依次为 客户总花费，客户平均花费，客户交易次数，客户收入，采用社交方式推送活动。前4个特征都是和客户经济能力有关系，说明经济能力越好的客户，收到优惠推送后有效交易的可能性较大。
  * 上述分类模型 只是判断了特定活动推送对于特定客户是否有效，但是由于缺乏足够信息，没有考察推送次数这一特征和用户有效完成的关系。还可以建立一个回归模型，预测不同推送活动对于不同用户的完成率，从而找出哪些用户在受到信息后可以多次有效消费。
