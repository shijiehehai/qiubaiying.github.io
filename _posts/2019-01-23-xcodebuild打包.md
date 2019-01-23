---
layout:     post
title:      xcodebuild打包
date:       2019-01-23
author:     sjhh
header-img: img/WechatIMG61.jpeg
catalog: 	 true
tags:
    - iOS
---



# xcodebuild打包步骤

以打ad-hoc包为例，打包主要分为下载代码，archive源代码，打ipa包几部，需要用到的准备文件有证书ID和描述文件以及导出ipa包时的ExportOptions.plist文件。

## 下载源代码

代码仓库拉取源代码，如果是pod管理的，最好执行pod install命令

## archive源代码

必须关闭xcode9的Automatically manage signing功能，如果必须，请手动指定证书Identity和描述文件UUID。

```
xcodebuild archive -workspace 项目名称.xcworkspace 未使用cocoapod,使用-project 项目名称.xcodeproj
                   -scheme 项目名称 
                   -target target名称
                   -configuration 构建配置 Debug Release
                   -archivePath archive包存储路径，精确到文件名 
                   CODE_SIGN_IDENTITY=证书 
                   PROVISIONING_PROFILE=描述文件UUID
```

xcodebuild -list查看虽有scheme和configuration

```
Information about project "YuQingTong":
    Targets:
        YuQingTong
        YuQingTongTests
        YuQingTong_Gansu

    Build Configurations:
        Debug
        Release

    If no build configuration is specified and -scheme is not passed then "Debug" is used.

    Schemes:
        YuQingTong
        YuQingTong_Gansu
        YuQingTongTests
        
```

### 证书Identity

keychain中选中证书，标题即证书Identity。

