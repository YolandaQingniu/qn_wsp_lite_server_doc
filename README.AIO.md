# Yolanda WSP Docker SDK 接入文档

> 最新SDK版本号: 1.2.5
>
> 更新日志：
>
> 1.2.5 => 支持扫码信息
> 
> 1.2.4 => 支持身高一体机

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
+ 秤通过HTTP请求发送加密数据到Docker. 
+ Docker解密并解析数据, 然后发回响应给秤.

3.Server <----> App
+ APP通过HTTP请求发送数据到服务器.
+ Server响应APP请求.

## 如何创建Docker服务?
1. 安装Docker. 参考 https://www.docker.com/get-started

2. 创建一个目录并进入目录.
```sh
mkdir yolanda_wsp_lite && cd yolanda_wsp_lite
```

3. vim Dockerfile
```dockerfile
FROM registry.cn-shenzhen.aliyuncs.com/yolanda_open/wsp-lite:v1.2.5
ENV CLIENT_URL="https://your-business-server.com"
ENV CLIENT_ID="A_CLIENT_ID_FROM_YOLANDA_PLEASE_CONTACT_US"
ENV SECRET_KEY="secret_key"
ENV TZ="Asia/Shanghai"
```

> 贴士: 默认时区是UTC, 可以通过环境变量`TZ`进行更改. 容器日志存储在 log/access.log, 按天进行切割, 并保留7天日志. 容器内端口号为8080, 外部访问端口号可在启动Docker时自行配置.

> 注：CLIENT_URL为接收方业务服务器地址；CLIENT_ID、SECRET_KEY为伊欧乐分配的ID和密钥


4. 构建并执行.
```shell
docker build . -t yolanda_wsp_lite
docker run -p 8080:8080 yolanda_wsp_lite
```

5. 映射应用为域名

示例: 0.0.0.0:8080 映射为 https://your-docker-server.com 或 http://your-docker-server.com

## 身高一体机接口 (Scale -> Docker)

接口:
```text
GET https://your-docker-server.com/yolanda/aios?code={encrypted_data}
```

## 接口 (Docker <-> Server)

### 上传数据（Docker -> Server)

#### 接口
```text
POST https://your-business-server.com/aios/measurements
```

#### 示例
```json
{
  "mac": "12:34:56:78:9A:BC",
  "timestamp": 1666841725,
  "weight": 79.7,
  "height": 172.5,
  "hmac": "FCAC4733C0DDE4AC1ACB3A46A2F81925BA2FEF981EAE5A0CADBE8663FF37FE3CC5C989ACC5EA8A436A952DFEEAF7B2D5D1FEABD1F5721B1664CAFFDD4BA42061B0C409CBA609664FFA70FD99B7C444D73BFD63D8D7652CF4F9D52B02DF7A984117E896976F860FB91497752A667750D1877B454F8A3F3C4485C3BF9DEDA6EE33",
  "scan_code": "6922711079073"
}
```

#### 详情
| 字段名     | 类型    | 是否必填 | 介绍               | 额外说明                     |
|------------|---------|----------|--------------------|------------------------------|
| mac        | string  | Y        | Mac                | mock: 12:34:56:78:9A:BC      |
| timestamp  | integer | Y        | 测量时间戳 (s)     | mock: 1666841725             |
| weight     | number  | Y        | 体重 (kg)          | mock: 79.7                   |
| height | integer | Y        | 身高 (cm)         | mock: 172.5                     |
| hmac       | string  | Y        | 签名               | mock: 183476B32E22B26989A... |
| scan_code  | string  | N        | 扫码信息            | mock: 6922711079073         |

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
### 计算指标（Server -> Docker)

#### 1、WIFI上传数据解析接口
```text
POST https://your-docker-server.com/yolanda/aios/reports
```

#### 示例
```json
{
  "weight": 79.7,
  "age": 25,
  "height": 172.5,
  "gender": 1,
  "hmac": "FCAC4733C0DDE4AC1ACB3A46A2F81925BA2FEF981EAE5A0CADBE8663FF37FE3CC5C989ACC5EA8A436A952DFEEAF7B2D5D1FEABD1F5721B1664CAFFDD4BA42061B0C409CBA609664FFA70FD99B7C444D73BFD63D8D7652CF4F9D52B02DF7A984117E896976F860FB91497752A667750D1877B454F8A3F3C4485C3BF9DEDA6EE33"
}
```

#### 详情
| 字段名 | 类型    | 是否必填 | 介绍                   | 额外说明                     |
|--------|---------|----------|------------------------|------------------------------|
| weight | number  | Y        | 体重 (kg)              | mock: 79.7                  |
| height | number  | Y        | 身高 (cm)              | mock: 172.5                  |
| age    | integer | Y        | 年龄                   | mock: 25                     |
| gender | integer | Y        | 性别(0: 女性，1: 男性) | mock: 1                      |
| hmac   | string  | Y        | 签名                   | mock: 183476B32E22B26989A... |


