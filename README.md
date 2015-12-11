#cordova-ionic-archetype
本原型由maven-archetype-archetype快速生成而来，用于快速开发基于Cordova+Ionic的Hybrid App

---

##设计理念(Design Philosophy)
**种子项目(Seed Project)** 来源于日常的项目开发，它内置代码资产、技术理念和团队文化，用于后续一类项目的快速开发，也可用于孵化开发平台和产品。
他的宗旨是**资产化(Asset)**: 产出可持续交付客户价值的模式、解决方案和利好团队战斗力的文化，而不仅是一堆应付的代码。

目前我们采用Maven Archetype的方式来开发种子项目，本项目由种子项目**cordova-ionic-archetype**快速生成而来。
他具有如下**特性(Features)** : 

- 拥抱领域驱动开发，支持按领域模块化的分析、开发、交付和维护
    - 支持模块热插拔
    - 内置通用模块(Cross Cutting Modules)，如登录
    - ...
- 完美集成现代Web开发和移动App开发    
    - 满足现代Web开发的要求
        - 支持以状态机的方式组合业务模块
        - 支持依赖注入业务模块和细粒度的服务
        - 支持Web资源的优化，如JS,CSS的压缩合并，HTML的压缩等
        - ...
    - 满足移动App开发的过程
        - 支持Android自动生成密钥文件、自动签名、ZipAlign
        - 支持iOS自动签名
        - 支持OTA，自动发布到蒲公英(http://www.pgyer.com) ，可通过扫码下载App
          此特性主要便于iOS应用真机测试时，在不越狱的情况下安装难的问题
        - ...  
    - 支持几乎零成本移植为微信公众平台项目 
- 发布是开发出来的，而不是手工重复劳动
    - 开发代码和发布代码分离
    - 支持以Maven Plugin和NodeJS自动化发布过程
    - ...

---

下面我们以种子从播种、培育、收获的过程形象的介绍如何基于种子项目开发业务项目
##通过种子项目生成业务项目 - 播种
```
mvn archetype:generate 
-DarchetypeGroupId=org.zhoujianhui.archetypes
-DarchetypeArtifactId=cordova-ionic-archetype
-DarchetypeVersion=0.0.1-SNAPSHOT
-DinteractiveMode=false 
-DgroupId=org.zhoujianhui
-DartifactId=TestCordovaIonicArchetype
-Dversion=0.0.1-SNAPSHOT 
```

为了避免通过原型生成项目耗时较长的问题（有可能是从公网遍历所有的原型导致）
第一次执行时，可添加archetypeCatalog参数指向原型所在的Nexus私服id（见Maven的settings.xml）
```
-DarchetypeCatalog=nexus
```

第二次执行时，可将archetypeCatalog参数指定为local，因为第一次已将原型下载到Maven的本地库
```
-DarchetypeCatalog=local
```

###项目的结构(Project Layout) - 种子的基因
![项目结构](http://zhoujianhui.bitbucket.org/maven-archetype/cordova-ionic-archetype-layout.png)

####hooks
Hook是Cordova用于定制构建过程的手段，Cordova会在构建生命周期的各阶段调用hook来执行自定义的构建动作。
Hook支持多种语言和多种配置方式，我们采用NodeJS编写hook将其放在**hooks**文件夹下，并在config.xml中配置在哪个阶段执行此hook

####init
init用于初始化Cordova项目。目前仅用于自动化安装Cordova的平台和插件，因为平台和插件不是我们开发的，所以不能纳入版本控制。
那么当发生变化时，如组员安装了一个新的插件，其他人能通过初始化自动安装此插件，而不是更新代码。init包含：

- init-config.json : 初始化配置文件，用于配置平台和插件列表
- platform-processor.js : 平台处理器，能够通过
  ```
  node platform-processor.js add/remove
  ```
  添加和删除平台
- platform-util.js : 平台处理工具，供platform-processor.js调用
- plugin-processor.js : 插件处理器，能够通过
  ```
  node plugin-processor.js add/remove
  ```
  添加和删除插件
- plugin-util.js : 插件处理工具，供plugin-processor.js调用  

####release
业务项目发布代码和发布时的配置文件，包含：

- boot-config.js : 业务项目配置文件（发布代码），发布时用于替换**www/boot-config.js**（开发代码）
- build-android.template.json : 构建Android时的配置模版
- build-ios.template.json : 构建iOS时的配置模版
- index.html : 业务项目首页文件（发布代码），发布时用于替换**www/index.html**（开发代码）
- wro.properties,wro.xml : Web资源优化工具Wro4j的配置文件

####scss
利用scss开发css,目前用于自定义Ionic样式

####www
包含以模块化方式组织的业务项目开发代码，开发过程见《发布业务项目/收获》

####.gitignore
我们推荐使用分布式版本控制Git，.gitignore用于标注哪些文件不纳入版本控制

####config.xml
Cordova的配置文件

####pom.xml
Maven的配置文件，我们采用多配置(Multiple Profiles)的方式，自定义多个构建过程：  
![Maven多配置](http://zhoujianhui.bitbucket.org/maven-archetype/cordova-ionic-archetype-multiple-profiles.png)    
具体使用参见《运行业务项目/检查种子质量》《发布业务项目/收获》章节

####README.md
项目自描述文件，采用Markdown的语法，记录项目**基因**

此时生成的业务项目是一个**完整可运行**的Cordova项目。下面我们介绍如何运行这个业务项目
###运行业务项目 - 检查种子质量
####添加平台和插件
上文我们介绍了手工命令来安装平台和插件，这里我们推荐Maven Plugin的方式来执行。
我们的多配置中**init-platforms**和**init-plugins**分别用于安装平台和插件，只需勾选执行即可。  
![Maven执行项目初始化](http://zhoujianhui.bitbucket.org/maven-archetype/cordova-ionic-archetype-init.png)   

当然也可以通过Maven命令执行:
```
mvn package -P init-platforms
```
值得一提的是我们在用Hook的方式在安装完平台后自动安装插件，配置见config.xml:
```xml
<hook type="after_platform_add" src="hooks/after-platform-add.js"/>
```
添加平台后，如果安装新的插件，可在**init/init-config.json**中配置插件信息，然后执行：
```
mvn package -P init-plugins
```

####运行项目
开发阶段我们推荐使用模拟器运行项目，没有问题后再安装到设备。
众所周知Android的模拟器启动异常缓慢，iOS的模拟器需要苹果电脑，而我们的项目使用Web技术栈开发移动应用，所以我们基于浏览器的模拟器[Apache Ripple](http://ripple.incubator.apache.org)
>Apache Ripple™ is a web based mobile environment simulator designed to enable rapid development of mobile web applications for various web application frameworks, such as Apache Cordova™ and BlackBerry® WebWorks™. 
>It can be paired with current web based mobile development workflows to decrease time spent developing and testing on real devices and/or simulators.

首先我们安装Apache Ripple，它属于NodeJS Module，通过NodeJS的模块管理器**npm**安装
```
npm install -g ripple-emulator
```

启动模拟器
```
ripple emulate
``` 
![Apache Ripple](http://zhoujianhui.bitbucket.org/maven-archetype/cordova-ionic-archetype-ripple.png) 
我们主要通过浏览器的开发者工具调试解决JS的错误

---

##开发业务项目 - 培育
TODO

---

##发布业务项目 - 收获
TODO