[![kAXrPP.md.png](https://s2.ax1x.com/2019/01/23/kAXrPP.md.png)](https://imgchr.com/i/kAXrPP)

### 描述文件UUID

Apple develop center导出包含项目APP ID和前文使用的证书的描述文件。双击描述文件安装到本地后，执行以下命令获取描述文件的UUID。

```
/usr/bin/security cms -D -i 098a87e3-11fe-463d-75aa-12345678adba.mobileprovision
ps:xcode8以上的描述文件的安装目录是/Users/用户名/Library/MobileDevice/Provisioning Profiles
```

![kAXjaR.png](https://s2.ax1x.com/2019/01/23/kAXjaR.png)

### Archive Commond Demo

```
xcodebuild archive -workspace workspaceName.xcworkspace  -scheme schemeName -archivePath /Path/to/xarchive/20190123-adhoc.xcarchive  -configuration Debug CODE_SIGN_IDENTITY='CODE_SIGN_IDENTITY' PROVISIONING_PROFILE=PROVISIONING_PROFILE
```

## 导出IPA包

导出IPA包依赖之前生成的archive文件，另外需要导出配置文件ExportOptions.plist

```
xcodebuild  -exportArchive -archivePath /Users/shijie/Desktop/test/20190123-YuQingTong-adhoc.xcarchive  -exportOptionsPlist /Users/shijie/Desktop/test/ExportOptions.plist -exportPath /Users/shijie/Desktop/test/Project_Name-adhoc.ipa
```

## 自动导出脚本

### 非WorkSpace工程

```
#注意：脚本目录和xxxx.xcodeproj要在同一个目录，如果放到其他目录，请自行修改脚本。
#工程名字(Target名字)
Project_Name="Target名字，系统默认和工程名字一样"
#配置环境，Release或者Debug
Configuration="Release"

#AdHoc版本的Bundle ID
AdHocBundleID="com.xxx"
#AppStore版本的Bundle ID
AppStoreBundleID="com.xxx"
#enterprise的Bundle ID
EnterpriseBundleID="com.xxx"

# ADHOC
#证书名#描述文件
ADHOCCODE_SIGN_IDENTITY="iPhone Distribution: xxxx"
ADHOCPROVISIONING_PROFILE_NAME="xxxx-xxxx-xxxx-xxxx"

#AppStore证书名#描述文件
APPSTORECODE_SIGN_IDENTITY="iPhone Distribution: xxxx"
APPSTOREROVISIONING_PROFILE_NAME="xxxx-xxxx-xxxx-xxxx"

#企业(enterprise)证书名#描述文件
ENTERPRISECODE_SIGN_IDENTITY="iPhone Distribution: xxxxx"
ENTERPRISEROVISIONING_PROFILE_NAME="xxxx-xxxx-xxxx-xxxx"

#加载各个版本的plist文件
ADHOCExportOptionsPlist=./ADHOCExportOptionsPlist.plist
AppStoreExportOptionsPlist=./AppStoreExportOptionsPlist.plist
EnterpriseExportOptionsPlist=./EnterpriseExportOptionsPlist.plist

ADHOCExportOptionsPlist=${ADHOCExportOptionsPlist}
AppStoreExportOptionsPlist=${AppStoreExportOptionsPlist}
EnterpriseExportOptionsPlist=${EnterpriseExportOptionsPlist}

echo "~~~~~~~~~~~~选择打包方式(输入序号)~~~~~~~~~~~~~~~"
echo "  1 appstore"
echo "  2 adhoc"
echo "  3 enterprise"

# 读取用户输入并存到变量里
read parameter
sleep 0.5
method="$parameter"

# 判读用户是否有输入
if [ -n "$method" ]
then

#clean下
xcodebuild clean -xcodeproj ./$Project_Name/$Project_Name.xcodeproj -configuration $Configuration -alltargets

    if [ "$method" = "1" ]
    then

#appstore脚本
xcodebuild -project $Project_Name.xcodeproj -scheme $Project_Name -configuration $Configuration -archivePath build/$Project_Name-appstore.xcarchive clean archive build  CODE_SIGN_IDENTITY="${APPSTORECODE_SIGN_IDENTITY}" PROVISIONING_PROFILE="${APPSTOREROVISIONING_PROFILE_NAME}" PRODUCT_BUNDLE_IDENTIFIER="${AppStoreBundleID}"
xcodebuild -exportArchive -archivePath build/$Project_Name-appstore.xcarchive -exportOptionsPlist $AppStoreExportOptionsPlist -exportPath ~/Desktop/$Project_Name-appstore.ipa
    elif [ "$method" = "2" ]
    then
#adhoc脚本
xcodebuild -project $Project_Name.xcodeproj -scheme $Project_Name -configuration $Configuration -archivePath build/$Project_Name-adhoc.xcarchive clean archive build CODE_SIGN_IDENTITY="${ADHOCCODE_SIGN_IDENTITY}" PROVISIONING_PROFILE="${ADHOCPROVISIONING_PROFILE_NAME}" PRODUCT_BUNDLE_IDENTIFIER="${AdHocBundleID}"
xcodebuild -exportArchive -archivePath build/$Project_Name-adhoc.xcarchive -exportOptionsPlist $ADHOCExportOptionsPlist -exportPath ~/Desktop/$Project_Name-adhoc.ipa
    elif [ "$method" = "3" ]
    then
#企业打包脚本
xcodebuild -project $Project_Name.xcodeproj -scheme $Project_Name -configuration $Configuration -archivePath build/$Project_Name-enterprise.xcarchive clean archive build CODE_SIGN_IDENTITY="${ENTERPRISECODE_SIGN_IDENTITY}" PROVISIONING_PROFILE="${ENTERPRISEROVISIONING_PROFILE_NAME}" PRODUCT_BUNDLE_IDENTIFIER="${EnterpriseBundleID}"
xcodebuild -exportArchive -archivePath build/$Project_Name-enterprise.xcarchive -exportOptionsPlist $EnterpriseExportOptionsPlist -exportPath ~/Desktop/$Project_Name-enterprise.ipa
else
    echo "参数无效...."
    exit 1
    fi
fi
```



### WorkSpace工程

```
#注意：脚本目录和WorkSpace目录在同一个目录
#工程名字(Target名字)
Project_Name="Target名字,系统默认等于工程名字"
#workspace的名字
Workspace_Name="WorkSpace名字"
#配置环境，Release或者Debug,默认release
Configuration="Release"

#AdHoc版本的Bundle ID
AdHocBundleID="com.xxxx"
#AppStore版本的Bundle ID
AppStoreBundleID="com.xxxx"
#enterprise的Bundle ID
EnterpriseBundleID="com.xxxx"

# ADHOC证书名#描述文件
ADHOCCODE_SIGN_IDENTITY="iPhone Distribution: xxxx"
ADHOCPROVISIONING_PROFILE_NAME="xxxxx-xxxx-xxxx-xxxx-xxxxxx"

#AppStore证书名#描述文件
APPSTORECODE_SIGN_IDENTITY="iPhone Distribution: xxxxx"
APPSTOREROVISIONING_PROFILE_NAME="xxxxx-xxxx-xxxx-xxxx-xxxxxx"

#企业(enterprise)证书名#描述文件
ENTERPRISECODE_SIGN_IDENTITY="iPhone Distribution: xxxx"
ENTERPRISEROVISIONING_PROFILE_NAME="xxxxx-xxxx-xxx-xxxx"

#加载各个版本的plist文件
ADHOCExportOptionsPlist=./ADHOCExportOptionsPlist.plist
AppStoreExportOptionsPlist=./AppStoreExportOptionsPlist.plist
EnterpriseExportOptionsPlist=./EnterpriseExportOptionsPlist.plist

ADHOCExportOptionsPlist=${ADHOCExportOptionsPlist}
AppStoreExportOptionsPlist=${AppStoreExportOptionsPlist}
EnterpriseExportOptionsPlist=${EnterpriseExportOptionsPlist}

echo "~~~~~~~~~~~~选择打包方式(输入序号)~~~~~~~~~~~~~~~"
echo "  1 adHoc"
echo "  2 AppStore"
echo "  3 Enterprise"

# 读取用户输入并存到变量里
read parameter
sleep 0.5
method="$parameter"

# 判读用户是否有输入
if [ -n "$method" ]
then
    if [ "$method" = "1" ]
    then
#adhoc脚本
xcodebuild -workspace $Workspace_Name.xcworkspace -scheme $Project_Name -configuration $Configuration -archivePath build/$Project_Name-adhoc.xcarchive clean archive build CODE_SIGN_IDENTITY="${ADHOCCODE_SIGN_IDENTITY}" PROVISIONING_PROFILE="${ADHOCPROVISIONING_PROFILE_NAME}" PRODUCT_BUNDLE_IDENTIFIER="${AdHocBundleID}"
xcodebuild  -exportArchive -archivePath build/$Project_Name-adhoc.xcarchive -exportOptionsPlist ${ADHOCExportOptionsPlist} -exportPath ~/Desktop/$Project_Name-adhoc.ipa

    elif [ "$method" = "2" ]
    then
#appstore脚本
xcodebuild -workspace $Workspace_Name.xcworkspace -scheme $Project_Name -configuration $Configuration -archivePath build/$Project_Name-appstore.xcarchive archive build CODE_SIGN_IDENTITY="${APPSTORECODE_SIGN_IDENTITY}" PROVISIONING_PROFILE="${APPSTOREROVISIONING_PROFILE_NAME}" PRODUCT_BUNDLE_IDENTIFIER="${AppStoreBundleID}"
xcodebuild  -exportArchive -archivePath build/$Project_Name-appstore.xcarchive -exportOptionsPlist ${AppStoreExportOptionsPlist} -exportPath ~/Desktop/$Project_Name-appstore.ipa

    elif [ "$method" = "3" ]
    then
#企业打包脚本
xcodebuild -workspace $Workspace_Name.xcworkspace -scheme $Project_Name -configuration $Configuration -archivePath build/$Project_Name-enterprise.xcarchive archive build CODE_SIGN_IDENTITY="${ENTERPRISECODE_SIGN_IDENTITY}" PROVISIONING_PROFILE="${ENTERPRISEROVISIONING_PROFILE_NAME}" PRODUCT_BUNDLE_IDENTIFIER="${EnterpriseBundleID}"
xcodebuild  -exportArchive -archivePath build/$Project_Name-enterprise.xcarchive -exportOptionsPlist ${EnterpriseExportOptionsPlist} -exportPath ~/Desktop/$Project_Name-enterprise.ipa
    else
    echo "参数无效...."
    exit 1
    fi
fi
```

以上脚本来自：https://github.com/qindeli/WorksapceShell