# Document for Yolanda-WSP-Lite Server SDK
[English](./README.BP.en.md) | [简体中文](./README.BP.zh-CN.md)

> The latest version: : 1.2.3
>
> Changelog
>
> 1.2.3 => Support sphygmomanometer。

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
+ The sphygmomanometer sends encrypted data to Docker through HTTP request.
+ Docker decrypts and analyzes the data, and then sends back the response to the sphygmomanometer.

3.Server <----> App
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
FROM registry.cn-shenzhen.aliyuncs.com/yolanda_open/wsp-lite:v1.2.3
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
GET https://your-docker-server.com/yolanda/bps?code={encrypted_data}
```

## Endpoints between Docker and Server (Docker <-> Server)

### Upload Data -> Server)

#### Endpoint
```text
POST https://your-business-server.com/bps/measurements
```

#### Example
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

#### Description
| name     | type    | required | description               | extra                     |
|------------|---------|----------|--------------------|------------------------------|
| user_index | integer  | Y        | User index (user 1 or user 2)    | mock: 1                 |
| systolic   | integer  | Y        | Systolic blood pressure/high pressure              | mock: 120               |
| diastolic  | integer  | Y        | Diastolic pressure/low pressure               | mock: 78                |
| blood_unit | string   | Y        | unit                  | mock: mmHg             |
| pulse      | integer  | Y        | pulse                     | mock: 77                |
| timestamp  | integer  | Y        | Measurement timestamp (s)            | mock: 1669975757        |
| mac        | string   | Y        | Mac                     | mock: 10:91:A8:3E:E1:CD  |

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