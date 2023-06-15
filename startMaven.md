1、从官网下载maven，记住maven版本要和idea版本对应，idea2021.1.3不支持maven3.8.5，所以下载maven3.6.3
2、修改maven的setting.xml
* 修改maven仓库路径
```xml
<!--在setting下加入这一行-->
<localRepository>E:/MyMavenLibrary/apache-maven-3.6.3-bin/repositories</localRepository>
```
* 修改镜像地址为阿里云
```xml
<!--在mirrors下加入以下代码-->
<mirror>
  <id>aliyunmaven</id>
  <mirrorOf>*</mirrorOf>
  <name>aliyunmaven</name>
  <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```
* 修改默认jdk
```xml
<!--在profiles下加入以下代码-->
<profile>
  <id>jdk1.8</id>
  <activation>
    <activeByDefault>true</activeByDefault>
    <jdk>1.8</jdk>
  </activation>
  <properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
  </properties>
</profile>
```
3、配置环境变量
* 新增环境变量：新建一个名叫MAVEN_HOME的环境变量。地址是bin所在目录。
* 修改Path变量：在Path中添加一条：%MAVEN_HOME%\bin
  
4、修改idea默认maven路径
* 退出所有工程
* 依次点击：Customize->Allsettings...->Build,Execution,Deployment->Build Tools->Maven
* 修改Maven home path为MAVEN_HOME所在地址。
* 依次修改User settings file与Local repository

如果不希望永久修改maven，则在打开某个项目的时候，再配置上面第4步即可