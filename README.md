# Maven系列技术文档

## 介绍
 

[【Maven系列】POM官网权威详解](https://zhuanlan.zhihu.com/p/693305560)

[【Maven系列】环境设置settings.xml官网详解](https://zhuanlan.zhihu.com/p/693332013)

[【Maven系列】Maven常见问题 FAQ学习总结](https://zhuanlan.zhihu.com/p/694058242)

[【Maven系列】如何解决Maven中工件的版本冲突 omitted for conflict](https://zhuanlan.zhihu.com/p/709192429)




## [【Maven系列】环境设置settings.xml官网详解](https://zhuanlan.zhihu.com/p/693332013/)

*源自专栏《*[*Gradle ScalaTest markdown idea Git中文实用教程目录*](https://zhuanlan.zhihu.com/p/679330466)*》*

## 快速概述

settings.xml 文件中的 settings 元素包含用于定义值的元素，这些值在各种方式下配置 Maven 的执行，类似于 pom.xml，但不应捆绑到任何特定项目或分发给用户。这些值包括本地仓库位置、备用远程仓库服务器和认证信息。

**settings.xml** **文件**可能存在于两个位置：

1. Maven安装目录：${maven.home}/conf/settings.xml
2.  用户安装目录：${user.home}/.m2/settings.xml 
3.  前者的 settings.xml 也称为全局设置，后者的 settings.xml 称为用户设置。 
4. 如果两个文件都存在，它们的**内容将合并**，用户特定的 settings.xml 文件将具有更高的优先级。

> **提示：**如果需要从头开始创建用户特定的设置，最简单的方法是将 Maven 安装目录中的全局设置复制到 ${user.home}/.m2 目录。Maven 的默认 settings.xml 是一个带有注释和示例的模板，因此您可以快速调整以满足您的需求。

以下是 settings 元素下的顶级元素的概述：

```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository/>
  <interactiveMode/>
  <offline/>
  <pluginGroups/>
  <servers/>
  <mirrors/>
  <proxies/>
  <profiles/>
  <activeProfiles/>
</settings>
```

settings.xml 文件的内容可以使用以下表达式进行插值： - ${user.home} 和所有其他系统属性（自Maven 3.0起） - ${env.HOME} 等环境变量

请注意，在 settings.xml 中定义的配置文件中的属性不能用于插值。

## 设置详细信息

## 简单值

顶级 settings 元素的一半是简单值，表示描述构建系统中始终活动的元素的各种值。

```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository>${user.home}/.m2/repository</localRepository>
  <interactiveMode>true</interactiveMode>
  <offline>false</offline>
  ...
</settings>
```

- localRepository：本构建系统的**本地仓库路径**。 
-  (1) **默认值**为 ${user.home}/.m2/repository。 
-  (2) 特别适用于**允许所有登录用户从共享本地仓库**构建的主要构建服务器。 

- interactiveMode：如果 Maven 应尝试与用户交互以获取输入，则为 true，否则为 false。默认为 true。 
- offline：如果此构建系统应**以离线模式运行**，则为 true，默认为 false。对于无法连接到远程仓库的构建服务器非常有用，可能是因为网络设置或安全原因。 

## 插件组（pluginGroup）

该元素包含一系列 pluginGroup 元素，每个元素包含一个 groupId。当使用插件时，且在命令行中未提供 groupId 时，将搜索该列表。此列表自动包含 org.apache.maven.plugins 和 org.codehaus.mojo。

例如：

```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
  ...
  <pluginGroups>
    <pluginGroup>org.eclipse.jetty</pluginGroup>
  </pluginGroups>
  ...
</settings>
```

基于上述设置，Maven命令行可以使用截断命令执行 org.eclipse.jetty:jetty-maven-plugin:run：

```
mvn jetty:run
```

## 服务器（servers）

下载和部署的仓库由 POM 的 repositories 和 distributionManagement 元素定义。但是，某些设置，如用户名和密码，不应与 pom.xml 一起分发。这些信息应存在于构建服务器上的 settings.xml 文件中。

例如：

```
<servers>
  <server>
    <id>server001</id>
    <username>my_login</username>
    <password>my_password</password>
    <privateKey>${user.home}/.ssh/id_dsa</privateKey>
    <passphrase>some_passphrase</passphrase>
    <filePermissions>664</filePermissions>
    <directoryPermissions>775</directoryPermissions>
    <configuration></configuration>
  </server>
</servers>
```

- id：这是**服务器的 ID**（而不是要登录的用户的 ID），匹配 Maven 尝试连接的仓库/镜像的 id 元素。
- username、password：这些元素成对出现，表示要对此服务器进行身份验证所需的登录名和密码。
- privateKey、passphrase：与前两个元素类似，此对指定私钥路径（默认为 ${user.home}/.ssh/id_dsa）和密码（如果需要）。将来，密码和密码元素可能会外部化，但目前必须在 settings.xml 文件中以明文形式设置。
- filePermissions、directoryPermissions：在部署时创建仓库文件或目录时要使用的权限。每个的合法值是与 *nix 文件权限对应的三位数，例如 664 或 775。

**注意**：如果使用私钥登录到服务器，请确保省略 <password> 元素。否则，密钥将被忽略。

### 密码加密

2.1.0+ 版本中添加了一个新功能 - 服务器密码和密码短语加密。请参阅此页面获取详细信息。

## 镜像（Mirrors）

通过仓库，您可以指定要从哪些位置下载特定的构件，例如依赖项和 Maven 插件。仓库可以在项目内声明，这意味着如果您有自己的自定义仓库，与您的项目共享的人可以轻松地获得正确的设置。但是，您可能希望为特定仓库使用替代镜像，而无需更改项目文件。

**一些使用镜像的原因**包括：

- 在互联网上有一个**同步的镜像**，地理**位置更近**且速度更快
- 您希望用自己**拥有更大控制权的内部仓库**替换**特定仓库**
- 您希望运行仓库管理器**以为镜像提供本地缓存**，并需要**使用其 URL**

**示例：**

```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
  ...
  <mirrors>
    <mirror>
      <id>planetmirror.com</id>
      <name>PlanetMirror Australia</name>
      <url>http://downloads.planetmirror.com/pub/maven2</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
  </mirrors>
  ...
</settings>
```

- id、name：此**镜像的唯一标识符**和**用户友好的名称**。id 用于区分镜像元素，并在连接到镜像时从 <servers> 部分中选择相应的凭据。
- url：此镜像的基本 URL。构建系统将使用此 URL 连接到仓库，而不是原始的仓库 URL。
- mirrorOf：此镜像所**镜像的仓库的ID**。例如，要指向 Maven 中央仓库的镜像（https://repo.maven.apache.org/maven2/），请将此元素设置为 central。**更高级的映射如** **repo1****、****repo2** **或** *******、****!inhouse** **也是可能的**。这个值不能与镜像的ID匹配。

### 使用镜像设置仓库

要配置给定仓库的镜像，您在设置文件（${user.home}/.m2/settings.xml）中提供它，为新仓库提供其自己的 ID 和 URL，并指定 mirrorOf 设置，该设置是您使用镜像的仓库的 ID。例如，默认包含的主要 Maven 中央仓库的 ID 是 central，因此要使用不同的镜像实例，您可以配置如下：

```
<settings>
  ...
  <mirrors>
    <mirror>
      <id>other-mirror</id>
      <name>Other Mirror Repository</name>
      <url>https://other-mirror.repo.other-company.com/maven2</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
  </mirrors>
  ...
</settings>
```

请注意，对于给定仓库最多可以有一个镜像。换句话说，您不能将单个仓库映射到一组定义了相同 <mirrorOf> 值的镜像中。Maven 不会聚合这些镜像，而只会选择第一个匹配项。如果要提供多个仓库的合并视图，请改用仓库管理器。

### 使用单个仓库

您可以通过使其镜像所有仓库请求来强制 Maven 使用单个仓库。该仓库必须包含所有所需的构件，或者能够将请求代理到其他仓库。当使用内部公司仓库和 Maven 仓库管理器代理外部请求时，此设置最有用。

要实现这一点，请将 mirrorOf 设置为 *。

注意：此功能仅适用于 Maven 2.0.5+。

```
<settings>
  ...
  <mirrors>
    <mirror>
      <id>internal-repository</id>
      <name>Maven Repository Manager running on repo.mycompany.com</name>
      <url>http://repo.mycompany.com/proxy</url>
      <mirrorOf>*</mirrorOf>
    </mirror>
  </mirrors>
  ...
</settings>
```

### 高级镜像规范

单个镜像**可以处理多个仓库**。这通常与仓库管理器一起使用，可以方便地集中配置其背后的仓库列表。

**语法如下：**

- \* 匹配所有仓库 ID。
- external:* 匹配除使用 localhost 或基于文件的仓库之外的所有仓库。这在您想要排除用于集成测试的重定向仓库时使用。
- 自 Maven 3.8.0 开始，external:http:* 匹配使用 HTTP 的所有仓库，但排除使用 localhost 的仓库。
- 可以使用逗号作为分隔符指定多个仓库
- 惊叹号可以与上述通配符之一结合使用，以排除一个仓库 ID

在逗号分隔的仓库标识符或通配符周围不要包含额外的空格。例如，将 <mirrorOf> 设置为 !repo1, * 将不会镜像任何内容，而 !repo1,* 将镜像除了 repo1 之外的所有内容。

通配符在仓库标识符的逗号分隔列表中的位置不重要，因为通配符会延迟到进一步处理，明确的包含或排除会停止处理，覆盖任何通配符匹配。

当使用高级语法并配置多个镜像时，声明顺序很重要。当 Maven 查找某个仓库的镜像时，它首先检查一个直接匹配该仓库标识符的镜像。如果找不到直接匹配，则 Maven 选择根据上述规则（如果有）第一个匹配的镜像声明。因此，您可以通过更改设置文件中定义的顺序来影响匹配顺序。

**示例：**

- \* = 所有内容
- external:* = 不在 localhost 和基于文件的仓库上的所有内容
- repo,repo1 = repo 或 repo1
- *,!repo1 = 除了 repo1 之外的所有内容

```
<settings>
  ...
  <mirrors>
    <mirror>
      <id>internal-repository</id>
      <name>Maven Repository Manager running on repo.mycompany.com</name>
      <url>http://repo.mycompany.com/proxy</url>
      <mirrorOf>external:*,!foo</mirrorOf>
    </mirror>
    <mirror>
      <id>foo-repository</id>
      <name>Foo</name>
      <url>http://repo.mycompany.com/foo</url>
      <mirrorOf>foo</mirrorOf>
    </mirror>
  </mirrors>
  ...
</settings>
```

**创建您自己的镜像** 中央仓库的大小稳步增长，为节省带宽并节省您的时间，**不允许镜像整个中央仓库**（这样做将被自动禁止）。相反，建议您设置仓库管理器作为代理。

## 代理（Proxies）

```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
  ...
  <proxies>
    <proxy>
      <id>myproxy</id>
      <active>true</active>
      <protocol>http</protocol>
      <host>proxy.somewhere.com</host>
      <port>8080</port>
      <username>proxyuser</username>
      <password>somepassword</password>
      <nonProxyHosts>*.google.com|ibiblio.org</nonProxyHosts>
    </proxy>
  </proxies>
  ...
</settings>
```

- id：此代理的唯一标识符，用于区分代理元素。
- active：如果此代理处于活动状态，则为 true。对于声明一组代理很有用，但一次只能激活一个。
- protocol、host、port：代理的协议://主机:端口，分开成独立元素。
- username、password：这些元素作为一对出现，表示要对此代理服务器进行身份验证所需的登录名和密码。
- nonProxyHosts：这是一个不应被代理的主机列表。列表的分隔符是代理服务器期望的类型；上面的示例是管道分隔符 - 逗号分隔也很常见。

## 配置文件（Profiles）

settings.xml 中的 profile 元素是 pom.xml 中 profile 元素的截断版本。它包含 activation、repositories、pluginRepositories 和 properties 元素。profile 元素仅包含这四个元素，因为它们关注整个构建系统（这是 settings.xml 文件的作用），而不是个别项目对象模型的设置。

如果从设置中激活了配置文件，其值将覆盖任何与之对应的 POM 或 profiles.xml 文件中具有相同 ID 的配置文件。

### 激活（Activation）

activation 是配置文件的关键。与 POM 的配置文件一样，配置文件的强大之处在于其能够仅在特定情况下修改某些值；这些情况通过 activation 元素指定。

```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
  ...
  <profiles>
    <profile>
      <id>test</id>
      <activation>
        <activeByDefault>false</activeByDefault>
        <jdk>1.5</jdk>
        <os>
          <name>Windows XP</name>
          <family>Windows</family>
          <arch>x86</arch>
          <version>5.1.2600</version>
        </os>
        <property>
          <name>mavenVersion</name>
          <value>2.0.3</value>
        </property>
        <file>
          <exists>${basedir}/file2.properties</exists>
          <missing>${basedir}/file1.properties</missing>
        </file>
      </activation>
      ...
    </profile>
  </profiles>
  ...
</settings>
```

激活发生在满足所有指定条件时，尽管不是所有条件同时都是必需的。

- jdk：activation 元素中具有内置的、面向 Java 的检查。如果测试在与给定前缀匹配的 JDK 版本号下运行，则会激活。在上面的示例中，1.5.0_06 将匹配。还支持范围。有关支持范围的更多详细信息，请参阅 maven-enforcer-plugin。
- os：os 元素可以定义一些操作系统特定的属性。有关 OS 值的更多详细信息，请参阅 maven-enforcer-plugin。
- property：如果 Maven 检测到对应的名称=值对的属性（一个可以在 POM 中通过 ${name} 进行解引用的值），则配置文件将被激活。
- file：最后，通过文件的存在或缺失，给定文件名可以激活配置文件。

activation 元素不是激活配置文件的唯一方式。settings.xml 文件的 activeProfile 元素可以包含配置文件的 ID。它们也可以通过命令行显式激活，通过在 -P 标志后使用逗号分隔的列表（例如 -P test）。

要查看在某个构建中哪个配置文件将被激活，请使用 maven-help-plugin。

```
mvn help:active-profiles
```

### 属性（Properties）

Maven 属性是类似于 Ant 中的属性的值占位符。它们的值可通过 ${X} 这种符号在 POM 中的任何地方访问，其中 X 是属性。它们有五种不同的样式，都可以从 settings.xml 文件中访问：

- env.X：在变量前加上“env.”将返回 shell 的环境变量。例如，${env.PATH} 包含 $path 环境变量（Windows 中为 %PATH%）。
- project.x：POM 中以点号（.）表示的路径将包含相应元素的值。例如：<project><version>1.0</version></project> 可通过 ${project.version} 访问。
- settings.x：在 settings.xml 中以点号（.）表示的路径将包含相应元素的值。例如：<settings><offline>false</offline></settings> 可通过 ${settings.offline} 访问。
- Java 系统属性：所有通过 java.lang.System.getProperties() 访问的属性都可作为 POM 属性使用，例如 ${java.home}。
- x：在 <properties /> 元素或外部文件中设置，其值可以作为 ${someVar} 使用。

```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
  ...
  <profiles>
    <profile>
      ...
      <properties>
        <user.install>${user.home}/our-project</user.install>
      </properties>
      ...
    </profile>
  </profiles>
  ...
</settings>
```

如果此配置文件处于活动状态，属性 ${user.install} 可在 POM 中访问。

### 仓库（Repositories）

仓库是远程项目的集合，Maven用于填充构建系统的本地仓库。Maven从这个本地仓库中调用它的插件和依赖项。不同的远程仓库可能包含不同的项目，并且在活动配置文件下，它们可能会被搜索以查找匹配的发布或快照构件。

```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
  ...
  <profiles>
    <profile>
      ...
      <repositories>
        <repository>
          <id>codehausSnapshots</id>
          <name>Codehaus Snapshots</name>
          <releases>
            <enabled>false</enabled>
            <updatePolicy>always</updatePolicy>
            <checksumPolicy>warn</checksumPolicy>
          </releases>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>never</updatePolicy>
            <checksumPolicy>fail</checksumPolicy>
          </snapshots>
          <url>http://snapshots.maven.codehaus.org/maven2</url>
          <layout>default</layout>
        </repository>
      </repositories>
      <pluginRepositories>
        <pluginRepository>
          <id>myPluginRepo</id>
          <name>My Plugins repo</name>
          <releases>
            <enabled>true</enabled>
          </releases>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
          <url>https://maven-central-eu....com/maven2/</url>
        </pluginRepository>
      </pluginRepositories>
      ...
    </profile>
  </profiles>
  ...
</settings>
```

- releases、snapshots：这些是每种构件类型（发布或快照）的策略。通过这两组，POM 可以独立于同一仓库中的其他类型改变每种类型的策略。例如，可能决定仅启用快照下载，可能用于开发目的。
- enabled：对于该仓库**是否为相应类型**（发布或快照）启用的 true 或 false。
- updatePolicy：此元素指定**更新应尝试发生的频率**。Maven将比较本地 POM 的时间戳（存储在仓库的 maven-metadata 文件中）与远程时间戳。选项有：always、daily（默认）、interval:X（其中 X 是以分钟为单位的整数）或 never。
- checksumPolicy：当 Maven 将文件部署到仓库时，它还会部署相应的校验和文件。您可以选择在缺失或不正确的校验和上忽略、失败或警告。
- layout：在仓库的上述描述中，提到它们都遵循一个共同的布局。这在大多数情况下是正确的。Maven 2 有一个默认的仓库布局；然而，Maven 1.x 有一个不同的布局。使用此元素来指定它是默认布局还是旧版布局。

### 插件仓库（Plugin Repositories）

仓库是两种主要类型的构件的家。第一种是作为其他构件的依赖项使用的构件。这些是驻留在中央仓库中的大多数构件。另一种类型的构件是插件。Maven 插件本身是一种特殊类型的构件。因此，插件仓库可以与其他仓库分开（尽管，我还没有听到令人信服的理由来这样做）。无论如何，pluginRepositories 元素块的结构类似于 repositories 元素。每个 pluginRepository 元素指定 Maven 可以找到新插件的远程位置。

## 活动配置文件（Active Profiles）

```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
  ...
  <activeProfiles>
    <activeProfile>env-test</activeProfile>
  </activeProfiles>
</settings>
```

activeProfiles 元素是 settings.xml 中的最后一部分。它包含一组 activeProfile 元素，每个元素具有配置文件的 ID 值。任何定义为 activeProfile 的配置文件 ID 将是活动的，无论环境设置如何。如果找不到匹配的配置文件，则不会发生任何事情。例如，如果 env-test 是一个 activeProfile，则 pom.xml 中的配置文件（或具有相应 ID 的 profile.xml）将被激活。如果找不到这样的配置文件，则执行将继续进行。

## 设置多个仓库

### 两种方法

您可以指定使用多个仓库的**两种不同方法**。

- 第一种方法是**在 POM 中指定您想要使用的仓库**。这在构建配置文件内外都受支持：

```
<project>
...
  <repositories>
    <repository>
      <id>my-repo1</id>
      <name>your custom repo</name>
      <url>http://jarsm2.dyndns.dk</url>
    </repository>
    <repository>
      <id>my-repo2</id>
      <name>your custom repo</name>
      <url>http://jarsm2.dyndns.dk</url>
    </repository>
  </repositories>
...
</project>
```

> **注意：**您还将获得在 Super POM 中定义的标准仓库集。

- 另一种指定多个仓库的方法是通过在 ${user.home}/.m2/settings.xml 或 ${maven.home}/conf/settings.xml 文件中创建如下所示的配置文件：

```
<settings>
 ...
 <profiles>
   ...
   <profile>
     <id>myprofile</id>
     <repositories>
       <repository>
         <id>my-repo2</id>
         <name>your custom repo</name>
         <url>http://jarsm2.dyndns.dk</url>
       </repository>
     </repositories>
   </profile>
   ...
 </profiles>

 <activeProfiles>
   <activeProfile>myprofile</activeProfile>
 </activeProfiles>
 ...
</settings>
```

如果在配置文件中指定了仓库，则**必须记住激活**该特定配置文件！如上所示，我们通过在 activeProfiles 元素中注册要激活的配置文件来实现这一点。

您还可以通过执行以下命令在**命令行上激活此配置文件**：

```
mvn -Pmyprofile ...
```

事实上，-P 选项将接受要激活的配置文件的 CSV 列表，如果您希望同时激活多个配置文件。

> **注意：**设置描述符文档可在 Maven 本地设置模型网站上找到。

### 仓库顺序

远程仓库 URL 查询顺序如下，直到返回有效结果为止：

**1. 有效设置：**

-  全局 settings.xml 
-  用户 settings.xml 

**2. 本地有效构建 POM：**

- 本地 pom.xml
- 递归的父 POM
- Super POM

**3. 依赖路径到构件的有效 POM**

对于这些位置，首先按照构建配置文件中概述的顺序查询配置文件中的仓库。

在从仓库下载之前，将应用镜像配置。

可以通过 mvn help:effective-settings 和 mvn help:effective-pom -Dverbose 命令轻松查看**考虑了配置文件的有效设置**和**本地构建 POM**，以**查看它们的仓库顺序**。

### 仓库 ID

**每个仓库必须有唯一的ID。**在有效设置或有效 POM 中出现**冲突的仓库 ID** 会导致构建失败。

但是，**来自 POM 的仓库会被有效设置中具有相同 ID 的仓库覆盖**。仓库 ID 也用于本地仓库元数据。

## [参考链接](https://maven.apache.org/settings.html)