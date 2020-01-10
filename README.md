# 无人值守maven-release-plugin插件使用

## 注意
1、开发版本必须是**xx.xx.x-SNAPSHOT**
2、

## 配置SCM信息
```xml
<!--版本库信息-->
<scm>
	<developerConnection>scm:git:git@github.com:hsun924/release-demo.git</developerConnection>
</scm>
```

## 配置仓库信息
可以实现自动发布到maven仓库

```xml
<!--maven仓库信息-->
<distributionManagement>
<repository>
  <id>xdja-deploy</id>
  <name>Releases</name>
  <url>http://120.194.4.152:8081/nexus/content/repositories/releases/</url>
</repository>
<snapshotRepository>
  <id>xdja-deploy</id>
  <name>snapshots</name>
  <url>http://120.194.4.152:8081/nexus/content/repositories/snapshots/</url>
</snapshotRepository>
</distributionManagement>
```

## 配置插件信息

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-release-plugin</artifactId>
	<version>2.5.3</version>
	<configuration>
	  <tagNameFormat>v@{project.version}</tagNameFormat>
	  <autoVersionSubmodules>true</autoVersionSubmodules>
	</configuration>
</plugin>
```

## 执行命令
```sh
mvn -B release:clean release:prepare release:perform
```
命令做了哪些操作

### release:clean
删除release.properties等发布过程文件

### release:prepare
* 生成release.properties文件
* 检查本地项目代码中是否有未提交的修改
* 检查项目的POM依赖或插件是否有SNAPSHOT版本
* 更新POM，将项目的version从1.0.0-SNAPSHOT改为发布版本1.0.0
* 确定项目tag v1.0.0
* 更新POM，转换POM中的SCM信息，包括更新tag
* 执行mvn clean verify
* 验证成功，第一次提交POM修改，并push到Git库的refs/heads/master
* 为当前项目打tag，并push到Git库的refs/tags/my_tag_name
* 为项目的version产生一个新的开发版本1.0.1-SNAPSHOT
* 更新POM为开发版本，并将更新后的POM再次提交到Git库

>发布失败后可以采用**release:rollback**回滚（基于releaseBackup文件）

### release:perform
* 依赖于release:prepare阶段生成的release.properties文件
* 基于SCM的URL检出prepare阶段刚刚打标签的代码
* 执行deploy
* 执行release:clean，删除release.properties等发布过程文件