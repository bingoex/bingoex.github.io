---
layout: post
title: Maven简介
categories: Java 工具
description: 
keywords: 
---


maven是一个项目管理和构建自动化工具，基于项目对象模型（POM），可以通过一小段描述信息来管理项目的构建，报告和文档的软件项目管理工具。
 
# 安装


<http://jingyan.baidu.com/article/3d69c55157068af0ce02d76f.html>

intellij 自带集成了maven工具，可配置IDE使用自己下载的maven版本（注意，maven2和3的区别还是很大的，某些老项目必须使用maven2构建）

# 生命周期


## clean生命周期

真正构建之前进行一些清理工作

## default生命周期

构建项目
- validate：验证项目是否正确及构建项目所需的信息是否正确；
- **compile**：编译项目源代码；
- test-compile：编译测试源代码；
- test：使用合适的单元测试框架运行测试；
- **package**：接受编译好的代码，打包成可发布的格式；
- **install**：将包安装至本地仓库，以让其它项目依赖；
- **deploy**：将最终的包复制到远程的仓库，以让其它开发人员与项目共享。

## site生命周期

生成项目报告，站点，发布站点


# 配置文件及其参数简介


## settings.xml

可配置本地仓库位置，远程仓库服务器以及认证信息等。

存在于两个地方。1又被叫做全局配置，2为用户配置。如果两者都存在，它们的内容将被合并，并且用户范围的settings.xml优先。
1. 安装目录/conf/settings.xml
2. ~/.m2/settings.xml
- **localRepository**：本地仓库的路径，默认值是${user.home} /.m2/ repository，如果想要把从远程仓库下载的jar包放在其他目录，只需要在这里修改成自己的目录即可；
- interactiveMode：如果Maven要试图与用户交互来得到输入就设置为true，否则就设置为false，默认为true。
- usePluginRegistry:如果需要让Maven使用文件${user.home}.m2/plugin-registry.xml来管理插件版本，则设为true。默认为false。
- offline：如果构建系统要在离线模式下工作，设置为true，默认为false。如果构建服务器因为网络故障或者安全问题不能与远程仓库相连，那么这个设置是非常有用的。
- pluginGroups:这个元素包含了一系列pluginGroup元素，每个又包含了一个groupId。当一个plugin被使用，而它的groupId没有被提供的时候，这个列表将被搜索。
- **servers**:如username和password;
- **mirrors**:配置构建系统使用的镜像服务器;
- proxies：配置代理服务器;
- profiles:settings.xml中的profile是pom.xml中的profile的简洁形式。它包含了激活(activation)，仓库(repositories)，插件仓库(pluginRepositories)和属性(properties)元素。profile元素仅包含这四个元素是因为他们涉及到整个的构建系统，而不是个别的POM配置。如果settings中的profile被激活，那么它的值将重载POM或者profiles.xml中的任何相等ID的profiles。
 
## pom.xml

pom.xml定义了项目的基本信息，用于描述项目如何构建，声明项目依赖

### 基本属性

- modelVersion:指定了当前POM模型的版本，对于maven2及maven3来说，它只能是4.0.0；**groupId,artifactId和version定义了一个项目的基本坐标**
- groupId:项目或者组织的唯一标志（项目组）;
- artifactId:项目名称;
- version:项目版本 

### 自定义属性

使用properties定义自己的属性，使用占位符“$”来进行引用。    

![](/images/posts/2017-07-01-maven-instruct.md/1.png)

![](/images/posts/2017-07-01-maven-instruct.md/2.png)

Maven提供了三个隐式的变量，可以用来访问环境变量，POM信息，和Maven Settings
- exv变量：可以用来引用操作系统环境变量，如${env.PATH}
- project变量：可以用来引用的是当前这个xml中project根元素下面的子元素的值，比如: <project><version>1.0-SNAPSHOT</version></project>，引用方式：${project.version}；
- settings变量：可以用来引用Maven本地配置文件xml文件中的属性，如：要引用settings下的本地仓库localRepository元素的值时，可以用${settings.localRepository}；
- java的系统属性:所有在java中使用java.lang.System.getProperties()能够获取到的属性都可以在pom.xml中引用，比如${java.home}; 

### pom关系


#### 1、依赖关系

![](/images/posts/2017-07-01-maven-instruct.md/3.png)

\<dependency\>下，可以添加自己项目对其他包的依赖，maven会自动将我们所依赖的包添加到项目中。

type:相应依赖产品包形式，如：jar,war;

scope:用于限制相应的依赖范围，包括以下几种变量：
- compile:默认范围，用于编译；
- provided:类似compile,依赖的包只有当JDK或者一个容器已提供该依赖之后才能使用；
- runtime:在运行和测试系统的时候需要，但在编译的时候不需要；
- test:在运行和测试系统的时候需要，但在编译的时候不需要;
- system: system范围依赖与provided类似，但是你必须显式的提供一个对于本地系统中jar文件的路径,如果要将一个依赖范围设置成系统范围，必须同时提供一个systemPath元素（该方式不推荐使用）;

