---
layout: post
title: "在线广告中的cookie matching"
date: 2015-02-12 13:45:00 +0800
comments: true
categories: ads
---

用户定向是在线广告的核心优势之一，数据是用户定向的基础，而cookie matching技术可以将用户在各个站点上的数据关联在一起，使得re-targeting成为可能。

cookie matching有很多的应用场景，典型的有2种，一种是在DMP(Data Management Platform)生态中，另一种是在RTB(Real-time bidding)中。下面介绍下在这2种场景中cookie matching是如何实现的。

![](http://dongguo.me/images/personal/ads/cookie_matching_DMP.png)

1. 用户U访问jd.com, jd从用户browser中获取jd_cookie_id(jd.com的cookies id);
2. jd的页面中预先嵌入了BlueKai的js脚本，会有一个302重定向请求转发给BlueKai, 用户的browser中会生成BlueKai的cookies,同时用户的jd_cookie_id会被发送给BlueKai;
3. BlueKai在其后端service中纪录下BlueKai_cookie_id和jd_cookie_id的映射关系
4. 用户U某一次去了yahoo.com浏览新闻，假设事先yahoo和jd签了一笔重定向的广告订单
5. yahoo的ad server在给用户U挑选广告前，访问BlueKai server，BlueKai会在其数据库中检索Bluekai_cookie_id对应了哪些站点的cookie_id
6. BlueKai给yahoo ad server返回用户U的tags（包含了哪些站点的cookie_id），如果其中包含了jd_cookie_id，则jd的广告可能会播放给该用户看



![](http://dongguo.me/images/personal/ads/cookie_matching_RTB.png)

1. 用户U访问jd.com, jd用从户browser中获取jd_cookie_id(jd.com的cookies id);
2. jd的页面预先嵌入了PinYou的脚本，同样的会为BlueKai生成cookie，同时请求Pinyou分配cookie mapping任务;
3. Pinyou给jd返回一个beacon，其中包含ad exchange地址，和用户U的Pinyou_cookie_id;
4. jd会通过该beacon向DoubleClick发送cookie matching请求，包含了pinyou_cookie_id;
5. doubleclick通过302重定向向Pinyou发送doubleclick_cookie_id;
6. Pinyou在其数据库中存储doubleclick_cookie_id和pinyou_cookie_id的映射关系；
7. 用户U某一次去yahoo.com浏览新闻，yahoo事先接入了double click广告平台售卖广告；
8. yahoo的ad server会向double click发送广告请求，double click会将用户U的doubleclick_cookie_id发送给Pinyou等DSP, Pinyou通过cookie matching数据库找到pinyou_cookie_id, 再检查其对应了哪些站点的cookie_id，如果包含了jd_cookie_id，Pinyou就可能会为jd的广告竞争该广告位
9. double click返回挑中的广告让yahoo播放