# maven-release-plugin插件使用

Release插件是Apache Maven团队提供的官方插件，能够为项目代码库打tag，并将项目代码库中的代码发布到SCM的新版本。

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
      <!--执行的命令  默认clean verify-->
      <preparationGoals>clean install</preparationGoals>
      <!--允许快照版本 默认false-->
      <allowTimestampedSnapshots>true</allowTimestampedSnapshots>
    </configuration>
</plugin>
```
###configuration可配置的参数

* allowReleasePluginSnapshot:默认值false，仅用于插件本身的开发测试
* username:访问SCM的用户名
* password:访问SCM的密码
* tagNameFormat:默认值@{project.artifactId}-@{project.version}
* autoVersionSubmodules:默认值false，提示为各个子模块输入版本；如果值true，表示所有子模块采用与父模块相同的版本
* preparationGoals:默认值clean verify，在转换到发布版本之后，但是在第一次commit之前执行
* completionGoals:没有默认值，在转换到下一个开发版本之后，但是在第二次commit之前执行
* updateDependencies:默认值true，更新依赖版本为下一个开发版本
* updateWorkingCopyVersions:默认值true，更新工作目录的版本为developmentVersion值
* developmentVersion:本地工作目录的下一个开发版本
* scmCommentPrefix:默认值[maven-release-plugin] ，所有SCM修改的消息前缀
* tagBase:SCM库中tag的基础目录
* tag或releaseLabel:使用的SCM tag
* releaseVersion:要发布的版本
* remoteTagging:默认值true，目前只对SCM, SVN有效。
* suppressCommitBeforeTag: 默认值false，tag创建完成之前拒绝提交修改，remoteTagging=false才有效
* waitBeforeTagging: 默认值0，等待创建tag的时间，单位秒

其他请参考官方网站

## 执行命令
```sh
mvn release:clean release:prepare release:perform
```
在Jenkins等自动化构建中，需要无人值守，可以增加**-B**参数，使用如下命令
```sh
mvn -B release:clean release:prepare release:perform
```

* 跳过单元测试: -Darguments="-DskipTests"
* 忽略javadoc编译错误:-Darguments="-Dmaven.javadoc.failOnError=false"

处理参数还可以在**pom.xml**文件中使用属性声明避免错误
```xml
<properties>
    <!-- 跳过javadoc生成 -->
    <maven.javadoc.skip>true</maven.javadoc.skip>
    <!-- 忽略文档生成错误 -->
    <maven.javadoc.failOnError>false</maven.javadoc.failOnError>
    <!-- 跳过单元测试 -->
    <maven.test.skip>true</maven.test.skip>
    <!-- 忽略单元测试错误 -->
    <maven.test.failure.ignore>true</maven.test.failure.ignore>
</properties>
```

## 命令做了哪些操作

### release:clean
删除release.properties等发布过程文件

### release:prepare
* 生成release.properties文件
* 检查本地项目代码中是否有未提交的修改
* 检查项目的POM依赖或插件是否有SNAPSHOT版本（allowTimestampedSnapshots）
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


## 注意

* 开发版本必须是**xx.xx.x-SNAPSHOT**