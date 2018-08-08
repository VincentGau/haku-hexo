---
title: 从零开始写五子棋AI
tags:
- alpha-bate pruning
- AI
---

在学校的时候上王老师的人工智能课，了解博弈树、启发式搜索、剪枝这些基本概念，很遗憾在当时没有完成五子棋AI。最近和X 下五子棋的时候突然又脑子一热，于是有了这篇博文和程序。程序源码：https://github.com/VincentGau/TicTacToe 包含井字棋和五子棋代码；

五子棋是一个博弈过程，双方都能获取局面的完备信息，交替行动，最终一人获胜或达到平局，典型的零和博弈。对于这类问题适合使用极大极小值搜索来求解。极大极小值搜索是实现这个五子棋AI的基础。

# 极大极小值搜索
搜索过程的本质是回溯，以深度优先方式遍历整棵树。
对于一个局面，一方有m 种可能的落子法，对于它的任何一种落子情况，对方都有对应的n 种落子，如此轮流行动，以博弈树形式表示，树的每一层代表一方，双方在树中交替出现。极大极小值搜索过程中，每一步假设对方也都按照它的最佳方案行动；一方要在可选的选项中选择将其优势最大化的选择，另一方则选择令对手优势最小化的方法。我们将博弈树中的层分成Max 层和Min 层，为了保证自己的收益最大化，在Max层始终选择子节点中值最大的点，Min层则相反，始终选择值最小的子节点；因此被称为Minimax 算法；

{% asset_img minimax.png %}

图中，方块和圆圈分别代表博弈双方，现在节点1有两种选择，它应该做出怎样的选择，就是Minimax 算法要解决的问题。  
节点1 处于Max 层，它要选择2、3 中评价值最高的那一个；节点2 处于Min 层，它取值为所有子节点中最小的值，也就是节点5，所以节点2 的值为 3；同理，节点3 的值为1； 节点1 的值为3；所以节点1 应该选择第一种下法；虽然第三层四个节点中，评价值最大的是节点7，但是如果节点1 选择第二种方案，到达节点3，到对方轮次，此时决定权已经到对方手中，为了使自己的利益最大化，对方会选择到节点6；

AI的目的是选择一处收益最高的位置落子；这里如何判断一个落子的好坏呢，我们需要自己设定一个评判标准，这就是后面我们会提到的`评价函数`。


了解了Minimax 算法之后，我们可以想象，随着搜索深度的增加，需要计算的节点呈指数增长，虽然相较围棋、象棋等复杂度较低，但是我们想要穷举搜索这颗博弈树的所有叶子结点（最终一方获胜或平局）依然是不可行的，因此我们需要对此做一个改进，这里的优化有两个思路，一个是设定搜索深度，这个深度即AI思索的步数，另一个是缩小可行落子点的范围，对于一些明显不适合的落点我们不进行扩展；
假定棋盘大小为15*15，

# alpha-beta 剪枝
## 理解α-β剪枝
alpha-beta 剪枝是在Minimax 算法基础上的优化，避免扩展哪些不会被选择的节点，减少无效操作从而提高搜索效率。继续使用前面的例子来理解剪枝的过程。

{% asset_img minimax.png %}

我们扩展节点2，得到它的值为3；继续扩展节点3，因为节点3 处于Min 层，且节点6 的值为1，所以节点3 的值不可能大于1， 因此不论节点3 后面的子节点的值是多少，节点1 一定会选择到节点2，在得知节点6 的值为1 之后，节点3 到节点7 这条分支可以直接忽略，这个过程称为剪枝；

