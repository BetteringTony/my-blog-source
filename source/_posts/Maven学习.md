---
title: Maven学习
date: 2025-11-19 13:42:03
categories: [后端个人学习]
tags: [maven]
---

## 1、Maven 安装及配置

网上相关教程很多，此处暂时省略，主要关注点在于：设置环境变量用于识别`mvn`命令、本地仓库文件夹保存位置修改、添加中央仓库镜像地址加速下载、按需调整配置文件`settings.xml`内容。

## 2、坐标和依赖详解

### 2.1、约定配置

Maven 提倡使用一个共同的标准目录结构，Maven 使用约定优于配置的原则，所以需要尽可能地遵守这样的目录结构，在后续的打包、运行、部署的时候就会比较方便，如下所示：

| 目录                               | 目的                                                         |
| :--------------------------------- | :----------------------------------------------------------- |
| ${basedir}                         | 存放pom.xml和所有的子目录                                    |
| ${basedir}/src/main/java           | 项目的java源代码                                             |
| ${basedir}/src/main/resources      | 项目的资源，比如说property文件，springmvc.xml                |
| ${basedir}/src/test/java           | 项目的测试类，比如说Junit代码                                |
| ${basedir}/src/test/resources      | 测试用的资源                                                 |
| ${basedir}/src/main/webapp/WEB-INF | web应用文件目录，web项目的信息，比如存放web.xml、本地图片、jsp视图页面 |
| ${basedir}/target                  | 打包输出目录                                                 |
| ${basedir}/target/classes          | 编译输出目录                                                 |
| ${basedir}/target/test-classes     | 测试编译输出目录                                             |
| ~/.m2/repository                   | Maven默认的本地仓库目录位置，可以修改settings.xml来自定义    |

### 2.2、pom文件

当需要用到maven来解决jar包依赖问题以及解决项目中的编译、测试、打包、部署时，项目中必须要有`pom.xml`文件，这些都是依靠pom的配置来完成的。POM( Project Object Model，项目对象模型 ) 是 Maven 工程的基本工作单元，是一个XML文件，包含了项目的基本信息，用于描述项目如何构件，声明项目依赖，等等。执行任务或目标时，Maven 会在当前目录中查找 POM。它读取 POM，获取所需的配置信息，然后执行目标。

| 区块    | 关键标签                                     | 作用                      | 常见值/示例                       |
| ------- | -------------------------------------------- | ------------------------- | --------------------------------- |
| 头部    | `<modelVersion>`                             | 固定 4.0.0，XML 语法版本  | 4.0.0                             |
| 坐标    | `groupId` `artifactId` `version` `packaging` | 唯一定位本项目            | com.foo / demo / 1.0.0 / jar      |
| 父子    | `parent`                                     | 复用依赖、插件版本        | spring-boot-starter-parent        |
| 属性    | `properties`                                 | 一键改版本、编码、JDK     | `<java.version>17</java.version>` |
| 依赖    | `dependencies` / `dependencyManagement`      | 声明或锁定依赖            | 略                                |
| 构建    | `build`                                      | 插件、资源过滤、finalName | 略                                |
| 发布    | `distributionManagement`                     | 推送到哪个私有仓库        | Nexus/Artifactory 地址            |
| profile | `profiles`                                   | 不同环境差异化构建        | dev/test/prod                     |

### 2.3、Maven坐标

maven中引入了坐标的概念，每个构件都有唯一的坐标，当使用maven创建一个项目需要标注其坐标信息，而项目中用到其他的一些构件，也需要知道这些构件的坐标信息。maven中构件坐标是通过一些元素定义的，他们是groupId、artifactId、version、packaging、classifier。

- goupId：定义当前构件所属的组，通常与域名反向一一对应。必须填写
- artifactId：项目组中构件的编号。必须填写
- version：当前构件的版本号，每个构件可能会发布多个版本，通过版本号来区分不同版本的构件。必须填写
- packaging：定义该构件的打包方式，可以省略，默认为jar包
- classifier：区分同版本不同变种，可省略

### 2.4、Maven导入依赖的构件

maven可以帮忙引入需要依赖的构件(jar等)，需要知道其坐标信息，然后将这些信息放入`pom.xml`文件中的`dependencies`元素中，常见的标签如下：

```xml
<dependencies>
    <!-- 在这里添加你的依赖 -->
    <dependency>
        <groupId></groupId>
        <artifactId></artifactId>
        <version></version>
        <type></type>
        <scope></scope>
        <optional></optional>
        <exclusions>
            <exclusion></exclusion>
            <exclusion></exclusion>
        </exclusions>
    </dependency>
</dependencies>
```

- `dependencies`元素中可以包含多个`dependency`，每个`dependency`就表示当前项目需要依赖的一个构件的信息
- `dependency`中`groupId`、`artifactId`、`version`是定位一个构件必须要提供的信息，所以这几个是必须的
- `type`：表示所要依赖的构件的类型，对应于被依赖的构件的`packaging`。大部分情况下，该元素不被声明，默认值为jar，表示被依赖的构件是一个jar包。
- `scope`：依赖的范围
- `option`：标记依赖是否可选
- `exclusions`：用来排除传递性的依赖

通常情况下依赖的都是一些jar包，所以在大多数情况下只需要提供`groupId`、`artifactId`、`version`信息就可以了。

### 2.5、maven依赖范围（scope）

考虑以下场景：

- 在开发项目的过程中，可能需要用 junit 来写一些测试用例，此时需要引入 junit 的jar包，但是当项目部署在线上运行了，测试代码不会再执行了，此时junit.jar是不需要了，所以junit.jar只是在编译测试代码和运行测试用例的时候用到，而上线之后用不到了，所以部署环境中是不需要的
- 开发了一个web项目，在项目中用到了servlet相关的jar包，但是部署的时候，将其部署在tomcat中，而tomcat中自带了servlet的jar包，那么需求是开发、编译、单元测试的过程中需要这些jar，上线之后，servlet相关的jar由web容器提供，也就是说打包的时候，不需要将servlet相关的jar打入war包了
- 像jdbc的驱动，只有在运行的时候才需要，编译的时候是不需要的

