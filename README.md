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


------------------------------------------------------------------------------------------------------------------------
# Actuator远程关闭服务"黑科技"：

## 学习目标：通过配置Actuator完成服务远程关闭。

## 配置远程关闭服务
由于Autuator内置了远程关闭服务功能，所以可以很简单的开启这一项“黑科技”,修改application.yml配置文件，如下所示：
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
              - shutdown
        # 开启远程关闭服务
        shutdown:
          enabled: true
通过management.endpoint.shutdown.enabled参数来进行设置，默认为false，默认不会开启远程关闭服务功能，然后把shutdown节点进
行开放，否则无法发送远程关机请求。

## 测试：
打开终端或者postman工具进行测试关机请求，如下是终端命令测试结果：
    curl -X POST http://localhost:8085/shutdown
通过curl命令发送POST请求类型到http://localhost:8085/shutdown，发送完成后会响应一段信息：
    {"message":"Shutting down, bye..."}

## 总结
本章配置比较简单，通过修改两个地方开启了远程关闭服务的操作。
不过建议没事不要打开，打开后也不要对公网开放，黑科技都是比较危险的。
spring-configuration-metadata.json文件中对应属性名成为：endpoints.shutdown.enabled，但是这样配置访问不到，要配置：
management.endpoint.shutdown.enabled属性才可以访问


------------------------------------------------------------------------------------------------------------------------
# Actuator自定义节点路径&监控服务自定义配置：
既然Actuator给我们内置提供了节点映射，我们为什么还需要进行修改呢？
正因为如此我们才需要进行修改！！！
路径都是一样的，很容易就会暴露出去，导致信息泄露，发生一些无法估计的事情，如果我们可以自定义节点的映射路径或者自定义监控
服务的管理信息，这样就不会轻易的暴露出去，Actuator已经为我们提供了对应的方法来解决这个问题

## 学习目标：自定义Actuator节点映射路径、监控服务配置信息等，提高监控服务的安全性

## 自定义监控节点映射路径
Actuator为我们提供了org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointProperties类，用于配置监控管
理web端的信息，映射路径也在该配置类中，通过修改management.endpoints.web.path-mapping配置来修改指定的节点映射路径，如下所示：
    # 管理节点配置
    management:
      endpoints:
        web:
          # 自定义路径映射
          path-mapping:
            # key=>value形式，原映射路径=>新映射路径
            health: healthcheck

## 自定义监控端口号
默认Actuator监控服务的端口号跟项目端口号一致，本项目的端口号为8085，所以通过http://localhost:8085前缀就可以访问到监
控节点，那该怎么修改监控节点端口号呢？
### ManagementServerProperties配置类
Acutator内置了配置类org.springframework.boot.actuate.autoconfigure.web.server.ManagementServerProperties来进行自定义设置
监控服务基本信息，该配置类内包含了端口号(port)、服务地址(address)、安全SSL（ssl）等。

通过修改management.server.port进行自定义监控端口号，如下所示：
    # 管理节点配置
    management:
      # 修改监控服务端的端口号
      server:
        port: 8081

## 总结
自定义Actuator开放的监控节点的映射路径，通过修改management.server.port参数进行自定义监控服务的管理端口号。