optional:标注可选，一般用在，假如你的项目为A,你在依赖类库B时，将optional设为true，则别人在依赖你的项目A时，B不会被传递依赖进去，如果别人也希望依赖这个类库，则也需要添加对类库B的依赖.

**路径最短，申明顺序其次**

##### 如何隔离jar包

###### 第一个很常用的exclusion来隔离jar包。

![](/images/posts/2017-07-01-maven-instruct.md/4.png)

项目依赖project-a,但是project-a依赖project-b，但是该项目不想依赖project-b，所以利用exclusion来排除对project-b的依赖。

###### 第二个不常用的方法就是创建一个空包。

空包的坐标和你需要隔离的Jar包坐标一样。项目中这个空包申明在pom文件靠前的地方，这样依据maven依赖原则，这个空包会优先被使用，后面所有无论是直接依赖还是间接依赖的相同坐标的jar包都不会被使用了。空包比exclusion的好处就是不用在所有间接依赖的地方去exclusion


#### 2、继承关系

![](/images/posts/2017-07-01-maven-instruct.md/5.png)

如果不想一遍又一遍的重复同样的依赖元素，可以通过继承parent元素，可避免这种重复。当一个项目指定一个父项目的时候，Maven在读取当前项目的POM之前，会使用这个父POM作为起始点。它继承所有东西，包括groupId和version。
 

#### 3、聚合关系（modoules）


![](/images/posts/2017-07-01-maven-instruct.md/6.png)

当Maven读取turorial1的POM的时候，它会检查它的modules元素，看到top-group引用了项目my-project1和my-project2。之后Maven会在它们的每个子目录中寻找pom.xml。Maven为每一个子模块重复这个过程.


### dependencyManagement

![](/images/posts/2017-07-01-maven-instruct.md/7.png)

![](/images/posts/2017-07-01-maven-instruct.md/8.png)


能让所有在子项目中引用一个依赖而不用显式的列出版本号。Maven会沿着父子层次向上走，直到找到一个拥有dependencyManagement元素的项目，然后它就会使用在这个dependencyManagement 元素中指定的版本号。

在子pom里只需要添加对mysql-connector-java的依赖，而无需声明版本。这样做的好处就是：如果有多个子项目都引用同一样依赖，**则可以避免在每个使用的子项目里都声明一个版本号**，这样当想升级或切换到另一个版本时，只需要在顶层父容器里更新，而不需要一个一个子项目的修改。另外如果某个子项目需要另外的一个版本，只需要声明自己的version就可。dependencyManagement里只是声明依赖，并不实现引入，因此子项目需要显式的声明需要用的依赖.
 
### build

![](/images/posts/2017-07-01-maven-instruct.md/9.png)

主要用于设置构建项目所需要的信息（如plugins），可以包括：
- defaultGoal: 定义默认的目标或者阶段。如install等
- directory: 编译输出的目录;
- finalName: 生成最后的文件的样式;
- filter: 定义过滤，用于替换相应的属性文件，使用maven定义的属性。设置所有placehold的值;

### resources

- resources：这个元素描述了项目相关的所有资源路径列表，例如和项目相关的属性文件，这些资源被包含在最终的打包文件里;
- resource：这个元素描述了项目相关或测试相关的所有资源路径;
- targetPath： 描述了资源的目标路径。该路径相对target/classes目录（例如${project.build.outputDirectory}）。举个例子，如果你想资源在特定的包里(org.apache.maven.messages)，你就必须该元素设置为org/apache/maven /messages。然而，如果你只是想把资源放到源码目录结构里，就不需要该配置。
- filtering：是否使用参数值代替参数名。参数值取自properties元素或者文件里配置的属性，文件在filters元素里列出。   
- directory：描述存放资源的目录，该路径相对POM路径   
- includes:包含的模式列表，例如**/*.xml.
- excludes:排除的模式列表，例如**/*.xml 

### plugins插件

- extensions:true or false，是否装载插件扩展。默认false;
- inherited:true or false，是否此插件配置将会应用于那些继承于此的项目;
- configuration:指定插件配置;
- dependencies:插件需要依赖的包
- executions:用于配置execution目标，一个插件可以有多个目标。


# 常用命令

命令 | 说明
mvn archetype:generate | 创建 Maven项目
mvn compile | 编译源代码
mvn test-compile | 编译测试代码
mvn test | 运行应用程序中的单元测试
mvn eclipse:eclipse | 生成 Eclipse项目文件
mvn eclipse:clean | 清除 Eclipse项目文件
mvn package | 打包，依据项目生成 jar文件
mvn install | 打包，并在本地 Repository中安装 jar
mvn deploy | 打包，并上传到maven远程仓库
mvn clean | 清除目标目录中的生成结果
mvn site | 生成项目相关信息的网站
mvn dependency:tree -Dverbose | 查看依赖
mvn dependency:resolve -Dclassifier=sources | 略
mvn -U install | 重新安装


# 其他
- 仓库：<http://mvnrepository.com/artifact/org.apache.directory.studio/org.apache.commons.collections/3.2.1>
- maven打包war：<https://www.cnblogs.com/zhujiabin/p/5115749.html>
- <https://www.zhihu.com/question/31551133>






