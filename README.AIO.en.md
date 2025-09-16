# Document for Yolanda-WSP-Lite Server SDK

> The latest version: 1.2.7
>
> Changelog
> 1.2.7 => Support CP30G
> 
> 1.2.6 => Update the barcode data parsing of the height scale
> 
> 1.2.5 => Support scanning information
> 
> 1.2.4 => Support for height all-in-one machine

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
FROM registry.cn-shenzhen.aliyuncs.com/yolanda_open/wsp-lite:v1.2.7
ENV CLIENT_URL="https://your-business-server.com"
ENV CLIENT_ID="A_CLIENT_ID_FROM_YOLANDA_PLEASE_CONTACT_US"
ENV SECRET_KEY="secret_key"
ENV TZ="Asia/Shanghai"
```

> tips: The default timezone is UTC, you can change it by environment variable TZ. The log of Docker stores in log/access.log on a daily basis and keeps 7 days. The internal port of the container is 8080, and the external access port can be configured when Docker is started.

> Note: CLIENT_ URL is the recipient's business server address; CLIENT_ ID、SECRET_ KEY is the ID and key assigned by Yolanda.


4. Build and run.
```shell
docker build . -t yolanda_wsp_lite
docker run -p 8080:8080 yolanda_wsp_lite
```

5. Map your application to a domain name

Example: 0.0.0.0:8080 maps to https://your-docker-server.com or http://your-docker-server.com

## Endpoints between Docker and Scale (Scale -> Docker)

Endpoint:
```text
GET https://your-docker-server.com/yolanda/aios?code=
```

## Endpoints between Docker and Server (Docker <-> Server)

### Upload Data （Docker -> Server)

#### Endpoint
```text
POST https://your-business-server.com/aios/measurements
```

#### Example
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

#### Description
| name     | type    | required | description               | extra                     |
|------------|---------|----------|--------------------|------------------------------|
| mac        | string  | Y        | Mac                | mock: 12:34:56:78:9A:BC      |
| timestamp  | integer | Y        | Measurement timestamp (s)     | mock: 1666841725             |
| weight     | number  | Y        | Weight (kg)          | mock: 79.7                   |
| height | integer | Y        |Height (cm)         | mock: 172.5                     |
| hmac       | string  | Y        | Signature hmac              | mock: 183476B32E22B26989A... |
| scan_code  | string  | N        | Scan code information            | mock: 6922711079073         |

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
### Calculation indicators（Server -> Docker)

#### 1、WIFI upload data parsing interface
```text
POST https://your-docker-server.com/yolanda/aios/reports
```

#### Example
```json
{
  "weight": 79.7,
  "age": 25,
  "height": 172.5,
  "gender": 1,
  "hmac": "FCAC4733C0DDE4AC1ACB3A46A2F81925BA2FEF981EAE5A0CADBE8663FF37FE3CC5C989ACC5EA8A436A952DFEEAF7B2D5D1FEABD1F5721B1664CAFFDD4BA42061B0C409CBA609664FFA70FD99B7C444D73BFD63D8D7652CF4F9D52B02DF7A984117E896976F860FB91497752A667750D1877B454F8A3F3C4485C3BF9DEDA6EE33"
}
```

#### Description
| name     | type    | required | description               | extra                     |
|--------|---------|----------|------------------------|------------------------------|
| weight | number  | Y        | Weight (kg)              | mock: 79.7                  |
| height | number  | Y        | Height (cm)              | mock: 172.5                  |
| age    | integer | Y        | Age                   | mock: 25                     |
| gender | integer | Y        | Gender(0: Female，1: Male) | mock: 1                      |
| hmac   | string  | Y        | Signature hmac                  | mock: 183476B32E22B26989A... |


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
| name     | type    | required | description               | extra                     |
| ------------- | ------- | -------- | -------- | ---------- |
| code          | integer | Y        | Code     | mock：0    |
| msg           | string  | Y        | Message  | mock：ok   |
| data          | object  | Y        | Data     | mock：{}   |
| reports       | object  | Y        | Reports  | mock：{}   |
| weight        | number  | Y        | Weight     | mock：58.2 |
| bmi           | number  | Y        | BMI      | mock：20.1 |
| bodyfat       | number  | Y        | BodyFat (%)   | mock：14   |
| lbm           | number  | Y        | Fat Free Weight | mock：50.1 |
| subfat        | number  | Y        | Subcutaneous Fat (%) | mock：12.7 |
| visfat        | number  | Y        | Visceral fat level | mock：3.46 |
| water         | number  | Y        | Water (%)     | mock：62.2 |
| bmr           | integer | Y        | BMR | mock：1451 |
| muscle        | number  | Y        | Muscle (%) | mock：55.6 |
| sinew         | number  | Y        | Sinew   | mock：47.5 |
| bone          | number  | Y        | Bone     | mock：2.51 |
| protein       | number  | Y        | Protein (%)   | mock：19.5 |
| score         | number  | Y        | Score     | mock：90.2 |
| body_age      | integer | Y        | Body Age   | mock：20   |
| body_shape    | integer | Y        | Body Shape     | mock：4    |
| heart_rate    | integer | Y        | Heart Rate     | mock：0    |
| cardiac_index | number  | Y        | Cardiac Index | mock：0    |


#### 2、Scale end real-time measurement data analysis interface
```text
POST https://your-docker-server.com/yolanda/aios/aio_scale_reports
```

#### Example
```json
{
  "age": 25,
  "gender": 1,
  "scale": "10105003E80101F401F6000000000025120f20a097c138c1a4380338000f58"
}
```

#### Description
| name     | type    | required | description               | extra                     |
|--------|---------|----------|------------------------|------------------------------|
| age    | integer | Y        | Age                   | mock: 25                     |
| gender | integer | Y        | Gender(0: Female，1: Male) | mock: 1                      |
| scale  | string  | Y        | The scale string is a concatenation of commands 10 and 12, with 10 leading| mock: 10105003E80101F401F6000000000025120f20a097c138c1a4380338000f58 |


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
| name     | type    | required | description               | extra                     |
| ------------- | ------- | -------- | -------- | ---------- |
| code          | integer | Y        | Code     | mock：0    |
| msg           | string  | Y        | Message  | mock：ok   |
| data          | object  | Y        | Data     | mock：{}   |
| reports       | object  | Y        | Reports  | mock：{}   |
| weight        | number  | Y        | Weight     | mock：58.2 |
| bmi           | number  | Y        | BMI      | mock：20.1 |
| bodyfat       | number  | Y        | BodyFat (%)   | mock：14   |
| lbm           | number  | Y        | Fat Free Weight | mock：50.1 |
| subfat        | number  | Y        | Subcutaneous Fat (%) | mock：12.7 |
| visfat        | number  | Y        | Visceral fat level | mock：3.46 |
| water         | number  | Y        | Water (%)     | mock：62.2 |
| bmr           | integer | Y        | BMR | mock：1451 |
| muscle        | number  | Y        | Muscle (%) | mock：55.6 |
| sinew         | number  | Y        | Sinew   | mock：47.5 |
| bone          | number  | Y        | Bone     | mock：2.51 |
| protein       | number  | Y        | Protein (%)   | mock：19.5 |
| score         | number  | Y        | Score     | mock：90.2 |
| body_age      | integer | Y        | Body Age   | mock：20   |
| body_shape    | integer | Y        | Body Shape     | mock：4    |
| heart_rate    | integer | Y        | Heart Rate     | mock：0    |
| cardiac_index | number  | Y        | Cardiac Index | mock：0    |
| height | number  | Y        | Height | mock：172.5    |

#### 3、Scale end historical measurement data analysis interface
```text
POST https://your-docker-server.com/yolanda/aios/aio_scale_history_reports
```

#### Example
```json
{
  "age": 25,
  "gender": 1,
  "scale": "23145001e9d7d9631f9a025e024f0006c20200b8120f5066478093174414130e0001c2"
}
```

#### Description
| name     | type    | required | description               | extra                     |
|--------|---------|----------|------------------------|------------------------------|
| age    | integer | Y        | age                   | mock: 25                     |
| gender | integer | Y        | gender(0: female，1: male) | mock: 1                      |
| scale  | string  | Y        | scale字符串为23和12命令的拼接，23在前           | mock: 23145001e9d7d9631f9a025e024f0006c20200b8120f5066478093174414130e0001c2 |


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
| name     | type    | required | description               | extra                     |
| ------------- | ------- | -------- | -------- | ---------- |
| code          | integer | Y        | Code     | mock：0    |
| msg           | string  | Y        | Message  | mock：ok   |
| data          | object  | Y        | Data     | mock：{}   |
| reports       | object  | Y        | Reports  | mock：{}   |
| weight        | number  | Y        | Weight     | mock：58.2 |
| bmi           | number  | Y        | BMI      | mock：20.1 |
| bodyfat       | number  | Y        | BodyFat (%)   | mock：14   |
| lbm           | number  | Y        | Fat Free Weight	 | mock：50.1 |
| subfat        | number  | Y        | Subcutaneous Fat (%)	 | mock：12.7 |
| visfat        | number  | Y        | Visceral fat level | mock：3.46 |
| water         | number  | Y        | Water (%)     | mock：62.2 |
| bmr           | integer | Y        | BMR | mock：1451 |
| muscle        | number  | Y        | Muscle (%) | mock：55.6 |
| sinew         | number  | Y        | Sinew   | mock：47.5 |
| bone          | number  | Y        | Bone     | mock：2.51 |
| protein       | number  | Y        | Protein (%)   | mock：19.5 |
| score         | number  | Y        | Score     | mock：90.2 |
| body_age      | integer | Y        | Body Age   | mock：20   |
| body_shape    | integer | Y        | Body Shape     | mock：4    |
| heart_rate    | integer | Y        | Heart Rate     | mock：0    |
| cardiac_index | number  | Y        | Cardiac Index | mock：0    |
| height | number  | Y        | Height | mock：172.5    |
