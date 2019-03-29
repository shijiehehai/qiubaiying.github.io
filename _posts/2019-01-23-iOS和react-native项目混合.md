---
layout:     post
title:      iOS和react-native项目混合
date:       2019-03-29
author:     sjhh
header-img: img/2019-03-29.jpg
catalog: 	 true
tags:
    - ios react-native
---

# react-native代码目录

​	gerrit仓库clone代码至本地，cd至项目根目录执行npm install。如果react-native环境未搭建，需要先搭建react-native环境。

- node安装

```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash
```

​	或者安装指定版本的node

```
nvm install vx.x.x 如v6.11.0 
```

​	查看版本

```
nvm ls-remote
```

- react模块

```
# 安装 react 模块
npm install react 

# 全局安装 react-native-cli 模块 
npm install -g react-native-cli

# 升级
npm install -g npm

# React Native 命令行工具 (react-native-cli)
npm install -g react-native-cli

# 安装Facebook 提供的监视文件变化的工具：Watchman
brew install watchman

# 安装静态的JS类型检查工具：flow
brew install flow
```

- react-native依赖安装

```
npm install 
```



# iOS项目目录

​	/path/to/react-native-project/ios下删除默认的所有文件，把已经存在的xcode工程内的文件和文件夹移至/path/to/react-native-project/ios目录。

- Pod文件更新

```
    pod 'React', :path => '../node_modules/react-native', :subspecs => [
    'Core',
    'RCTText',
    'RCTImage',
    'RCTActionSheet',
    'RCTGeolocation',
    'RCTNetwork',
    'RCTSettings',
    'RCTVibration',
    'CxxBridge',
    'RCTWebSocket',
    'ART',
    'RCTAnimation',
    'RCTBlob',
    'RCTCameraRoll',
    'RCTPushNotification',
    'RCTLinkingIOS',
    'DevSupport'
    ]
    
    pod "yoga", :path => "../node_modules/react-native/ReactCommon/yoga"
    pod 'DoubleConversion', podspec: '../node_modules/react-native/third-party-podspecs/DoubleConversion.podspec'
    pod 'Folly', podspec: '../node_modules/react-native/third-party-podspecs/Folly.podspec'
    pod 'glog', podspec: '../node_modules/react-native/third-party-podspecs/glog.podspec' 
    
    #RNVectorIcons依赖的安装，需要先在/path/to/react-native-project目录下执行下边注释中的两条命令
    #npm install react-native-vector-icons --save
	#react-native link react-native-vector-icons
  	pod 'RNVectorIcons', :path => '../node_modules/react-native-vector-icons'
```

- Pod 命令执行

```
pod install

#如果pod仓库过旧，执行 pod repo update
#可能需要修改podfile支持的最低iOS系统版本
#安装报错可以删除podfile.lock文件，重新安装所有的依赖
```





# 运行项目

- 本地运行方式

  1. /path/to/react-native-project目录下执行

  1. ```
     npm start	
     ```

  2. 启动本地服务，iOS代码中访问react-native资源基于如下形式

     ```
     NSString * strUrl = @"http://localhost:8081/index.bundle?platform=ios&dev=true";
     NSURL *jsCodeLocation = [NSURL URLWithString:strUrl];
     RCTRootView * rootView = [[RCTRootView alloc] initWithBundleURL:jsCodeLocation
                                                              moduleName:@"yqt"
                                                       initialProperties:nil
                                                           launchOptions:nil];
     ```

- react-native打包方式

  1. /path/to/react-native-project/ios/project_name目录下创建存放包文件目录react_native_bundle

     ```
     cd /path/to/react-native-project/ios;
     mkdir bundle;
     ```

  2. /path/to/react-native-project目录下执行命令

     ```
     react-native bundle --entry-file index.js --platform ios --dev false --bundle-output ./ios/bundle/index.jsbundle --assets-dest ./ios/bundle
     ```

  3. iOS代码中访问react-native资源基于如下形式

     ```
     NSURL *jsCodeLocation = [[NSBundle mainBundle] URLForResource:@"main" withExtension:@"jsbundle"];
     RCTRootView * rootView = [[RCTRootView alloc] initWithBundleURL:jsCodeLocation
                                                              moduleName:@"yqt"
                                                       initialProperties:nil
                                                           launchOptions:nil];
     ```

ps：以上两种方式运行xcode项目，既可以采用传统的直接打开xcode工具运行项目，也可以采用react-native run-ios命令行方式。