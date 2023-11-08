---
title: Go Language Error Code Design and Management Practices
date: 2023-11-06 12:04:00
categories: 
  - Go
tags: 
  - Backend Technology Sharing
  - defined
  - third-party
  - recognize
  - development
  - encountered
  - network
  - messages
description: If we defined the same error once every time we encountered it with a similar errors.New(). Not only would there be a lot of duplicate code, but it would also be very difficult to sort through our error messages to web developers or third-party platforms. So we thought of unifying our error messages
cover: https://raw.githubusercontent.com/zqwuming/blogimage/img/img/512cff201dee4b49b77eb75ca6cd3796.jpeg
---
# 1. Introduction

## 1.1 Background

Recently, I've been working on a service that directly interacts with front-end and third-party platforms (which can be simply understood as other departments of the company or client software), involving modules such as user registration, login, data processing, etc. The architecture diagram is roughly as follows. The architecture diagram is roughly as follows:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/b7063229b44642e3877adef40d200b0b%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

After getting the requirements, combined with the team's internal familiar technology stack, we determined that the backend service [business logic layer] using Golang language to develop, the framework used are Gin to do HTTP interaction, Swaggo automatic generation of interface documents, Redis and MySQL as K-V and DB storage.

It is worth noting that the application requires us to **specify and normalize the errors on the third-party platform and the Web side**, for example: the error code information on the Web side is also available to the third-party platform.

Therefore, the design and management of error code specification becomes our primary problem.

## 1.2 Features

The Go language provides a simple error handling mechanism: `error &#x7C7B;&#x578B;`. error is an interface type defined as follows:

```go
type error interface {
    Error() string
}
```

The use of error can be seen everywhere in the code, e.g., the database triple Gorm auto-incrementing tables, Gin getting parameters, etc:

```go

func (db *DB) AutoMigrate(dst ...interface{}) error {
    return db.Migrator().AutoMigrate(dst...)
}
```

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/6382ed53d355471e93791df4ba693e12%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

In addition to Go itself and the use of three-way packages, we can also implement specific error messages through `errors.New()`:

```css
func div(a, b int) (float, error) {
   if b == 0 {
       return 0, errrors.New("除数不能为0")
  }
   return float(a)/float(b)
}
```

However, a new problem arises.

If we define the same error with a similar `errors.New()` definition every time we encounter it, we will not only have a lot of duplicate code, but it will also be very difficult to sort out our error messages for web development or third party platforms. Not only would there be a lot of duplicate code, but it would also be very difficult to comb through our error messages to web-side development or third-party platforms.

**Imagine 100,000 lines of code, and it would be more or less unseemly to go through them one by one looking for `errors.New()` information! **

# 2. Defining Error Codes and Messages

## 2.1 Error Code Design Specifications

So we thought of unifying error messages and uniquely identifying them with error codes. That is: ** an error code corresponds to an error message **, every time you need it, just use the error code directly.

The error code in the industry adopts `5~7` bit integer (space-saving) constants to define, so we adopt **5 bit numeric error code, Chinese error message**, and divide the range of error code according to the business module.

### Module Description

Module description 1****1 starts with service level error code, such as internal service error, wrong parameter information, etc. 2****2 starts with: business module level error code 201***201 starts with error code of dataset module 202***202: user management module 203***203: pre-training management module

## 2.2 Error code definition

Create a new `err_code` package with a new `error_handle.go` file:

```go
package err_code

import "github.com/pkg/errors"

type Response struct {
    Code      ErrCode `json:"code"`       
    Msg       string  `json:"msg"`           
    RequestId string  `json:"request_id"` 
}
```

Added error codes and error messages:

```java
type ErrCode int 

const (

ServerError    ErrCode = 10001
ParamBindError ErrCode = 10002

IllegalDatasetName ErrCode = 20101 
ParamNameError     ErrCode = 20102 

IllegalPhoneNum         ErrCode = 20201 
IllegalVerifyCode       ErrCode = 20202 
PhoneRepeatedRegistered ErrCode = 20203 
PhoneIsNotRegistered    ErrCode = 20204 
PhoneRepeatedApproved   ErrCode = 20205 
PhoneIsNotApproved      ErrCode = 20206 

IllegalModelName 20301 
)
```

## 2.2 Map Mapping Error Messages

Based on the error code, we use Map mapping to define the **Chinese error message**:

```go

var errorMsg = map[int]string{
ServerError:          "服务内部错误",
ParamBindError:     "参数信息有误",
IllegalDatasetName: "无效的数据集名称",
ParamNameError:     "参数name错误",
IllegalPhoneNum:    "手机号格式不正确",
IllegalModelName:   "非法模型名称",
}

func Text(code int) string {
    return errorMsg[code]
}

func NewCustomError(code ErrCode) error {

    return errors.Wrap(&Response{
        Code: code,
        Msg:  code.String(),
    }, "")
}
```

Use the error code information:

