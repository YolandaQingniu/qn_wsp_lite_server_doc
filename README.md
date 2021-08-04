# Yolanda WSP Docker SDK 接入文档

最新SDK版本号: 1.0.0

## 交互原理
1. 通过本SDK搭建WSP服务器
2. 接入8电极的客户参照本SDK文档-8电极服务器交互实现客户自身的服务器; 其他客户参照本SDK文档-通用服务器交互实现客户自身的服务器
3. 秤通过APP-SDK配置的固定URL访问WSP服务器, WSP服务器解析秤端加密的指令, 并传输给到三方服务器
4. 第三方服务器发送"指标计算"请求到WSP服务器获取指标信息

## 部署WSP服务器
第一步: 创建目录并进入目录

```sh
mkdir yolanda_wsp_lite && cd yolanda_wsp_lite
```

第二步: 创建Dockerfile文件并写入

```dockerfile
FROM registry.cn-shenzhen.aliyuncs.com/yolanda/wsp-lite:v1.0.0
ENV CLIENT_URL="https://your-business-server.com"
ENV CLIENT_KEY="Yolanda提供的16位SDK密钥"
```

第三步: 启动容器

```shell
docker build . -t yolanda_wsp_lite
docker run -p 8080:8080 yolanda_wsp_lite
```

第四步: 将容器服务对外映射为一个域名, 如"https://your-wsp-server.com",
根据是否配置https, 将秤端交互地址提供给APP开发人员.
+ "https://your-wsp-server.com:443/yolanda/wsp?code="
+ "http://your-wsp-server.com:80/yolanda/wsp?code="

补充说明:
1. Docker的日志存于容器内log/access.log, 按天进行存储
2. Docker不依赖数据库和Redis缓存
3. Docker内端口号为8080, 外部容器号可在启动Docker时自行配置

---

## 秤端交互

请求方式：`GET `

WSP域名(需配置)：`"https://your-wsp-server.com" `

URI：`/yolanda/wsp?code=`

---

## 通用服务器交互

#### 上传测量数据（WSP服务器 => 业务服务器)

请求方式：`POST`

业务域名：`https://your-business-server.com`

URI：`/wsp/measurements`

请求头：`Content-Type：application/json`

请求体：

```json
{
  "mac": "12:34:56:78:9A:BC",
  "model_id": "0E2B",
  "timestamp": 1527855059,
  "weight": 55.0,
  "heart_rate": 70,
  "hmac": "183476B32E22B26989AD4744C3AE13CCA857EB021C9290CFC83FF46A544D999BFF09F3DE988D58C2F435BFC70DB4C54CF018E35A5AE2A63A1D094350EFB2540908026F908CA690B81D5C13E1A4371D0E710F91DA67CFBDD55F50D593D1075CA23A2572F541B5047FD6FB4E59870740197628E2680EDCFFE4FB2F3F8CB31C8F22B16787F9AE44C1CB1E5D99714211F5370EBE1CB9CD8A760E8BAE668A515E2B85"
}
```

请求参数说明:

| 名称       | 类型    | 是否必须 | 备注             | 其他信息                     |
| ---------- | ------- | -------- | ---------------- | ---------------------------- |
| mac        | string  | 必须     | Mac地址          | mock: 12:34:56:78:9A:BC      |
| model_id   | string  | 必须     | 型号ID(4位)      | mock: 0E2B                   |
| timestamp  | integer | 必须     | 测量时间戳(秒级) | mock: 1582698882             |
| weight     | number  | 必须     | 体重(单位:kg)    | mock: 55.0                   |
| heart_rate | integer | 必须     | 心率(次/分)      | mock: 70                     |
| hmac       | string  | 必须     | 签名(Hmac)       | mock: 183476B32E22B26989A... |



业务服务器成功响应：

- 响应头

  `Content-Type：text/plain; charset=utf-8`

- 响应体：
  	`success`



业务服务器失败响应：

- 响应头

  `Content-Type：text/plain; charset=utf-8`

- 响应体：
  	`fail`



