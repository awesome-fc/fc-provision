# 函数计算预留模式示例项目

该项目是一个函数计算预留模式的示例模板项目，借助于 fun 工具，可以快速初始化并部署一个支持预留模式的函数。该项目帮助用户理解如何在 template.yml 文件中描述预留模式。

**备注: 本文介绍的技巧需要 Fun 版本大于等于 3.6.9。**

## 依赖工具

本项目是在 MacOS 下开发的，涉及到的工具是平台无关的，对于 Linux 和 Windows 桌面系统应该也同样适用。在开始本例之前请确保如下工具已经正确的安装，更新到最新版本，并进行正确的配置。

* [Docker](https://www.docker.com/)
* [Fun](https://github.com/aliyun/fun)

Fun 工具依赖于 docker 来模拟本地环境。

对于 MacOS 用户可以使用 [homebrew](https://brew.sh/) 进行安装：

```bash
brew cask install docker
brew tap vangie/formula
brew install fun
```

Windows 和 Linux 用户安装请参考：

1. https://github.com/aliyun/fun/blob/master/docs/usage/installation.md
2. https://github.com/aliyun/fcli/releases

安装好后，记得先执行 `fun config` 初始化一下配置。

## 初始化

使用 fun init 命令可以快捷的将本模板项目初始化到本地。

```bash
fun init https://github.com/awesome-fc/fc-provision.git
```

template.yml 文件内容如下

```yaml
ROSTemplateFormatVersion: '2015-09-01'
Transform: 'Aliyun::Serverless-2018-04-03'
Resources:
  provision-svr:
    Type: 'Aliyun::Serverless::Service'
    Properties:
      Description: 'helloworld'
    provision-fun:
      Type: 'Aliyun::Serverless::Function'
      Properties:
        Handler: index.handler
        Runtime: nodejs10
        CodeUri: './'
  version1:
    Type: "ALIYUN::FC::Version"
    Properties:
      ServiceName: 
        Fn::GetAtt:
          - provision-svr
          - ServiceName
  alias1:
    Type: "ALIYUN::FC::Alias"
    Properties:
      VersionId: 
        Fn::GetAtt:
          - version1
          - VersionId
      ServiceName: 
        Fn::GetAtt:
          - provision-svr
          - ServiceName
      AliasName: prod
  provision1:
    Type: "ALIYUN::FC::ProvisionConfig"
    Properties:
      ServiceName: 
        Fn::GetAtt:
          - provision-svr
          - ServiceName
      FunctionName:
        Fn::GetAtt:
          - provision-svrprovision-fun
          - FunctionName
      Target: 1
      Qualifier:
        Fn::GetAtt:
          - alias1
          - AliasName
```
其中 `Type: "ALIYUN::FC::ProvisionConfig"` 声明了预留资源，其属性说明如下

* ServiceName： 指定需要预留的服务名称
* FunctionName： 指定需要预留的函数名称
* Target： 设定预留实例数
* Qualifier： 设定预留服务的别名

`Fn::GetAtt` 是 ROS 的求值语法，用于获得 template.yml 内其他资源的属性值，以避免硬编码。

## 打包

package 会将本地的代码打包并上传到 OSS，用于后面的 ROS 部署。

```bash
fun package
```

## 部署

```bash
fun deploy --use-ros
```