```go

func CheckMobile(phone string) bool {

regRuler := "^1[345789]{1}\d{9}$"

reg := regexp.MustCompile(regRuler)

return reg.MatchString(phone)

}

func savePhoneNum(phone string) error {
   if phone == "" || !CheckMobile(phone) {

return NewCustomError(err_code.IllegalPhoneNum)
}
}
```

In this way, our error code mechanism is effectively set up, with the benefits of:

* Solve the problem of difficult to manage error information: all in a `err_code` package, at a glance you can know what error information the service has, ** easy to collect and error localization **;
* solved the problem of uneven error code, arbitrary definition: according to the business module divided into different numerical ranges of error code, ** according to the error code you can know which module is the problem, to avoid tearing the skin **;

However, some smart and studious friends may have found it. Every time you define a new error code, you need to add the error code number and Map mapping error information, is there a more concise way to define it?

The answer is yes! As a programmer who often tries to be lazy, **simple and efficient automation** is the goal we are pursuing.

# 3. Automated generation of error codes and error messages

## 3.1 stringer

`stringer` is a toolkit open-sourced for the Go language, and the installation command is:

> go install golang.org/x/tools/cmd/stringer

In addition to the toolkit, we also need Go's `iota` counter for automatic accumulation of constant numbers:

> PS: `iota` is the **go language's constant counter** and can only be used in constant expressions.
> Its value starts at 0, and grows by 1 for each new line in const. iota increases its value by 1 until it encounters the next const keyword, when its value is reset to 0.

## 3.2 Defining Error Messages

```go
package err_code

import "github.com/pkg/errors"

type Response struct {
  Code      ErrCode `json:"code"`       
  Msg       string  `json:"msg"`        
  RequestId string  `json:"request_id"` 
}

func (e *Response) Error() string {
  return e.Code.String()
}

type ErrCode int 

const (

ServerError     ErrCode = iota + 10001 
ParamBindError                         
TokenAuthFail                          
TokenIsNotExist                        
)

const (

IllegalDatasetName ErrCode = iota + 20101 
)

const (

IllegalPhoneNum         ErrCode = iota + 20201 
IllegalVerifyCode                              
PhoneRepeatedRegistered                        
PhoneIsNotRegistered                           
PhoneRepeatedApproved                          
PhoneIsNotApproved                             
)

const (

IllegalModelName ErrCode = iota + 20301 
)

func NewCustomError(code ErrCode) error {

return errors.Wrap(&Response{
Code: code,
Msg:  code.String(),
}, "")
}
```

With the above definition of the error code **const constant + error code name + error message comment**, where `iota` is automatically constant-accumulated.

I.e. `ParamBindError` is `10002` and `TokenAuthFail` is `10003`:

```go

const (

   ServerError     ErrCode = iota + 10001
   ParamBindError
   TokenAuthFail
   TokenIsNotExist
)
```

There are two ways we can generate error messages for error code mapping.

### 1) Run the `stringer` utility in `Goland

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/8541198b92fd426893da327ff1bdd3f2%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

### 2) Execute the command to run the `stringer` utility

We run the following command on the `err_code/error_handle.go` file:

> go generate internal/protocols/err_code/error_handle.go

This generates a new `errcode_string.go` file with a mapping of `err_code` to `err_msg`:

```ini
// Code generated by "stringer -type ErrCode -linecomment"

package err_code

import "strconv"

func _() {
   // An "invalid array index" compiler error signifies that the constant values have changed.

   // Re-run the stringer command to generate them again.

   var x [1]struct{}
   _ = x[ServerError-10001]
   _ = x[ParamBindError-10002]
   _ = x[TokenAuthFail-10003]
   _ = x[TokenIsNotExist-10004]
   _ = x[IllegalDatasetName-20101]
   _ = x[IllegalPhoneNum-20201]
   _ = x[IllegalVerifyCode-20202]
   _ = x[PhoneRepeatedRegistered-20203]
   _ = x[PhoneIsNotRegistered-20204]
   _ = x[PhoneRepeatedApproved-20205]
   _ = x[PhoneIsNotApproved-20206]
   _ = x[IllegalModelName-20301]
}

const (
   _ErrCode_name_0 = "服务内部错误参数信息有误Token鉴权失败Token不存在"
   _ErrCode_name_1 = "非法数据集名称"
   _ErrCode_name_2 = "手机号格式不正确无效的验证码手机号不可重复注册该手机号未注册手机号不可重复审批该手机号未审批"
   _ErrCode_name_3 = "非法模型名称"
)

var (
   _ErrCode_index_0 = [...]uint8{0, 18, 36, 53, 67}
   _ErrCode_index_2 = [...]uint8{0, 24, 42, 69, 90, 117, 138}
)

func (i ErrCode) String() string {
   switch {
   case 10001 -= 10001
      return _ErrCode_name_0[_ErrCode_index_0[i]:_ErrCode_index_0[i+1]]
   case i == 20101:
      return _ErrCode_name_1
   case 20201 -= 20201
      return _ErrCode_name_2[_ErrCode_index_2[i]:_ErrCode_index_2[i+1]]
   case i == 20301:
      return _ErrCode_name_3
   default:
      return "ErrCode(" + strconv.FormatInt(int64(i), 10) + ")"
   }
}
```

