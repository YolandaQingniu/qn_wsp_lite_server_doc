# Document for Yolanda-WSP-Lite Server SDK 

[English](./README.en.md) | [中文](./README.md)

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

> tips: The default timezone is UTC, you can change it by environment variable `TZ`. The log of Docker stores in log/access.log on a daily basis and keeps 7 days.


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
GET https://your-wsp-server.com/yolanda/wsp?code=
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

#### Calc Report（Server -> Docker)

#### Endpoint
```text
GET https://your-wsp-server.com/yolanda/wsp/reports
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
| name            | type    | required | description        | extra      |
|-----------------|---------|----------|--------------------|------------|
| code            | integer | Y        | Code               | mock：0    |
| msg             | string  | Y        | Message            | mock：ok   |
| data            | object  | Y        | Data               | mock：{}   |
| reports         | object  | Y        | Reports            | mock：{}   |
| weight          | number  | Y        | Weight             | mock：58.2 |
| bmi             | number  | Y        | BMI                | mock：20.1 |
| bodyfat         | number  | Y        | BodyFat            | mock：14   |
| fat_free_weight | number  | Y        | Fat Free Weight    | mock：50.1 |
| subfat          | number  | Y        | Subcutaneous Fat   | mock：12.7 |
| visfat          | number  | Y        | Visceral fat level | mock：3.46 |
| water           | number  | Y        | Water              | mock：62.2 |
| bmr             | integer | Y        | BMR                | mock：1451 |
| muscle          | number  | Y        | Muscle             | mock：55.6 |
| sinew           | number  | Y        | Sinew              | mock：47.5 |
| bone            | number  | Y        | Bone               | mock：2.51 |
| protein         | number  | Y        | Protein            | mock：19.5 |
| score           | number  | Y        | Score              | mock：90.2 |
| bodyage         | integer | Y        | Body Age           | mock：20   |
| body_shape      | integer | Y        | Body Shape         | mock：4    |
| heart_rate      | integer | Y        | Heart Rate         | mock：0    |
| cardiac_index   | number  | Y        | Cardiac Index      | mock：0    |

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

### Eight Scale Report（Docker -> Server)

#### Endpoint
```text
GET https://your-wsp-server.com/yolanda/wsp/eight_reports
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
| hmac   | string  | Y        | Signature Hmac             | mock: 183476B32E22B26989A... |


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
      "bodyfat_trunk": 0.8
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
| name              | type    | required | description          | extra      |
|-------------------|---------|----------|----------------------|------------|
| code              | integer | Y        | Code                 | mock：0    |
| msg               | string  | Y        | Message              | mock：ok   |
| data              | object  | Y        | Data                 | mock：{}   |
| reports           | object  | Y        | Reports              | mock：{}   |
| weight            | number  | Y        | Weight (kg)          | mock：58.2 |
| bmi               | number  | Y        | BMI                  | mock：20.1 |
| bodyfat           | number  | Y        | BodyFat              | mock：14   |
| fat_free_weight   | number  | Y        | Fat Free Weight      | mock：50.1 |
| subfat            | number  | Y        | Subcutaneous Fat     | mock：12.7 |
| visfat            | number  | Y        | Visceral fat level   | mock：3.46 |
| water             | number  | Y        | Water                | mock：62.2 |
| bmr               | integer | Y        | BMR                  | mock：1451 |
| muscle            | number  | Y        | Muscle               | mock：55.6 |
| sinew             | number  | Y        | Sinew                | mock：47.5 |
| bone              | number  | Y        | Bone                 | mock：2.51 |
| protein           | number  | Y        | Protein              | mock：19.5 |
| score             | number  | Y        | Score                | mock：90.2 |
| bodyage           | integer | Y        | Body Age             | mock：20   |
| body_shape        | integer | Y        | Body Shape           | mock：4    |
| sinew_right_arm   | number  | Y        | Sinew (Right Arm)    | mock：0.8  |
| sinew_left_arm    | number  | Y        | Sinew (Left Arm)     | mock：0.8  |
| sinew_right_leg   | number  | Y        | Sinew (Right Leg)    | mock：0.8  |
| sinew_left_leg    | number  | Y        | Sinew (Left Leg)     | mock：0.8  |
| sinew_trunk       | number  | Y        | Sinew (Trunk)        | mock：0.8  |
| bodyfat_right_arm | number  | Y        | Body Fat (Right Arm) | mock：0.8  |
| bodyfat_left_arm  | number  | Y        | Body Fat (Left Arm)  | mock：0.8  |
| bodyfat_right_leg | number  | Y        | Body Fat (Right Leg) | mock：0.8  |
| bodyfat_left_leg  | number  | Y        | Body Fat (Left Leg)  | mock：0.8  |
| bodyfat_trunk     | number  | Y        | Body Fat (Trunk)     | mock：0.8  |