我们知道，java中编译代码、运行代码都需要用到classpath变量，classpath简单理解就是Java 虚拟机在执行时要去哪里找 .class 文件和 jar 包的一张清单。maven用到classpath的地方有：编译源码、编译测试代码、运行测试代码、运行项目这几个步骤。针对上面的场景，编译、测试、运行需要的classpath对应的值可能是不一样的，这时maven中的scope就提供了支持，scope是用来控制被依赖的构件与classpath的关系（编译、测试、运行打包所用到的classpath），决定 jar 包要出现在哪些classpath里。`scope`有以下几种值：

- compile：编译依赖范围，如果没有指定，默认使用该依赖范围，该范围下的maven依赖对于编译源码、编译测试代码、测试、运行打包4种classpath都有效。
- test：测试依赖范围，使用此依赖范围的maven依赖，只对编译测试、运行测试的classpath有效，在编译主代码、运行项目时无法使用此类依赖。
- provided：已提供依赖范围。表示项目的运行环境中已经提供了所需要的构件，对于此依赖范围的maven依赖，对于编译源码、编译测试、运行测试中classpath有效，但在运行打包时无效。比如servlet-api，这个在编译和测试的时候需要用到，但是在运行的时候，web容器已经提供了，就不需要maven帮忙引入了。
- runtime：运行时依赖范围，使用此依赖范围的maven依赖，对于编译测试、运行测试和运行项目的classpath有效，但在编译主代码时无效，比如jdbc驱动实现，运行的时候才需要具体的jdbc驱动实现。
- system：已废弃，尽量不使用
- import：暂时略过

常用依赖范围表格总结如下：

| 依赖范围 | 编译源码 | 编译测试代码 | 运行测试 | 运行项目 | 示例        |
| -------- | -------- | ------------ | -------- | -------- | ----------- |
| compile  | Y        | Y            | Y        | Y        | spring-web  |
| test     | -        | Y            | Y        | -        | junit       |
| provided | Y        | Y            | Y        | -        | servlet-api |
| runtime  | -        | Y            | Y        | Y        | jdbc驱动    |

即：scope如果对于运行范围有效，意思是指依赖的jar包会被打包到项目的运行包中，最后运行的时候会被添加到classpath中运行。如果scope对于运行项目无效，那么项目打包的时候，这些依赖不会被打包到运行包中。

### 2.6、依赖的传递

当在项目中引入A依赖，而A依赖自身又依赖于B、C两个依赖，那么maven会自动将所有相关依赖都引入项目，这种叫做依赖的传递。此时`scope`标签的值会对这种传递依赖会有影响。

假设构件A依赖于B，B依赖于C，我们说A对于B是第一直接依赖，B对于C是第二直接依赖，而A对于C是传递性依赖，而第一直接依赖的scope和第二直接依赖的scope决定了传递依赖的范围，即决定了A对于C的scope的值。

下面用表格来列一下这种依赖的效果，表格最左边一列表示第一直接依赖（即A->B的scope的值）,而表格中的第一行表示第二直接依赖（即B->C的scope的值），行列交叉的值显示的是A对于C最后产生的依赖效果。

|          | compile  | test | provided | runtime  |
| -------- | -------- | ---- | -------- | -------- |
| compile  | compile  | -    | provided | runtime  |
| test     | test     | -    | -        | test     |
| provided | provided | -    | -        | provided |
| runtime  | runtime  | -    | -        | runtime  |

举例解释如下：

1. 比如A->B的scope是`compile`，而B->C的scope是`test`，那么按照上面表格中，对应第2行第3列的值`-`，那么A对于C是没有依赖的，A对C的依赖没有从B->C传递过来，所以A中是无法使用C的
2. 比如A->B的scope是`compile`，而B->C的scope是`runtime`，那么按照上面表格中，对应第2行第5列的值为`runtime`，那么A对于C是的依赖范围是`runtime`，表示A只有在运行的时候C才会被添加到A的classpath中，即对A进行运行打包的时候，C会被打包到A的包中

（说明：这里A、B、C表示三个独立Maven构件artifact。A项目引入了已发布的B构件这一依赖，那么A->B的scope指的是A项目中的pom文件里对于B依赖的引入的scope标签内容；B->C同样方式理解）

### 2.7、maven依赖调解功能

现实中可能存在这样的情况，A->B->C->Y(1.0)，A->D->Y(2.0)，此时Y出现了2个版本，1.0和2.0，此时maven会选择Y的哪个版本？

解决这种问题，maven有2个原则：

- 路径最近原则：上面`A->B->C->Y(1.0)，A->D->Y(2.0)`，Y的2.0版本距离A更近一些，所以maven会选择2.0。
- 最先声明原则：如果出现了路径距离是一样的，如：`A->B->Y(1.0)，A->D->Y(2.0)`，此时会看A的pom.xml中所依赖的B、D在`dependencies`标签体中的位置，谁的声明在最前面，就以谁的为主，比如`A->B`在前面，那么最后Y会选择1.0版本。

### 2.8、可选依赖

有这么一种情况：A->B中scope是compile，B->C中scope是compile。按照上面介绍的依赖传递性，C会传递给A，被A依赖。假如B不想让C被A自动依赖，那么可以这样设置：dependency标签元素下面有个optional，是一个boolean值，表示是一个可选依赖，B->C时将这个值置为true，这样C不会被A自动引入。

### 2.9、排除依赖

A项目的pom.xml中

```xml
<dependency>
    <groupId>com.javacode</groupId>
    <artifactId>B</artifactId>
    <version>1.0</version>
</dependency>
```

B项目1.0版本的pom.xml中

```xml
<dependency>
    <groupId>com.javacode</groupId>
    <artifactId>C</artifactId>
    <version>1.0</version>
</dependency>
```

上面A->B的1.0版本，B->C的1.0版本，而scope都是默认的compile，根据前面讲的依赖传递性，C会传递给A，会被A自动依赖，但是C此时有个更新的版本2.0，A想使用2.0的版本，那么此时A的pom.xml中可以这么写：