This way, we don't have to manually create a new Map to maintain the mapping relationship!

> Note: After each addition, deletion or modification of error codes, you need to execute `go generate` to generate a new mapping file `errcode_string.go`.
> This file is the mapping file for error codes and error messages, do not modify or delete it manually!

# 4. Error Code Practice

In summary, we have defined the error code message. Next,  interface to briefly demonstrate the usage.

A portion of the `go.mod` dependencies are listed below:

```bash
module wanx-llm-server

go 1.20

require (
    github.com/gin-gonic/gin v1.9.1
    github.com/pkg/errors v0.9.1
    github.com/spf13/viper v1.16.0
    github.com/swaggo/gin-swagger v1.6.0
    github.com/swaggo/swag v1.16.1
    go.uber.org/zap v1.25.0
    golang.org/x/arch v0.4.0 // indirect
    golang.org/x/tools v0.12.0 // indirect
    google.golang.org/protobuf v1.31.0 // indirect
    gorm.io/driver/mysql v1.5.1
    gorm.io/gorm v1.25.4
)
```

Add `main.go` as the service startup entry with the following code:

```go
package main

import (
   "flag"
   "fmt"
   "os"

   "go.uber.org/zap"
   _ "wanx-llm-server/docs"
   "wanx-llm-server/internal/cmd"
   "wanx-llm-server/internal/global"
   "wanx-llm-server/internal/initialize"
   util "wanx-llm-server/internal/utils"
)

func main() {
   configPath := flag.String("conf", "./config/config.yaml", "config path")
   flag.Parse()

   err := initialize.Init(*configPath)
   if err != nil {
      global.Logger.Error("server init failed", zap.Any(util.ErrKey, err))
      fmt.Printf("server init failed, %v\n", err)
      os.Exit(1)
   }

   r := cmd.SetupRouter()

   addr := fmt.Sprintf(":%v", 8088)
   if err := r.Run(addr); err != nil {
      global.Logger.Error(fmt.Sprintf("gin run failed, %v", err))
      return
   }
}
```

`server.go` As a `HTTP` request entry, the key code is as follows：

```scss
func SetupRouter() *gin.Engine {
    r := gin.Default()
    r.POST("/api/v1/user/register", userRegister)
    return r
}

func userRegister(c *gin.Context) {
	requestId := c.Writer.Header().Get("X-Request-Id")
	resp := &err_code.Response{RequestId: requestId}

	defer func() {
		if resp.Code != 0 {
			c.JSONP(http.StatusOK, &user.GenerateCodeResp{Response: resp})
		}
	}()

	req := &user.RegisterUserReq{}
	err := c.BindJSON(req)
	if err != nil {
		errors.As(err_code.NewCustomError(err_code.ParamBindError), &resp)
		return
	}

	err = service.RegisterUser(requestId, req)
	if err != nil {

		if !errors.As(err, &resp) {
			errors.As(err_code.NewCustomError(err_code.ServerError), &resp)
		}
		return
	}

	c.JSONP(http.StatusOK, &user.RegisterUserResp{
		Response: resp,
		Data:     user.RegisterUser{State: service.RegisteredState},
	})
}
```

`service/user.go` To realize the specific business, the key code is as follows：

```go

func RegisterUser(requestId string, req *user.RegisterUserReq) error {
	if req.Phone == "" || !CheckMobile(req.Phone) {
		return err_code.NewCustomError(err_code.IllegalPhoneNum)
	}

	smsOBj := &sms.SMS{
		Phone:      req.Phone,
		Code:       req.Code,
		CodeExpire: global.Config.CodeSMS.VerifyCodeExpire,
	}
	codePass, msg, err := smsOBj.VerifyCode(global.RedisClient)
	if err != nil {
		return err_code.NewCustomError(err_code.ServerError)
	}

	if !codePass {
		return err_code.NewCustomError(err_code.IllegalVerifyCode)
	}

	exist, err := (&model.ApprovedTable{}).IsExistByPhone(req.Phone)
	if err != nil {
		return err_code.NewCustomError(err_code.ServerError)
	}

	if exist {
		return err_code.NewCustomError(err_code.PhoneRepeatedRegistered)
	}

	ur := &model.UserApproved{
		Phone: req.Phone,
		State: RegisteredState,
	}

	_, err = (&model.ApprovedTable{}).Insert(ur)
	if err != nil {
		return err_code.NewCustomError(err_code.ServerError)
	}
	return nil
}
```

In the example, by making direct calls to the error code, we avoid the frequent steps of throwing and receiving errors, followed by `error_code` collocation.

In this way, a standardized system of error codes is created!

End of story, sprinkle flowers!
