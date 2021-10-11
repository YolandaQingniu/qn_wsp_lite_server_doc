# Document for Yolanda-WSP-Lite Server SDK 

[English](./README.en.md) | [简体中文](./README.zh-CN.md)

> The latest version: 1.0.0

The SDK is released by Docker.

Docker is an open source containerization platform.

It enables developers to package applications into containers. 

Containers simplify delivery of distributed applications.

## Roles
There are 4 roles in the SDK.
1. Scale
2. App
3. Docker
4. Server

## The Request Flow
1.App <----> Scale
+ App sends data to Scale via bluetooth, to tell what is the domain of Docker.
+ Scale sends data to App via bluetooth.

2.Scale ----> Docker
+ Scale sends encrypted data to Docker Server via HTTP request. 
+ Docker Server decrypts and parses the data, then responses to Scale.

3.Docker <----> Server
+ Docker sends data to Server via HTTP request.
+ Server receives the data from Docker.
+ Server sends data to Docker via HTTP request, to gain more indicators.

4.Server <----> App
+ App sends data to Server via HTTP request.
+ Server responses to App.

## How to create Docker Server?
1. Install Docker. See https://www.docker.com/get-started

2. Make a directory and change into it.
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

> tips: The default timezone is UTC, you can change it by environment variable `TZ`. The log of Docker stores in log/access.log on a daily basis and keeps 7 days. The internal port of the container is 8080, and the external access port can be configured when Docker is started.


4. Build and run.
```shell
docker build . -t yolanda_wsp_lite
docker run -p 8080:8080 yolanda_wsp_lite
```

5. Map your application to a domain name

Example: 0.0.0.0:8080 maps to https://your-docker-server.com or http://your-docker-server.com


## Endpoints between Docker and Scale 

Endpoint:
```text
GET https://your-docker-server.com/yolanda/wsp?code=
```

## Endpoints between Docker and Server 

### Upload Measurement（Docker -> Server)

#### Endpoint
```text
POST https://your-business-server.com/wsp/measurements
```

#### Example
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

#### Description
| name       | type    | required | description           | extra                        |
|------------|---------|----------|-----------------------|------------------------------|
| mac        | string  | Y        | Mac                   | mock: 12:34:56:78:9A:BC      |
| model_id   | string  | Y        | Device Type (4 chars) | mock: 0E2B                   |
| timestamp  | integer | Y        | Measure At (s)        | mock: 1582698882             |
| weight     | number  | Y        | Weight (kg)           | mock: 55.0                   |
| heart_rate | integer | Y        | Height Rate (BPM)     | mock: 70                     |
| hmac       | string  | Y        | Signature hmac        | mock: 183476B32E22B26989A... |

#### Success Response
+ Header
```text
Content-Type：text/plain; charset=utf-8
```

+ Body
```text
success
```

#### Fail Response
+ Header
```text
Content-Type：text/plain; charset=utf-8
```

+ Body
```text
fail
```

### Calc Report（Server -> Docker)

#### Endpoint
```text
POST https://your-docker-server.com/yolanda/wsp/reports
```

#### Example
```json
{
  "weight": 58.9,
  "height": 175.2,
  "age": 20,
  "gender": 1,
  "hmac": "183476B32E22B26989AD4744C3AE13CCA857EB021C9290CFC83FF46A544D999BFF09F3DE988D58C2F435BFC70DB4C54CF018E35A5AE2A63A1D094350EFB2540908026F908CA690B81D5C13E1A4371D0E710F91DA67CFBDD55F50D593D1075CA23A2572F541B5047FD6FB4E59870740197628E2680EDCFFE4FB2F3F8CB31C8F22B16787F9AE44C1CB1E5D99714211F5370EBE1CB9CD8A760E8BAE668A515E2B85"
}
```

#### Description
| name   | type    | required | description                | extra                        |
|--------|---------|----------|----------------------------|------------------------------|
| weight | number  | Y        | Weight (kg)                | mock: 58.9                   |
| height | number  | Y        | Height (cm)                | mock: 175.2                  |
| age    | integer | Y        | Age                        | mock: 20                     |
| gender | integer | Y        | Gender(0: Female，1: Male) | mock: 1                      |
| hmac   | string  | Y        | Signature hmac             | mock: 183476B32E22B26989A... |

#### Success Response
+ Header
```text
Content-Type：text/plain; charset=utf-8
```

+ Body
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

#### Fail Response
+ Header
```text
Content-Type: application/json
```

+ Body
```json
{
  "code": 10002,
  "data": {},
  "msg": "Missing params or params error"
}
```