#### 成功响应
+ 头部
```text
Content-Type：text/plain; charset=utf-8
```

+ 正文
```json
{
  "code": 0,
  "data": {
    "reports": {
      "weight": 79.7,
      "bmi": 26.8,
      "bodyfat": 26.5,
      "lbm": 58.6,
      "subfat": 23.4,
      "visfat": 9.707964860323466,
      "water": 53.1,
      "bmr": 1635,
      "muscle": 47.5,
      "sinew": 55.6,
      "bone": 2.93,
      "protein": 16.7,
      "score": 75,
      "body_age": 26,
      "body_shape": 6,
      "heart_rate": 0,
      "cardiac_index": 0
    }
  },
  "msg": "ok"
}
```

#### 失败响应
+ 头部
```text
Content-Type: application/json
```

+ 正文
```json
{
  "code": 10002,
  "data": {},
  "msg": "Missing params or params error"
}
```

#### 详情
| 字段名        | 类型    | 是否必填 | 介绍     | 额外说明   |
| ------------- | ------- | -------- | -------- | ---------- |
| code          | integer | Y        | Code     | mock：0    |
| msg           | string  | Y        | Message  | mock：ok   |
| data          | object  | Y        | Data     | mock：{}   |
| reports       | object  | Y        | Reports  | mock：{}   |
| weight        | number  | Y        | 体重     | mock：58.2 |
| bmi           | number  | Y        | BMI      | mock：20.1 |
| bodyfat       | number  | Y        | 体脂率   | mock：14   |
| lbm           | number  | Y        | 去脂体重 | mock：50.1 |
| subfat        | number  | Y        | 皮下脂肪 | mock：12.7 |
| visfat        | number  | Y        | 内脏脂肪 | mock：3.46 |
| water         | number  | Y        | 水分     | mock：62.2 |
| bmr           | integer | Y        | 基础代谢 | mock：1451 |
| muscle        | number  | Y        | 骨骼肌率 | mock：55.6 |
| sinew         | number  | Y        | 肌肉量   | mock：47.5 |
| bone          | number  | Y        | 骨量     | mock：2.51 |
| protein       | number  | Y        | 蛋白质   | mock：19.5 |
| score         | number  | Y        | 分数     | mock：90.2 |
| body_age      | integer | Y        | 体年龄   | mock：20   |
| body_shape    | integer | Y        | 体型     | mock：4    |
| heart_rate    | integer | Y        | 心率     | mock：0    |
| cardiac_index | number  | Y        | 心脏指数 | mock：0    |


#### 2、秤端实时测量数据解析接口
```text
POST https://your-docker-server.com/yolanda/aios/aio_scale_reports
```

#### 示例
```json
{
  "age": 25,
  "gender": 1,
  "scale": "10105003E80101F401F6000000000025120f20a097c138c1a4380338000f58"
}
```

#### 详情
| 字段名 | 类型    | 是否必填 | 介绍                   | 额外说明                     |
|--------|---------|----------|------------------------|------------------------------|
| age    | integer | Y        | 年龄                   | mock: 25                     |
| gender | integer | Y        | 性别(0: 女性，1: 男性) | mock: 1                      |
| scale  | string  | Y        | scale字符串为10和12命令的拼接，10在前           | mock: 10105003E80101F401F6000000000025120f20a097c138c1a4380338000f58 |


#### 成功响应
+ 头部
```text
Content-Type：text/plain; charset=utf-8
```

+ 正文
```json
{
  "code": 0,
  "data": {
    "reports": {
      "weight": 79.7,
      "bmi": 26.8,
      "bodyfat": 26.5,
      "lbm": 58.6,
      "subfat": 23.4,
      "visfat": 9.707964860323466,
      "water": 53.1,
      "bmr": 1635,
      "muscle": 47.5,
      "sinew": 55.6,
      "bone": 2.93,
      "protein": 16.7,
      "score": 75,
      "body_age": 26,
      "body_shape": 6,
      "heart_rate": 0,
      "cardiac_index": 0,
      "height": 172.5
    }
  },
  "msg": "ok"
}
```

#### 失败响应
+ 头部
```text
Content-Type: application/json
```

+ 正文
```json
{
  "code": 10002,
  "data": {},
  "msg": "Missing params or params error"
}
```

