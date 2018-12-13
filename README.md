# spring-boot-actuator
使用Actuator监控应用程序

------------------------------------------------------------------------------------------------------------------------
# 探究Actuator的默认开放节点&详细健康状态：
统的监控在分布式的设计中显得尤为重要，因为分开部署的缘故，并不能及时的了解到程序运行的实时状况，所以SpringBoot也给我提供
了一套自动监控的API，可以无缝整合spring-boot-admin来完成图形化的展示，先来熟悉下actuator系统监控相关内容。

## 学习目标：通过spring-boot-actuator完成系统运行监控，实时了解程序运行的环境是否健康

## 构建项目
    <!--Web-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--Actuator-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

## 配置绑定映射类
有关开放节点的配置都被映射到属性配置类org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointProperties中，
通过@ConfigurationProperties注解自动映射application.yml配置文件内以前缀management.endpoints.web的属性配置信息。

## 默认开放的节点
Actuator默认开放了两个节点信息，分别是：
    health：健康监测节点
    健康节点我们在访问时默认只可以查看当前系统的运行状态，如下所示：
        {
            "status": "UP"
        }
    如果不开放相关的配置无法查看详细的运行健康信息，比如：硬盘等，具体的开放方法在本章查看详细健康状态

    info：基本信息查看节点

我们在属性类WebEndpointProperties内也并没有看到health、info作为初始化的值赋值给exposure.include，那么是怎么进行赋值的呢？

### 元数据配置文件
spring-configuration-metadata.json(元数据配置文件)位于spring-boot-actuator-autoconfigure-2.1.1.RELEASE.jar依赖的META-INF目录下，
主要作用是对应WebEndpointProperties等属性配置类的字段类型、描述、默认值、对应目标字段的定义，找到name为
management.endpoints.web.exposure.include的配置如下：
    .....
    {
      "sourceType": "org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointProperties$Exposure",
      "defaultValue": [
        "health",
        "info"
      ],
      "name": "management.endpoints.web.exposure.include",
      "description": "Endpoint IDs that should be included or '*' for all.",
      "type": "java.util.Set<java.lang.String>"
    },
    .....
通过配置的defaultValue来自动映射到WebEndpointProperties属性配置类的Exposure#include字段，下面简单介绍上面的字段：
    sourceType：该配置字段所关联配置类的类型（全限定名）
    defaultValue：默认值，根据type来设置，如上java.util.Set<java.lang.String>类型的默认值就可以通过["health","info"]设置
    name：字段的名称，对应配置类内的field名称
    description：该配置字段的描述信息，可以是中文，填写后idea工具会自动识别并提示
    type：该字段的类型的全限定名，如：java.lang.String

## 查看详细健康状态
开启查看详细健康状态比较简单，通过配置参数management.endpoint.health.show-details来进行修改，该参数的值由
org.springframework.boot.actuate.health.ShowDetails枚举提供配置值，ShowDetails源码如下所示：
    /**
     * Options for showing details in responses from the {@link HealthEndpoint} web
     * extensions.
     *
     * @author Andy Wilkinson
     * @since 2.0.0
     */
    public enum ShowDetails {

    	/**
    	 * Never show details in the response.
    	 */
    	NEVER,

    	/**
    	 * Show details in the response when accessed by an authorized user.
    	 */
    	WHEN_AUTHORIZED,

    	/**
    	 * Always show details in the response.
    	 */
    	ALWAYS

    }

在spring-configuration-metadata.json元数据文件内，配置的showDetails的默认值为never，也就是不显示详细信息，配置如下所示：
    ````
    {
      "name": "management.endpoint.health.show-details",
      "type": "org.springframework.boot.actuate.health.ShowDetails",
      "description": "When to show full health details.",
      "sourceType": "org.springframework.boot.actuate.autoconfigure.health.HealthEndpointProperties",
      "defaultValue": "never"
    }
    ````

修改management.endpoint.health.show-details参数为always后重新项目再次访问/actuator/health就可以获取如下的信息：
    {
        "status": "UP",
        "details": {
            "diskSpace": {
                "status": "UP",
                "details": {
                    "total": 250790436864,
                    "free": 77088698368,
                    "threshold": 10485760
                }
            }
        }
    }

## 自定义节点访问前缀
默认地址：http://localhost:8085/actuator
actuator默认的所有节点的访问前缀都是/actuator，在application.yml配置文件内设置management.endpoints.web.basePath参数进行修改，如下所示：
    # 管理节点配置
    management:
      endpoints:
        web:
          # actuator的前缀地址
          base-path: /
basePath字段位于WebEndpointProperties属性配置类内，修改完成重启项目就可以使用修改后的路径进行访问，上述直接映射到了/下。
配置根路径为“/”之后不能访问根目录

## 总结
spring-boot-actuator默认开放了health、info两个节点，通过配置健康检查详细信息可以查看硬盘相关的运行健康状态。