#### Description
| name          | type    | required | description        | extra      |
| ------------- | ------- | -------- | ------------------ | ---------- |
| code          | integer | Y        | Code               | mock：0    |
| msg           | string  | Y        | Message            | mock：ok   |
| data          | object  | Y        | Data               | mock：{}   |
| reports       | object  | Y        | Reports            | mock：{}   |
| weight        | number  | Y        | Weight             | mock：58.2 |
| bmi           | number  | Y        | BMI                | mock：20.1 |
| bodyfat       | number  | Y        | BodyFat            | mock：14   |
| lbm           | number  | Y        | Fat Free Weight    | mock：50.1 |
| subfat        | number  | Y        | Subcutaneous Fat   | mock：12.7 |
| visfat        | number  | Y        | Visceral fat level | mock：3.46 |
| water         | number  | Y        | Water              | mock：62.2 |
| bmr           | integer | Y        | BMR                | mock：1451 |
| muscle        | number  | Y        | Muscle             | mock：55.6 |
| sinew         | number  | Y        | Sinew              | mock：47.5 |
| bone          | number  | Y        | Bone               | mock：2.51 |
| protein       | number  | Y        | Protein            | mock：19.5 |
| score         | number  | Y        | Score              | mock：90.2 |
| body_age      | integer | Y        | Body Age           | mock：20   |
| body_shape    | integer | Y        | Body Shape         | mock：4    |
| heart_rate    | integer | Y        | Heart Rate         | mock：0    |
| cardiac_index | number  | Y        | Cardiac Index      | mock：0    |

## Endpoints For Eight Scale

### Eight Scale Upload Measurement（Docker -> Server)

#### Endpoint
```text
POST https://your-business-server.com/wsp/eight_measurements
```

#### Example
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

#### Description
| name       | type    | required | description           | extra                        |
|------------|---------|----------|-----------------------|------------------------------|
| mac        | string  | Y        | Mac                   | mock: 12:34:56:78:9A:BC      |
| model_id   | string  | Y        | Device Type (4 chars) | mock: 0E2B                   |
| timestamp  | integer | Y        | Measure At (s)        | mock: 1582698882             |
| weight     | number  | Y        | Weight (kg)           | mock: 55.0                   |
| heart_rate | integer | Y        | Height Rate (BPM)     | mock: 70                     |
| hmac       | string  | Y        | Signature hmac        | mock: 183476B32E22B26989A... |


#### Success Response
+ Header
```text
Content-Type：text/plain; charset=utf-8
```

+ Body
```text
success
```

#### Fail Response

+ Header
```text
Content-Type：text/plain; charset=utf-8
```

+ Body
```text
fail
```

### Eight Scale Report（Server -> Docker)

#### Endpoint
```text
POST https://your-docker-server.com/yolanda/wsp/eight_reports
```

#### Example
```json
{
  "weight": 58.9,
  "height": 175.2,
  "age": 20,
  "gender": 1,
  "hmac": "70D57038CC39FEA98FBB9A4132B35422752520A027E97ADAE55BD57F9A6EB13A27AD4447D806316227EF96A5979C6F529D15DAF42D7ADD17436739ABE38820CEBF15D58ABAB1F30DAA1EC67362F463A412AAB8BCBA80A594F0B194C197B4C1620BFCF4F7BA5A3743825A7156AEB1F575E22CA4E5BED9237DF838365DFD3E88BA74EDB0AD7F0809DF7DCF4C2F3EA67387C8EE85C25E83044940675FB31BF693239A37E23247F60E651ABD3885BBEAE29CC8EE85C25E83044940675FB31BF69323DA10A0956A1709A0E895FA7740EEB9A1AC7AC7A1F0A2D8365BD261D0FA6C50398A2AF8A4D7003A75E5178E2B52A7502BAE53A619BDE39B1C8098C6A00D4220AF589D389ABEE16D80F5C7E2109E79B95BC7AE52DD296697750F6484B0AB29AD92B5FCABF2E71CDB5F15D89A04A5B01EE13EB4C3F4DB748ACFFBDF4F227478ED69A59A1592642C7420F52D6787F337ADAE9106A14693CEC5AD814D14CEACFC8005BD39289E27ECE343F4B0CF99BCB4DBC950F76FEFB0867ED0932DE5635872AA65CA97902A2898C323861883E1B51E9F5B"
}
```

#### Description
| name   | type    | required | description                | extra                        |
|--------|---------|----------|----------------------------|------------------------------|
| weight | number  | Y        | Weight (kg)                | mock: 58.9                   |
| height | number  | Y        | Height (cm)                | mock: 175.2                  |
| age    | integer | Y        | Age                        | mock: 20                     |
| gender | integer | Y        | Gender(0: Female，1: Male) | mock: 1                      |
| hmac   | string  | Y        | Signature hmac             | mock: 183476B32E22B26989A... |