```xml
<dependency>
    <groupId>com.javacode</groupId>
    <artifactId>B</artifactId>
    <version>1.0</version>
    <exclusions>
        <exclusion>
            <groupId>com.javacode</groupId>
            <artifactId>C</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

上面使用`exclusions`标签元素排除了B->C依赖的传递，也就是B->C不会被传递到A中。

`exclusions`中可以有多个`exclusion`元素，可以排除一个或者多个依赖的传递，声明`exclusion`时只需要写上`groupId`、`artifactId`就可以了，`version`可以省略。

## 3、仓库

Maven是通过采用引用的方式将依赖的jar引入进来，项目只需要在pom.xml中通过maven坐标的方式来对这些jar进行引用，不对真实的jar进行拷贝到项目目录下。若对jar包进行删除或者升级版本直接修改pom.xml就可以了，但是打包的时候，运行需要用到的jar都会被拷贝到安装包中。

Maven如何寻找我们希望的特定jar包呢？默认情况下，当项目中引入依赖的jar包时，maven先在本地仓库检索jar，若本地仓库没有，maven再去从中央仓库寻找，然后从中央仓库中将依赖的构件下载到本地仓库，然后才可以使用，如果2个地方都没有，maven会报错。

这里提到了“仓库”的概念，maven中仓库的含义是什么呢？

在 Maven 中，任何一个依赖、插件或者项目构件的输出，都可以称之为构件。而仓库是一个位置，这个位置是用来存放各种第三方构件的，所有使用同一个maven的项目可以共享这个仓库中的构件。仓库又分成两个大类：本地仓库和远程仓库。而远程仓库又可以进一步细分为：中央仓库、私服、其他公共远程仓库。

### 3.1、本地仓库

默认情况下，maven本地仓库默认地址是`${user.home}/.m2/respository`目录，这个默认地址也可以在`~/.m2/settings.xml`文件中进行修改，比如可以把本地仓库设置在D盘某个位置：

```xml
<localRepository>D:\ProgramEnv\MavenRepo</localRepository>
```

这样设置之后，依赖的构件都会从远程仓库下载到本地仓库目录中。注意，默认情况下，`${user.home}/.m2/settings.xml`这个文件是不存在的，需要从Maven安装目录中拷贝`conf/settings.xml`文件到`${user.home}/.m2`目录中，然后对`${user.home}/.m2/settings.xml`进行编辑，这个是用户级别的，只会影响当前用户。`conf/settings.xml`这个文件其实也是可以使用的，不过不建议直接使用，因为它是一个全局配置文件，这个修改可能会影响同一机器其他所有使用者。所以建议将maven安装目录中的`settings.xml`拷贝到`${user.home}/.m2`中进行编辑。

### 3.2、远程仓库

远程仓库可以进一步细分为：中央仓库、私服、其他公共远程仓库。介绍如下：

- 中央仓库

  由于maven刚安装好的时候，本地仓库是空的，此时什么都没有配置，去执行maven命令的时候，会看到maven默认执行了一些下载操作，这个下载地址就是中央仓库的地址，是maven内置的一个默认的远程仓库地址，不需要用户去配置。常用的一些地址如下：

  中央仓库地址：https://repo.maven.apache.org/maven2

  搜索某个构件地址：https://search.maven.org/

- 私服

  暂时跳过

- 其他远程仓库

  中央仓库是在国外的，访问速度不是特别快，所以有很多比较大的公司做了一些好事，自己搭建了一些maven仓库服务器，公开出来给其他开发者使用，比如像阿里、网易等等，他们对外提供了一些maven仓库给全球开发者使用，在国内的访问速度相对于maven中央仓库来说还是快了不少。可以通过配置远程镜像仓库的方式使用，通过修改`settings.xml`文件来进行配置对所有使用该配置的maven项目起效，以阿里云配置为例，如下：

  ```xml
  <mirrors>  
      <mirror>
          <id>aliyunmaven</id>
          <mirrorOf>*</mirrorOf>
          <name>maven-aliyun</name>
          <url>https://maven.aliyun.com/repository/public</url>
      </mirror>
  </mirrors>
  ```

## 4、生命周期和插件

### 4.1、生命周期

在开发一个项目的时候，通常有这些环节：创建项目、编写代码、清理已编译的代码、编译代码、执行单元测试、打包、集成测试、验证、部署、生成站点等，这些环节组成了项目的生命周期。maven出现之前，项目的结构没有一个统一的标准，所以生命周期中各个环节对应的自动化脚本也是各种各样，maven出来之后，项目生命周期中的这些环节都被规范化，因为maven约定好了项目的结构，源码的位置、资源文件的位置、测试代码的位置、测试用到的资源文件的位置、静态资源的位置、打包之后文件的位置等，所以清理代码用一个命令`mvn clean`就可以完成，不需要我们去配置清理的目标目录；用`mvn compile`命令就可以完成编译的操作；用`mvn test`就可以自动运行测试用例；用`mvn package`就可以将项目打包为`jar、war`格式的包。能够如此简单，主要还是maven中约定大于配置的结果。

maven将项目的生命周期抽象成了3套生命周期，每套生命周期又包含多个阶段，每套中具体包含哪些阶段是maven已经约定好的，但是每个阶段具体需要做什么，是用户可以自己指定的。maven中定义的三套生命周期为：clean生命周期、default生命周期、site生命周期。这3套生命周期是相互独立的，没有依赖关系的，而每套生命周期中有多个阶段，每套中的多个阶段是有先后顺序的，并且后面的阶段依赖于前面的阶段，而用户可以直接使用`mvn`命令来调用这些阶段去完成项目生命周期中具体的操作，通俗理解就是：maven中的3套生命周期相当于maven定义了3个类来解决项目生命周期中需要的各种操作，每个类中有多个方法，这些方法就是指具体的阶段，方法名称就是阶段的名称，每个类的方法是有顺序的，当执行某个方法的时候，这个方法前面的方法也会执行。具体每个方法中需要执行什么，这个是通过插件的方式让用户去配置的，所以非常灵活。用户执行`mvn 阶段名称`就相当于调用了具体的某个方法。

下面介绍三个生命周期包含哪些阶段（也就是通俗理解中的每个类有哪些方法，顺序是什么样的）：

#### 4.1.1、clean生命周期

clean生命周期的目的是清理项目，它包含三个阶段：

| 生命周期阶段 | 描述                                  |
| ------------ | ------------------------------------- |
| pre-clean    | 执行一些需要在clean之前完成的工作     |
| clean        | 移除所有上一次构建生成的文件          |
| post-clean   | 执行一些需要在clean之后立刻完成的工作 |

用户可以通过`mvn pre-clean`来调用clean生命周期中的`pre-clean`阶段需要执行的操作。调用`mvn post-clean`会执行上面3个阶段所有的操作，因为每个生命周期中的后面的阶段会依赖于前面的阶段，当执行某个阶段的时候，会先执行其前面的阶段。

#### 4.1.2、default生命周期

这个是maven主要的生命周期，主要被用于构建应用，包含了23个阶段。

| 生命周期阶段            | 描述                                                         |
| :---------------------- | :----------------------------------------------------------- |
| validate                | 校验：校验项目是否正确并且所有必要的信息可以完成项目的构建过程。 |
| initialize              | 初始化：初始化构建状态，比如设置属性值。                     |
| generate-sources        | 生成源代码：生成包含在编译阶段中的任何源代码。               |
| process-sources         | 处理源代码：处理源代码，比如说，过滤任意值。                 |
| generate-resources      | 生成资源文件：生成将会包含在项目包中的资源文件。             |
| process-resources       | 编译：复制和处理资源到目标目录，为打包阶段最好准备。         |
| compile                 | 处理类文件：编译项目的源代码。                               |
| process-classes         | 处理类文件：处理编译生成的文件，比如说对Java class文件做字节码改善优化。 |
| generate-test-sources   | 生成测试源代码：生成包含在编译阶段中的任何测试源代码。       |
| process-test-sources    | 处理测试源代码：处理测试源代码，比如说，过滤任意值。         |
| generate-test-resources | 生成测试源文件：为测试创建资源文件。                         |
| process-test-resources  | 处理测试源文件：复制和处理测试资源到目标目录。               |
| test-compile            | 编译测试源码：编译测试源代码到测试目标目录.                  |
| process-test-classes    | 处理测试类文件：处理测试源码编译生成的文件。                 |
| test                    | 测试：使用合适的单元测试框架运行测试（Juint是其中之一）。    |
| prepare-package         | 准备打包：在实际打包之前，执行任何的必要的操作为打包做准备。 |
| package                 | 打包：将编译后的代码打包成可分发格式的文件，比如JAR、WAR或者EAR文件。 |
| pre-integration-test    | 集成测试前：在执行集成测试前进行必要的动作。比如说，搭建需要的环境。 |
| integration-test        | 集成测试：处理和部署项目到可以运行集成测试环境中。           |
| post-integration-test   | 集成测试后：在执行集成测试完成后进行必要的动作。比如说，清理集成测试环境。 |
| verify                  | 验证：运行任意的检查来验证项目包有效且达到质量标准。         |
| install                 | 安装：安装项目包到本地仓库，这样项目包可以用作其他本地项目的依赖。 |
| deploy                  | 部署：将最终的项目包复制到远程仓库中与其他开发者和项目共享。 |

#### 4.1.3、site生命周期

site生命周期的目的是建立和发布项目站点，Maven能够基于pom.xml所包含的信息，自动生成一个友好的站点，方便团队交流和发布项目信息。主要包含以下4个阶段：

| 阶段        | 描述                                                       |
| ----------- | ---------------------------------------------------------- |
| pre-site    | 执行一些需要在生成站点文档之前完成的工作                   |
| site        | 生成项目的站点文档                                         |
| post-site   | 执行一些需要在生成站点文档之后完成的工作，并且为部署做准备 |
| site-deploy | 将生成的站点文档部署到特定的服务器上                       |

#### 4.1.4、mvn命令和生命周期

从命令行执行maven任务的最主要方式就是调用maven生命周期的阶段，需要注意的是，每套生命周期是相互独立的，但是每套生命周期中阶段是有前后依赖关系的，执行某个的时候，会按序先执行其前面所有的。mvn执行阶段的命令格式是：`mvn 阶段1 [阶段2] ··· [阶段n]`，多个阶段的名称之间用空格隔开。常见例子说明如下：

- `mvn clean`

  该命令是调用clean生命周期的clean阶段，实际执行的阶段为clean生命周期中的pre-clean和clean阶段。

- `mvn test`

  该命令调用default生命周期的test阶段，实际上会从default生命周期的第一个阶段（`validate`）开始执行一直到`test`阶段结束。这里面包含了代码的编译，运行测试用例。

- `mvn clean install`

  这个命令中执行了两个阶段：`clean`和`install`，从上面3个生命周期的阶段列表中找一下，可以看出`clean`位于`clean`生命周期的表格中，`install`位于`default`生命周期的表格中，所以这个命令会先从`clean`生命周期中的`pre-clean`阶段开始执行一直到`clean`生命周期的`clean`阶段；然后会继续从`default`生命周期的`validate`阶段开始执行一直到default生命周期的`install`阶段。这里面包含了清理上次构建的结果，编译代码，测试，打包，将打好的包安装到本地仓库。

- `mvn clean deploy`

  这个命令会先按顺序执行`clean`生命周期的`[pre-clean,clean]`这个闭区间内所有的阶段，然后按序执行`default`生命周期的`[validate,deploy]`这个闭区间内的所有阶段（也就是`default`生命周期中的所有阶段）。这个命令内部包含了清理上次构建的结果、编译代码、运行单元测试、打包、将打好的包安装到本地仓库、将打好的包发布到私服仓库。

### 4.2、Maven插件

maven插件主要是为maven中生命周期中的阶段服务的，maven中只是定义了3套生命周期，以及每套生命周期中有哪些阶段，具体每个阶段中执行什么操作，完全是交给插件去干的。maven中的插件就相当于一些工具，比如编译代码的工具，运行测试用例的工具，打包代码的工具，将代码上传到本地仓库的工具，将代码部署到远程仓库的工具等等，这些都是maven中的插件。插件可以通过`mvn`命令的方式调用直接运行，或者将插件和maven生命周期的阶段进行绑定，然后通过`mvn 阶段`的方式执行阶段的时候，会自动执行和这些阶段绑定的插件。

#### 4.2.1、插件目标

maven中的插件以jar的方式存在于仓库中，和其他构件是一样的，也是通过坐标进行访问，每个插件中可能为了代码可以重用，一个插件可能包含了多个功能，比如编译代码的插件，可以编译源代码、也可以编译测试代码。插件中的每个功能就叫做插件的目标（Plugin Goal），每个插件中可能包含一个或者多个插件目标。

插件目标是用来执行任务的，那么执行任务肯定是有参数配的，这些就是目标的参数。

关于插件常用的命令如下：

- 列出插件的所有目标：

  1、`mvn 插件goupId:插件artifactId[:插件version]:help`

  2、`mvn 插件前缀:help`

  比如执行`mvn org.apache.maven.plugins:maven-clean-plugin:help`，就能展示出`maven-clean-plugin`这个插件所有的目标，会有2个，分别是`clean:clean、clean:help`，分号后面是目标名称，分号前面是插件的前缀，每个目标的后面包含对这个目标的详细解释说明，

- 查看插件某个目标的详细参数列表：

  1、`mvn 插件goupId:插件artifactId[:插件version]:help -Dgoal=目标名称 -Ddetail`

  2、`mvn 插件前缀:help -Dgoal=目标名称 -Ddetail`

  比如执行`mvn org.apache.maven.plugins:maven-clean-plugin:help -Dgoal=help -Ddetail`，列出了`clean`插件的`help`目标的详细参数信息。

- 命令行运行插件：

  1、`mvn 插件goupId:插件artifactId[:插件version]:插件目标 [-D目标参数1] [-D目标参数2] ··· [-D目标参数n]`

  2、`mvn 插件前缀:插件目标  [-D目标参数1] [-D目标参数2] ··· [-D目标参数n]`

  比如执行`mvn org.apache.maven.plugins:maven-surefire-plugin:test -Dmaven.test.skip=true`，通过`maven.test.skip`这个属性来进行命令行传参，将其传递给`test`目标的`skip`属性，这样即可跳过项目中的单元测试。

- 获取插件目标的详细信息的另一种方式：

  1、`mvn help:describe -Dplugin=插件goupId:插件artifactId[:插件version] -Dgoal=目标名称 -Ddetail`

  2、`mvn help:describe -Dplugin=插件前缀 -Dgoal=目标名称 -Ddetail`

  这两个命令实际调用的是help插件的`describe`这个目标，这个目标可以列出其他指定插件目标的详细信息，比如执行`mvn help:describe -Dplugin=org.apache.maven.plugins:maven-surefire-plugin -Dgoal=test -Ddetail`，就列出了`test`目标的详细信息。

#### 4.2.2、插件前缀

运行插件的时候，可以通过指定插件坐标的方式运行，但是插件的坐标信息过于复杂，也不方便写和记忆，所以maven中给插件定义了一些简捷的插件前缀，可以通过插件前缀来运行指定的插件。可以通过`mvn help:describe -Dplugin=插件goupId:插件artifactId[:插件version]`命令来查看某个插件的前缀。

#### 4.2.3、插件和生命周期阶段绑定

maven只是定义了生命周期中的阶段，而没有定义每个阶段中具体的实现，这些实现是由插件的目标来完成的，所以需要将阶段和插件目标进行绑定，来让插件目标帮助生命周期的阶段做具体的工作，生命周期中的每个阶段支持绑定多个插件的多个目标。当我们将生命周期中的阶段和插件的目标进行绑定的时候，执行`mvn 阶段`就可以执行和这些阶段绑定的插件目标。

Maven中已经内置了一些插件和阶段的绑定，列举如下：

- clean生命周期

  | 生命周期阶段 | 插件:目标                |
  | ------------ | ------------------------ |
  | pre-clean    |                          |
  | clean        | maven-clean-plugin:clean |
  | post-clean   |                          |

- default生命周期

  这里只列出该生命周期中有默认绑定的一些阶段，其他没有列出的没有绑定任何插件，因此没有任何实际的行为。

  | 生命周期阶段           | 插件:目标                            | 执行任务                       |
  | ---------------------- | ------------------------------------ | ------------------------------ |
  | process-resources      | maven-resources-plugin:resources     | 复制主资源文件至主输出目录     |
  | compile                | maven-compiler-plugin:compile        | 编译主代码至主输出目录         |
  | process-test-resources | maven-resources-plugin:testResources | 复制测试资源文件至测试输出目录 |
  | test-compile           | maven-compiler-plugin:testCompile    | 编译测试代码至测试输出目录     |
  | test                   | maven-surefile-plugin:test           | 执行测试用例                   |
  | package                | maven-jar-plugin:jar                 | 创建项目jar包                  |
  | install                | maven-install-plugin:install         | 将输出构件安装到本地仓库       |
  | deploy                 | maven-deploy-plugin:deploy           | 将输出的构件部署到远程仓库     |

- site生命周期

  | 生命周期阶段 | 插件:目标                |
  | ------------ | ------------------------ |
  | pre-site     |                          |
  | site         | maven-site-plugin:site   |
  | post-site    |                          |
  | site-deploy  | maven-site-plugin:deploy |

#### 4.2.4、自定义绑定

除了默认绑定的一些操作，我们自己也可以将一些阶段绑定到指定的插件目标上来完成一些操作，这种自定义绑定让maven项目在构件的过程中可以执行更多更丰富的操作。

常见的一个案例是：创建项目的源码jar包，将其安装到仓库中，内置插件绑定关系中没有涉及到这一步的任务，所以需要用户自己配置。插件`maven-source-plugin`的`jar-no-fork`可以帮助我们完成该任务，我们将这个目标绑定在`default`生命周期的`verify`阶段上面，这个阶段没有任何默认绑定，`verify`是在测试完成之后并将构件安装到本地仓库之前执行的阶段，在这个阶段我们生成源码。在项目的`pom.xml`文件中配置如下：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>3.2.0</version>
            <executions>
                <!-- 使用插件需要执行的任务 -->
                <execution>
                    <!-- 任务id，需要唯一，不指定则默认为default -->
                    <id>attach-source</id>
                    <!-- 任务中插件的目标，可以指定多个 -->
                    <goals>
                        <goal>jar-no-fork</goal>
                    </goals>
                    <!-- 绑定的生命周期的阶段 -->
                    <phase>verify</phase>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

配置之后，后续经过verify阶段即可看见该插件的该目标的执行效果

## 5、聚合、继承、裁剪

考虑以下场景：使用java做一个电商网站，涉及到：pc端网站、h5微站、移动端接口部分，那么可以使用maven创建3个项目用于这3块业务的开发，这几个项目经常一起上线，上线过程，先由开发人员进行打包，然后进行发布。使用`mvn package`进行打包，需要在每个项目的`pom.xml`所在目录都去执行一次这个命令，也就是说需要执行3次，这个电商项目还会涉及到后台系统、bi系统、监控系统等等，可能最后会多达10个小项目，那时候每次上线都需要执行10次打包操作，这个过程是相当繁琐的。

那么maven有没有更好的办法来解决这个事情呢？这里用到的就是本次要说的maven中的聚合。相比于上述将各个部分作为独立的小项目方式，可以将整个电商网站视为一个大项目，各个部分是该项目中的具体的不同模块。然后只需要在电商网站这个大项目下执行`mvn`命令，就会自动为其所管理的模块执行同样的`mvn`命令。

### 5.1、Maven聚合

maven聚合需要创建一个新的位于顶层的maven项目， 用来管理其他的maven构件模块，新的maven项目中加入如下配置：

```xml
<modules>
    <module>模块1</module>
    <module>模块2</module>
    <module>模块n</module>
