# 使用com.novoda:bintray-release上传多模块到bintray jcenter

[TOC]

# 注册bintray账号
1. 官网注册账号 https://bintray.com，切记点击右边的For an Open Source Account。
2. 创建一个名为maven的Maven类型的Repository。
3. 在右上角，头像的下拉选项中，Edit Profile. 记录下自己的APP KEY和User Name。

# 使用

因为是多模块，我们统一配置

## 1. 在项目的build.gradle里（非模块）添加
    
    
```
buildscript {
    
    repositories {
        mavenCentral()
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.1'
        classpath 'com.novoda:bintray-release:0.8.0'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}
```

**classpath 里添加classpath 'com.novoda:bintray-release:0.8.0'**


```
ext {
    userOrg = 'davy'
    groupId = 'davy.utils'
    uploadName = 'annotaion_utils'
    publishVersion = '1.0.0'
    desc = 'a simple tool for android'
    website = 'https://github.com/davyjoneswang/AndroidAnnotatonProcessor'
    licences = ['Apache-2.0']
}
```

**这些添加并修改为自己的配置。**
> - userOrg 为用户名，
> - groupId 为项目的groupId， 可以理解为包名。
> - uploadName 为上传后，在bintray显示的名字。
> - publishVersion 为统一的版本号
> - desc 项目描述
> - website 项目网址，填写自己的github 项目网址，【验证时会用，填写能打开的项目网址】
> - licences 项目的licences。

## 2.  模块的的build.gradle配置
  
```
apply plugin: 'com.novoda.bintray-release'

publish {
    artifactId = 'annotation'
    userOrg = rootProject.userOrg
    groupId = rootProject.groupId
    uploadName = rootProject.uploadName
    publishVersion = rootProject.publishVersion
    desc = rootProject.description
    website = rootProject.website
    licences = rootProject.licences
}
```

**唯一区别就是artifactId不同。**

## 3. 上传执行。

```
./gradlew clean generatePomFileForReleasePublication build bintrayUpload -PbintrayUser=*** -PbintrayKey=******** -PdryRun=false
```
使用自己的UserName和Key.

上传成功后，还不能直接使用，需要审核，通过后才能使用。

进入bintray，maven 仓库，项目。如果正常，你就可以看到你上传的文件，右侧有add to Jcenter按钮, 点击，填写信息，并勾选就可以等待审核了。

# 填坑
1.  **没有add to Jcenter按钮**

请确认，是否按照上面的步骤注册的账号，Jcenter注册分为个人和企业。我们需要注册个人版。如果错了，请在右上角用户配置里，删除账号，重新注册。

2. **Could not create package HTTP/1.1 404 Not Found[message:Repo ‘maven’ was not found]**
  
请创建maven仓库
3.  **提示 Please fix the following before submitting a JCenter inclusion request: Add a POM file to the latest version of your package.**

这是因为插件没有创建pom文件，请确认是否是按照上面执行的 上传命令，直接使用插件官网的命令就会出现问题。 个人观察，如果你的工程包含Android模块，经常会这样。


参考:
 - http://blog.csdn.net/lxd_android/article/details/79076312