#### Success Response
+ Header
```text
Content-Type: application/json
```

+ Body
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
      "weight": 78
    }
  },
  "msg": "ok"
}
```

#### Fail Response
+ Header
```text
Content-Type：text/plain; charset=utf-8
```

+ Body
```json
{
  "code": 10002,
  "data": {},
  "msg": "Missing params or params error"
}
```

#### Description
| name                     | type    | required | description               | extra      |
| ------------------------ | ------- | -------- | ------------------------- | ---------- |
| code                     | integer | Y        | Code                      | mock：0    |
| msg                      | string  | Y        | Message                   | mock：ok   |
| data                     | object  | Y        | Data                      | mock：{}   |
| reports                  | object  | Y        | Reports                   | mock：{}   |
| extras                   | object  | Y        | Extras                    | mock：{}   |
| weight                   | number  | Y        | Weight (kg)               | mock：58.2 |
| bmi                      | number  | Y        | BMI                       | mock：20.1 |
| bodyfat                  | number  | Y        | BodyFat                   | mock：14   |
| lbm                      | number  | Y        | Fat Free Weight           | mock：50.1 |
| subfat                   | number  | Y        | Subcutaneous Fat          | mock：12.7 |
| visfat                   | number  | Y        | Visceral fat level        | mock：3.46 |
| water                    | number  | Y        | Water                     | mock：62.2 |
| bmr                      | integer | Y        | BMR                       | mock：1451 |
| muscle                   | number  | Y        | Muscle                    | mock：55.6 |
| sinew                    | number  | Y        | Sinew                     | mock：47.5 |
| bone                     | number  | Y        | Bone                      | mock：2.51 |
| protein                  | number  | Y        | Protein                   | mock：19.5 |
| score                    | number  | Y        | Score                     | mock：90.2 |
| body_age                 | integer | Y        | Body Age                  | mock：20   |
| body_shape               | integer | Y        | Body Shape                | mock：4    |
| right_arm_muscle_weight  | number  | Y        | Muscle Weight (Right Arm) | mock：0.8  |
| left_arm_muscle_weight   | number  | Y        | Muscle Weight (Left Arm)  | mock：0.8  |
| right_leg_muscle_weight  | number  | Y        | Muscle Weight (Right Leg) | mock：0.8  |
| left_leg_muscle_weight   | number  | Y        | Muscle Weight (Left Leg)  | mock：0.8  |
| trunk_muscle_weight      | number  | Y        | Muscle Weight (Trunk)     | mock：0.8  |
| right_arm_fat            | number  | Y        | Fat (Right Arm)           | mock：0.8  |
| left_arm_fat             | number  | Y        | Fat (Left Arm)            | mock：0.8  |
| right_leg_fat            | number  | Y        | Fat (Right Leg)           | mock：0.8  |
| left_leg_fat             | number  | Y        | Fat (Left Leg)            | mock：0.8  |
| trunk_fat                | number  | Y        | Fat (Trunk)               | mock：0.8  |
| water_mass               | number  | Y        | Water Mass                | mock: 0.8  |
| protein_mass             | number  | Y        | Protein Mass              | mock: 0.8  |
| bone_mass                | number  | Y        | Bone Mass                 | mock: 0.8  |
| bodyfat_mass             | number  | Y        | Bodyfat Mass              | mock: 0.8  |
| obesity_bmi              | number  | Y        | Obesity Bmi               | mock: 0.8  |
| fatty_liver_risk_control | number  | Y        | Fatty Liver Risk Control  | mock: 0.8  |
| obesity_degree           | number  | Y        | Obesity Degree            | mock: 0.8  |
| weight_control           | number  | Y        | Weight Control            | mock: 0.8  |
| bodyfat_control          | number  | Y        | Bodyfat Control           | mock: 0.8  |
| muscle_control           | number  | Y        | Muscle Control            | mock: 0.8  |
| health_score             | number  | Y        | Health Score              | mock: 0.8  |
| health_body_shape        | number  | Y        | Health Body Shape         | mock: 0.8  |
| left_arm_fat_mass        | number  | Y        | Left Arm Fat Mass         | mock: 0.8  |
| right_arm_fat_mass       | number  | Y        | Right Arm Fat Mass        | mock: 0.8  |
| left_leg_fat_mass        | number  | Y        | Left Leg Fat Mass         | mock: 0.8  |
| right_leg_fat_mass       | number  | Y        | Right Leg Fat Mass        | mock: 0.8  |
| trunk_fat_mass           | number  | Y        | Trunk Fat Mass            | mock: 0.8  |
