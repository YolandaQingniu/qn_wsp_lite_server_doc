# Yolanda WSP Docker SDK 接入文档

> 最新SDK版本号: 1.2.3
>
> 更新日志：
>
> 1.2.3 => 支持血压计。

本SDK通过Docker发布.

Docker是开源的容器平台.

它让开发者可以打包应用进容器. 

容器简化了应用的分发.

## 角色
本SDK存在4类角色.
1. Scale
2. App
3. Docker
4. Server

## 请求流程
1.App <----> Scale
+ APP通过蓝牙发送数据到Scale, 告知Scale Docker的域名.
+ Scale通过蓝牙发送数据到APP.

2.Scale ----> Docker
+ 血压计通过HTTP请求发送加密数据到Docker. 
+ Docker解密并解析数据, 然后发回响应给血压计.

3.Server <----> App
+ APP通过HTTP请求发送数据到服务器.
+ Server响应APP请求.


## 血压计接口 (Scale -> Docker)

接口:
```text
GET https://your-docker-server.com/yolanda/bps?code={encrypted_data}
```

## 接口 (Docker <-> Server)

### 上传数据（Docker -> Server)

#### 接口
```text
POST https://your-business-server.com/bps/measurements
```

#### 示例
```json
{
	"blood_pressures": [{
		"user_index": 1,
		"systolic": 103,
		"diastolic": 78,
		"blood_unit": "mmHg",
		"pulse": 73,
		"timestamp": 1669975757,
		"mac": "10:91:A8:3E:E1:CD"
	}, {
		"user_index": 1,
		"systolic": 123,
		"diastolic": 75,
		"blood_unit": "mmHg",
		"pulse": 74,
		"timestamp": 1669976062,
		"mac": "10:91:A8:3E:E1:CD"
	}]
}
```

#### 详情
| 字段名     | 类型    | 是否必填 | 介绍               | 额外说明                     |
|------------|---------|----------|--------------------|------------------------------|
| user_index | integer  | Y        | 用户索引（用户1或用户2）    | mock: 1                 |
| systolic   | integer  | Y        | 收缩压/高压               | mock: 120               |
| diastolic  | integer  | Y        | 舒张压/低压               | mock: 78                |
| blood_unit | string   | Y        | 血压单位                  | mock: mmHg             |
| pulse      | integer  | Y        | 脉搏                     | mock: 77                |
| timestamp  | integer  | Y        | 测量时间戳 (s)            | mock: 1669975757        |
| mac        | string   | Y        | Mac                     | mock: 10:91:A8:3E:E1:CD  |

#### 成功响应
+ 头部
```text
Content-Type：text/plain; charset=utf-8
```

+ 正文
```text
success
```

#### 失败响应
+ 头部
```text
Content-Type：text/plain; charset=utf-8
```

+ 正文
```text
fail
```
