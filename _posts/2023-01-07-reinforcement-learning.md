---
layout:     post
title:      "Reading notes -- Reinforcement Learning 2"
subtitle:   "读书笔记 -- 强化学习2"
date:       2022-11-30 10:15:00
author:     "Di Chen"
catalog:    true
header-img: "img/in-post/cover/rlbook.jpg"
tags:
    - algorithm
    - AI
    - Reading
---

## 背景

上个月看了一下强化学习的权威教材 [《Reinforcement Learning: An Introduction》](http://www.incompleteideas.net/book/the-book-2nd.html)，希望对强化学习有个比较基础但全面的认识，可以供我开始利用强化学习的算法。

看书的时候顺手记下一些自己的思考，和具体的算法无关，因为具体的算法细节在书里已经讲的很好了，我没必要画蛇添足再复述一遍，建议大家还是看书更好。但书中提到的一些思想，以及和我的思考，可能是比较容易在其他方面借鉴的。

## 强化学习在传统算法技术上的影响

### 强化学习和传统机器学习
传统机器学习指的是有监督学习和无监督学习，例如常见的分类、回归、模式识别、预测等任务。我理解强化学习的核心区别，其实是把机器学习建模的范围扩大了，直接建模了环境和智能体的概念，从而把适用范围扩大到了几乎所有问题上。

#### 强化学习的核心
我理解强化学习的核心，其实是**环境模型的建模**和**动作价值函数的训练**：
  - 环境：环境可能很复杂，例如一个环境中有什么物体，这些物体是什么状态，这些信息要怎么以数字的形式表现出来。如果维度过大，例如有几 PB 的参数，则无法放在计算机中处理，所以将环境降维并建模是必然的。如果输入是文字、图片的识别，已经有成熟的编码方案了，比如 word embedding，或者 CNN 的网络，可以直接拿来套用可能效果就比较好。
  - 环境中的动作：环境的建模同时也包括对动作的建模，例如鼠标移动这个操作在建模时，是将屏幕上的每个像素都作为一个可选择的操作呢？还是将可点击区域作为可选择的目标呢？
  - 动作价值函数：在环境降维建模完成后，针对每个环境的状态，学习一个对应的动作价值函数，即在当前状态下，智能体做的每个动作的价值高低。这样智能体就只要选择做出价值最高的动作就行了。在传统的机器学习中，也需要对输入输出进行拟合，只不过输出的数据往往没有前后关系，例如一个图片的识别结果不会影响到其他图片。但在强化学习中，一个动作的价值函数往往是会收到其后续状态的价值函数影响的。而不同的强化学习算法主要是在于如何更有效率地利用采样结果，将模型的参数收敛到更好的位置。

动作价值函数的训练，和传统的机器学习其实很像。智能体在多次尝试之后，就生成了训练集，只要尝试次数足够多，就能采样获得每个动作的价值，这个采样也是一个难点，往往需要通过仿真程序运行很多次才能生成。但是只要有了采样的结果，就可以套用传统机器学习的算法，对 【环境输入 + 动作 -> 价值】这个输入输出进行拟合训练。可以把这个动作价值函数想象成有监督学习中的标签，只不过这个情况下没有人能做 “标注员” 的工作，因为每个动作的价值是怎样的，可能一眼无法直观地看出来，只有通过多次尝试并采样才能有个答案。

虽然强化学习中也有一些算法并不直接使用价值函数，比如 Policy Gradient，但是我认为本质上还是一个价值函数，只不过比直接用数字表示的价值函数更抽象一些，带入了一些概率的思维。

#### 共性的训练技巧

在书中提到的很多拟合方法和技巧其实在传统机器学习中也有运用过。例如 Kernel-base function approximation 其实和支持向量机 (SVM) 的思想是一样的，在强化学习中选用价值函数的拟合方式时，也可以借用传统机器学习中的方法。例如 SVM、Neural Network 在处理同质化特征时表现更好，而决策树模型例如随机森林和 GBDT 在处理非同质化特征和异常值时表现更好，收敛更快。

### 强化学习的优势

强化学习在这里的优势其实是一个全流程的反馈，它可以把不同阶段的选择统一进行考虑。例如前一个状态做的决策会如何影响下一个状态，通过动作价值函数采样更新的过程，会在动作价值函数中体现出来。所以也可以考虑在已经训练好的任务上，使用强化学习的方式使其在最终的场景中表现得更好。例如最近很火的 chatgpt，就是在传统的机器学习生成的语言模型基础上，增加了对答流程应用的训练，使得语言模型在最终的对话场景下可以发挥出最好的效果。

有一些问题难以用有监督学习解决，因为难以找到合适的 “标签”。例如寻路问题：在进行寻路算法的设计时，如果要用有监督学习，那么就需要得到一个最佳路径作为标签。如果计算这个最佳路径的难度很大，就无法获得标签，也就无法使用有监督学习。但是强化学习则没有这个要求，通过采样和尝试，也许不能找到最佳路径，但是可以找到多条次优路径。在 NP hard 问题的求解上，也是可以运用强化学习的。

## 强化学习的落地难点

强化学习虽然有几个例如 AlphaGo 这样比较出名的应用案例，但是在工业界落地的例子仍然并不是很多。除了学习这个知识领域本身的门槛之外，我觉得可能还有一些其他的因素，导致强化学习不能直接开箱即用地套上去。

### 许多问题领域都需要实现环境模拟器

和有监督学习不同，强化学习在训练的流程中，多了一个对模拟器的依赖。比如我们要把AI的算法落地到人脸相关的场景中时，如果使用有监督学习做人脸识别，只要收集并标注好人脸信息，就可以套用已有的算法库进行训练评估了。但比如要使用强化学习智能调整摄像头姿态，我们则需要实现一个模拟器，例如用 Unity 来模拟摄像头看到人脸并转动的过程。

而每一个需要落地的问题，可能都需要实现这样一个环境模拟器，根据落地的行业不同，可能是金融、工业、半导体、电商，每个行业需要实现的模拟器也不同。所以尝试在一个领域使用强化学习的成本并不低，不像有监督学习一样，从数据库中导出一些数据就可以套算法做测试了。

### 部分场景无法实现模拟器

如果可以通过仿真模拟来实现一个虚拟环境，例如自动驾驶，已经是很好的情况了。当环境信息包含人这样的因素时，我们可能无法实现一个虚拟环境。例如面对广告点击问题，或者电商展示优化问题时，提供反馈 reward 的是人类，而人类的偏好和行为目前是无法用代码来模拟的。

没有了模拟器，如果要在这类场景中训练一个强化学习的算法，就得用 **Off Policy 的策略**，即采样时用的策略和实际训练的策略不同。Off Policy 的策略在收敛的速度和质量上都会差一些，无法达到 On Policy 同样的效果。但这很多时候也是唯一的办法，例如广告点击问题中使用的算法，每一次修改测试可能都需要很高昂的成本，**这个成本的来源并不是计算资源，而是商业上的机会成本**：原本某个广告位可以用原本的算法提供 2 美元的收入，但为了测试和训练强化学习，可能需要牺牲这一部分收入。

### 训练流程很可能受 CPU 限制

和有监督学习不同，一个新的策略如果要进一步提升，则需要跟环境有更多地互动，从而针对这个策略进行 Policy Evalution。以往积累的其他策略的环境交互结果对本策略的帮助并不是很大。而跟环境互动这一步，可能就需要花很多 CPU 计算资源，因为模拟器的实现逻辑比较复杂，只能在 CPU 上执行。

如果我们要跟环境互动100万次才能获得一个相对合理的智能体，那么每一次训练可能就需要执行100万次环境模拟器，这时候环境模拟器的实现性能很可能就决定了训练的效率。

### 建模对最终的效果影响很大

和传统的机器学习一样，对问题的建模方式好坏比算法的影响可能更大。在传统的机器学习问题中，如果能构造出好的特征集，logistic regression 也可以表现得比 deep learning 更好，因为好的建模可以使因果关系更明显。而在强化学习的领域，这也很重要。下面举几个建模可能影响训练效果的例子：

#### 状态重叠

强化学习问题中，有不少算法是通过学习 state-action 的价值函数来优化智能体的表现的。而价值函数是跟状态直接相关的，如果建模的过程有问题，两个不同的状态有相同的 vector 表示，state-action 的价值函数就必然无法收敛到最佳值，也就表现不佳了。例如要建模一个钢琴按键的状态，如果只用 0 和 1 来表示其按下和抬起的状态，那么抬起到30%和按下到70%的状态，就可能重叠了，也就无法在这个状态时做出有区分性的动作了。

#### 动作空间影响采样效率

在我们建模时，需要说明有哪些动作是可以操作的。当动作空间越大时，每一次采样的效率也就越低。例如对鼠标操作进行建模时，如果以屏幕上的像素x，y 值作为可以放置的位置，那么可以移动的位置就会很灵活，动作空间就会很大。但是当智能体开始训练时，可能需要在百万次无效的操作中，才能有一次有效的操作。而如果我们把屏幕上的可点击区域作为离散的动作空间，那么动作空间就少得多，智能体在随机尝试的过程中也更容易获取触发有效操作。

#### 训练奖励的 shaping 设计需要领域知识

当训练奖励和训练的动作空间关联度不大时，比如要控制一个机械臂投篮，可能一开始让机械臂随机地尝试上百万次也不能投进一次。这时候如果只有投进篮筐才能获得奖励的话，智能体就一直无法获得奖励，算法也就无法提升智能体。

奖励函数的设计常会用到一个叫 **shaping** 的技巧，就是将奖励函数平滑、阶梯化，把离散稀疏的奖励变更成连续、密集的，使得智能体只要有一点提高就可以获得奖励。比如智能体只要大致朝篮筐抛出球就可以获得奖励，奖励数值根据与篮筐的距离越近则越大。这样一来智能体的策略收敛速度就会更快，我们更容易获得有意义的智能体。但是这个方法就需要很强的先验知识和领域知识，当在特定领域落地强化学习时，例如金融、电商等行业，就需要交叉型人才的参与才能有比较好的效果。

## 强化学习有趣的落地场景

书中也提到了一些有趣的落地场景，之前没想到还可以这样用，可以借鉴一下，开拓解决问题的思路：

### DRAM 储存系统的指令调度问题

当 CPU、DRAM 这些底层硬件的芯片要执行指令的时候，每个指令的耗时和顺序都不太一样，有时候灵活调整顺序不会影响最终结果，而且还能提高性能。以前很多工程师设计各种算法来优化这类算法。书中提到了一些利用强化学习算法来优化指令调度的工作，环境的输入就是代执行的指令队列，可选的动作空间就是底层的指令集，最终调度的结果就是让算法选择对应的指令并最大化吞吐量。结果证明这个效果是比传统算法更好的。

但是这个最终没有应用到芯片上，书中提到是算法烧录到硬件上的成本过高。

### 飞行器操控

在滑翔翼滑翔时，可以靠上升的热气流来提高高度，从而获取更多势能。那么怎么找到上升的热气流，并在热气流中保持盘旋，就是一个控制问题了。

除了通过流体力学的分析，并总结出一些算法之外，我们还可以通过强化学习 + 仿真模拟的方式来解决这个问题。在书中提到了通过编写仿真程序模拟1立方公里内的气流状态，从而提供一个智能体训练的环境，通过获取环境中温度、风速等信息作为输入，调整俯仰角作为可选的动作空间，构造了一个强化学习的模型并解决了这个问题。

## 其他的随笔

### 采样预估 vs 期望计算

在对一个状态的价值函数进行预估的时候，有两种办法：
  - 一种是采样预估，对未来可能发生的状态和采取的行动进行模拟，模拟很多次之后，对模拟的结果进行统计平均，这样就可以得到一个预估值。
  - 另一个是通过期望的计算，例如一个状态下可能发生变化的概率加权求和，利用其后续状态的价值函数的期望计算得到。

**这两种方案乍一看，感觉利用概率加权求和的方法会更好，因为利用概率计算可以避免采样时的偏差，但是在实际测试下发现并不是这样。**采样预估的方案在可以用计算机模拟的情况下，可以短时间采样很多次。例如抛硬币之类的活动，采样成本很低。而期望预估的过程中，需要对每个状态以及其子状态进行计算，当子状态的 fan out ratio 很大的情况下，例如每个状态有 100个子状态，那么计算4层就需要 10 的 8 次方个计算操作，计算的量非常大。但是采样4层，可能只需要10万次就可以有很高的精度了。

这个区别的一个原因是，采样的过程其实将计算资源加权分配了，对于更可能出现的状态，采样到的概率更大，也就分配了更多的采样计算资源。而期望计算中，不论期望值的大小，计算资源都是平均分配的。如果我们放弃进一步递归预估概率比较低的分支，可能期望计算的计算量也会更低。但是我们无法知道概率比较低的分支上会不会出现比较高的收益值，使得其就算概率低，期望值也并不低，所以这个优化方法也并不能使两个方案的计算量同步。

放在我们日常的生活中来看，这条经验可能就不适用。因为我们生活中的采样成本非常高，例如我们很难在短时间内尝试1000次上班的过程，但是我们可以很容易地分析出来哪个上班路线或者上班的方案是最好的。但有时分析难度太大时，试点可能是最好的办法。例如一个政策的结果可能受很多因素影响，每个因素影响的程度也不一样，单独分析无法面面俱到，找到最合适的方案。这时候在几个城市试点可能就是“采样”的过程，可以最快地看到最终效果。

### 学习未必需要奖励

强化学习从生物学中抽象出了一个 做出动作 -> 获取奖励 -> 更新学习 的流程。但是除了通过奖励来指引我们学习的方向，自然界中也有很多不以 reward 为目的的学习。例如我们看到一些意外的事情：树突然倒了，车突然震动，六月下雪。我们觉得惊讶的过程其实就是在预测和学习，当我们看到和经历的事情和我们预测的不同时，我们就会感到惊讶。

这种训练的方式和目前强化学习的建模并不相同，而与无监督学习更像一些。比如 copilot 的代码模型、BERT 的语言模型，都是给了一堆训练的素材，但是没有告诉算法要学习什么，算法自己在这堆素材里面寻找规律。当算法学到一些规律之后，再利用这些规律做进一步有用的事情，比如回答问题、补全代码、推荐搜索结果等等。

无监督学习也被证明了是许多学习任务的基石。例如基于无监督学习学到的语言模型，在经过有目的性地调优后，比单纯基于一个任务的训练集表现要好得多。这个思路在强化学习领域也是一样可以用运用的，AlphaGo 的模型中也使用了从人类棋谱中通过无监督学习初始化的价值函数，后续自己和自己下棋的过程，其实也类似于无监督学习的过程。

### 多巴胺是快乐的一阶导

神经学家在研究了脑补多巴胺的运作机制时，也发现了强化学习的 TD (Temporal-Difference) 模型可以很好地解释多巴胺的工作方式。这其中让我诧异的是：触发多巴胺分泌的根源，并不是因为人获得了当下的益处，而是**获得了超出预期的益处。**比如我们在第一次吃到巧克力的时候，大脑会大量分泌多巴胺，但是在后面吃巧克力的时候，每多次一次，就会少分泌一些，直到我们习惯吃巧克力后，就不会再分泌多巴胺了。这样的机制可以鼓励人在保持当前生活习惯的前提下，去追求更好的生活习惯。所以多巴胺并不仅仅是快乐，它其实是快乐的一阶导数，指引我们往更好的生活走。

这其实也解释了为什么毒品的戒断这么难。毒品的机制其实绕过了大脑皮层的许多机制，强行提高了多巴胺的水平。这其实相当于破坏了一个精密设计的仪器，原本是靠一滴滴水来平衡内外压强的，突然被一个巨大的水龙头给冲毁了。这个过程和强化学习中价值函数的发散也很像：当机制设计的有问题时，不断地采样训练不光无法将价值函数收敛到稳定的值，还会使得价值函数不断发散，就像吸毒久了的人，在他眼里毒品的价值就会不断提高，其他东西的价值就低了很多。

还很有意思的是，帕金森综合症其实也是由于多巴胺分泌不足。当多巴胺分泌不足时，没有了足够的刺激，大脑也会慢慢退化，记忆力和智力都会慢慢下降。

### 神经元的背景噪音

我们知道机器学习中的神经网络是根据人的神经元结构模仿得来的。在书中介绍到神经元本身是有一个背景噪音的，就是在没有被激活的时候，也会有一些类似噪声的电信号。那么为什么要有这个噪声电信号呢？如果用机器学习的模拟角度来看就容易理解了，在神经网络中往往用一个实数来表示电信号，实数可以是正数也可以是负数。在人脑中，神经元的电信号却只能是正数，为了能像神经网络一样表示负数，或者表达负反馈，神经元就有了一个背景噪音。当神经元要表达负反馈时，降低背景的电信号噪音，就表达出了类似负数的含义。







