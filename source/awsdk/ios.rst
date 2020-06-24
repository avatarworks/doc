iOS 端 SDK 接入指南
=======================

概述
--------------------
AWSDK 是一个适用于 iOS 的虚拟人解决方案SDK。

功能以及版本
~~~~~~~~



阅读对象
--------------------

- 具备基本的 iOS 开发能力
- 准备接入虚拟人SDK

开发准备
--------------------

设备以及系统
~~~~~~~~

- iPhone4s 及以上设备
- iOS 9 及以上操作系统

前置条件
~~~~~~~~

- 已申请到了SDK的license

快速开始
--------------------
开发环境配置
~~~~~~~~
- Xcode 开发工具。App Store `下载地址`_

.. _下载地址: https://apps.apple.com/us/app/xcode/id497799835?ls=1&mt=12

导入 SDK
~~~~~~~~
- 在工程配置中，将 AWSDK.framework 导入工程中，勾选Embed方式为``Embed & Sign``，如下：
.. image:: /_static/img/awsdk_xcode_config.png

- 找到 Product -> Schemes -> Edit Scheme... -> Run -> Options， 将``Metal API Validation``选项勾选为``Disabled``
.. image:: /_static/img/awsdk_xcode_scheme.png

功能使用
--------------------

