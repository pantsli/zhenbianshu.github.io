---
layout: post
title: "聊聊单元测试"
date: 2019-10-09 10:00:06 +0800
category: blog
tags: [Unit Test, Code]
comments: true
---
## 单元测试
---
写的代码能一次正确执行是每个程序员的追求，但世事皆不能尽如人意，我们的代码经常会有 Bug，这就需要测试的存在。

测试有黑盒和白盒之分。黑盒测试，测试时认为被测程序就像一个漆黑的盒子，虽然不明白其中的运行原理，但知道怎么输入有对应的输出。QA (Quality assurance)，也就是我们的测试部门一般负责对程序进行黑盒测试，调用接口时传确定的参数，再校验接口响应值符合某种预期。

与黑盒测试对应的是白盒测试，白盒测试要求被测试人员了解被测程序的构造，从而构造测试用例校验程序各个分支逻辑。从这一方面来说，单元测试就是一种白盒测试。

单元测试，又称为模块测试，是针对程序模块（软件设计的最小单位）来进行正确性检验的测试工作，一般对面向对象语言来说，这个最小单位是类或重要的类方法，它不仅可以用作功能测试，将单元测试集成到依赖集成工具之后，它们还可以在模块编译时执行，进行模块的回归测试。

工作以来，我对单元测试的认知不断刷新，从拒绝到勉强接受，再到理解其中妙处，最近竟然有些理解我以前觉得不可思议的 TDD（测试驱动开发）了，我觉得有必要把其中心路历程整理一下，无论是之后回忆还是翻出来打脸，应该都很有意思。      

{{ site.article.copyright }}                                       

## 初遇单测
---
#### 不写单测
刚开始工作时的公司是一个小型公司，项目小且业务简单，部门也没有要求，我是不写单元测试的。

不接触就无法理解它，这时候我对单元测试的认识就是收益很低的"测试工具"，认为代码是自己写的，自己再编造一些数据去迎合这些代码，根本测试不出来什么，而且编造数据还要花费大量的尽力，收益和付出完全不成正比，程序的正确性靠 QA 就完全够了。

可能跟我有类似想法的同事也有很多，后来我们干脆把所有的类方法都改成了静态方法，程序运行时不用再去创建服务对象了，这样，代码就变成了披着面向对象外衣的函数式程序。

当然，这也进一步导致了单元测试不可能实行了，因为方法是层层调用的，想要构造出一组能正确运行的数据都非常困难，就更不用说再测试各种分支逻辑了。

#### 随波逐流
后来换到了目前所在的岗位，部门强制要求每段逻辑都要有对应的单测 case，这样我才真正接触到了单元测试。刚开始写单元测试时内心是非常煎熬的，不认同还要被强制要求做，当然很不爽，更重要的是不习惯。

原来功能开发完成后，自己改改参数调两遍接口测下就行，现在还要再写同样多甚至更多的代码来校验功能代码是否正确，花费的不仅是时间，还有心力。而且有些代码就没法写单测，费尽心思构造出数据，可能还没测出功能代码的问题，改单测 case 的 bug 就能让人崩溃了。

这时我采用的策略就是仿着原来的单测 case 写，现在看来之前的很多单测 case 也没什么质量，又由于自己水平低，导致最终写出来的单测 case 基本没什么意义，要么是重要逻辑覆盖不到，要么是只能覆盖一些通用逻辑，就算有问题，调接口测一下也能测出来。

收获也不能说完全没有，毕竟有非常明显的 bug 还是能够测出来的，而且有时候误改了之前的代码，也能够在 QA 反馈前及时解决，但总体来说，这样写单元测试是不划算的。

## 单测的意义
---
#### 缘由
后来 case 越写越多，在越来越熟练地满足单测覆盖的要求时，我也在不停思考这样的工作有什么意义，直到有一天被 leader review 代码，我感觉有些开悟了。

被 review 代码的功能是将一个 json 字符串解析为服务里的配置模型，考虑到它只是一个解析字符串的功能，我把它定义为一个"工具类"，里面用静态方法实现，这样调用解析方法时不用注入 bean，使用类名.方法即可。

我以为这个类虽然称不上完美无缺，但命名、日志、异常处理都很完善，至少算得上中规中矩，没想到被 leader 一顿狠批，因为他直接跳到这个工具类的上层类，发现这个类没有单测覆盖。

为什么上层类没有写单元测试呢，不是因为上层的逻辑太过复杂，而且因为如果我想测这个上层类，就需要构造出一个能够解析为配置模型的大型字符串，还要传上一堆配置参数到这个解析工具类里，这个字符串不好构造不说，即使构造出来了，写在代码里这么一堆，也异常难看，如果之后要变更这个配置模型的结构，更是一场噩梦。

#### 思考
被教育一顿后，我终于明白了，上层代码的单元测试难写，是因为这么一个工具类，工具类的静态方法无法 Mock 返回值，这就需要我构造大量的真实数据，费力也讨不了好，简而言之，是因为我的设计烂导致单测不好写。

接着我观察了更多据说写不出单测的代码，反思了之前遇到的单测难写的代码，拿这些代码去对比代码设计原则，发现了这些代码的一个共同点：它们都严重违背了一种或更多设计原则，也就是说，它们都是烂代码。"不好写单测的代码都是烂代码"，我觉得我理解了单元测试的（部分）意义，单元测试不仅用来测试代码功能，还可以用来测试代码设计。

从此之后，我开始更重视单元测试了，单元测试的名字不再用 "testMethodName" 这么敷衍的名字，也开始考虑设计单测的边界值，每次写单测时也在不停问自己，这个 case 写起来费劲吗，我的设计合理吗？终于，单测对我来说不再只是负担了。

## TDD 的思考
---
TDD，测试驱动开发，是一种先写单元测试再根据单元测试写功能代码的开发模式。一直以来都觉得这种开发模式很不可思议，在自己都不知道类和方法怎么拆分时，怎么能写出单元测试呢，就算强行写出了，可是结果又跟 QA 写的测试 case 有什么区别呢？都是假想有这么一个黑盒能够对某种输出响应某种输出。

在理解了单测的意义后，再往深处去想，TDD 是十分可行的。首先对于良好的代码设计，恰当的功能拆分来说，要测试的模块是基本确定的，这就提供了先写单元测试的可能性，而且这种开发方式也能有效避免业务代码开发完成后，写单元测试时发现设计不可理的窘境。

虽然我不熟悉 TDD，也没有想真正实践 TDD 的开发模式，但 TDD 也能给我一些启发。我不会在功能开发完成前写单元测试，但我可以在进行代码设计前先考虑单测 case 的结构，或者先预定单测 case 的方法，功能开发完后再补充单测的方法体，这也是 TDD 的另一种实践方式吧。

## 小结
---
现在能明白为什么 leader 一直要求大家代码必须补充单测了，但盲目地写收益真的很小，思想上不认同的话，懒惰小人也会不停地跳出来阻止你写单元测试。

如果你也跟我之前一样挣扎在单测的苦海，那你可以重新思考一下单测的意义，如果你是充满"灵性"的程序员，对我的看法有意义的话，欢迎评论。

{{ site.article.summary }}