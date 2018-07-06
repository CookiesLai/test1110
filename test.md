<!-- TOC -->

- [DMP流程测试](#dmp)
    - [一.lps收数统计](#lps)
        - [1.分类型前的处理](#1)
        - [2.utmId对应的redis数据](#2utmidredis)
        - [3.LPS_LandingView](#3lps-landingview)
            - [(1).查重](#1)
            - [(2) 有utmId，入库lps_main td_lps td_main](#2-utmidlps-main-td-lps-td-main)
                - [[1].url参数：tp\nid\utm](#1urltpnidutm)
                - [[2].url参数：tp\nid\utm\uid\lid\v](#2urltpnidutmuidlidv)
            - [(3)无utmid, 入库 lps_main td_main](#3utmid---lps-main-td-main)
                - [[1].url参数：tp\nid](#1urltpnid)
                - [[2].url参数：tp\nid\uid\lid\v](#2urltpniduidlidv)
        - [4.LPS_LandingStay](#4lps-landingstay)
            - [(1) 有utmId，入库lps_main td_lps](#1-utmidlps-main-td-lps)
                - [[1].url参数：tp\utm\st](#1urltputmst)
                - [[2].url参数：tp\utm\st\uid\lid\v](#2urltputmstuidlidv)
            - [(3)无utmid, 入库 lps_main](#3utmid---lps-main)
                - [[1].url参数：tp\st](#1urltpst)
                - [[2].url参数：tp\nid\st\uid\lid\v](#2urltpnidstuidlidv)
        - [5.LPS_LandingLinkClick](#5lps-landinglinkclick)
            - [(1).查重](#1)
            - [(2) 有utmId，入库lps_main td_lps](#2-utmidlps-main-td-lps)
                - [[1].url参数：tp\nid\utm](#1urltpnidutm)
                - [[2].url参数：tp\nid\utm\uid\lid\v](#2urltpnidutmuidlidv)
            - [(3)无utmid, 入库 lps_main](#3utmid---lps-main)
                - [[1].url参数：tp\nid](#1urltpnid)
                - [[2].url参数：tp\nid\uid\lid\v](#2urltpniduidlidv)
        - [6.LPS_PointerClick](#6lps-pointerclick)
            - [(1).查重](#1)
            - [(2) 有utmId，入库lps_main td_lps](#2-utmidlps-main-td-lps)
                - [[1].url参数：tp\nid\utm\en](#1urltpnidutmen)
                - [[2].url参数：tp\nid\utm\uid\lid\v\en](#2urltpnidutmuidlidven)
            - [(3)无utmid, 入库 lps_main](#3utmid---lps-main)
                - [[1].url参数：tp\nid\en](#1urltpniden)
                - [[2].url参数：tp\nid\uid\lid\v\en](#2urltpniduidlidven)
        - [7.LPS_PointerVideo](#7lps-pointervideo)
            - [(1).查重](#1)
            - [(2) 有utmId，入库lps_main td_lps](#2-utmidlps-main-td-lps)
                - [[1].url参数：tp\nid\utm\en](#1urltpnidutmen)
                - [[2].url参数：tp\nid\utm\en\uid\lid\v](#2urltpnidutmenuidlidv)
            - [(3)无utmid, 入库 lps_main](#3utmid---lps-main)
                - [[1].url参数：tp\nid\en](#1urltpniden)
                - [[2].url参数：tp\nid\en\uid\lid\v](#2urltpnidenuidlidv)
        - [8.LPS_PointerVideoStay](#8lps-pointervideostay)
            - [(1) 有utmId，入库lps_main td_lps](#1-utmidlps-main-td-lps)
                - [[1].url参数：tp\utm\en\vdt](#1urltputmenvdt)
                - [[2].url参数：tp\utm\en\vdt\uid\lid\v](#2urltputmenvdtuidlidv)
            - [(2)无utmid, 入库 lps_main](#2utmid---lps-main)
                - [[1].url参数：tp\en\vdt](#1urltpenvdt)
                - [[2].url参数：tp\en\vdt\uid\lid\v](#2urltpenvdtuidlidv)
        - [9.LPS_JSTransfer js脚步调用](#9lps-jstransfer-js)
            - [(1).处理逻辑](#1)
            - [(2).url参数：tp\en\utm](#2urltpenutm)
    - [二.td统计](#td)
        - [1.bi (TD_Bi)](#1bi-td-bi)
            - [(1).HandleMsg日志](#1handlemsg)
            - [(2) TDLog](#2-tdlog)
            - [(3).入库td_lps td_main](#3td-lps-td-main)
        - [2. 业务数据 (TD_Business)](#2--td-business)
            - [(1).TDBusinessMsg日志](#1tdbusinessmsg)
            - [(2) TDLog](#2-tdlog)
            - [(3).入库td_main](#3td-main)
        - [3.行为日志(TD_Clicks)](#3td-clicks)
            - [(1).td日志](#1td)
            - [(2).入库td_main](#2td-main)

<!-- /TOC -->

# DMP流程测试
## 一.lps收数统计
### 1.分类型前的处理

(1). 检查参数合法性

- 类型1 LPS_LandingView无参数检查
- 类型2 LPS_LandingStay： StayTime == 0 ，不处理
- 类型3 LPS_LandingLinkClick无参数检查
- 类型4 LPS_PointerClick： EventName == ""，不处理
- 类型5 LPS_PointerVideo： EventName == ""，不处理
- 类型6 LPS_PointerVideoStay： EventName == "" || EventDurationTime==0，不处理

(2). 从请求里获取uuid(对应cookie的utm_uuid) ：如果有uuid，则isNewUUID = false。如果没有uuid，则生成uuid，isNewUUID = true。

(3). 获取UtmId：先判断url里是否有utm，如果无，就从请求里获取（对应cookie的 __lps__utm_id）

(4). 无UtmId：从请求里获取ip 、ua  

(5). 有UtmId：
- 根据td:utm_id:utmid，从redis获取信息；redis没有对应数据，结束收数处理。
- isNewUUID == true，且redis里有Uuid：则把它赋值给dim.Uuid 
- 从请求里获取ip 、ua；
- 检查ip、ua：和redis里的信息是否一致，如不一致，且redis数据有utm_id值：则用set_ip接口保存ip、ua

(6). setCookies (对应cookie的utm_uuid: Uuid) 

(7). 用请求里的数据替换掉redis读取的数据：

  paramMap.LPSUserId = dim.UserId

  paramMap.LPSLandingPageId = dim.LandingPageId

  paramMap.LPSVersion = dim.Version

(8).埋点映射

  根据eventName + lpsLandingPageId 生成 key ,将key、eventName 保存到MYSQL redis

(9).下面按类型处理

### 2.utmId对应的redis数据

```json
{
    "business_id": 88001,
    "campaign_id": 88002,
    "channel_id": 88003,
    "creative_id": 88004,
    "date": 20180704,
    "dmp_user_id": 0,
    "hour": 9,
    "ip": "192.168.9.182",
    "landing_page_id": 88005,
    "lps_id": 88006,
    "lps_user_id": 88007,
    "lps_version": 88008,
    "medium_id": 88009,
    "plan_id": 88010,
    "referer": null,
    "spot_id": 88011,
    "targeting_id": 88012,
    "tda_user_id": 88013,
    "ts": "1530668431",
    "user_agent": "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.79 Safari/537.36",
    "user_id": 88014,
    "utm_id": "1530668431031d862af9f81a",
    "utm_source": null,
    "uuid": "15301532238979cdd8c4f8fd"
}

```

### 3.LPS_LandingView
#### (1).查重
#### (2) 有utmId，入库lps_main td_lps  td_main
##### [1].url参数：tp\nid\utm

- url: http://127.0.0.1:9009/lps/a.gif?tp=1&nid=test&utm=1530668431031d862af9f81a

- lps日志

```json
{
    "log_type":1,
    "hash":"",
    "user_id":88007,
    "lps_landing_page_id":88006,
    "version":88008,
    "date":20180704,//接收请求的日期
    "hour":10, //接收请求的时间
    "ts":1530670760,//接收请求的时间
    "event_id":0,
    "event_type_id":0,
    "uuid":"15301532238979cdd8c4f8fd", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":1,
    "visitors":0,
    "stay_time":0,
    "link_clicks":0,
    "event_duration_time":0,
    "event_clicks":0,
    "event_video_views":0,
    "source":0
    "utm_id":"1530668431031d862af9f81a"
    "tda_user_id":88013,
    "business_id":88001,
    "channel_id":88003,
    "campaign_id":88002,
    "td_landing_page_id":88005,
    "targeting_id": 88012,
}
```
- td日志

```json
{
    "log_type":1,
    "hash":"",
    "tda_user_id":88013,
    "user_id":88014,
    "business_id":88001,
    "channel_id":88003,
    "spot_id":88011,
    "medium_id":88009,
    "plan_id":88010,
    "campaign_id":88002,
    "creative_id":88004,
    "landing_page_id":88005,
    "targeting_id": 88012,
    "date":20180704, //接收请求的日期
    "hour":10, //接收请求的时间
    "ts":1530670760,//接收请求的时间
    "uuid":"15301532238979cdd8c4f8fd",
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36", //请求的user-agent
    "ip":"127.0.0.1",//请求的ip
    "referer":"",//请求的Referer
    "impressions":0,
    "clicks":0,
    "cost":0,
    "b_page_views":1,
    "b_visitors":0,
    "l_visitors":0,
    "dim_0":0,
    "dim_1":0,
    "dim_2":0,
    "dim_3":0,
    "dim_4":0,
    "dim_5":0,
    "dim_6":0,
    "dim_7":0,
    "dim_8":0,
    "dim_9":0,
    "metric_0":0,
    "metric_1":0,
    "metric_2":0,
    "metric_3":0,
    "metric_4":0,
    "metric_5":0,
    "metric_6":0,
    "metric_7":0,
    "metric_8":0,
    "metric_9":0,
     "lps_landing_page_id":0,
     "td_landing_page_id":0,
     "version":0,
     "event_id":0,
     "event_type_id":0,
     "visitors":0,
     "stay_time":0,
     "link_clicks":0,
     "event_duration_time":0,
     "event_clicks":0,
     "event_video_views":0,
     "page_views":0,
}
```

- SQL语句校验数据：

select * from lps_main where ts = 1530670760

select * from td_main where ts = 1530670760

select * from td_lps where ts =1530670760

##### [2].url参数：tp\nid\utm\uid\lid\v 

- 先删除浏览器cookie，url: http://127.0.0.1:9009/lps/a.gif?tp=1&nid=test001&utm=1530668431031d862af9f81a2&uid=99007&lid=99006&v=99008
- lps日志
```json
{
    "log_type":1,
    "hash":"",
    "user_id":99007,
    "lps_landing_page_id":99006,
    "version":99008,
    "date":20180704,//接收请求的日期
    "hour":10, //接收请求的时间
    "ts":1530671743,//接收请求的时间
    "event_id":0,
    "event_type_id":0,
    "uuid":"15301532238979cdd8c4f8fd", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":1,
    "visitors":0,
    "stay_time":0,
    "link_clicks":0,
    "event_duration_time":0,
    "event_clicks":0,
    "event_video_views":0,
    "source":0
    "utm_id":"1530668431031d862af9f81a"
    "tda_user_id":88013,
    "business_id":88001,
    "channel_id":88003,
    "campaign_id":88002,
    "td_landing_page_id":88005,
    "targeting_id": 88012,
}
```
- td日志
```json
{
    "log_type":1,
    "hash":"",
    "tda_user_id":88013,
    "user_id":88014,
    "business_id":88001,
    "channel_id":88003,
    "spot_id":88011,
    "medium_id":88009,
    "plan_id":88010,
    "campaign_id":88002,
    "creative_id":88004,
    "landing_page_id":88005,
    "targeting_id": 88012,
    "date":20180704, //接收请求的日期
    "hour":10, //接收请求的时间
    "ts":1530671743,//接收请求的时间
    "uuid":"15301532238979cdd8c4f8fd",
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36", //请求的user-agent
    "ip":"127.0.0.1",//请求的ip
    "referer":"",//请求的Referer
    "impressions":0,
    "clicks":0,
    "cost":0,
    "b_page_views":1,
    "b_visitors":0,
    "l_visitors":0,
    "dim_0":0,
    "dim_1":0,
    "dim_2":0,
    "dim_3":0,
    "dim_4":0,
    "dim_5":0,
    "dim_6":0,
    "dim_7":0,
    "dim_8":0,
    "dim_9":0,
    "metric_0":0,
    "metric_1":0,
    "metric_2":0,
    "metric_3":0,
    "metric_4":0,
    "metric_5":0,
    "metric_6":0,
    "metric_7":0,
    "metric_8":0,
    "metric_9":0,
     "lps_landing_page_id":0,
     "td_landing_page_id":0,
     "version":0,
     "event_id":0,
     "event_type_id":0,
     "visitors":0,
     "stay_time":0,
     "link_clicks":0,
     "event_duration_time":0,
     "event_clicks":0,
     "event_video_views":0,
     "page_views":0,
}
```
- SQL语句校验数据：

select * from lps_main where ts = 1530671743;

select * from td_main where ts = 1530671743;

select * from td_lps where ts =1530671743;

#### (3)无utmid, 入库 lps_main   td_main
##### [1].url参数：tp\nid
- 先删除浏览器cookie，url:http://127.0.0.1:9009/lps/a.gif?tp=1&nid=test002
- lps日志
```json
{
    "log_type":1,
    "hash":"",
    "user_id":0,
    "lps_landing_page_id":0,
    "version":0,
    "date":20180704,//接收请求的日期
    "hour":10, //接收请求的时间
    "ts":1530672059,//接收请求的时间
    "event_id":0,
    "event_type_id":0,
    "uuid":"1530672059279d117d43ba47", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":1,
    "visitors":0,
    "stay_time":0,
    "link_clicks":0,
    "event_duration_time":0,
    "event_clicks":0,
    "event_video_views":0,
    "source":0
    "utm_id":""
    "tda_user_id":0,
    "business_id":0,
    "channel_id":0,
    "campaign_id":0,
    "td_landing_page_id":0,
    "targeting_id": 0,
}
```
- td日志
```json
{
    "log_type":1,
    "hash":"",
    "tda_user_id":0,
    "user_id":0,
    "business_id":0,
    "channel_id":0,
    "spot_id":0,
    "medium_id":0,
    "plan_id":0,
    "campaign_id":0,
    "creative_id":0,
    "landing_page_id":0,
    "targeting_id": 0,
    "date":20180704, //接收请求的日期
    "hour":10, //接收请求的时间
    "ts":1530672059,//接收请求的时间
    "uuid":"1530672059279d117d43ba47",
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36", //请求的user-agent
    "ip":"127.0.0.1",//请求的ip
    "referer":"",//请求的Referer
    "impressions":0,
    "clicks":0,
    "cost":0,
    "b_page_views":1,
    "b_visitors":0,
    "l_visitors":0,
    "dim_0":0,
    "dim_1":0,
    "dim_2":0,
    "dim_3":0,
    "dim_4":0,
    "dim_5":0,
    "dim_6":0,
    "dim_7":0,
    "dim_8":0,
    "dim_9":0,
    "metric_0":0,
    "metric_1":0,
    "metric_2":0,
    "metric_3":0,
    "metric_4":0,
    "metric_5":0,
    "metric_6":0,
    "metric_7":0,
    "metric_8":0,
    "metric_9":0,
     "lps_landing_page_id":0,
     "td_landing_page_id":0,
     "version":0,
     "event_id":0,
     "event_type_id":0,
     "visitors":0,
     "stay_time":0,
     "link_clicks":0,
     "event_duration_time":0,
     "event_clicks":0,
     "event_video_views":0,
     "page_views":0,
}
```
- SQL语句校验数据：

select * from lps_main where ts = 1530672059;

select * from td_main where ts = 1530672059;

##### [2].url参数：tp\nid\uid\lid\v 
- 先删除浏览器cookie，url:http://127.0.0.1:9009/lps/a.gif?tp=1&nid=test003&uid=99007&lid=99006&v=99008
- lps日志
```json
{
    "log_type":1,
    "hash":"",
    "user_id":99007,
    "lps_landing_page_id":99006,
    "version":99008,
    "date":20180704,//接收请求的日期
    "hour":10, //接收请求的时间
    "ts":1530672285,//接收请求的时间
    "event_id":0,
    "event_type_id":0,
    "uuid":"1530672285293996cdc263a0",   //每次生成的都不一样
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":1,
    "visitors":0,
    "stay_time":0,
    "link_clicks":0,
    "event_duration_time":0,
    "event_clicks":0,
    "event_video_views":0,
    "source":0
    "utm_id":""
    "tda_user_id":0,
    "business_id":0,
    "channel_id":0,
    "campaign_id":0,
    "td_landing_page_id":0,
    "targeting_id": 0,
}
```
- td日志
```json
{
    "log_type":1,
    "hash":"",
    "tda_user_id":,
    "user_id":0,
    "business_id":0,
    "channel_id":0,
    "spot_id":0,
    "medium_id":0,
    "plan_id":0,
    "campaign_id":0,
    "creative_id":0,
    "landing_page_id":0,
    "targeting_id": 0,
    "date":20180704, //接收请求的日期
    "hour":10, //接收请求的时间
    "ts":1530672285,//接收请求的时间
    "uuid":"1530672285293996cdc263a0",
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36", //请求的user-agent
    "ip":"127.0.0.1",//请求的ip
    "referer":"",//请求的Referer
    "impressions":0,
    "clicks":0,
    "cost":0,
    "b_page_views":1,
    "b_visitors":0,
    "l_visitors":0,
    "dim_0":0,
    "dim_1":0,
    "dim_2":0,
    "dim_3":0,
    "dim_4":0,
    "dim_5":0,
    "dim_6":0,
    "dim_7":0,
    "dim_8":0,
    "dim_9":0,
    "metric_0":0,
    "metric_1":0,
    "metric_2":0,
    "metric_3":0,
    "metric_4":0,
    "metric_5":0,
    "metric_6":0,
    "metric_7":0,
    "metric_8":0,
    "metric_9":0,
     "lps_landing_page_id":0,
     "td_landing_page_id":0,
     "version":0,
     "event_id":0,
     "event_type_id":0,
     "visitors":0,
     "stay_time":0,
     "link_clicks":0,
     "event_duration_time":0,
     "event_clicks":0,
     "event_video_views":0,
     "page_views":0,
}
```
- SQL语句校验数据：

select * from lps_main where ts = 1530672285;

select * from td_main where ts = 1530672285;

### 4.LPS_LandingStay
#### (1) 有utmId，入库lps_main td_lps 

没有清空浏览器cookies，所以uuid 的值和上面的uuid一样。

##### [1].url参数：tp\utm\st

- url: http://127.0.0.1:9009/lps/a.gif?tp=2&utm=1530668431031d862af9f81a&st=6
- lps日志
```json
{
    "log_type":2,
    "hash":"",
    "user_id":88007,
    "lps_landing_page_id":88006,
    "version":88008,
    "date":20180704,//接收请求的日期
    "hour":10, //接收请求的时间
    "ts":1530672770,//接收请求的时间
    "event_id":0,
    "event_type_id":0,
    "uuid":"1530672285293996cdc263a0", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":0,
    "visitors":0,
    "stay_time":6,
    "link_clicks":0,
    "event_duration_time":0,
    "event_clicks":0,
    "event_video_views":0,
    "source":0
    "utm_id":"1530668431031d862af9f81a"
    "tda_user_id":88013,
    "business_id":88001,
    "channel_id":88003,
    "campaign_id":88002,
    "td_landing_page_id":88005,
    "targeting_id": 88012,
}
```
- SQL语句校验数据：

select * from lps_main where ts = 1530672770

select * from td_lps where ts =1530672770

##### [2].url参数：tp\utm\st\uid\lid\v
- url: http://127.0.0.1:9009/lps/a.gif?tp=2&utm=1530668431031d862af9f81a&st=6&uid=99007&lid=99006&v=99008

- lps日志
```json
{
    "log_type":2,
    "hash":"",
    "user_id":99007,
    "lps_landing_page_id":99006,
    "version":99008,
    "date":20180704,//接收请求的日期
    "hour":10, //接收请求的时间
    "ts":1530672980,//接收请求的时间
    "event_id":0,
    "event_type_id":0,
    "uuid":"1530672285293996cdc263a0", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":0,
    "visitors":0,
    "stay_time":6,
    "link_clicks":0,
    "event_duration_time":0,
    "event_clicks":0,
    "event_video_views":0,
    "source":0
    "utm_id":"1530668431031d862af9f81a"
    "tda_user_id":88013,
    "business_id":88001,
    "channel_id":88003,
    "campaign_id":88002,
    "td_landing_page_id":88005,
    "targeting_id": 88012,
}
```
- SQL语句校验数据：

select * from lps_main where ts = 1530672980;

select * from td_lps where ts =1530672980;

#### (3)无utmid, 入库 lps_main 
##### [1].url参数：tp\st

- url:http://127.0.0.1:9009/lps/a.gif?tp=2&st=6
- lps日志

```json
{
    "log_type":2,
    "hash":"",
    "user_id":0,
    "lps_landing_page_id":0,
    "version":0,
    "date":20180704,//接收请求的日期
    "hour":10, //接收请求的时间
    "ts":1530673100,//接收请求的时间
    "event_id":0,
    "event_type_id":0,
    "uuid":"1530672285293996cdc263a0", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":0,
    "visitors":0,
    "stay_time":6,
    "link_clicks":0,
    "event_duration_time":0,
    "event_clicks":0,
    "event_video_views":0,
    "source":0
    "utm_id":""
    "tda_user_id":0,
    "business_id":0,
    "channel_id":0,
    "campaign_id":0,
    "td_landing_page_id":0,
    "targeting_id": 0,
}
```
- SQL语句校验数据：

select * from lps_main where ts = 1530673100;

##### [2].url参数：tp\nid\st\uid\lid\v 
- url:http://127.0.0.1:9009/lps/a.gif?tp=2&st=6&uid=99007&lid=99006&v=99008
- lps日志
```json
{
    "log_type":2,
    "hash":"",
    "user_id":99007,
    "lps_landing_page_id":99006,
    "version":99008,
    "date":20180704,//接收请求的日期
    "hour":11, //接收请求的时间
    "ts":1530673226,//接收请求的时间
    "event_id":0,
    "event_type_id":0,
    "uuid":"1530672285293996cdc263a0", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":0,
    "visitors":0,
    "stay_time":6,
    "link_clicks":0,
    "event_duration_time":0,
    "event_clicks":0,
    "event_video_views":0,
    "source":0
    "utm_id":""
    "tda_user_id":0,
    "business_id":0,
    "channel_id":0,
    "campaign_id":0,
    "td_landing_page_id":0,
    "targeting_id": 0,
}
```
- SQL语句校验数据：

select * from lps_main where ts = 1530673226;

### 5.LPS_LandingLinkClick 

#### (1).查重

#### (2) 有utmId，入库lps_main td_lps

##### [1].url参数：tp\nid\utm

- url: http://127.0.0.1:9009/lps/a.gif?tp=3&nid=test3001&utm=1530668431031d862af9f81a
- lps日志

```json
{
    "log_type":3,
    "hash":"",
    "user_id":88007,
    "lps_landing_page_id":88006,
    "version":88008,
    "date":20180704,//接收请求的日期
    "hour":11, //接收请求的时间
    "ts":1530673433,//接收请求的时间
    "event_id":0,
    "event_type_id":0,
    "uuid":"1530672285293996cdc263a0", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":0,
    "visitors":0,
    "stay_time":0,
    "link_clicks":1,
    "event_duration_time":0,
    "event_clicks":0,
    "event_video_views":0,
    "source":0
    "utm_id":"1530668431031d862af9f81a"
    "tda_user_id":88013,
    "business_id":88001,
    "channel_id":88003,
    "campaign_id":88002,
    "td_landing_page_id":88005,
    "targeting_id": 88012,
}
```
- SQL语句校验数据：

select * from lps_main where ts = 1530673433;

select * from td_lps where ts =1530673433;

##### [2].url参数：tp\nid\utm\uid\lid\v

- url: http://127.0.0.1:9009/lps/a.gif?tp=3&nid=test3002&utm=1530668431031d862af9f81a&uid=99007&lid=99006&v=99008
- lps日志

```json
{
    "log_type":3,
    "hash":"",
    "user_id":99007,
    "lps_landing_page_id":99006,
    "version":99008,
    "date":20180704,//接收请求的日期
    "hour":11, //接收请求的时间
    "ts":1530673515,//接收请求的时间
    "event_id":0,
    "event_type_id":0,
    "uuid":"1530672285293996cdc263a0", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":0,
    "visitors":0,
    "stay_time":0,
    "link_clicks":1,
    "event_duration_time":0,
    "event_clicks":0,
    "event_video_views":0,
    "source":0
    "utm_id":"1530668431031d862af9f81a"
    "tda_user_id":88013,
    "business_id":88001,
    "channel_id":88003,
    "campaign_id":88002,
    "td_landing_page_id":88005,
    "targeting_id": 88012,
}
```

- SQL语句校验数据：

select * from lps_main where ts = 1530673515;

select * from td_lps where ts =1530673515;

#### (3)无utmid, 入库 lps_main 

##### [1].url参数：tp\nid

- url:http://127.0.0.1:9009/lps/a.gif?tp=3&nid=test3003
- lps日志

```json
{
    "log_type":3,
    "hash":"",
    "user_id":0,
    "lps_landing_page_id":0,
    "version":0,
    "date":20180704,//接收请求的日期
    "hour":11, //接收请求的时间
    "ts":1530673667,//接收请求的时间
    "event_id":0,
    "event_type_id":0,
    "uuid":"1530672285293996cdc263a0", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":0,
    "visitors":0,
    "stay_time":0,
    "link_clicks":1,
    "event_duration_time":0,
    "event_clicks":0,
    "event_video_views":0,
    "source":0
    "utm_id":""
    "tda_user_id":0,
    "business_id":0,
    "channel_id":0,
    "campaign_id":0,
    "td_landing_page_id":0,
    "targeting_id": 0,
}
```
- SQL语句校验数据：

select * from lps_main where ts = 1530673667;

##### [2].url参数：tp\nid\uid\lid\v

- url:http://127.0.0.1:9009/lps/a.gif?tp=3&nid=test3004&uid=99007&lid=99006&v=99008
- lps日志

```json
{
    "log_type":3,
    "hash":"",
    "user_id":99007,
    "lps_landing_page_id":99006,
    "version":99008,
    "date":20180704,//接收请求的日期
    "hour":11, //接收请求的时间
    "ts":1530673759,//接收请求的时间
    "event_id":0,
    "event_type_id":0,
    "uuid":"1530672285293996cdc263a0", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":0,
    "visitors":0,
    "stay_time":0,
    "link_clicks":1,
    "event_duration_time":0,
    "event_clicks":0,
    "event_video_views":0,
    "source":0
    "utm_id":""
    "tda_user_id":0,
    "business_id":0,
    "channel_id":0,
    "campaign_id":0,
    "td_landing_page_id":0,
    "targeting_id": 0,
}
```

- SQL语句校验数据：

select * from lps_main where ts = 1530673759;

### 6.LPS_PointerClick 
#### (1).查重

#### (2) 有utmId，入库lps_main td_lps

##### [1].url参数：tp\nid\utm\en

- url: http://127.0.0.1:9009/lps/a.gif?tp=4&nid=test4001&utm=1530668431031d862af9f81a&en=pointerClick20180704001
- lps日志

```json
{
    "log_type":4,
    "hash":"",
    "user_id":88007,
    "lps_landing_page_id":88006,
    "version":88008,
    "date":20180704,//接收请求的日期
    "hour":11, //接收请求的时间
    "ts":1530673878,//接收请求的时间
    "event_id":167021687134170,
    "event_type_id":1,
    "uuid":"1530672285293996cdc263a0", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":0,
    "visitors":0,
    "stay_time":0,
    "link_clicks":0,
    "event_duration_time":0,
    "event_clicks":1,
    "event_video_views":0,
    "source":0
    "utm_id":"1530668431031d862af9f81a"
    "tda_user_id":88013,
    "business_id":88001,
    "channel_id":88003,
    "campaign_id":88002,
    "td_landing_page_id":88005,
    "targeting_id": 88012,
}
```

- SQL语句校验数据：

select * from lps_main where ts = 1530673878;

select * from td_lps where ts =1530673878;

select * from event_mapping where id =167021687134170 and value=“pointerClick20180704001”;

##### [2].url参数：tp\nid\utm\uid\lid\v\en

- url: http://127.0.0.1:9009/lps/a.gif?tp=4&nid=test4002&utm=1530668431031d862af9f81a&uid=99007&lid=99006&v=99008&en=pointerClick20180704002
- lps日志

```json
{
    "log_type":4,
    "hash":"",
    "user_id":99007,
    "lps_landing_page_id":99006,
    "version":99008,
    "date":20180704,//接收请求的日期
    "hour":11, //接收请求的时间
    "ts":1530673995,//接收请求的时间
    "event_id":37727569642265,
    "event_type_id":1,
    "uuid":"1530672285293996cdc263a0", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":0,
    "visitors":0,
    "stay_time":0,
    "link_clicks":0,
    "event_duration_time":0,
    "event_clicks":1,
    "event_video_views":0,
    "source":0
    "utm_id":"1530668431031d862af9f81a"
    "tda_user_id":88013,
    "business_id":88001,
    "channel_id":88003,
    "campaign_id":88002,
    "td_landing_page_id":88005,
    "targeting_id": 88012,
}
```
- SQL语句校验数据：

select * from lps_main where ts = 1530673995;

select * from td_lps where ts =1530673995;

select * from event_mapping where id =37727569642265 and `value`="pointerClick20180704002"

#### (3)无utmid, 入库 lps_main 

##### [1].url参数：tp\nid\en

- url:http://127.0.0.1:9009/lps/a.gif?tp=4&nid=test4003&en=pointerClick20180704003
- lps日志

```json
{
    "log_type":4,
    "hash":"",
    "user_id":0,
    "lps_landing_page_id":0,
    "version":0,
    "date":20180704,//接收请求的日期
    "hour":11, //接收请求的时间
    "ts":1530674109,//接收请求的时间
    "event_id":52378432806730,
    "event_type_id":1,
    "uuid":"1530672285293996cdc263a0", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":0,
    "visitors":0,
    "stay_time":0,
    "link_clicks":0,
    "event_duration_time":0,
    "event_clicks":1,
    "event_video_views":0,
    "source":0
    "utm_id":""
    "tda_user_id":0,
    "business_id":0,
    "channel_id":0,
    "campaign_id":0,
    "td_landing_page_id":0,
    "targeting_id": 0,
}
```
- SQL语句校验数据：

select * from lps_main where ts = 1530674109;

select * from event_mapping where id =52378432806730 and `value`="pointerClick20180704003"

##### [2].url参数：tp\nid\uid\lid\v\en

- url:http://127.0.0.1:9009/lps/a.gif?tp=4&nid=test4004&uid=99007&lid=99006&v=99008&en=pointerClick20180704004
- lps日志

```json
{
    "log_type":4,
    "hash":"",
    "user_id":99007,
    "lps_landing_page_id":99006,
    "version":99008,
    "date":20180704,//接收请求的日期
    "hour":11, //接收请求的时间
    "ts":1530674229,//接收请求的时间
    "event_id":126656804357916,
    "event_type_id":1,
    "uuid":"1530672285293996cdc263a0", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":0,
    "visitors":0,
    "stay_time":0,
    "link_clicks":0,
    "event_duration_time":0,
    "event_clicks":1,
    "event_video_views":0,
    "source":0
    "utm_id":""
    "tda_user_id":0,
    "business_id":0,
    "channel_id":0,
    "campaign_id":0,
    "td_landing_page_id":0,
    "targeting_id": 0,
}
```

- SQL语句校验数据：

select * from lps_main where ts = 1530674229;

select * from event_mapping where id =126656804357916 and `value`="pointerClick20180704004"

### 7.LPS_PointerVideo 
#### (1).查重

#### (2) 有utmId，入库lps_main td_lps

##### [1].url参数：tp\nid\utm\en

- url: http://127.0.0.1:9009/lps/a.gif?tp=5&nid=test5001&utm=1530668431031d862af9f81a&en=pointerVideo20180704001
- lps日志

```json
{
    "log_type":5,
    "hash":"",
    "user_id":88007,
    "lps_landing_page_id":88006,
    "version":88008,
    "date":20180704,//接收请求的日期
    "hour":11, //接收请求的时间
    "ts":1530674434,//接收请求的时间
    "event_id":92198463329356,
    "event_type_id":2,
    "uuid":"1530672285293996cdc263a0", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":0,
    "visitors":0,
    "stay_time":0,
    "link_clicks":0,
    "event_duration_time":0,
    "event_clicks":0,
    "event_video_views":1,
    "source":0
    "utm_id":"1530668431031d862af9f81a"
    "tda_user_id":88013,
    "business_id":88001,
    "channel_id":88003,
    "campaign_id":88002,
    "td_landing_page_id":88005,
    "targeting_id": 88012,
}
```

- SQL语句校验数据：

select * from lps_main where ts = 1530674434;

select * from td_lps where ts =1530674434;

select * from event_mapping where id =92198463329356 and `value`="pointerVideo20180704001";

##### [2].url参数：tp\nid\utm\en\uid\lid\v

- url: http://127.0.0.1:9009/lps/a.gif?tp=5&nid=test5002&utm=1530668431031d862af9f81a&en=pointerVideo20180704002&uid=99007&lid=99006&v=99008
- lps日志

```json
{
    "log_type":5,
    "hash":"",
    "user_id":99007,
    "lps_landing_page_id":99006,
    "version":99008,
    "date":20180704,//接收请求的日期
    "hour":11, //接收请求的时间
    "ts":1530674670,//接收请求的时间
    "event_id":37729609779223,
    "event_type_id":2,
    "uuid":"1530672285293996cdc263a0", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":0,
    "visitors":0,
    "stay_time":0,
    "link_clicks":0,
    "event_duration_time":0,
    "event_clicks":0,
    "event_video_views":1,
    "source":0
    "utm_id":"1530668431031d862af9f81a"
    "tda_user_id":88013,
    "business_id":88001,
    "channel_id":88003,
    "campaign_id":88002,
    "td_landing_page_id":88005,
    "targeting_id": 88012,
}
```

- SQL语句校验数据：

select * from lps_main where ts = 1530674670;

select * from td_lps where ts =1530674670;

select * from event_mapping where id =37729609779223 and `value`="pointerVideo20180704002"

#### (3)无utmid, 入库 lps_main 

##### [1].url参数：tp\nid\en

- url:http://127.0.0.1:9009/lps/a.gif?tp=5&nid=test5003&en=pointerVideo20180704003
- lps日志

```json
{
    "log_type":5,
    "hash":"",
    "user_id":0,
    "lps_landing_page_id":0,
    "version":0,
    "date":20180704,//接收请求的日期
    "hour":11, //接收请求的时间
    "ts":1530674839,//接收请求的时间
    "event_id":131938100252117,
    "event_type_id":2,
    "uuid":"1530672285293996cdc263a0", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":0,
    "visitors":0,
    "stay_time":0,
    "link_clicks":0,
    "event_duration_time":0,
    "event_clicks":0,
    "event_video_views":1,
    "source":0
    "utm_id":""
    "tda_user_id":0,
    "business_id":0,
    "channel_id":0,
    "campaign_id":0,
    "td_landing_page_id":0,
    "targeting_id": 0,
}
```

- SQL语句校验数据：

select * from lps_main where ts = 1530674839;

select * from event_mapping where id =131938100252117 and `value`="pointerVideo20180704003"

##### [2].url参数：tp\nid\en\uid\lid\v

- url:http://127.0.0.1:9009/lps/a.gif?tp=5&nid=test5004&en=pointerVideo20180704004&uid=99007&lid=99006&v=99008
- lps日志

```json
{
    "log_type":5,
    "hash":"",
    "user_id":99007,
    "lps_landing_page_id":99006,
    "version":99008,
    "date":20180704,//接收请求的日期
    "hour":11, //接收请求的时间
    "ts":1530675066,//接收请求的时间
    "event_id":12379681894373 ,
    "event_type_id":2,
    "uuid":"1530672285293996cdc263a0", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":0,
    "visitors":0,
    "stay_time":0,
    "link_clicks":0,
    "event_duration_time":0,
    "event_clicks":0,
    "event_video_views":1,
    "source":0
    "utm_id":""
    "tda_user_id":0,
    "business_id":0,
    "channel_id":0,
    "campaign_id":0,
    "td_landing_page_id":0,
    "targeting_id": 0,
}
```

- SQL语句校验数据：

select * from lps_main where ts = 1530675066;

select * from event_mapping where id =12379681894373 and `value`="pointerVideo20180704004"

### 8.LPS_PointerVideoStay 
#### (1) 有utmId，入库lps_main td_lps

##### [1].url参数：tp\utm\en\vdt

- url: http://127.0.0.1:9009/lps/a.gif?tp=6&utm=1530668431031d862af9f81a&en=pointerVideoStay20180704001&vdt=20
- lps日志

```json
{
    "log_type":6,
    "hash":"",
    "user_id":88007,
    "lps_landing_page_id":88006,
    "version":88008,
    "date":20180704,//接收请求的日期
    "hour":11, //接收请求的时间
    "ts":1530675355,//接收请求的时间
    "event_id":163957924650195,
    "event_type_id":2,
    "uuid":"1530672285293996cdc263a0", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":0,
    "visitors":0,
    "stay_time":0,
    "link_clicks":0,
    "event_duration_time":20,
    "event_clicks":0,
    "event_video_views":0,
    "source":0
    "utm_id":"1530668431031d862af9f81a"
    "tda_user_id":88013,
    "business_id":88001,
    "channel_id":88003,
    "campaign_id":88002,
    "td_landing_page_id":88005,
    "targeting_id": 88012,
}
```

- SQL语句校验数据：

select * from lps_main where ts = 1530675355;

select * from td_lps where ts =1530675355;

select * from event_mapping where id =163957924650195  and `value`="pointerVideoStay20180704001"

##### [2].url参数：tp\utm\en\vdt\uid\lid\v

- url: http://127.0.0.1:9009/lps/a.gif?tp=6&utm=1530668431031d862af9f81a&en=pointerVideoStay20180704002&vdt=20&uid=99007&lid=99006&v=99008
- lps日志

```json
{
    "log_type":6,
    "hash":"",
    "user_id":99007,
    "lps_landing_page_id":99006,
    "version":99008,
    "date":20180704,//接收请求的日期
    "hour":11, //接收请求的时间
    "ts":1530675792,//接收请求的时间
    "event_id":36973092365866,
    "event_type_id":2,
    "uuid":"1530672285293996cdc263a0", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":0,
    "visitors":0,
    "stay_time":0,
    "link_clicks":0,
    "event_duration_time":20,
    "event_clicks":0,
    "event_video_views":0,
    "source":0
    "utm_id":"1530668431031d862af9f81a"
    "tda_user_id":88013,
    "business_id":88001,
    "channel_id":88003,
    "campaign_id":88002,
    "td_landing_page_id":88005,
    "targeting_id": 88012,
}
```

- SQL语句校验数据：

select * from lps_main where ts = 1530675792;

select * from td_lps where ts =1530675792;

select * from event_mapping where id =36973092365866  and `value`="pointerVideoStay20180704002"

#### (2)无utmid, 入库 lps_main 

##### [1].url参数：tp\en\vdt

- url:http://127.0.0.1:9009/lps/a.gif?tp=6&en=pointerVideoStay20180704003&vdt=20
- lps日志

```json
{
    "log_type":6,
    "hash":"",
    "user_id":0,
    "lps_landing_page_id":0,
    "version":0,
    "date":20180704,//接收请求的日期
    "hour":11, //接收请求的时间
    "ts":1530675914,//接收请求的时间
    "event_id":213651727800883,
    "event_type_id":2,
    "uuid":"1530672285293996cdc263a0", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":0,
    "visitors":0,
    "stay_time":0,
    "link_clicks":0,
    "event_duration_time":20,
    "event_clicks":0,
    "event_video_views":0,
    "source":0
    "utm_id":""
    "tda_user_id":0,
    "business_id":0,
    "channel_id":0,
    "campaign_id":0,
    "td_landing_page_id":0,
    "targeting_id": 0,
}
```

- SQL语句校验数据：

select * from lps_main where ts = 1530675914;

select * from event_mapping where id =213651727800883 and `value`="pointerVideoStay20180704003"

##### [2].url参数：tp\en\vdt\uid\lid\v

- url:http://127.0.0.1:9009/lps/a.gif?tp=6&en=pointerVideoStay20180704004&vdt=20&uid=99007&lid=99006&v=99008
- lps日志

```json
{
    "log_type":6,
    "hash":"",
    "user_id":99007,
    "lps_landing_page_id":99006,
    "version":99008,
    "date":20180704,//接收请求的日期
    "hour":11, //接收请求的时间
    "ts":1530676098,//接收请求的时间
    "event_id":70322671598767,
    "event_type_id":2,
    "uuid":"1530672285293996cdc263a0", 
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
    "ip": "127.0.0.1",
    "referer": "",
    "page_views":0,
    "visitors":0,
    "stay_time":0,
    "link_clicks":0,
    "event_duration_time":20,
    "event_clicks":0,
    "event_video_views":0,
    "source":0
    "utm_id":""
    "tda_user_id":0,
    "business_id":0,
    "channel_id":0,
    "campaign_id":0,
    "td_landing_page_id":0,
    "targeting_id": 0,
}
```

- SQL语句校验数据：

select * from lps_main where ts = 1530676098;

select * from event_mapping where id =70322671598767 and `value`="pointerVideoStay20180704004";

### 9.LPS_JSTransfer  js脚步调用

#### (1).处理逻辑
- 生成eventid; 
- 更新utm里面的eventid（对应字段是lps_event_id）
#### (2).url参数：tp\en\utm
- url:http://127.0.0.1:9009/lps/a.gif?tp=7&en=jsTransfer&utm=1530668431031d862af9f81a
- 生成event_id：22549877616260，更新redis的数据（对应字段是lps_event_id）
- SQL数据校验

select * from event_mapping where id =22549877616260 and value="jsTransfer"

## 二.td统计

### 1.bi  (TD_Bi)

#### (1).HandleMsg日志
```json
{
    "data_type": "BIMsg",
    "data":
    {
        "targeting_id": 6604001,
        "tda_user_id": 6604002,
        "user_id": 6604003,
        "business_id": 6604004,
        "channel_id": 6604005,
        "spot_id": 6604006,
        "medium_id": 6604007,
        "plan_id": 6604008,
        "campaign_id": 6604009,
        "creative_id": 6604010,
        "landing_page_id": 6604011,
        "date": 20180705,
        "hour": 10,
        "ts": 1530758003,

        "uuid": "153075568737244957d49904",
        "user_agent": "UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36",
        "ip": "168.10.25.33",
        "referer": "referer test",

        "dim_0": "weidu88000",
        "dim_1": "weidu88001",
        "dim_2": "weidu88002",
        "dim_3": "weidu88003",
        "dim_4": "weidu88004",
        "dim_5": "weidu88005",
        "dim_6": "weidu88006",
        "dim_7": "weidu88007",
        "dim_8": "weidu88008",
        "dim_9": "weidu88009",

        "metric_0": 8,
        "metric_1": 18,
        "metric_2": 28,
        "metric_3": 38,
        "metric_4": 48,
        "metric_5": 58,
        "metric_6": 68,
        "metric_7": 78,
        "metric_8": 88,
        "metric_9": 98,
    }
}
```
#### (2) TDLog
```json
{
    "log_type":3,(TD_Bi)
    "hash":"",
    "tda_user_id":6604002,
    "user_id":6604003,
    "business_id":6604004,
    "channel_id":6604005,
    "spot_id":6604006,
    "medium_id":6604007,
    "plan_id":6604008,
    "campaign_id":6604009,
    "creative_id":6604010,
    "landing_page_id":6604011,
    "targeting_id": 6604001,
    "date":20180705, //接收请求的日期
    "hour":10, //接收请求的时间
    "ts":1530758003,//接收请求的时间
    "uuid":"153075568737244957d49911",
    "user_agent":"UserAgent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181Safari/537.36", //请求的user-agent
    "ip":"168.10.25.33",//请求的ip
    "referer":"referer test",//请求的Referer
    "impressions":0,
    "clicks":0,
    "cost":0,
    "b_page_views":0,
    "b_visitors":0,
    "l_visitors":0,
    "dim_0":93535954831855,
    "dim_1":69668588532898,
    "dim_2":131699993768474,
    "dim_3":239501549786422,
    "dim_4":45987890529851,
    "dim_5":142127123595050,
    "dim_6":82421897857485,
    "dim_7":58432242197977,
    "dim_8":136064277200012,
    "dim_9":67982601925585,
    "metric_0":8,
    "metric_1":18,
    "metric_2":28,
    "metric_3":38,
    "metric_4":48,
    "metric_5":58,
    "metric_6":68,
    "metric_7":48,
    "metric_8":88,
    "metric_9":98,
     "lps_landing_page_id":0,
     "td_landing_page_id":0,
     "version":0,
     "event_id":0,
     "event_type_id":0,
     "visitors":0,
     "stay_time":0,
     "link_clicks":0,
     "event_duration_time":0,
     "event_clicks":0,
     "event_video_views":0,
     "page_views":0,
}
```
#### (3).入库td_lps  td_main

select * from td_main where ts=1530758003 and landing_page_id = 6604011;

select * from td_lps where ts=1530758003 and landing_page_id = 6604011;

select * from bi_mapping where id in (93535954831855,69668588532898,131699993768474,239501549786422,45987890529851,
142127123595050,82421897857485,58432242197977,136064277200012,67982601925585);

### 2. 业务数据 (TD_Business)

#### (1).TDBusinessMsg日志
```json
	{
		"data_type": "TDBusinessMsg",
		"data": {
		    "log_type":2,
			"targeting_id": 5505001,
		    "tda_user_id":5505002,
		    "user_id":5505003,
		    "business_id":5505004,
		    "channel_id":5505005,
		    "spot_id":5505006,
		    "medium_id":5505007,
		    "plan_id":5505008,
		    "campaign_id":5505009,
		    "creative_id":5505010,
		    "landing_page_id":5505011,
		    "date":20180705, 
			"hour":11, 
			"ts":1530760798,
		    "impressions":66,
		    "cost":30,  
	    }
	}
```
#### (2) TDLog
```json
{
    "log_type":2,(TD_Business)
    "hash":"",
    "tda_user_id":5505002,
    "user_id":5505003,
    "business_id":5505004,
    "channel_id":5505005,
    "spot_id":5505006,
    "medium_id":5505007,
    "plan_id":5505008,
    "campaign_id":5505009,
    "creative_id":5505010,
    "landing_page_id":5505011,
    "targeting_id": 5505001,
    "date":20180705, //接收请求的日期
    "hour":11, //接收请求的时间
    "ts":1530760798,//接收请求的时间
    "uuid":"",
    "user_agent":"", 
    "ip":"",
    "referer":"",
    "impressions":66,
    "clicks":0,
    "cost":30,
    "b_page_views":0,
    "b_visitors":0,
    "l_visitors":0,
    "dim_0":0,
    "dim_1":0,
    "dim_2":0,
    "dim_3":0,
    "dim_4":0,
    "dim_5":0,
    "dim_6":0,
    "dim_7":0,
    "dim_8":0,
    "dim_9":0,
    "metric_0":0,
    "metric_1":0,
    "metric_2":0,
    "metric_3":0,
    "metric_4":0,
    "metric_5":0,
    "metric_6":0,
    "metric_7":0,
    "metric_8":0,
    "metric_9":0,
     "lps_landing_page_id":0,
     "td_landing_page_id":0,
     "version":0,
     "event_id":0,
     "event_type_id":0,
     "visitors":0,
     "stay_time":0,
     "link_clicks":0,
     "event_duration_time":0,
     "event_clicks":0,
     "event_video_views":0,
     "page_views":0,
}
```
#### (3).入库td_main

select * from td_main where ts in (1530760798);

### 3.行为日志(TD_Clicks)

#### (1).td日志
```json
	{
		INDEX: "1",
		SOURCE_TYPE: "2",
		FILE_NAME: "3",

		SOURCE_HOST: "4",
		AGENT_TIMESTAMP: "5",
       LOG: "{"log_type":1,"tda_user_id":1001,"user_id":2001,"business_id":3001,"channel_id":4001,"spot_id":5001,"medium_id":6001,"plan_id":7001,"campaign_id":8001,"creative_id":9001,"landing_page_id":11001,"targeting_id":12001,"date":20180705,"hour":14,"ts":1530771773,"clicks":12}",
		TOPIC: "6",
		FILE_PATH: "7",
	}
```
```json
{
    "log_type":1,
    "tda_user_id":1001,
    "user_id":2001,
    "business_id":3001,
    "channel_id":4001,
    "spot_id":5001,
    "medium_id":6001,
    "plan_id":7001,
    "campaign_id":8001,
    "creative_id":9001,
    "landing_page_id":11001,
    "targeting_id": 12001,
    "date":20180705, 
    "hour":14, 
    "ts":1530771773,
    "clicks":12, 
}
```

#### (2).入库td_main

select * from td_main where ts in (1530771773);
