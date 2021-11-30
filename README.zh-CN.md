# Yolanda WSP Docker SDK 接入文档

[English](./README.en.md) | [简体中文](./README.zh-CN.md)

> 最新SDK版本号: 1.0.0

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

3.Docker <----> Server
+ Docker通过HTTP请求发送数据到Server.
+ Server接收到Docker的数据.
+ Server通过HTTP请求发送数据到Docker, 以此获取更多指标数据.

4.Server <----> App
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
FROM registry.cn-shenzhen.aliyuncs.com/yolanda_open/wsp-lite:v1.0.0
ENV CLIENT_URL="https://your-business-server.com"
ENV CLIENT_ID="A_CLIENT_ID_FROM_YOLANDA_PLEASE_CONTACT_US"
ENV TZ="Asia/Shanghai"
```

> 贴士: 默认时区是UTC, 可以通过环境变量`TZ`进行更改. 容器日志存储在 log/access.log, 按天进行切割, 并保留7天日志. 容器内端口号为8080, 外部访问端口号可在启动Docker时自行配置.


4. 构建并执行.
```shell
docker build . -t yolanda_wsp_lite
docker run -p 8080:8080 yolanda_wsp_lite
```

5. 映射应用为域名

示例: 0.0.0.0:8080 映射为 https://your-docker-server.com 或 http://your-docker-server.com


## 接口 (Scale -> Docker)

接口:
```text
GET https://your-docker-server.com/yolanda/wsp?code={encrypted_data}
```

## 接口 (Docker <-> Server)

### 上传数据（Docker -> Server)

#### 接口
```text
POST https://your-business-server.com/wsp/measurements
```

#### 示例
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

#### 详情
| 字段名     | 类型    | 是否必填 | 介绍               | 额外说明                     |
|------------|---------|----------|--------------------|------------------------------|
| mac        | string  | Y        | Mac                | mock: 12:34:56:78:9A:BC      |
| model_id   | string  | Y        | 设备型号 (4 chars) | mock: 0E2B                   |
| timestamp  | integer | Y        | 测量时间戳 (s)     | mock: 1582698882             |
| weight     | number  | Y        | 体重 (kg)          | mock: 55.0                   |
| heart_rate | integer | Y        | 心率 (BPM)         | mock: 70                     |
| hmac       | string  | Y        | 签名               | mock: 183476B32E22B26989A... |

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

#### 接口
```text
POST https://your-docker-server.com/yolanda/wsp/reports
```

#### 示例
```json
{
  "weight": 58.9,
  "height": 175.2,
  "age": 20,
  "gender": 1,
  "hmac": "183476B32E22B26989AD4744C3AE13CCA857EB021C9290CFC83FF46A544D999BFF09F3DE988D58C2F435BFC70DB4C54CF018E35A5AE2A63A1D094350EFB2540908026F908CA690B81D5C13E1A4371D0E710F91DA67CFBDD55F50D593D1075CA23A2572F541B5047FD6FB4E59870740197628E2680EDCFFE4FB2F3F8CB31C8F22B16787F9AE44C1CB1E5D99714211F5370EBE1CB9CD8A760E8BAE668A515E2B85"
}
```

#### 详情
| 字段名 | 类型    | 是否必填 | 介绍                   | 额外说明                     |
|--------|---------|----------|------------------------|------------------------------|
| weight | number  | Y        | 体重 (kg)              | mock: 58.9                   |
| height | number  | Y        | 身高 (cm)              | mock: 175.2                  |
| age    | integer | Y        | 年龄                   | mock: 20                     |
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
      "weight": 58.2,
      "bmi": 20.1,
      "bodyfat": 14,
      "lbm": 50.1,
      "subfat": 12.7,
      "visfat": 3.4671535888517173,
      "water": 62.2,
      "bmr": 1451,
      "muscle": 55.6,
      "sinew": 47.5,
      "bone": 2.51,
      "protein": 19.5,
      "score": 90.2,
      "body_age": 20,
      "body_shape": 4,
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


## 接口(8电极)

### 8电极上传数据（Docker -> Server)

#### 接口
```text
POST https://your-business-server.com/wsp/eight_measurements
```

#### 示例
```json
{
  "mac": "12:34:56:78:9A:BC",
  "model_id": "0E2B",
  "timestamp": 1527855059,
  "weight": 55.0,
  "heart_rate": 70,
  "hmac": "70D57038CC39FEA98FBB9A4132B35422752520A027E97ADAE55BD57F9A6EB13A27AD4447D806316227EF96A5979C6F529D15DAF42D7ADD17436739ABE38820CEBF15D58ABAB1F30DAA1EC67362F463A412AAB8BCBA80A594F0B194C197B4C1620BFCF4F7BA5A3743825A7156AEB1F575E22CA4E5BED9237DF838365DFD3E88BA74EDB0AD7F0809DF7DCF4C2F3EA67387C8EE85C25E83044940675FB31BF693239A37E23247F60E651ABD3885BBEAE29CC8EE85C25E83044940675FB31BF69323DA10A0956A1709A0E895FA7740EEB9A1AC7AC7A1F0A2D8365BD261D0FA6C50398A2AF8A4D7003A75E5178E2B52A7502BAE53A619BDE39B1C8098C6A00D4220AF589D389ABEE16D80F5C7E2109E79B95BC7AE52DD296697750F6484B0AB29AD92B5FCABF2E71CDB5F15D89A04A5B01EE13EB4C3F4DB748ACFFBDF4F227478ED69A59A1592642C7420F52D6787F337ADAE9106A14693CEC5AD814D14CEACFC8005BD39289E27ECE343F4B0CF99BCB4DBC950F76FEFB0867ED0932DE5635872AA65CA97902A2898C323861883E1B51E9F5B"
}
```

#### 详情
| 字段名          | 类型    | 是否必填 | 介绍                  | 额外说明                     |
| --------------- | ------- | -------- | --------------------- | ---------------------------- |
| mac             | string  | Y        | Mac                   | mock: 12:34:56:78:9A:BC      |
| model_id        | string  | Y        | 设备类型 (4 字符)     | mock: 0E2B                   |
| timestamp       | integer | Y        | 测量时间戳 (s)        | mock: 1582698882             |
| weight          | number  | Y        | 体重 (kg)             | mock: 55.0                   |
| heart_rate      | integer | Y        | 心率 (BPM)            | mock: 70                     |
| hmac            | string  | Y        | 签名                  | mock: 183476B32E22B26989A... |
| resistance_flag | integer | Y        | 是否有效电阻 (1是0否) | mock: 1                      |



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

### 8电极报告（Server -> Docker)

#### 接口
```text
POST https://your-docker-server.com/yolanda/wsp/eight_reports
```

#### 示例
```json
{
  "weight": 58.9,
  "height": 175.2,
  "age": 20,
  "gender": 1,
  "hmac": "70D57038CC39FEA98FBB9A4132B35422752520A027E97ADAE55BD57F9A6EB13A27AD4447D806316227EF96A5979C6F529D15DAF42D7ADD17436739ABE38820CEBF15D58ABAB1F30DAA1EC67362F463A412AAB8BCBA80A594F0B194C197B4C1620BFCF4F7BA5A3743825A7156AEB1F575E22CA4E5BED9237DF838365DFD3E88BA74EDB0AD7F0809DF7DCF4C2F3EA67387C8EE85C25E83044940675FB31BF693239A37E23247F60E651ABD3885BBEAE29CC8EE85C25E83044940675FB31BF69323DA10A0956A1709A0E895FA7740EEB9A1AC7AC7A1F0A2D8365BD261D0FA6C50398A2AF8A4D7003A75E5178E2B52A7502BAE53A619BDE39B1C8098C6A00D4220AF589D389ABEE16D80F5C7E2109E79B95BC7AE52DD296697750F6484B0AB29AD92B5FCABF2E71CDB5F15D89A04A5B01EE13EB4C3F4DB748ACFFBDF4F227478ED69A59A1592642C7420F52D6787F337ADAE9106A14693CEC5AD814D14CEACFC8005BD39289E27ECE343F4B0CF99BCB4DBC950F76FEFB0867ED0932DE5635872AA65CA97902A2898C323861883E1B51E9F5B"
}
```

#### 详情
| 字段名 | 类型    | 是否必填 | 介绍                   | 额外说明                     |
|--------|---------|----------|------------------------|------------------------------|
| weight | number  | Y        | 体重 (kg)              | mock: 58.9                   |
| height | number  | Y        | 身高 (cm)              | mock: 175.2                  |
| age    | integer | Y        | 年龄                   | mock: 20                     |
| gender | integer | Y        | 性别(0: 女性，1: 男性) | mock: 1                      |
| hmac   | string  | Y        | 签名                   | mock: 183476B32E22B26989A... |



#### 成功响应
+ 头部
```text
Content-Type: application/json
```

+ 正文
```json
{
  "code": 0,
  "data": {
    "extras": {
      "water_mass": 63.46153846153846,
      "protein_mass": 17.17948717948718,
      "bone_mass": 4.833333333333333,
      "bodyfat_mass": 41.282051282051285,
      "obesity_bmi": 0,
      "fatty_liver_risk_control": 2,
      "obesity_degree": 119.84365013029159,
      "weight_control": -12.874697524705908,
      "bodyfat_control": -15.347204628705889,
      "muscle_control": 2.4725071039999804,
      "health_score": 68.904493984,
      "health_body_shape": 3,
      "left_arm_muscle_weight": 2.76834,
      "right_arm_muscle_weight": 2.83209,
      "left_leg_muscle_weight": 8.68123,
      "right_leg_muscle_weight": 8.72593,
      "trunk_muscle_weight": 23.85385,
      "left_arm_fat": 2.18987,
      "right_arm_fat": 2.18414,
      "left_leg_fat": 4.96095,
      "right_leg_fat": 4.97996,
      "trunk_fat": 16.16568,
      "left_arm_fat_mass": 1.7080985999999998,
      "right_arm_fat_mass": 1.7036292,
      "left_leg_fat_mass": 3.8695410000000003,
      "right_leg_fat_mass": 3.8843688000000003,
      "trunk_fat_mass": 12.6092304
    },
    "reports": {
      "bodyfat": 32.2,
      "subfat": 28.9,
      "visfat": 10,
      "water": 49.5,
      "bmr": 1511,
      "muscle": 36.7,
      "bone": 3.77,
      "lbm": 52.9,
      "protein": 13.4,
      "sinew": 49.1,
      "bmi": 26.4,
      "score": 62.6,
      "body_age": 33,
      "body_shape": 1,
      "right_arm_muscle_weight": 2.83209,
      "left_arm_muscle_weight": 2.76834,
      "right_leg_muscle_weight": 8.72593,
      "left_leg_muscle_weight": 8.68123,
      "trunk_muscle_weight": 23.85385,
      "right_arm_fat": 2.18414,
      "left_arm_fat": 2.18987,
      "right_leg_fat": 4.97996,
      "left_leg_fat": 4.96095,
      "trunk_fat": 16.16568,
      "weight": 78,
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
Content-Type：text/plain; charset=utf-8
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
| 字段名                   | 类型    | 是否必填 | 介绍                | 额外说明   |
| ------------------------ | ------- | -------- | ------------------- | ---------- |
| code                     | integer | Y        | Code                | mock：0    |
| msg                      | string  | Y        | Message             | mock：ok   |
| data                     | object  | Y        | Data                | mock：{}   |
| reports                  | object  | Y        | 报告指标            | mock：{}   |
| extras                   | object  | Y        | 扩展指标            | mock：{}   |
| weight                   | number  | Y        | 体重 (kg)           | mock：58.2 |
| bmi                      | number  | Y        | BMI                 | mock：20.1 |
| bodyfat                  | number  | Y        | 体脂率 (%)          | mock：14   |
| lbm                      | number  | Y        | 去脂体重            | mock：50.1 |
| subfat                   | number  | Y        | 皮下脂肪率 (%)      | mock：12.7 |
| visfat                   | number  | Y        | 内脏脂肪等级        | mock：3.46 |
| water                    | number  | Y        | 体水分 (%)          | mock：62.2 |
| bmr                      | integer | Y        | 基础代谢            | mock：1451 |
| muscle                   | number  | Y        | 骨骼肌率 (%)        | mock：55.6 |
| sinew                    | number  | Y        | 肌肉量              | mock：47.5 |
| bone                     | number  | Y        | 骨量                | mock：2.51 |
| protein                  | number  | Y        | 蛋白质 (%)          | mock：19.5 |
| score                    | number  | Y        | 分数                | mock：90.2 |
| body_age                 | integer | Y        | 体年龄              | mock：20   |
| body_shape               | integer | Y        | 体型                | mock：4    |
| right_arm_muscle_weight  | number  | Y        | 肌肉量 (右上肢)     | mock：0.8  |
| left_arm_muscle_weight   | number  | Y        | 肌肉量 (左上肢)     | mock：0.8  |
| right_leg_muscle_weight  | number  | Y        | 肌肉量 (右下肢)     | mock：0.8  |
| left_leg_muscle_weight   | number  | Y        | 肌肉量 (左下肢)     | mock：0.8  |
| trunk_muscle_weight      | number  | Y        | 肌肉量 (躯干)       | mock：0.8  |
| right_arm_fat            | number  | Y        | 脂肪百分比 (右上肢) | mock：0.8  |
| left_arm_fat             | number  | Y        | 脂肪百分比 (左上肢) | mock：0.8  |
| right_leg_fat            | number  | Y        | 脂肪百分比 (右下肢) | mock：0.8  |
| left_leg_fat             | number  | Y        | 脂肪百分比 (左下肢) | mock：0.8  |
| trunk_fat                | number  | Y        | 脂肪百分比 (躯干)   | mock：0.8  |
| water_mass               | number  | Y        | 水分 (L)            | mock: 0.8  |
| protein_mass             | number  | Y        | 蛋白质重量          | mock: 0.8  |
| bone_mass                | number  | Y        | 骨量百分比          | mock: 0.8  |
| bodyfat_mass             | number  | Y        | 脂肪重量            | mock: 0.8  |
| obesity_bmi              | number  | Y        | 肥胖度 BMI          | mock: 0.8  |
| fatty_liver_risk_control | number  | Y        | 脂肪肝风险等级      | mock: 0.8  |
| obesity_degree           | number  | Y        | 肥胖度              | mock: 0.8  |
| weight_control           | number  | Y        | 体重控制            | mock: 0.8  |
| bodyfat_control          | number  | Y        | 脂肪控制            | mock: 0.8  |
| muscle_control           | number  | Y        | 肌肉控制            | mock: 0.8  |
| health_body_shape        | number  | Y        | 健康体型            | mock: 0.8  |
| left_arm_fat_mass        | number  | Y        | 脂肪重量 (右上肢)   | mock: 0.8  |
| right_arm_fat_mass       | number  | Y        | 脂肪重量 (左上肢)   | mock: 0.8  |
| left_leg_fat_mass        | number  | Y        | 脂肪重量 (左上肢)   | mock: 0.8  |
| right_leg_fat_mass       | number  | Y        | 脂肪重量 (左下肢)   | mock: 0.8  |
| trunk_fat_mass           | number  | Y        | 脂肪重量 (躯干)     | mock: 0.8  |
| heart_rate               | integer | Y        | 心率                | mock：0    |
| cardiac_index            | number  | Y        | 心脏指数            | mock：0    |
