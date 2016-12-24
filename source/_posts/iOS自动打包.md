---
title: iOS自动打包
date: 2016-12-24 11:46:45
tags: [iOS, 打包]
---

做iOS开发这么久，一直给测试打包都是使用Xcode先进行Archive，然后再将对应的xcarchive文件Export为一个ipa文件。当然做了这么久感觉也没有什么不对的，但是前段时间我们老大希望能让我准备一个脚本，然后以后就让运维通过脚本打包，这样一想也的确省去不少麻烦，就算不让运维打包，我们自己用脚本也省去了不断点下一步的麻烦操作，于是我就开始通过谷歌和度娘查找相关资料，根据网上资料，网上流传的打包脚本无外两种：

第一种是先通过xcodebuild编译工程得到一个XXX.app文件，脚本如下：

```
xcodebuild -project XXXXXX.xcodeproj -scheme XXXXXX -configuration Release -sdk iphoneos build CODE_SIGN_IDENTITY="XXXXXX" PROVISIONING_PROFILE="XXXXXXX"
xcodebuild -workspace XXXXXX.xcworkspace -scheme XXXXXX -configuration Release -sdk iphoneos build CODE_SIGN_IDENTITY="XXXXXX" PROVISIONING_PROFILE="XXXXXXX"
```

这里有两句脚本，这里的两句是二选一的，如果你是需要打包的是一个普通的project就是用第一句，如果你需要打包的是一个workspace（当你使用了CocoaPods添加第三方库，你就需要使用第二句）就用第二句。
然后在使用xcrun将XXX.app文件转为XXX.ipa，脚本如下：

```
xcrun -sdk iphoneos -v PackageApplication ./Build/Products/Release-iphoneos/XXXXXX.app -o bin/XXXXXX.ipa
```

这里CODE_SIGN_IDENTITY等号后面的就是对应的开发者证书，PROVISIONING_PROFILE等号后面对应的是PROVISIONING_PROFILE的uuid。  
这种方式打包相当于先在Xcode进行build，然后将Products目录下的app文件直接拖入iTunes所得，但是我并不建议该种方法，因为这种方法并不完全同于我们以前的打包流程。

第二种就是先通过xcodebuild archive将项目工程打包为xcarchive文件，脚本如下：

```
xcodebuild archive -project XXXX.xcodeproj -scheme XXXX -archivePath bin/XXXX.xcarchive
xcodebuild archive -workspace XXXX.xcworkspace -scheme XXXX -archivePath bin/XXXX.xcarchive
```

这里同样是两句选一句，分别针对project和workspace
然后使用xcodebuild -exportArchive将xcarchive文件export为对应该ipa文件，脚本如下：

```
xcodebuild -exportArchive -archivePath bin/XXXX.xcarchive -exportPath bin/$XXXX -exportFormat IPA -exportProvisioningProfile "[PROVISION_PROFILE]"
```

这里就是将的xcarchive通过使用PROVISION_PROFILE文件打包为ipa文件，这里[PROVISION_PROFILE]代表项目对应的PROVISION_PROFILE文件的文件名。  
这种方式的打包是最接近于我之前手动打包的方式。

这里附上完整脚本：

```
#!/bin/sh

PROJECT_NAME="XXXXX"
WORKSPACE_NAME="XXXXX"
PROVISION_PROFILE="XXXXXX"

rm -rf bin

# 如果是打包xcodeproj就使用这句
# xcodebuild archive -project ${PROJECT_NAME}.xcodeproj -scheme ${PROJECT_NAME} -archivePath bin/${PROJECT_NAME}.xcarchive

# 如果是打包xcworkspace就使用这句
xcodebuild archive -workspace ${WORKSPACE_NAME}.xcworkspace -scheme ${WORKSPACE_NAME} -archivePath bin/${WORKSPACE_NAME}.xcarchive

xcodebuild -exportArchive -archivePath bin/${WORKSPACE_NAME}.xcarchive -exportPath bin/${WORKSPACE_NAME} -exportFormat IPA -exportProvisioningProfile "${PROVISION_PROFILE}"
```

这里只需修改PROJECT_NAME，WORKSPACE_NAME和PROVISION_PROFILE即可。  
脚本地址：[ios_auto_build](https://github.com/MingleChang/ios_auto_build.git)

