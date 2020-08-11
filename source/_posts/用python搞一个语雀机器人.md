---
title: 用python搞一个语雀机器人
date: 2019-03-19 21:16:12
tags:
- python
categories: 
- python
---
<!-- more -->
# 用python搞一个语雀机器人

这其实是个愚蠢的事情。因为钉钉机器人是有webhook的，只要在语雀知识库绑定一下就能获取更新推送。<br />![image.png](\uploads\lark.jpg)<br />咳咳。。但是我~~裤子都脱了~~（环境都装上了），总得搞点事情出来吧？

所以。。我试了试API，输出文章列表。[笑哭]

```python
import requests
import json

author = {
    'WH':'https://yuque.antfin-inc.com/api/v2/repos/kd71e4/dg9s97/docs',
}

robot_hook = 'https://oapi.dingtalk.com/robot/send?access_token=b6428f'
lark_headers = {'X-Auth-Token': 'wVzEauX5Sh5ZK76hoaO58as','User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.11 (KHTML, like Gecko) Chrome/23.0.1271.64 Safari/537.11'}
robot_headers = {'Content-Type':'application/json'}
robot_payload = {"msgtype": "text", "text": {"content": "emmmm.."}}

article_pool = {}

for mate in author:
    r = requests.get(author[mate], headers=lark_headers)
    art_obj = json.loads(r.text)
    article_pool[mate] = []
    
    for article in art_obj['data']:
        article_pool[mate].append(article['title'])

print(article_pool)

robot_payload["text"]['content'] = article_pool
p = requests.post(robot_hook, data=json.dumps(robot_payload), headers=robot_headers)

print(p.text)

```

emmmm。。效果还是有的。

```python
{'WH': ['DDoS高防架构浅析', '流量清洗', 'DDoS攻击的类型和防御', '关于luban+umi框架融合的思考']}
{"errmsg":"ok","errcode":0}
```

虽然并没有什么卵用。等我想到什么卵用再加功能吧。