#### 详情
| 字段名        | 类型    | 是否必填 | 介绍     | 额外说明   |
| ------------- | ------- | -------- | -------- | ---------- |
| code          | integer | Y        | Code     | mock：0    |
| msg           | string  | Y        | Message  | mock：ok   |
| data          | object  | Y        | Data     | mock：{}   |
| reports       | object  | Y        | Reports  | mock：{}   |
| weight        | number  | Y        | 体重     | mock：58.2 |
| bmi           | number  | Y        | BMI      | mock：20.1 |
| bodyfat       | number  | Y        | 体脂率   | mock：14   |
| lbm           | number  | Y        | 去脂体重 | mock：50.1 |
| subfat        | number  | Y        | 皮下脂肪 | mock：12.7 |
| visfat        | number  | Y        | 内脏脂肪 | mock：3.46 |
| water         | number  | Y        | 水分     | mock：62.2 |
| bmr           | integer | Y        | 基础代谢 | mock：1451 |
| muscle        | number  | Y        | 骨骼肌率 | mock：55.6 |
| sinew         | number  | Y        | 肌肉量   | mock：47.5 |
| bone          | number  | Y        | 骨量     | mock：2.51 |
| protein       | number  | Y        | 蛋白质   | mock：19.5 |
| score         | number  | Y        | 分数     | mock：90.2 |
| body_age      | integer | Y        | 体年龄   | mock：20   |
| body_shape    | integer | Y        | 体型     | mock：4    |
| heart_rate    | integer | Y        | 心率     | mock：0    |
| cardiac_index | number  | Y        | 心脏指数 | mock：0    |
| height | number  | Y        | 身高 | mock：172.5    |

#### 3、秤端历史测量数据解析接口
```text
POST https://your-docker-server.com/yolanda/aios/aio_scale_history_reports
```

#### 示例
```json
{
  "age": 25,
  "gender": 1,
  "scale": "23145001e9d7d9631f9a025e024f0006c20200b8120f5066478093174414130e0001c2"
}
```

#### 详情
| 字段名 | 类型    | 是否必填 | 介绍                   | 额外说明                     |
|--------|---------|----------|------------------------|------------------------------|
| age    | integer | Y        | 年龄                   | mock: 25                     |
| gender | integer | Y        | 性别(0: 女性，1: 男性) | mock: 1                      |
| scale  | string  | Y        | scale字符串为23和12命令的拼接，23在前           | mock: 23145001e9d7d9631f9a025e024f0006c20200b8120f5066478093174414130e0001c2 |


#### 成功响应
+ 头部
```text
Content-Type：text/plain; charset=utf-8
```

+ 正文
```json
{
    "code": 0,
    "data": {
      "reports": {
        "weight": 80.9,
        "bmi": 27,
        "bodyfat": 26.8,
        "lbm": 59.2,
        "subfat": 23.7,
        "visfat": 9.939511912860437,
        "water": 52.8,
        "bmr": 1649,
        "muscle": 47.3,
        "sinew": 56.3,
        "bone": 2.96,
        "protein": 16.7,
        "score": 73.9,
        "body_age": 27,
        "body_shape": 6,
        "heart_rate": 0,
        "cardiac_index": 0,
        "height": 173
      }
    },
    "msg": "ok"
}
```

#### 失败响应
+ 头部
```text
Content-Type: application/json
```

+ 正文
```json
{
  "code": 10002,
  "data": {},
  "msg": "Missing params or params error"
}
```

#### 详情
| 字段名        | 类型    | 是否必填 | 介绍     | 额外说明   |
| ------------- | ------- | -------- | -------- | ---------- |
| code          | integer | Y        | Code     | mock：0    |
| msg           | string  | Y        | Message  | mock：ok   |
| data          | object  | Y        | Data     | mock：{}   |
| reports       | object  | Y        | Reports  | mock：{}   |
| weight        | number  | Y        | 体重     | mock：58.2 |
| bmi           | number  | Y        | BMI      | mock：20.1 |
| bodyfat       | number  | Y        | 体脂率   | mock：14   |
| lbm           | number  | Y        | 去脂体重 | mock：50.1 |
| subfat        | number  | Y        | 皮下脂肪 | mock：12.7 |
| visfat        | number  | Y        | 内脏脂肪 | mock：3.46 |
| water         | number  | Y        | 水分     | mock：62.2 |
| bmr           | integer | Y        | 基础代谢 | mock：1451 |
| muscle        | number  | Y        | 骨骼肌率 | mock：55.6 |
| sinew         | number  | Y        | 肌肉量   | mock：47.5 |
| bone          | number  | Y        | 骨量     | mock：2.51 |
| protein       | number  | Y        | 蛋白质   | mock：19.5 |
| score         | number  | Y        | 分数     | mock：90.2 |
| body_age      | integer | Y        | 体年龄   | mock：20   |
| body_shape    | integer | Y        | 体型     | mock：4    |
| heart_rate    | integer | Y        | 心率     | mock：0    |
| cardiac_index | number  | Y        | 心脏指数 | mock：0    |
| height | number  | Y        | 身高 | mock：172.5    |