</modules>
<package>pom</package>
```

新的项目中执行任何`mvn`命令，都会`modules`中包含的所有模块执行同样的命令，而被包含的模块不需要做任何特殊的配置，正常的maven项目就行。注意上面的`module`元素，这部分是被聚合的模块`pom.xml`所在目录的相对路径。package的值必须为pom，这个需要注意。

### 5.2、Maven继承

对于上述场景，使用模块化设计之后，简化了`mvn`命令操作，一个命令能自动的被多个模块同时执行，但是可能存在以下现象：可能各个模块都会引入某些完全一致的依赖，就会在各自的pom文件中都引入，这样就会让`pom.xml`文件部分内容重复，更好的方式是将重复内容提取出来只引入一次。可以使用Maven的继承功能。

主要分为3个步骤：

1. 用maven创建一个顶层父项目，将公共的依赖信息放在pom.xml中，如下：

   ```xml
   <dependencies>
       <dependency>依赖的构件的坐标信息</dependency>
       <dependency>依赖的构件的坐标信息</dependency>
       <dependency>依赖的构件的坐标信息</dependency>
   </dependencies>
   ```

2. 将顶层父项目的`package`元素的值置为`pom`，如下：

   ```xml
   <packaging>pom</packaging>
   ```

3. 在该项目中创建需要管理的各个子模块，同时pom.xml引入父项目的配置，如下：

   ```xml
   <parent>
       <groupId>父构件groupId</groupId>
       <artifactId>父构件artifactId</artifactId>
       <version>父构件的版本号</version>
   </parent>
   ```

这样设置之后，在父构件中的pom文件中引入依赖，子构件的pom文件不需要任何改动也能使用这些依赖。

### 5.3、依赖管理

当然，上面的继承依然存在一个问题，如果继续新增一个子模块，父构件`pom.xml`中配置的这些依赖可能某个子项目只是想使用其中一个构件，但是上面的继承关系却把所有的依赖都给传递到子构件中了，这种显然是不合适的。这时可以在父构件中的`pom`文件使用`dependencyManagement`元素来解决这个问题。

maven提供的`dependencyManagement`元素既能让子模块继承到父模块的依赖配置，又能保证子模块依赖使用的灵活性，在`dependencyManagement`元素下声明的依赖不会引入实际的依赖，只是声明了这些依赖，不过它可以对`dependencies`中使用的依赖起到一些约束作用。

例如在父构件的`pom.xml`文件配置如下：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>5.3.1</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

此时执行`mvn dependency:tree`命令可以发现父子构件中都没有导入实际的依赖，说明父构件`pom.xml`中`dependencyManagement`管理的依赖没有被子模块依赖进去。子模块如果想用到这些配置，可以`dependencies`进行引用，引用之后，依赖才会真正的起效。

在子模块的`pom.xml`文件引用如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
    </dependency>
</dependencies>
```

