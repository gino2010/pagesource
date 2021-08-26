---
title: Haskell Study 2
date: 2021-08-15 12:32:40
tags: Haskell
categories: tech
---

继续上次阅读进度，进行后续内容的学习
感觉学习速度明显减慢，后面的内容难度上来了，开始变得复杂需要更多的实验来理解

本来此文章15号就创建了，以为2周内可以看完的书，结果又多读了一周多，本周二（24号）终于看完了一边
花了3周多的时间终于看完了 [Learn You a Haskell for Great Good!](http://learnyouahaskell.com/) ，中间有两章看的十分慢，因为概念有点涩，难懂，还有2～3点没有理解的十分清楚
虽然已经看完了一边，书中代码基本也都敲了一遍，但还是感觉有点云里雾里的，囫囵吞枣，很多细节还需要通过实践进行深入，还在等待中......

先说说我的整体感觉吧，不一定准确，仅供参考
<!-- more -->
Haskell 的语言发展背景我没有太多了解，直观感觉是一帮计算机或数学方面的领域专家设计的，因为如下特点：
1. 纯函数式编程思路，复杂的嵌套调用规则，类似数学公式一般的推到过程。（这也是我第一次在写基础练习时，为了搞清楚代码调用关系，使用了草稿纸进行规则公式般的推导，让我想起了高等数学）
2. 类似高等数学公式一样的书写方式 () $ >>= :- -> 等等，还让我第一次知道了 [Reverse Polish Notation](https://en.wikipedia.org/wiki/Reverse_Polish_notation)
3. 大量的递归操作思路，不仅仅在逻辑实现上，也在类型定义和方法实现上，和传统解决思路有很大的区别
```haskell
data Tree a = Empty | Node a (Tree a) (Tree a) deriving (Show) 

data Direction = L | R deriving (Show)  
type Directions = [Direction]  
  
changeToP :: Directions-> Tree Char -> Tree Char  
changeToP (L:ds) (Node x l r) = Node x (changeToP ds l) r  
changeToP (R:ds) (Node x l r) = Node x l (changeToP ds r)  
changeToP [] (Node _ l r) = Node 'P' l r 
```
4. 和传统编程语言相比，除了递归，当你在haskell中遍历操作一个数据集合，很少看到传统的Loop循环操作
```haskell
map sum $ transpose [[0,3,5,9],[10,0,0,9],[8,5,1,-1]] 
```
5. 一个逻辑多种实现途径，至于选哪个？呵呵，我可能记不全，也没有太好的感觉，还需要磨练一下
6. 很多易与传统编程混淆的概念，例如 data type newtype class instance，千万不要用传统语言直接理解他们，和你知道的不一样

## 书中问题
[Learn You a Haskell for Great Good!](http://learnyouahaskell.com/) 中还是有不少需要纠正的地方，猜测Haskell版本更新导致了一些变化，本书并没有及时更新。
（为何不把它share到wiki上，大家一起维护？💡）

前几章中应该也有几个小问题，我没有记录。下面列举的问题，是从 Input and Output 章节开始记录
1. 使用random，是需要安装 random package 的，cabal install --lib random
2. random (mkStdGen 100)  show内容和书中已经不一样了
3. random reads 我没有能正常调用它，如何fix还需要再看看
4. B.pack [99,104,105] 展示内容不再包含empty
5. 异常 catch 需要导入 import Control.Exception.Base
6. vowels 这个方法我没有找到，不知道如何使用
```haskell
import Data.Monoid  
  
lengthCompare :: String -> String -> Ordering  
lengthCompare x y = (length x `compare` length y) `mappend`  
                    (vowels x `compare` vowels y) `mappend`  
                    (x `compare` y)  
    where vowels = length . filter (`elem` "aeiou") 
```
7. fail 没有找到
```haskell
instance Monad Prob where  
   return x = Prob [(x,1%1)]  
   m >>= f = flatten (fmap f m)  
   fail _ = Prob []
```
8. Monad包中的内容发生改变，原来import Control.Monad.Writer，现在需要import Control.Monad.Trans.Writer
9. Error 类型已经废弃，替换使用Except import Control.Monad.Trans.Except

书中还有几处的代码可以优化的，并没有特别记录下来，稍后可以分享一下我敲的书中实例代码，哪里有。

## 开发体验
* visual studio code + Haskell 插件，这个组合还是很给力的，语法高亮，代码优化推荐，quick fix提醒，快速查看文档，对于我这个初学者受益匪浅
* https://hoogle.haskell.org/ 这个网站很重要，可以查询方法使用，部分包含实例，也可以定位源码，便于深入理解
* Idea + Haskell 插件和 VS Code 相比差了很多，暂不推荐
* ghci中常用指令 :m :l :t :k :i :q 大量时间依赖他们去理解代码
* 编译指令和解释运行指令
```shell
ghc --make <file>.hs
runhaskell <file>.hs
```
* 至今不会运行时的debug，如何解？

## JVM 运行 Haskell
你想在JVM中运行 Haskel 语法的代码吗？ 找到两个利用JVM运行Haskell代码的方式
* [ETA](https://eta-lang.org/) 通过构建说明，无法加载eta资源，没有运行成功。如有需要，稍后再尝试了
* [Frege](https://github.com/Frege/frege) 这个搭建成功，成功运行Haskell示例代码，并且它还有一个还在更新的idea插件支持，感觉还不错

感觉如果今后要是想把Haskell迁移到JVM环境，这是个思路。 两个资源都是BSD 3-clause开源协议

亦或是学习一下[scala](https://www.scala-lang.org/) ?直接迁移，谁知道呢，知识尚浅，稍后再比较了
P.S.目前的我，如果不写Java，还是更喜欢写Python，呵呵。

## 后话
从网上找了两本Haskell编程的书，2019年出版的还是很新的，其中应用场景为金融数据分析，和我后面要做的事情应该比较接近。
已经小一个月了，后续的事情还没有确定，好事多磨，好事多磨啊～ 希望后续一切顺利吧
确定后，会开始深入Haskell的一个具体应用。

最近工作和学习内容一起忙乎，并且连续两个周末都在家学习，有点疲惫。
这几天没什么事情的话，就只想好好提升一下英语，计划周末去听一个2个小时的英语专场，去看Ryan Reynolds 演的《Free Guy》😛

这个周末对我很重要，很重要～～～～

## 题外话
个人投资的部分基金回撤严重，这次算是学到了一些，选择的行业，问题还不是很大，只好耐心操作，补仓拉低成本，等待回调

收～～期待一个普通又不普通的周末