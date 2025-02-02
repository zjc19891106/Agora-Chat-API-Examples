# Chat App Server

## Introduction

This repository contains sample project for chat app server, which include user registration and login.
When you plan to use agora chat service, you may need mapping your user profile with agora chat account and generate token for chat account. This project demonstrate how to create chat account and map to your own user profile and generate the token for chat account

Contact [support@agora.io](mailto:support@agora.io) to enable automatic user registration. Once enabled, you can use `chatUsername` to generate a user-permission token. When logging into Chat, if `chatUsername` is not registered, the Chat server will automatically use `chatUsername` to register the Chat user and log in. `chatUsername` must be consistent with the username. 

* workflow for create account

![user-register](https://github.com/user-attachments/assets/d84e914f-0618-4be4-b1b3-72fde73a7a2c)


---

* workflow for login

![user-login](https://github.com/user-attachments/assets/69836f5d-7a29-4712-a531-6d3666338d9d)


## Features

- App Server support user registration and will create a chat account and map it to the user.
- App Server support user login and generate a token for chat service(use aogra appId, appcert, chat username).
- App Server support store user information with database, which include account name, acount password, chat username.


## Technical

This project developed based on Spring Boot.

* [Spring Boot](https://spring.io/projects/spring-boot)

## Component

* [AgoraTools](https://github.com/AgoraIO/Tools/tree/master/DynamicKey/AgoraDynamicKey/java/src/main/java/io/agora)
* MySQL

## Prepare

Before start, you need prepare agora chat appkey, agora AppId and agora AppCert.

* Setup aogra chat and get the AppKey：
  - Please login agora developer console, you can reference the link for detail. [Here](https://docs.agora.io/en/agora-chat/get-started/enable)

* You need setup your auth mechanism for your own user profile.

## Database
You need to create a database and a table.

```
CREATE DATABASE app_server CHARACTER SET utf8mb4;
	
CREATE TABLE `app_user_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `user_account` varchar(32) NOT NULL COMMENT 'user account',
  `user_password` varchar(32) DEFAULT NULL COMMENT 'user password',
  `agora_chat_user_name` varchar(32) NOT NULL COMMENT 'Agora Chat user name',
  PRIMARY KEY (`id`,`user_account`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

## Configure

Configure the below file with appkey, AppId and AppCert you get from the above steps.

* Configure file is：[application.properties](./agora-app-server/src/main/resources/application.properties)

  ```
 
      ## configure with your own appid
      application.agoraAppId=xxx
      ## config with your own appcert
      application.agoraAppCert=xxx
      ## token valid duration(suggest not over one day)
      agora.token.expire.period.seconds=86400
      
      ## data source
      spring.datasource.driver-class-name=com.mysql.jdbc.Driver
      spring.datasource.url=jdbc:mysql://127.0.0.1:3306/app_server?useSSL=false&useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true
      spring.datasource.username=root
      spring.datasource.password=123456789
      spring.datasource.hikari.maximum-pool-size=50
      spring.datasource.hikari.minimum-idle=20
  
      ## jpa
      spring.jpa.show_sql=false
      spring.jpa.properties.hibernate.format_sql=true
      spring.jpa.properties.hibernate.generate_statistics=false
      spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
      spring.jpa.hibernate.ddl-auto=validate
      
  ```

## Run

When you finish the configure, you can just run this app server.

## API

### register user


This api is used to register a user for your app. User name and password is used in this sample project, you can use any other format for your user account ,such as phone number.

**Path:** `http://localhost:8086/app/chat/user/register`

**HTTP Method:** `POST`

**Request Headers:** 

| Param        | description      |
| ------------ | ---------------- |
| Content-Type | application/json |
| Accept | application/json |

**Request Body example:** 
{"userAccount":"jack", "userPassword":"123"}

**Request Body params:** 

| Param        | Data Type | description   |
| ------------ | --------- | ------------- |
| userAccount  | String    | user account  |
| userPassword | String    | user password |


**request example:**

```
curl -X POST -H 'Content-Type: application/json' -H 'Accept: application/json' 'http://localhost:8086/app/chat/user/register' -d '{"userAccount": "jack","userPassword":"123"}'
```

**Response Parameters:**

| Param           | Data Type | description          |
| --------------- | --------- | -------------------- |
| code            | Int    | response status code |

**response example:**

```json
{
    "code": 200
}
```

---

### User Login

User login on your app server and get a agora token for chat service.

**Path:** `http://localhost:8086/app/chat/user/login`

**HTTP Method:** `POST`

**Request Headers:** 

| Param        | description      |
| ------------ | ---------------- |
| Content-Type | application/json |
| Accept | application/json |

**Request Body example:** 
{"userAccount":"jack", "userPassword":"123"}

**Request Body params:** 

| Param        | Data Type | description   |
| ------------ | --------- | ------------- |
| userAccount  | String    | user account  |
| userPassword | String    | user password |

**request example:**

```
curl -X POST -H 'Content-Type: application/json' -H 'Accept: application/json' 'http://localhost:8086/app/chat/user/login' -d '{"userAccount": "jack","userPassword":"123"}'
```

**Response Parameters:**

| Param           | Data Type | description                |
| --------------- | --------- | -------------------------- |
| code            | Int    | response status code       |
| accessToken     | String    | token                      |
| expireTimestamp | Long      | timestamp for token expire |
| chatUserName | String    | chat user id               |

**response example:**

```json
{
    "code": 200,
    "accessToken": "xxx",
    "expireTimestamp": 1628245967857,
    "chatUserName": "jack"
}
```
---

### Get token

Get a token including user and rtc privileges.

**Path:** `http://localhost:8086/token`

**HTTP Method:** `GET`

**Request Headers:** 

| Param        | description      |
| ------------ | ---------------- |
| Accept | application/json |

**Request params:** 

| Param        | Data Type | description   | required |
| ------------ | --------- | ------------- | -------- |
| userAccount  | String    | user account  |    Yes   |
| channelName  | String    | channel name  |    Yes   |
| publisherRole  | Boolean    | whether the rtc privileges is the publisher role, the default is false   |    No    |

**request example:**

```
curl -X GET -H 'Accept: application/json' 'http://localhost:8086/token?userAccount={userAccount}&channelName={channelName}&publisherRole=true'
```

**Response Parameters:**

| Param           | Data Type | description                |
| --------------- | --------- | -------------------------- |
| code            | Int    | response status code       |
| accessToken     | String    | token                      |
| expireTimestamp | Long      | timestamp for token expire |
| chatUserName | String    | chat user id               |
| agoraUid | String    | agora uid              |

**response example:**

```json
{
    "code": 200,
    "accessToken": "xxx",
    "expireTimestamp": 1628245967857,
    "chatUserName": "jack",
    "agoraUid": "12356"
}
```