再执行`mvn dependency:tree`，可以发现子构件中引入了真正的依赖。

总结一下就是：`dependencyManagement`不会引入实际的依赖，只有在子模块中使用`dependencies`来引入父`dependencyManagement`声明的依赖之后，依赖的构建才会被真正的引入。

使用`dependencyManagement`来解决继承的问题，子`pom.xml`中只用写`groupId,artifactId`就可以了，其他信息都会从父`dependencyManagement`中声明的依赖关系中传递过来，通常使用这种方式将所有依赖的构件在父`pom.xml`中定义好，子构件中只需要通过`groupId,artifactId`就可以引入依赖的构件，而不需要写`version`，可以很好的确保多个子模块中依赖构件的版本一致性，对应依赖构件版本的升级也非常方便，只需要在父`pom.xml`中修改一下就可以了。

### 5.4、插件管理

maven中提供了`dependencyManagement`来解决继承的问题，同样也提供了解决插件继承问题的`pluginManagement`元素，在父构件`pom`中可以在这个元素中声明插件的配置信息，但是子模块`pom.xml`中不会引入此插件的配置信息，只有在子构件`pom.xml`中使用`plugins->plugin`元素引入这些声明的插件的时候，插件才会起效，子插件中只需要写`groupId、artifactId`，其他信息都可以从父构件中传递过来。