这里只对剪枝做了大概的理解，更多参见[α-β剪枝](http://web.cs.ucla.edu/~rosen/161/notes/alphabeta.html)

## 算法实现

```C#
public int minimax()
{
    
    // Max, computer's turn 
    if (turn == 0)
    {
        int maxInChild = int.MinValue;
        foreach (var move in availableMoves)
        {
            int childScore;
            BoardState childState = new BoardState(newBoard(move, Helper.AIMark), changeTurn(turn), depth + 1, alpha, beta);
            if (depth >= Helper.searchDepth || checkFinished(childState.board).Count == 5)
            {
                childScore = evaluateBoard(childState.board);

                nextState = childState;
            }
            else
            {
                childScore = childState.minimax();
            }


            if (childScore > maxInChild)
            {
                maxInChild = childScore;
                alpha = maxInChild;
                nextState = childState;
                nextMove = move;
            }

            if (alpha >= beta)
                break;
        }
        return maxInChild;
    }

    //Min, player's turn
    else if (turn == 1)
    {
        int minInChild = int.MaxValue;
        foreach (var move in availableMoves)
        {
            BoardState childState = new BoardState(newBoard(move, Helper.playerMark), changeTurn(turn), depth + 1, alpha, beta);
            int childScore;
            if (depth >= Helper.searchDepth || checkFinished(childState.board).Count == 5)
            {
                childScore = evaluateBoard(childState.board);
                
                nextState = childState;
            }

            else
            {
                childScore = childState.minimax();
            }


            if (childScore < minInChild)
            {
                minInChild = childScore;
                beta = minInChild;
                nextState = childState;
                nextMove = move;
            }

            if (alpha >= beta)
                break;
        }

        return minInChild;
    }

    return 0;

}
```

# 评价函数

# 算法优化

## 缩小候选点范围
五子棋盘有 15 * 15个点，我们不可能对每种可能的落子情况都进行扩展，这些点中有一些明显不符合最优解，比如边角处，远离棋子圈处，我们可以通过缩小候选落子点范围来提升效率。在实践中，我选择那些周围有棋子存在的空位作为可能的落子点，比如棋盘上对方下了第一步，我们只考虑它周围一圈的八个点，因为只有这八个点周围有棋子（棋盘上的唯一一颗棋子），这样极大第减少了需要扩展的点，这样选取候选点没有经过科学验证，虽然可能失去潜在的最佳方案，但是直觉上落子在远离其他棋子的地方不会获得太多收益，
## 候选点排序



五子棋盘有15 * 15 个落子点，这些点有一些明显不符合最优解，比如边角处，远离棋子圈处，缩小候选落子点范围能有效提升处理速度和效率；通常棋子圈附近是落子的好选择；刚开局时落子；
候选落子位置根据已落子点周围一圈时确定，搜索深度为四层时，时间过长，只计算到达第四层的盘面估值和已经分出胜负的盘面估值；即使一步即可获胜，也需要等到前面的节点完成深度搜索；可以尝试对第一层进行估值，使用最大堆保存估值较高的K 个节点进行深度搜索，可更快发现一步获胜局面；

棋型：
连五：出现连续五个同色棋子；
活四，有两点可以连五；
冲四：有一点可以连五；
活三：可以形成活四的连三；
眠三：只能形成冲四的连三；
活二：可以形成活三的连二；
眠二：只能形成眠三的连二；

评价函数是影响决策的关键：
是对整个盘面进行评估还是对单个落子点进行评估？ 评估是否应该与轮次挂钩？  考虑一种情况：黑子（max）形成冲四，如果是黑子轮，黑子下在第五个延伸位置即可获得连五的分数，如果是白子轮，它下在第五个延伸位置对自己成五可能并没有收益，但是它还是应该下在这个位置防止对方获胜；评价函数用二者之差来表示，博弈树中黑子作为max层追求最大的估值，白子作为min层追求最小的估值，黑子落第五个位置可获得10000分，假设白子落此位置可获得 -10 分（白子连五获得-10000分），

剪枝算法的效率依赖于着法的寻找顺序。如果总是先去搜索最坏的着法，那么剪枝就不会发生，最终会找遍整个博弈树，这时该算法就如同极大极小算法一样，效率非常低。