#### 计算测量指标（业务服务器 => WSP服务器)

请求方式：`GET`

WSP域名(需配置)：`"https://your-wsp-server.com" `

URI：`/yolanda/wsp/reports`

请求头：`Content-Type：application/json`

请求体：

```json
{
  "weight": 58.9,
  "height": 175.2,
  "age": 20,
  "gender": 1,
  "hmac": "183476B32E22B26989AD4744C3AE13CCA857EB021C9290CFC83FF46A544D999BFF09F3DE988D58C2F435BFC70DB4C54CF018E35A5AE2A63A1D094350EFB2540908026F908CA690B81D5C13E1A4371D0E710F91DA67CFBDD55F50D593D1075CA23A2572F541B5047FD6FB4E59870740197628E2680EDCFFE4FB2F3F8CB31C8F22B16787F9AE44C1CB1E5D99714211F5370EBE1CB9CD8A760E8BAE668A515E2B85"
}
```

请求参数说明:

| 名称   | 类型    | 是否必须 | 备注                   | 其他信息                     |
| ------ | ------- | -------- | ---------------------- | ---------------------------- |
| weight | number  | 必须     | 体重(单位:kg)          | mock: 58.9                   |
| height | number  | 必须     | 身高(单位:cm)          | mock: 175.2                  |
| age    | integer | 必须     | 年龄                   | mock: 20                     |
| gender | integer | 必须     | 性别(0表示女，1表示男) | mock: 1                      |
| hmac   | string  | 必须     | 签名(Hmac)             | mock: 183476B32E22B26989A... |



WSP服务器成功响应：

- 响应头

  `Content-Type：text/plain; charset=utf-8`

- 响应体：

  ```json
  {
    "code": 0,
    "data": {
      "reports": {
        "weight": 58.2,
        "bmi": 20.1,
        "bodyfat": 14,
        "fat_free_weight": 50.1,
        "subfat": 12.7,
        "visfat": 3.4671535888517173,
        "water": 62.2,
        "bmr": 1451,
        "muscle": 55.6,
        "sinew": 47.5,
        "bone": 2.51,
        "protein": 19.5,
        "score": 90.2,
        "bodyage": 20,
        "body_shape": 4,
        "heart_rate": 0,
        "cardiac_index": 0
      }
    },
    "msg": "ok"
  }
  ```



WSP服务器失败响应：

- 响应头

  `Content-Type: application/json`

- 响应体：

  ```json
  {
    "code": 10002,
    "data": {},
    "msg": "参数类型错误或缺少参数"
  }
  ```



返回值说明:

| 名称            | 类型    | 是否必须 | 备注         | 其他信息   |
| --------------- | ------- | -------- | ------------ | ---------- |
| code            | integer | 必须     | 错误码       | mock：0    |
| msg             | string  | 必须     | 错误说明     | mock：ok   |
| data            | object  | 必须     | 业务数据     | mock：{}   |
| reports         | object  | 必须     | 测量指标     | mock：{}   |
| weight          | number  | 必须     | 体重         | mock：58.2 |
| bmi             | number  | 必须     | 身体质量指数 | mock：20.1 |
| bodyfat         | number  | 必须     | 体脂率       | mock：14   |
| fat_free_weight | number  | 必须     | 去脂体重     | mock：50.1 |
| subfat          | number  | 必须     | 皮下脂肪     | mock：12.7 |
| visfat          | number  | 必须     | 内脏脂肪     | mock：3.46 |
| water           | number  | 必须     | 体水分       | mock：62.2 |
| bmr             | integer | 必须     | 基础代谢率   | mock：1451 |
| muscle          | number  | 必须     | 骨骼肌率     | mock：55.6 |
| sinew           | number  | 必须     | 肌肉量       | mock：47.5 |
| bone            | number  | 必须     | 骨量         | mock：2.51 |
| protein         | number  | 必须     | 蛋白质       | mock：19.5 |
| score           | number  | 必须     | 分数         | mock：90.2 |
| bodyage         | integer | 必须     | 体年龄       | mock：20   |
| body_shape      | integer | 必须     | 体型         | mock：4    |
| heart_rate      | integer | 必须     | 心率         | mock：0    |
| cardiac_index   | number  | 必须     | 心脏指数     | mock：0    |