例如在父构件中pom.xml配置如下：

```xml
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <version>3.2.0</version>
                <executions>
                    <execution>
                        <id>attach-source</id>
                        <phase>verify</phase>
                        <goals>
                            <goal>jar-no-fork</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```

此时在子模块目录下执行相关的`mvn`命令并不会看见上述插件的运行效果，说明并没有真正继承父构件中的配置的插件信息，在子构件的`pom.xml`文件中加入以下配置：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

再次执行与该插件相关的`mvn`命令，就能看见在配置的生命周期的阶段时该插件起作用了。类似地，只需要写插件的`groupId、artifactId`，插件其他信息都可以从父构件中传递过来。

## 6、多模块任意构建

考虑以下场景：需要做一个电商项目，一般都会做成微服务的形式，按业务进行划分，本次主要以账户业务和订单业务为例，我们将这两块业务分别作为2个大的模块来开发，而订单模块又会调用账户模块中的接口，所以对账户模块接口部分会有依赖。用maven搭建的项目结构如下：

```pla
b2b
    b2b-account
        b2b-account-api
        b2b-account-service
    b2b-order
        b2b-order-api
        b2b-order-service
```

`b2b-account`：账户模块，其中包含2个小模块：`b2b-account-api`和`b2b-account-service`

`b2b-account-api`：账户模块对外暴露的接口部分，以供外部调用

