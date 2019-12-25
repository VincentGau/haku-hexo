---
title: 飞花令助手
date: 2019-12-25 14:15:47
tags:
- 诗词
categories:
- 诗词
- 飞花令
---


曾有一个小小的黑色本子，记录了高中时期痴迷的诗词，还夹有已经干枯的叶子和花，仿佛给墨迹添了一圈香气。我把它视若珍宝，无奈终于在一次意外中遗失，一起遗失的还有一叠名为「遗忘」的信纸，至今耿耿于怀。

近来尤其喜欢苏轼的《临江仙·夜归临皋》，从前最喜欢的是这首词结尾两句：「小舟从此逝，江海寄余生」，现在重读却更为「家童鼻息已雷鸣，敲门都不应，倚杖听江声」句倾倒，真像极了辛帅写的「少年不识愁滋味，爱上层楼。爱上层楼，为赋新词强说愁」，从前哪里懂得「小舟从此逝，江海寄余生」的境界，只是被其中洒脱的仙气感染，觉得读上两句就能像向苏子一样超凡脱俗。

看过《中国诗词大会》和《中华好诗词》，尤其喜欢大学季恰同学少年专题节目，酣畅淋漓。后来偶然知道除了飞花令，还有单双飞花令，射覆飞花令，接龙各种诗词游戏玩法，和一群达人玩过一段时间后，偶尔会在一些诗句上卡壳，恰好前段时间在网上抓了一些诗词数据，一时兴起就想写一个自动生成双飞花和射覆游戏的小程序，实现了在群里自动飞花的程序。

{% asset_img demo.jpg %}

# 素材准备
首先准备好诗词素材，这里我用的是从互联网上抓取的2万余作者信息的50余万篇诗词文赋，囊括了从古至今绝大部分的作者和作品。由于50多万作品中有很大一部分对我们是陌生的，因此我对它进行了筛选，尽量选择大家熟悉的作品，最终选定了14175篇放入素材池。

# 接口开发
使用Springboot开发Restful接口，通过docker部署到云服务器，接口调用方式如下：
`http://tc.hakucc.com/api/shuangfei?clue=萧瑟秋风今又是 换了人间`   
`http://tc.hakucc.com/api/shefu?clue=月明人倚楼 云7`

# 自动回复
由于QQBot所依赖的SmartQQ协议停用，最后选择酷Q作为自动回复的工具。酷Q使用教程[点此](酷Q SDK-X)，配置完成后，编写`CQPlusHandler.py`如下：

```python
# -*- coding:utf-8 -*-

import cqplus
import requests
import random

    
class MainHandler(cqplus.CQPlusHandler):
    def handle_event(self, event, params):
        if event == 'on_private_msg':
            self.OnEvent_PrivateMsg(params)
        elif event == 'on_group_msg':
            self.OnEvent_GroupMsg(params)

    def OnEvent_PrivateMsg(self, params):        
        msg = params['msg']
        self.api.send_private_msg(params['from_qq'],msg)

    def OnEvent_GroupMsg(self,params):
        # 只针对特定的群
        if params['from_group'] in [642540069, 790583076]:
            result = self.shuangfei(params['msg'])
            if result != '':
                self.api.send_group_msg(params['from_group'], result)

    def shuangfei(self, clue):
        url = r'http://tc.hakucc.com/api/shuangfei?clue=' + clue
        r = requests.get(url)
        if r.json()['code'] == '0000':
            return r.json()['data'][random.randint(0, 20)]
        else:
            return ''
```

完成后无需执行`CQPlusHandler.py`，在酷Q中启用该应用即可。

参考
[酷Q SDK-X](https://gitee.com/muxiaofei/coolq_sdk_x/wikis/pages)