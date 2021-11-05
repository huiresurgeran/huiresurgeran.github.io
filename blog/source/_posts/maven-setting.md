---
title: maven_setting
date: 2021-03-03 14:13:15
tags: java
categories: java
---

1. settings.xml文件用途
  * 设置maven参数
  * 包含本地仓库路径，远程仓库路径，认证信息等
2. settings.xml文件位置
  * user.home/.m2/settings.xml
3. 元素
  * 顶级元素
    * LocalRepository: 本地仓库路径
      `<localRepository>${user.home}/.m2/repository</localRepository>`
    * InteractiveMode: maven是否需要和用户交互获得输入
      `<interactiveMode>true</interactiveMode>`
    * UsePluginRegistry:
    * Offline: maven是否需要在离线模式下运行
      `<offline>true</offline>`
    * PluginGroups
    * Servers: 仓库账号信息
      ```
        <!--配置服务端的一些设置。一些设置如安全证书不应该和pom.xml一起分发。这种类型的信息应该存在于构建服务器上的settings.xml文件中。 -->
        <servers>
          <!--服务器元素包含配置服务器时需要的信息 -->
          <server>
            <!--这是server的id（注意不是用户登陆的id），该id与distributionManagement中repository元素的id相匹配。 -->
            <id>server001</id>
            <!--鉴权用户名。鉴权用户名和鉴权密码表示服务器认证所需要的登录名和密码。 -->
            <username>my_login</username>
            <!--鉴权密码 。鉴权用户名和鉴权密码表示服务器认证所需要的登录名和密码。密码加密功能已被添加到2.1.0 +。详情请访问密码加密页面 -->
            <password>my_password</password>
            <!--鉴权时使用的私钥位置。和前两个元素类似，私钥位置和私钥密码指定了一个私钥的路径（默认是${user.home}/.ssh/id_dsa）以及如果需要的话，一个密语。将来passphrase和password元素可能会被提取到外部，但目前它们必须在settings.xml文件以纯文本的形式声明。 -->
            <privateKey>${usr.home}/.ssh/id_dsa</privateKey>
            <!--鉴权时使用的私钥密码。 -->
            <passphrase>some_passphrase</passphrase>
            <!--文件被创建时的权限。如果在部署的时候会创建一个仓库文件或者目录，这时候就可以使用权限（permission）。这两个元素合法的值是一个三位数字，其对应了unix文件系统的权限，如664，或者775。 -->
            <filePermissions>664</filePermissions>
            <!--目录被创建时的权限。 -->
            <directoryPermissions>775</directoryPermissions>
          </server>
        </servers>
      ```
    * Mirrors: 仓库列表的下载镜像列表
      ```
      <mirrors>
        <!-- 给定仓库的下载镜像。 -->
        <mirror>
          <!-- 该镜像的唯一标识符。id用来区分不同的mirror元素。 -->
          <id>planetmirror.com</id>
          <!-- 镜像名称 -->
          <name>PlanetMirror Australia</name>
          <!-- 该镜像的URL。构建系统会优先考虑使用该URL，而非使用默认的服务器URL。 -->
          <url>http://downloads.planetmirror.com/pub/maven2</url>
          <!-- 被镜像的服务器的id。例如，如果我们要设置了一个Maven中央仓库（http://repo.maven.apache.org/maven2/）的镜像，就需要将该元素设置成central。这必须和中央仓库的id central完全一致。 -->
          <mirrorOf>central</mirrorOf>
        </mirror>
      </mirrors>
      ```
    * Proxies: 代理
      ```
      <proxies>
        <!--代理元素包含配置代理时需要的信息 -->
        <proxy>
          <!--代理的唯一定义符，用来区分不同的代理元素。 -->
          <id>myproxy</id>
          <!--该代理是否是激活的那个。true则激活代理。当我们声明了一组代理，而某个时候只需要激活一个代理的时候，该元素就可以派上用处。 -->
          <active>true</active>
          <!--代理的协议。 协议://主机名:端口，分隔成离散的元素以方便配置。 -->
          <protocol>http</protocol>
          <!--代理的主机名。协议://主机名:端口，分隔成离散的元素以方便配置。 -->
          <host>proxy.somewhere.com</host>
          <!--代理的端口。协议://主机名:端口，分隔成离散的元素以方便配置。 -->
          <port>8080</port>
          <!--代理的用户名，用户名和密码表示代理服务器认证的登录名和密码。 -->
          <username>proxyuser</username>
          <!--代理的密码，用户名和密码表示代理服务器认证的登录名和密码。 -->
          <password>somepassword</password>
          <!--不该被代理的主机名列表。该列表的分隔符由代理服务器指定；例子中使用了竖线分隔符，使用逗号分隔也很常见。 -->
          <nonProxyHosts>*.google.com|ibiblio.org</nonProxyHosts>
        </proxy>
      </proxies>
      ```
    * Profiles: 根据环境参数调整构建配置的列表
      ```
      <profiles>
        <profile>
          <!-- profile的唯一标识 -->
          <id>test</id>
          <!-- 自动触发profile的条件逻辑 -->
          <activation />
          <!-- 扩展属性列表 -->
          <properties />
          <!-- 远程仓库列表 -->
          <repositories />
          <!-- 插件仓库列表 -->
          <pluginRepositories />
        </profile>
      </profiles>
      ```
        * activation: 自动触发profile的条件逻辑
        * properties: 扩展属性列表，可以用${X}来使用
        * repositories: 远程仓库列表
          ```
          <repositories>
            <!--包含需要连接到远程仓库的信息 -->
            <repository>
              <!--远程仓库唯一标识 -->
              <id>codehausSnapshots</id>
              <!--远程仓库名称 -->
              <name>Codehaus Snapshots</name>
              <!--如何处理远程仓库里发布版本的下载 -->
              <releases>
                <!--true或者false表示该仓库是否为下载某种类型构件（发布版，快照版）开启。 -->
                <enabled>false</enabled>
                <!--该元素指定更新发生的频率。Maven会比较本地POM和远程POM的时间戳。这里的选项是：always（一直），daily（默认，每日），interval：X（这里X是以分钟为单位的时间间隔），或者never（从不）。 -->
                <updatePolicy>always</updatePolicy>
                <!--当Maven验证构件校验文件失败时该怎么做-ignore（忽略），fail（失败），或者warn（警告）。 -->
                <checksumPolicy>warn</checksumPolicy>
              </releases>
              <!--如何处理远程仓库里快照版本的下载。有了releases和snapshots这两组配置，POM就可以在每个单独的仓库中，为每种类型的构件采取不同的策略。例如，可能有人会决定只为开发目的开启对快照版本下载的支持。参见repositories/repository/releases元素 -->
              <snapshots>
                <enabled />
                <updatePolicy />
                <checksumPolicy />
              </snapshots>
              <!--远程仓库URL，按protocol://hostname/path形式 -->
              <url>http://snapshots.maven.codehaus.org/maven2</url>
              <!--用于定位和排序构件的仓库布局类型-可以是default（默认）或者legacy（遗留）。Maven 2为其仓库提供了一个默认的布局；然而，Maven 1.x有一种不同的布局。我们可以使用该元素指定布局是default（默认）还是legacy（遗留）。 -->
              <layout>default</layout>
            </repository>
          </repositories>
          ```
        * pluginRepositories: 发现插件的远程仓库列表
    * activeProfiles: 手动激活profiles的列表
      ```<activeProfiles>
          <!-- 要激活的profile id -->
          <activeProfile>env-test</activeProfile>
        </activeProfiles>
  ```