`b2b-account-service`：账户模块具体业务实现，依赖于`b2b-account-api`模块

`b2b-order`：订单模块，也是包含2个小模块：`b2b-order-api`和`b2b-order-service`

`b2b-order-api`：订单模块对外暴露的接口部分，以供外部调用

`b2b-order-service`：订单模块具体业务实现，依赖2个模块`b2b-order-api、b2b-account-api`

假设上述各个模块已经开发完毕，需要将这些构建下载到本地仓库，于是在`b2b`目录下执行`mvn clean install`命令，`mvn`命令对多模块构件时，会根据模块的依赖关系而得到模块的构建顺序，这个功能就是maven的反应堆（reactor）做的事情，反应堆会根据模块之间的依赖关系、聚合关系、继承关系等等，从而计算得出一个合理的模块构建顺序，所以反应堆的功能是相当强大的。

### 6.1、按需随意构建

有这样的一种场景：`b2b-account-api`被`b2b-account-service`和`b2b-order-service`依赖了，所以当`b2b-account-api`有修改的时候，我们希望他们3个都能够被重新构建一次，而不是去对所有的模块都进行重新构建，也就是说只希望被影响的模块都能够参与重新构建。需要的是按需构建，需要构建哪些模块让我们自己能够随意指定，这样也可以加快构建的速度。maven反应堆考虑到了这种情况，`mvn`命令提供了一些功能可以帮我们实现这些操作，常用命令如下：

- `-pl,--projects <arg>`：只构建指定的模块，`arg`表示多个模块，之间用逗号分开，模块有两种写法：
  1. `-pl 模块1相对路径 [,模块2相对路径] [,模块n相对路径]`
  2. `-pl [模块1的groupId]:模块1的artifactId [,[模块2的groupId]:模块2的artifactId] [,[模块n的groupId]:模块n的artifactId]`
- `-rf,--resume-from <arg>`：从指定的模块位置裁剪反应堆，然后构建保留的反应堆模块
- `-am,--also-make`：同时构建所列模块的所依赖的模块
- `-amd,--also-make-dependents`：同时构建依赖于所列模块的模块

## 7、多环境构建支持

在开发一个较大项目的时候，可能会有开发环境、测试环境、线上环境，每个环境中配置文件可能都是不一样的，比如：数据库的配置，静态资源的配置等等，所以希望构建工具能够适应不同环境的构建工作，能够灵活处理，并且操作足够简单。Maven作为一款优秀的构建工具，这方面做的足够好了，能够很好的适应不同环境的构建工作，这里主要讲解maven如何灵活的处理各种不同环境的构建工作。

### 7.1、Maven属性

#### 7.1.1、自定义属性

maven属性前面有用到过，可以自定义一些属性进行重用，如下：

```xml
<properties>
    <spring.verion>5.2.1.RELEASE</spring.verion>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.verion}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>${spring.verion}</version>
    </dependency>
</dependencies>
```

这种maven自定义属性，需要先在`properties`中定义，然后才可以在其他地方使用`${属性元素名称}`进行引用。这样写的好处是可以简化相同版本的书写，同时方便统一所使用的依赖版本

#### 7.1.2、内置属性

主要有两个：

- `${basedir}`：表示项目根目录，即包含pom.xml文件的目录
- `${version}`：表示项目的版本号

#### 7.1.3、POM属性

用户可以使用这些属性引用`pom.xml`文件中对应元素的值，常用的有：

1. `${pom.build.sourceDirectory}`：项目的主源码目录，默认为src/main/java/
2. `${project.build.testSourceDirectory}`：项目的测试源码目录，默认为src/test/java/
3. `${project.build.directory}`：项目构建输出目录，默认为target/
4. `${project.build.outputDirectory}`：项目主代码编译输出目录，默认为target/classes
5. `${project.build.testOutputDirectory}`：项目测试代码编译输出目录，默认为target/test-classes
6. `${project.groupId}`：项目的groupId
7. `${project.artifactId}`：项目的artifactId
8. `${project.version}`：项目的version，与`${version}`等价
9. `${project.build.finalName}`：项目打包输出文件的名称，默认为`${project.artifactId}-${project.version}`

#### 7.1.4、Settings属性

这种属性以settings.开头来引用`~/.m2/settings.xml`中的内容，如`${settings.localRepository}`指向用户本地仓库的地址

#### 7.1.5、Java系统属性

所有java系统属性都可以使用maven属性来进行引用，例如`${user.home}`指向了当前用户目录。java系统属性可以通过`mvn help:system`命令看到。

#### 7.1.6、环境变量属性

所有的环境变量都可以使用env.开头的方式来进行引用，如`${env.JAVA_HOME}`可以获取环境变量`JAVA_HOME`的值。用户可以使用`mvn help:system`命令查看所有环境变量的值。

### 7.2、多套环境构建问题

考虑以下场景：`b2b-account-service`会操作数据库，所以需要一个配置文件来放数据库的配置信息，配置文件一般都放在`src/main/resources`中，在这个目录中新建一个`jdbc.properties`文件，内容如下：

```
jdbc.url=jdbc:mysql://localhost:3306/mavenlearn?characterEncoding=UTF-8
jdbc.username=root
jdbc.password=root
```

现在系统需要打包，运行下面命令`mvn clean package -pl :b2b-account-service`，这里有一个问题：

上面jdbc的配置的是开发库的db信息，打包之后生成的jar中也是上面开发环境的配置，那么上测试环境是不是需要修改上面的配置，最终上线的时候，上面的配置是不是又得重新修改一次，比较麻烦。

那能不能写3套环境的jdbc配置，打包的时候去指定具体使用那套配置？maven支持这么做，pom.xml的`project`元素下面提供了一个`profiles`元素可以用来对多套环境进行配置。

在讲profiles的使用之前，需要先了解资源文件打包的过程。

### 7.3、理解资源文件打包过程