### 八电极服务器交互

#### 八电极秤上传测量数据（WSP服务器 => 业务服务器)

请求方式：`POST`

业务域名：`https://your-business-server.com`

URI：`/wsp/eight_measurements`

请求头：`Content-Type: application/json`

请求体：

```json
{
  "mac": "12:34:56:78:9A:BC",
  "model_id": "0E2B",
  "timestamp": 1527855059,
  "weight": 55.0,
  "heart_rate": 70,
  "hmac": "183476B32E22B26989AD4744C3AE13CCA857EB021C9290CFC83FF46A544D999BFF09F3DE988D58C2F435BFC70DB4C54CF018E35A5AE2A63A1D094350EFB2540908026F908CA690B81D5C13E1A4371D0E710F91DA67CFBDD55F50D593D1075CA23A2572F541B5047FD6FB4E59870740197628E2680EDCFFE4FB2F3F8CB31C8F22B16787F9AE44C1CB1E5D99714211F5370EBE1CB9CD8A760E8BAE668A515E2B85"
}
```

请求参数说明:

| 名称       | 类型    | 是否必须 | 备注             | 其他信息                     |
| ---------- | ------- | -------- | ---------------- | ---------------------------- |
| mac        | string  | 必须     | Mac地址          | mock: 12:34:56:78:9A:BC      |
| model_id   | string  | 必须     | 型号ID(4位)      | mock: 0E2B                   |
| timestamp  | integer | 必须     | 测量时间戳(秒级) | mock: 1582698882             |
| weight     | number  | 必须     | 体重(单位:kg)    | mock: 55.0                   |
| heart_rate | integer | 必须     | 心率(次/分)      | mock: 70                     |
| hmac       | string  | 必须     | 签名(Hmac)       | mock: 183476B32E22B26989A... |



业务服务器成功响应：

- 响应头

  `Content-Type：text/plain; charset=utf-8`

- 响应体：
  	`success`



业务服务器失败响应：

- 响应头

  `Content-Type：text/plain; charset=utf-8`

- 响应体：
  	`fail`



#### 八电极秤计算测量指标（业务服务器 => WSP服务器)

请求方式：`GET`

WSP域名(需配置)：`"https://your-wsp-server.com" `

URI：`/yolanda/wsp/eight_reports`

请求头：`Content-Type：application/json`

请求体：

```json
{
  "weight": 58.9,
  "height": 175.2,
  "age": 20,
  "gender": 1,
  "hmac": "183476B32E22B26989AD4744C3AE13CCA857EB021C9290CFC83FF46A544D999BFF09F3DE988D58C2F435BFC70DB4C54CF018E35A5AE2A63A1D094350EFB2540908026F908CA690B81D5C13E1A4371D0E710F91DA67CFBDD55F50D593D1075CA23A2572F541B5047FD6FB4E59870740197628E2680EDCFFE4FB2F3F8CB31C8F22B16787F9AE44C1CB1E5D99714211F5370EBE1CB9CD8A760E8BAE668A515E2B85"
}
```

- 请求参数说明:

| 名称   | 类型    | 是否必须 | 备注                   | 其他信息                     |
| ------ | ------- | -------- | ---------------------- | ---------------------------- |
| weight | number  | 必须     | 体重(单位:kg)          | mock: 58.9                   |
| height | number  | 必须     | 身高(单位:cm)          | mock: 175.2                  |
| age    | integer | 必须     | 年龄                   | mock: 20                     |
| gender | integer | 必须     | 性别(0表示女，1表示男) | mock: 1                      |
| hmac   | string  | 必须     | 签名(Hmac)             | mock: 183476B32E22B26989A... |



WSP服务器成功响应：

- 响应头

  `Content-Type: application/json`

