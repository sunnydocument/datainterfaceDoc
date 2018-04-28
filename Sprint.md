# Sprint识别后台管理系统
![达闼logo](./dtlogo.png)

### Revision history

| NO. | Name | Change detail |  Revision    |
|---:|:---|:---:|---|
| 1| Sunny|2018/03/09 | V0.1|
| 2| Sunny|2018/04/11 | V0.2|
| 3| Sunny|2018/04/24 | V0.3 按照英文文档重新开发|

## Menu

-  [项目目的](#项目目的)
- [项目运行过程](#项目运行过程)
- [项目进度](#项目进度)
- [1）用户信息添加接口](#1用户信息添加接口)
- [2）把用户信息添加到热搜接口](#2把用户信息添加到热搜接口)
- [3）删除一条用户信息接口](#3删除一条用户信息接口)
- [4）图像识别接口](#4图像识别接口)
- [5）Unknown信息添加接口](#5unknown信息添加接口)
- [6）Unknown信息修改接口](#6unknown信息修改接口)
- [7）删除一条用户信息接口](#7删除一条用户信息接口)


## 项目目的
#### 为了给Sprint提供人脸识别接口。主要的问题在于有有限识别的问题，这块需要考虑清楚什么人可以放入优先识别库。

## 原数据库

|No.|Name|Type|IsNull|Memo|
|---:|:---|:---:|:---:|:---|
| 1| id|int |不可为空|id|
| 2| uid|string |可为空| uid|
| 3| avatar|string |不可为空| 头像存储的地址|
| 4| name|string |不可为空| 名称|
| 5| hot|bool |不可为空| 是否需要快速搜索|
| 6| time|timestamp |可为空| 什么时候来|
| 7| expire|timestamp |可为空| 过期时间|


## 新数据库

|No.|Name|Type|IsNull|Memo|
|---:|:---|:---:|:---:|:---|
| 1| id|int |不可为空|id|
| 2| 	engage_location_id|string（25） |可为空| uid|
| 3| customer_photo|longblob |不可为空| 头像存储，base64|
| 4| 	customer_name|string |不可为空| 名称|
| 5| phone_number|string |不可为空| 电话号码|
| 6| known_customer|bool |不可为空| 是否需要快速搜索|
| 7| time_stamp|timestamp |可为空| 什么时候来|
| 8| time_span|int |可为空|过期时间以分钟计算|
| 9| trigger_type|string |可为空|触发类型beacon|
|10| device_type|string |可为空|设备类型android|
|11| appointment|bool |不可为空|是否预约|
| 12| hot|bool |不可为空| 是否需要快速搜索|
| 13| expire|timestamp |可为空| 过期时间|


## 项目运行过程

- 手动生成2个库
- 把两个库写到app.conf里面


#### 手动生成2个库（postman）

	10.11.35.201:1688/api/v1/faceset/register
	app_key = NoNmDWavecRVryE195w819410oUDJ6O3
    app_secret = 5R0uHPZNwZ4jpUJtl8AyhcGfrcWEfrvx
    创建主库
    {"code":0,"date":"2018-03-12 10:07:03","faceset_id":"4B8iGcu5qwf5FL87rrA5lxe22GSH7050"}
    创建热搜库
    {"code":0,"date":"2018-03-12 10:10:57","faceset_id":"haXgad4ENjXSu0oF928C7P709Jsk096e"}
	创建unknow库
	{"code":0,"date":"2018-04-10 15:34:28","faceset_id":"Ya20Jbhls8L7vGUjk6BVdAfFtePFW4L6"}

## 项目进度
  * [ ] 未开始
  * [x] 已完成


## Sprint人脸识别部分接口

  - [x] 信息录入
  - [x] 图像识别接口
  - [x] 删除用户信息
  - [x] 优先识别添加部分
  - [x] unknown的添加
  - [ ] 做一个任务计划



***

## 注意一件事情由于有了unknown这个情况，返回的数据需要区分这个人是客户还是unknown，* 有uid的是客户，没有uid的就是unknown*。

***

## 1）用户信息添加接口
## 基本描述
|接口说明|用户信息添加接口|
|---|---|
|请求URL|加密链接 https://{host-name}/sprint/add <br />普通链接： http://{host-name}/sprint/add|
|协议方法|Post (multipart/form-data)|

### 请求参数说明

|参数名|类型|参数限制|描述|
|:-----|:----|:-------|:----|
|customer_photo |base64 string |必填 |需要上传的照片，目前只支持PNG和JPEG |
|engage_location_id|string|选填 |engage_location_id|
|customer_name |String |必填 |用户名称|
|phone_number |String|必填|电话号码|
|device_type |String|必填|设备类型 比如：Android|
|trigger_type |String|必填|触发 比如：beacon|
|known_customer |bool|必填|是否已知客户|
|appointment |bool|必填|是否预约|
|time_stamp |string|必填|什么时候来|
|time_span |int|必填|什么时候过期，是以分钟为单位的|

```go
{
	"engage_location_id": "St20001",
	"time_stamp": "2018-04-24T15:00:00.000Z",
	"time_span": 60,
	"trigger_type":"beacon",
	"known_customer": false,
	"customer_name": "John Doe",
	"phone_number": "1891",
	"device_type": "Android",
	"appointment": true,
	"customer_photo":"data:image/jpeg;base64,/9j/2wCEAAg……………………………………………………"
	
}
```

### 返回参数说明

|参数名|类型|参数限制|描述|
|:-----|:----|:-------|:----|
|status|String |必填|响应码:true 或者 false 除了0，都是false|
|msg|String |必填|返回信息。返回信息。<br />0-成功<br />1-未得到图像<br />2-数据传输不正确<br />  3-AI服务器异常！<br />4-无权访问<br />5-Uid不可以为空或已经存在<br />6-用户名称不能为空。<br />7-结束时间小于当前时间。<br />9-其它异常。|


### 例子
	curl -H "Content-Type:application/json" -X POST -d   '{   "engage_location_id": "St20001",    "time_stamp": "2018-04-24T15:00:00.000Z",	"time_span": 60,    "trigger_type":"beacon",	"known_customer": false,    "customer_name": "John Doe",   "phone_number": "1891",	"device_type": "Android",	"appointment": true,	"customer_photo":"data:image/jpe…………}'   "http://127.0.0.1:8088/sprint/add" -i

返回结果
```go
	{
        "data":nil,
        "msg": "0-OK！",
        "status": true
    }
```
## 2）把用户信息添加到热搜接口
## 基本描述
|接口说明|用户信息添加接口|
|---|---|
|请求URL|加密链接 https://{host-name}/sprint/Hotadd <br />普通链接： http://{host-name}/sprint/Hotadd|
|协议方法|Post (multipart/form-data)|


### 时间说明

```go
const (
    ANSIC       = "Mon Jan _2 15:04:05 2006"
    UnixDate    = "Mon Jan _2 15:04:05 MST 2006"
    RubyDate    = "Mon Jan 02 15:04:05 -0700 2006"
    RFC822      = "02 Jan 06 15:04 MST"
    RFC822Z     = "02 Jan 06 15:04 -0700" // RFC822 with numeric zone
    RFC850      = "Monday, 02-Jan-06 15:04:05 MST"
    RFC1123     = "Mon, 02 Jan 2006 15:04:05 MST"
    RFC1123Z    = "Mon, 02 Jan 2006 15:04:05 -0700" // RFC1123 with numeric zone
    RFC3339     = "2006-01-02T15:04:05Z07:00"
    RFC3339Nano = "2006-01-02T15:04:05.999999999Z07:00"
    Kitchen     = "3:04PM"
    // Handy time stamps.
    Stamp      = "Jan _2 15:04:05"
    StampMilli = "Jan _2 15:04:05.000"
    StampMicro = "Jan _2 15:04:05.000000"
    StampNano  = "Jan _2 15:04:05.000000000"
)
```
### 请求参数说明

|参数名|类型|参数限制|描述|
|:-----|:----|:-------|:----|
|customer_photo |base64 string |必填 |需要上传的照片，目前只支持PNG和JPEG |
|engage_location_id|string|选填 |engage_location_id|
|customer_name |String |必填 |用户名称|
|phone_number |String|必填|电话号码|
|device_type |String|必填|设备类型 比如：Android|
|trigger_type |String|必填|触发 比如：beacon|
|known_customer |bool|必填|是否已知客户|
|appointment |bool|必填|是否预约|
|time_stamp |string|必填|什么时候来|
|time_span |int|必填|什么时候过期，是以分钟为单位的|

```go
{
	"engage_location_id": "St20001",
	"time_stamp": "2018-04-24T15:00:00.000Z",
	"time_span": 60,
	"trigger_type":"beacon",
	"known_customer": false,
	"customer_name": "John Doe",
	"phone_number": "1891",
	"device_type": "Android",
	"appointment": true,
	"customer_photo":"data:image/jpeg;base64,/9j/2wCEAAg……………………………………………………"
	
}
```

### 返回参数说明

|参数名|类型|参数限制|描述|
|:-----|:----|:-------|:----|
|status|String |必填|响应码:true 或者 false 除了0，都是false|
|msg|String |必填|返回信息。返回信息。<br />0-成功<br />1-未得到图像<br />2-数据传输不正确<br />  3-AI服务器异常！<br />4-无权访问<br />5-Uid不可以为空或已经存在<br />6-用户名称不能为空。<br />7-结束时间小于当前时间。<br />9-其它异常。|


### 例子
	curl -H "Content-Type:application/json" -X POST -d   '{   "engage_location_id": "St20001",    "time_stamp": "2018-04-24T15:00:00.000Z",	"time_span": 60,    "trigger_type":"beacon",	"known_customer": false,    "customer_name": "John Doe",   "phone_number": "1891",	"device_type": "Android",	"appointment": true,	"customer_photo":"data:image/jpe…………}'   "http://127.0.0.1:8088/sprint/hotadd" -i


返回结果
```go
	{
        "data":nil,
        "msg": "0-OK！",
        "status": true
    }
```

## 3）删除一条用户信息接口

## 基本描述

| 接口说明 | 类型信息取得接口 |
|:---|:---|
| 请求URL|  http://{host-name}/sprint/delete|
| 协议方法 | Delete |

### 请求参数说明

|参数名|类型|参数限制|描述|
|:-----|:----|:-------|:----|
|uid|string|必填 |这个用户的id号码 |

### 返回参数说明

| 参数名 | 类型 | 参数限制 |  描述    |
|:---:|:---|:---:|---|
|status|String |必填|响应码:true 或者 false|
|msg|String |必填|返回信息。<br />0-成功<br />4-无权访问<br />5-Uid不可以为空！|

### 例子

	curl -X DELETE -F "uid=1" http://127.0.0.1:8088/sprint/delete -i


## 4）图像识别接口

## 基本描述

| 接口说明 | 类型信息取得接口 |
|:---|:---|
| 请求URL|  http://{host-name}/sprint/detect|
| 协议方法 | Post |

### 请求参数说明

|参数名|类型|参数限制|描述|
|:-----|:----|:-------|:----|
|avatar|file|必填 |需要识别的图片 |


### 返回参数说明

| 参数名 | 类型 | 参数限制 |  描述    |
|:---:|:---|:---:|---|
|status|String |必填|响应码:true 或者 false|
|msg|String |必填|返回信息。<br />0-OK<br />1-Unknown<br/>2-数据传输不正确，图片未传输<br />3--AI服务器异常！<br />4-无权访问<br />9-识别错误|
|data | json| 必填 | 返回列表,注意这个是一个数组 |


### 例子

	curl -X POST -F "avatar=@gates.jpeg" http://127.0.0.1:8088/sprint/detect -i

### 返回列表

```go
	{
    "data": [
		{
		    "sprint": {
			"id": 21,
			"engage_location_id": "S0001",
			"customer_name": "John Doe",
			"phone_number": "1891",
			"known_customer": false,
			"time_stamp": "2018-04-24T23:00:00+08:00",
			"time_span": 0,
			"trigger_type": "beacon",
			"device_type": "Android",
			"appointment": true
		    },
		    "similarity": 1,
		    "aitype": "hot", // 此处返回 hot ,primary，unknown
		    "rect": {
			"height": 41,
			"width": 41,
			"x": 64,
			"y": 33
		    }
		}
	    ],
	    "msg": "0-OK",
	    "status": true
	}
```

## 5）Unknown信息添加接口
## 基本描述
|接口说明|用户信息添加接口|
|---|---|
|请求URL|加密链接 https://{host-name}/sprint/unknowadd <br />普通链接： http://{host-name}/sprint/unknowadd|
|协议方法|Post (multipart/form-data)|

### 请求参数说明



|参数名|类型|参数限制|描述|
|:-----|:----|:-------|:----|
|avatar |file |必填 |需要上传的照片 |
|name |String |选填 |用户名称|
|phone |String|选填|电话号码|
|expire |string|必填|什么时候过期|

### 返回参数说明

|参数名|类型|参数限制|描述|
|:-----|:----|:-------|:----|
|status|String |必填|响应码:true 或者 false 除了0，都是false|
|msg|String |必填|返回信息。<br />0-成功<br />1-未得到图像<br />2-数据传输不正确<br />  3-AI服务器异常！<br />4-无权访问<br />5-过期时间不能为空<br />7-结束时间小于当前时间<br />9-其它异常。|




### 例子
	  curl  -X POST  -F "avatar=@gates.jpeg" -F "uid=1" -F "name=张三" -F "phone=13901234567" -F "expire=2019-01-01 00:00:00" "http://127.0.0.1:8088/sprint/unknowadd" -i


### 返回结果

```go
	{
        "data":nil,
        "msg": "0-OK！",
        "status": true
    }
```



## 6）Unknown信息修改接口
## 基本描述
|接口说明|用户信息添加接口|
|---|---|
|请求URL|加密链接 https://{host-name}/sprint/unknowupdate <br />普通链接： http://{host-name}/sprint/unknowupdate|
|协议方法|Post (multipart/form-data)|

### 请求参数说明



|参数名|类型|参数限制|描述|
|:-----|:----|:-------|:----|
|id |int |必填 |需要上传的照片 |
|name |String |选填 |用户名称|
|phone |String|选填|电话号码|

### 返回参数说明

|参数名|类型|参数限制|描述|
|:-----|:----|:-------|:----|
|status|String |必填|响应码:true 或者 false 除了0，都是false|
|msg|String |必填|返回信息。<br />0-成功<br />2-数据传输不正确<br />  3-AI服务器异常！<br />4-无权访问<br />9-其它异常。|




### 例子
	  curl  -X POST  -F "id=1" -F "name=张三" -F "phone=13901234567" "http://127.0.0.1:8088/sprint/unknowupdate" -i


### 返回结果

```go
	{
	    "data": [
		{
		    "sprint": {
		        "id": 39,
		        "customer_name": "",
		        "phone_number": ""
		    },
		    "similarity": 0,
		    "aitype": "",
		    "rect": {
		        "height": 0,
		        "width": 0,
		        "x": 0,
		        "y": 0
		    }
		}
	    ],
	    "msg": "0-OK!",
	    "status": true
	}
```


## 7）删除一条用户信息接口

## 基本描述

| 接口说明 | 类型信息取得接口 |
|:---|:---|
| 请求URL|  http://{host-name}/sprint/unknowdelete|
| 协议方法 | Delete |

### 请求参数说明

|参数名|类型|参数限制|描述|
|:-----|:----|:-------|:----|
|uid|string|必填 |这个用户的id号码 |

### 返回参数说明

| 参数名 | 类型 | 参数限制 |  描述    |
|:---:|:---|:---:|---|
|status|String |必填|响应码:true 或者 false|
|msg|String |必填|返回信息。<br />0-成功<br />4-无权访问<br />5-id不可以为空！|

### 例子

	curl -X DELETE -F "uid=1" http://127.0.0.1:8088/sprint/unknowdelete  -i