举例说明如下：将`src/main/resouces/jdbc.properties`复制一份到`src/test/resources`目录，然后在项目根目录下执行命令`mvn clean package -pl :b2b-account-service`，可以发现输出过程显示使用了插件`maven-resources-plugin`的`resources`目标，将`src/main/resouces`目录中的资源文件复制到了`target/classess`目录，同时也使用了插件`maven-resources-plugin`的`testResources`目标，将`src/test/resouces`目录中的资源文件复制到了`target/testClassess`目录。

这里可以想到一个问题：复制的过程中是否能够对资源文件进行替换，比如在资源文件中使用一些占位符，然后复制过程中对这些占位符进行替换？实际上该插件能够实现这样的功能，介绍如下：

#### 7.3.1、设置资源文件内容动态替换

资源文件中可以通过`${maven属性}`来引用maven属性中的值，打包的过程中这些会被替换掉，替换的过程默认是不开启的，需要手动开启配置。

现在修改`src/main/resource/jdbc.properties`内容如下：

```
jdbc.url=${jdbc.url}
jdbc.username=${jdbc.username}
jdbc.password=${jdbc.password}
```

修改`src/test/resource/jdbc.properties`内容如下：

```
jdbc.url=${jdbc.url}
jdbc.username=${jdbc.username}
jdbc.password=${jdbc.password}
```

然后在`b2b-account-service/pom.xml`中加入下面内容：

```xml
<properties>
    <jdbc.url>jdbc:mysql://localhost:3306/mavenlearn?characterEncoding=UTF-8</jdbc.url>
    <jdbc.username>root</jdbc.username>
    <jdbc.password>root</jdbc.password>
</properties>
```

开启动态替换配置，需要在pom.xml中加入下面配置：

```xml
<build>
    <resources>
        <resource>
            <!-- 指定资源文件的目录 -->
            <directory>${project.basedir}/src/main/resources</directory>
            <!-- 是否开启过滤替换配置，默认是不开启的 -->
            <filtering>true</filtering>
        </resource>
    </resources>
    <testResources>
        <testResource>
            <!-- 指定资源文件的目录 -->
            <directory>${project.basedir}/src/test/resources</directory>
            <!-- 是否开启过滤替换配置，默认是不开启的 -->
            <filtering>true</filtering>
        </testResource>
    </testResources>
</build>
```

`resources`元素中可以包含多个`resource`，每个`resource`表示一个资源的配置信息，一般使用来控制主资源的复制的。`testResources`元素和`testResource`类似，是用来控制测试资源复制的。

此时再次运行`mvn clean package -pl :b2b-account-service`命令，可以看见`target`对应目录下2个资源文件内容被动态替换掉了。

### 7.4、使用profiles处理多环境构建问题

maven支持配置多套环境，每套环境中可以指定自己的maven属性，`mvn`命令对模块进行构建的时候可以通过`-P`参数来指定具体使用哪个环境的配置。`pom.xml`文件配置代码如下：

```xml
<profiles>
    <profile>测试环境配置信息</profile>
    <profile>开发环境配置信息</profile>
    <profile>线上环境配置信息</profile>
    <profile>环境n配置信息</profile>
</profiles>
```

例如配置一个开发环境，就可以如下：

```xml
<profile>
    <id>dev</id>
    <properties>
        <jdbc.url>dev jdbc url</jdbc.url>
        <jdbc.username>dev jdbc username</jdbc.username>
        <jdbc.password>dev jdbc password</jdbc.password>
    </properties>
</profile>
```

`id`标签表示这套环境的标识信息，`properties`可以定义环境中会使用到的属性列表。执行`mvn`命令编译的时候可以带上一个`-P profileid`来使用指定的环境进行构建。

如果在`pom.xml`文件中配置了多套环境，并且在执行构建命令时不指定特定环境，那么就会导致资源文件的占位符不会被替换，这时可以利用`<activation>`标签指定一个默认的构建环境，修改`pom.xml`文件如下：

```xml
<!-- 配置多套环境 -->
    <profiles>
        <!-- 开发环境使用的配置 -->
        <profile>
            <id>dev</id>
            <!-- 开启默认构建环境 -->
            <activation>
        		<activeByDefault>true</activeByDefault>
    		</activation>
            <properties>
                <jdbc.url>dev jdbc url</jdbc.url>
                <jdbc.username>dev jdbc username</jdbc.username>
                <jdbc.password>dev jdbc password</jdbc.password>
            </properties>
        </profile>
        <!-- 线上环境使用的配置 -->
        <profile>
            <id>prod</id>
            <properties>
                <jdbc.url>prod jdbc url</jdbc.url>
                <jdbc.username>prod jdbc username</jdbc.username>
                <jdbc.password>prod jdbc password</jdbc.password>
            </properties>
        </profile>
    </profiles>
```

也可以在启动时指定多个环境，可以在`-P`参数后跟多个环境的`id`，多个之间用逗号隔开，当使用多套环境的时候，多套环境中的maven属性会进行合并，如果多套环境中属性有一样的，后面的会覆盖前面的。

可以通过命令`mvn help:all-profiles`查看目前有哪些环境，可以通过命令`mvn help:active-profiles`查看激活的是哪些环境

这里又有一个新的问题出现：配置文件比较分散。上面介绍了`b2b-account-service`不同环境的构建操作，是在`pom.xml`中进行配置的，`b2b-order-service`中数据的配置也可以这么做，如果以后有更多的模块都需要连接不同的数据库，是不是每个模块中都需要配置这样的`pom.xml`，那时候数据库的配置分散在几十个`pom.xml`文件中，如果需要修改数据库的配置的时候，需要去每个模块中去修改`pom.xml`中的属性，非常麻烦。

针对上述问题，可以将数据库所有的配置放在一个文件中，若有需要可以只修改这一个文件，然后执行构建操作，重新发布上线。maven支持这么做，可以在`profile`中指定一个外部属性文件`xx.properties`，文件内容是这种格式的：

```
key1=value1
key2=value2
···
keyn=valuen
```

然后在`profile`元素中加入下面配置：

```xml
<build>
    <filters>
        <filter>xx.properties文件路径（相对路径或者完整路径）</filter>
    </filters>
</build>
```

上面的`filter`元素可以指定多个，当有多个外部属性配置文件的时候，可以指定多个来进行引用。然后资源文件复制的时候就可以使用`xxx=${key1}`方式引用外部资源文件的内容。

## 8、自定义插件

略
