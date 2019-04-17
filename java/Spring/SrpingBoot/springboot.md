** Servlet containers ** 内置的Servlet容器
Tomcat、Jetty、Undertow

@RestController 这个注解是用来标识Web中的Controller，当有请求过来时会查找这个注解的类。  
@RequestMapping 注释提供了“路由”信息。它告诉Spring，任何路径为“/”的HTTP请求都应该映射到home方法。@RestController注释告诉Spring将结果字符串直接呈现回调用者

@EnableAutoConfiguration 第二个类级注释，这个注释告诉Spring
引导到“启动器”您想要如何配置Spring，基于您已经添加的jar依赖项。由于***Spring -boot-starter-web**添加了Tomcat和Spring MVC，因此自动配置将假定您正在开发一个web应用程序并相应地设置Spring。
自动配置被设计为可以很好地与“启动器”一起工作，但是这两个概念并不直接相关。您可以在启动程序之外自由地选择jar依赖项，Spring Boot仍然会尽力自动配置您的应用程序。

main 应用程序的最后一部分是主方法。这只是应用程序入口点遵循Java约定的标准方法。我们的主方法委托给Spring Boot
通过调用run来调用SpringApplication类。SpringApplication将引导我们的应用程序，启动Spring，而Spring又将启动自动配置的Tomcat web服务器。我们需要通过
类作为run方法的参数来告诉SpringApplication哪个是主方法

2,构建系统建

&ensp;&ensp;强烈建议您选择一个支持依赖关系管理的构建系统，以及一个可以使用发布到“Maven中心”存储库的构件的构建系统。我们建议您选择Maven或Gradle。让Spring Boot与其他构建系统(例如Ant)一起工作是可能的，但是它们不会得到特别好的支持。

&ensp;&ensp;管理列表包含所有spring引导可以使用的spring模块，以及第三方库的精细化列表。该列表作为标准的物料清单(**spring-boot-dependencies**)提供，还提供了对Maven和Gradle的额外专用支持
2.1, Maven   
Maven用户可以从**spring-boot-starter-parent**项目继承，以获得合理的默认值。  
** 父项目提供以下功能:**
* Java 1.6作为默认编译器级别。
* UTF-8源编码。
* 依赖项管理部分，允许您省略公共依赖项的<version>标记，这些依赖项继承自spring-boot-dependencies POM。
* 合理的资源过滤。
合理的插件配置(exec plugin, surefire, Git提交ID, shade)。
* 应用程序的合理资源过滤。属性和应用程序。包含特定于概要文件的文件(例如application-foo)。属性和application-foo.yml)
>关于最后一点:由于默认配置文件接受Spring样式的占位符(${…})，Maven过滤被更改为使用@..@占位符(您可以使用Maven属性resource.delimiter覆盖它)。

2.2 继承启动父级
&ensp;&ensp;要将项目配置为继承自spring-boot-starter-parent，只需设置父类:
```
<!-- Inherit defaults from Spring Boot -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.2.RELEASE</version>
</parent>
```
>您应该只需要在此依赖项上指定Spring引导版本号。如果导入其他启动程序，可以安全地省略版本号。

&ensp;&ensp;使用该设置，您还可以通过覆盖您自己项目中的属性来覆盖各个依赖项。例如，要升级到另一个Spring数据发布培训，您需要将以下内容添加到pom.xml中
```
<properties>
    <spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
</properties>
```
**在没有父POM的情况下使用Spring Boot**  
&ensp;&ensp;并不是每个人都喜欢继承spring-boot-starter-parent POM。您可能需要使用自己的企业标准父级，或者您可能更愿意显式地声明您的所有父级Maven配置。  
&ensp;&ensp;如果你不想使用spring-boot-starter-parent，你仍然可以使用scope=import dependency来保持依赖管理的好处(但不是插件管理)
```
<dependencyManagement>
    <dependencies>
        <dependency>
        <!-- Import dependency management from Spring Boot -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>1.5.2.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
该设置不允许使用如上所述的属性覆盖单个依赖项。
要实现相同的结果，您需要在spring-boot-dependencies条目之前在项目的dependencyManagement中添加一个条目。例如，要升级到另一个Spring Data release，您需要将以下内容添加到pom.xml中
```
<dependencyManagement>
    <dependencies>
    <!-- Override Spring Data release train provided by Spring Boot -->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-releasetrain</artifactId>
            <version>Fowler-SR2</version>
            <scope>import</scope>
            <type>pom</type>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>1.5.2.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
