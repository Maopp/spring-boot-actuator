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


------------------------------------------------------------------------------------------------------------------------
# 了解Actuator开放指定监控节点：

## 学习目标：开放spring-boot-actuator的指定节点访问。

## 开放指定节点
management.endpoints.web.exposure.include的配置字段我们已经了解到了在
org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointProperties属性配置类内，而且exposure.include的值默认为["health","info"]。
除此之外通过spring-configuration-metadata.json元数据配置文件内还知道了management.endpoints.web.exposure.include配置参数
的类型为java.util.Set<java.lang.String>，这样我们就知道了该如何进行修改配置了，修改application.yml配置文件如下所示：
    # 管理节点配置
    management:
      endpoints:
        web:
          # actuator的前缀地址
          base-path: /
          # 开放指定节点
          exposure:
            include:
              - health
              - info
              - mappings
              - env
由于management.endpoints.web.exposure.include是java.util.Set<java.lang.String>类型，那么我就可以采用中横线换行形式进行配
置(这是SpringBoot采用yaml配置文件风格的约定)，一个"- xxx"代表一个配置的值。
当然我们采用数组的形式也是可以的，如下所示：
    # 管理节点配置
    management:
      endpoints:
        web:
          # actuator的前缀地址
          base-path: /
          # 开放指定节点
          exposure:
            include: ["health","info","mappings","env"]


## 开放全部节点
如果不做节点的开放限制，可以将management.endpoints.web.exposure.include配置为*，那么这样就可以开放全部的对外监控的节点，如下所示：
    # 管理节点配置
    management:
      endpoints:
        web:
          # actuator的前缀地址
          base-path: /
          # 开放指定节点
          exposure:
            include: "*"

## 内置节点列表
开放全部节点后在项目启动时，控制台会打印已经映射的路径列表，spring-boot-actuator内置了丰富的常用监控节点，详见如下表格：
节点	        节点描述	                                                                                是否默认启用
auditevents	    公开当前应用程序的审核事件信息。	                                                        是
beans	        显示应用程序中所有Spring bean的完整列表。	                                                是
conditions	    显示在配置和自动配置类上评估的条件以及它们匹配或不匹配的原因。	                            是
configprops	    显示所有的整理列表@ConfigurationProperties。	                                            是
env	            露出Spring的属性ConfigurableEnvironment。	                                                是
health	        显示应用健康信息。	                                                                        是
httptrace	    显示HTTP跟踪信息（默认情况下，最后100个HTTP请求 - 响应交换）。	                            是
info	        显示任意应用信息。	                                                                        是
loggers	        显示和修改应用程序中记录器的配置。	                                                        是
metrics	        显示当前应用程序的“指标”信息。	                                                        是
mappings	    显示所有@RequestMapping路径的整理列表。	                                                    是
scheduledtasks	显示应用程序中的计划任务。	                                                                是
shutdown	    允许应用程序正常关闭。	                                                                    否
threaddump	    执行线程转储。	                                                                            是
sessions	    允许从Spring Session支持的会话存储中检索和删除用户会话。使用Spring Session对响应式Web应用程序的支持时不可用。	是

## 总结
通过本章学习可以知道你需要开发的节点了，根据具体的业务需求开放不同的节点。
注意：节点开放一定要慎重，不要过度开放接口，也不要盲目填写*开放全部节点。