- 响应体：

  ```json
  {
    "code": 0,
    "data": {
      "reports": {
        "weight": 58.9,
        "bmi": 20.1,
        "bodyfat": 14,
        "fat_free_weight": 50.1,
        "subfat": 12.7,
        "visfat": 3.4671535888517173,
        "water": 62.2,
        "bmr": 1451,
        "muscle": 55.6,
        "sinew": 47.5,
        "bone": 2.51,
        "protein": 19.5,
        "score": 90.2,
        "bodyage": 20,
        "body_shape": 4,
        "sinew_right_arm": 0.8,
        "sinew_left_arm": 0.8,
        "sinew_right_leg": 0.8,
        "sinew_left_leg": 0.8,
        "sinew_trunk": 0.8,
        "bodyfat_right_arm": 0.8,
        "bodyfat_left_arm": 0.8,
        "bodyfat_right_leg": 0.8,
        "bodyfat_left_leg": 0.8,
        "bodyfat_trunk": 0.8,
        "paunch": 0
      }
    },
    "msg": "ok"
  }
  ```



WSP服务器失败响应：

- 响应头

  `Content-Type：text/plain; charset=utf-8`

- 响应体：

  ```json
  {
    "code": 10002,
    "data": {},
    "msg": "参数类型错误或缺少参数"
  }
  ```



返回值说明:

| 名称              | 类型    | 是否必须 | 备注          | 其他信息   |
| ----------------- | ------- | -------- | ------------- | ---------- |
| code              | integer | 必须     | 错误码        | mock：0    |
| msg               | string  | 必须     | 错误说明      | mock：ok   |
| data              | object  | 必须     | 业务数据      | mock：{}   |
| reports           | object  | 必须     | 测量指标      | mock：{}   |
| weight            | number  | 必须     | 体重(单位:kg) | mock：58.2 |
| bmi               | number  | 必须     | 身体质量指数  | mock：20.1 |
| bodyfat           | number  | 必须     | 体脂率        | mock：14   |
| fat_free_weight   | number  | 必须     | 去脂体重      | mock：50.1 |
| subfat            | number  | 必须     | 皮下脂肪      | mock：12.7 |
| visfat            | number  | 必须     | 内脏脂肪      | mock：3.46 |
| water             | number  | 必须     | 体水分        | mock：62.2 |
| bmr               | integer | 必须     | 基础代谢率    | mock：1451 |
| muscle            | number  | 必须     | 骨骼肌率      | mock：55.6 |
| sinew             | number  | 必须     | 肌肉量        | mock：47.5 |
| bone              | number  | 必须     | 骨量          | mock：2.51 |
| protein           | number  | 必须     | 蛋白质        | mock：19.5 |
| score             | number  | 必须     | 分数          | mock：90.2 |
| bodyage           | integer | 必须     | 体年龄        | mock：20   |
| body_shape        | integer | 必须     | 体型          | mock：4    |
| sinew_right_arm   | number  | 必须     | 右上肢肌肉量  | mock：0.8  |
| sinew_left_arm    | number  | 必须     | 左上肢肌肉量  | mock：0.8  |
| sinew_right_leg   | number  | 必须     | 右下肢肌肉量  | mock：0.8  |
| sinew_left_leg    | number  | 必须     | 左下肢肌肉量  | mock：0.8  |
| sinew_trunk       | number  | 必须     | 躯干肌肉量    | mock：0.8  |
| bodyfat_right_arm | number  | 必须     | 右上肢脂肪量  | mock：0.8  |
| bodyfat_left_arm  | number  | 必须     | 左上肢脂肪量  | mock：0.8  |
| bodyfat_right_leg | number  | 必须     | 右下肢脂肪量  | mock：0.8  |
| bodyfat_left_leg  | number  | 必须     | 左下肢脂肪量  | mock：0.8  |
| bodyfat_trunk     | number  | 必须     | 躯干脂肪量    | mock：0.8  |
| pauch             | number  | 必须     | 肥胖度        | mock：0    |