>在上面的例子中，我们指定了一个BOM，但是任何依赖类型都可以通过这种方式被覆盖

2.3 更改Java版本
&ensp;&ensp;spring-boot-starter-parent选择了相当保守的Java兼容性。如果您想遵循我们的建议并使用较晚的Java版本，您可以添加一个Java。版本属性:
```
<properties>
    <java.version>1.8</java.version>
</properties>
```
2.4 使用Spring Boot Maven插件
&ensp;&ensp;Spring Boot包含一个Maven插件，可以将项目打包为可执行jar。如果您想使用插件，请将插件添加到您的<plugins>节点:
```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```
>如果您使用Spring Boot starter父pom，您只需要添加插件，不需要配置它，除非您想更改父pom中定义的设置。

2.4 Gradle
Gradle用户可以直接在依赖项部分导入“starter”。与Maven不同，没有要导入“super parent”以共享某些配置
```
repositories {
  jcenter()
}
dependencies {
  compile("org.springframework.boot:spring-boot-starter-web:1.5.2.RELEASE")
}
```
&ensp;&ensp;spring-boot-gradle插件也是可用的，它提供了创建可执行jar和从源代码运行项目的任务。它还提供了依赖关系管理，在其他功能中，允许您省略任何由Spring Boot管理的依赖关系的版本号:
```
plugins {
  id 'org.springframework.boot' version '1.5.2.RELEASE'
  id 'java'
}
repositories {
  jcenter()
}
dependencies {
  compile("org.springframework.boot:spring-boot-starter-web")
  testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

3 Configuration classes  @Configuration
  Auto-configuration
  如果您发现正在应用您不想要的特定自动配置类，您可以使用@EnableAutoConfiguration的exclude属性禁用它们
  ```
@Configuration
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {
}
  ```
  如果类不在类路径上，则可以使用注释的exclude ename属性并指定完全限定名。最后，还可以通过spring.autoconfigure控制要排除的自动配置类列表。排除属性。  
**4, Spring bean和依赖项注入**  
&ensp;&ensp;您可以自由地使用任何标准Spring框架技术来定义bean及其注入的依赖项。为了简单起见，我们经常发现使用@ComponentScan来查找bean，与@Autowired构造函数注入结合使用效果很好  
&ensp;&ensp;如果按照上面的建议构造代码(将应用程序类定位在根包中)，可以添加@ComponentScan，而不需要任何参数。所有应用程序组件(@Component，@Service、@Repository、@Controller等)将自动注册为Spring bean。  
&ensp;&ensp;下面是一个示例@Service Bean，它使用构造函数注入来获得所需的RiskAssessor Bean
```
@Service
public class DatabaseAccountService implements AccountService {
    private final RiskAssessor riskAssessor;
    @Autowired
    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }
    // ...
}
```
&ensp;&ensp;如果bean有一个构造函数，可以省略@Autowired。
```
@Service
public class DatabaseAccountService implements AccountService {
    private final RiskAssessor riskAssessor;
    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }
    // ...
}
```
>注意，如何使用构造函数注入允许riskAssessor字段被标记为final，表示它随后不能被更改。

**5,@SpringBootApplication注释**
&ensp;&ensp;许多Spring Boot开发人员总是用@Configuration注释他们的主类，@EnableAutoConfiguration @ComponentScan。由于这些注释经常一起使用(特别是如果您遵循上面的最佳实践)，Spring Boot提供了一种方便的方法@SpringBootApplication选择。  
&ensp;&ensp;@SpringBootApplication注释等价于使用@Configuration，@EnableAutoConfiguration和@ComponentScan的默认属性
**6，Running as a packaged application**
```
$>java -jar target/myproject-0.0.1-SNAPSHOT.jar
```
还可以在启用远程调试支持的情况下运行打包的应用程序。这允许您附加调试器到您的打包应用程序:
```
$ java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n \
-jar target/myproject-0.0.1-SNAPSHOT.jar
```
**7,热插拔 Hot swapping**
