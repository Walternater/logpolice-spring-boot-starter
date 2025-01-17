# 日志异常消息通知的spring-boot-start框架：logpolice-spring-boot-starter

## 注意：
此版本为<动态加载>报警配置版本，若不需要，可切换master分支，支持application.properties

## 背景：

对于项目工程来说，bug是不可能避免的。生产环境并不能像本地环境一样方便调试，无法第一时间知道线上事故。所以我们就需要项目的异常通知功能，在用户发现bug之前，开发者自可以提前发现问题点，避免不必要的线线上事故。

如果捕获项目全局异常，部分异常不是我们想关注的，这时候可以考虑基于日志的log.error()主动触发异常提示开发者，并精确获取异常堆栈信息，在获取异常消息推送的避免消息轰炸，可以根据自定义配置决定日志推送策略。

本项目基于以上需求并 结合DDD领域开发设计而成，简单接入消息报功能

## 功能
1. 监听log.error()，异步推送堆栈信息，快速接入
2. 提供推送策略，避免消息轰炸（超频/超时）
3. 提供本地，redis 数据存储，按需配置（默认本地）
4. 提供钉钉，邮件 推送类型，按需配置（默认钉钉）
5. 提供异常过滤白名单
6. 此版本提供报警属性动态配置

## 系统需求

![jdk版本](https://img.shields.io/badge/java-1.8%2B-red.svg?style=for-the-badge&logo=appveyor)
![maven版本](https://img.shields.io/badge/maven-3.2.5%2B-red.svg?style=for-the-badge&logo=appveyor)
![spring boot](https://img.shields.io/badge/spring%20boot-2.0.3.RELEASE%2B-red.svg?style=for-the-badge&logo=appveyor)

## 当前版本

![目前工程版本](https://img.shields.io/badge/version-1.0.0%20acm%2B-red.svg?style=for-the-badge&logo=appveyor)


## 快速接入(默认本地缓存&钉钉推送)
（默认钉钉推送，本地缓存。有需求可以更改配置，邮箱或redis异常存储）
1. 工程``mvn clean install``打包本地仓库。
2. 在引用工程中的``pom.xml``中做如下依赖
```
    <dependency>
        <groupId>com.logpolice</groupId>
        <artifactId>logpolice-spring-boot-starter</artifactId>
        <version>1.3.0-hot</version>
    </dependency>
    
```
3. 项目新增类文件，实现LogpoliceProperties, LogpoliceDingDingProperties：
```
    @Component
    public class LogpoliceAcmProperties implements LogpoliceProperties, LogpoliceDingDingProperties {
    
        @Override
        public String getAppCode() {
            return "工程名";
        }
    
        @Override
        public String getLocalIp() {
            return "工程地址";
        }
    
        @Override
        public Boolean getEnabled() {
            return true;
        }
    
        @Override
        public Long getTimeInterval() {
            return 60 * 5;
        }
    
        @Override
        public Boolean getEnableRedisStorage() {
            return true;
        }
    
        @Override
        public NoticeSendEnum getNoticeSendType() {
            return NoticeSendEnum.DING_DING;
        }
    
        @Override
        public Set<String> getExceptionWhiteList() {
            return new HashSet<>();
        }
    
        @Override
        public Set<String> getClassWhiteList() {
            return new HashSet<>();
        }
    
    }
```
4. 钉钉配置：[钉钉机器人](https://open-doc.dingtalk.com/microapp/serverapi2/krgddi "自定义机器人")
5. 以上配置好以后就可以写demo测试啦，首先创建logback.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <!-- encoder 默认配置为PatternLayoutEncoder -->
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="LogDingDingAppender" class="com.logpolice.port.LogSendAppender"/>

    <root level="ERROR">
        <appender-ref ref="LogDingDingAppender"/>
    </root>
</configuration>
```
注：如果已有logback.xml，引用LogSendAppender类即可
```
    <appender name="LogDingDingAppender" class="com.logpolice.port.LogSendAppender"/>

    <root level="ERROR">
        <appender-ref ref="LogDingDingAppender"/>
    </root>
```
6. 然后编写测试类，需要主动打印exception堆栈信息，否则日志获取不到：
```
    @RunWith(SpringRunner.class)
    @SpringBootTest(classes = DemoApplication.class)
    public class DemoApplicationTest1s {
    
        private Logger log = LoggerFactory.getLogger(DemoApplicationTest1s.class);
    
        @Test
        public void test1() {
            try {
                int i = 1 / 0;
            } catch (Exception e) {
                log.error("哈哈哈哈，param:{}, error:{}", 1, e);
            }
        }
    
    }
```
log.error()写入异常，推送效果（钉钉/邮箱）

![效果](/src/main/resources/wx_20190916162148.png)
![效果](/src/main/resources/wx_20190916162204.png)
![效果](/src/main/resources/wx_20190916194724.png)

log.error()未写入异常，推送效果（钉钉/邮箱）

![效果](/src/main/resources/wx_20190916163218.png)
![效果](/src/main/resources/wx_20190916194628.png)

## 全部配置
1. 通用配置（*必填）, 需实现：LogpoliceProperties接口
```
     * 1. 工程名          
     * 2. 工程地址
       3. 日志报警开关 （默认：关闭）
       4. 日志报警清除时间 （默认：21600秒 = 6小時）                 
       5. 通知频率类型：超时/超频 （默认：超时）
       6. 超时间隔时间 （默认：300秒 = 5分钟）
       7. 超频间隔次数 （默认：10次）
       8. 消息推送类型：钉钉/邮件 （默认：钉钉）
       9. redis配置开关 （默认：关闭）
       10. redis缓存key （默认：logpolice_exception_statistic:）
       11. 异常白名单 （默认：无)
       12. 类文件白名单 （默认：无)
       13. 日志模板（默认：无）
 ```
 
 2. 钉钉配置（若接入，*必填）, 需实现：LogpoliceDingDingProperties接口
```
     * 钉钉webhook
       被@人的手机号（默认：无)
       此消息类型为固定text（默认：text)
       所有人@时：true，否则为false（默认：false)
```
 3. 邮箱配置（若接入，*必填）, 需实现：LogpoliceMailProperties接口
```
       #报警配置，接口实现
     * 发件人 （应配置spring邮箱配置一致）
     * 收件人  
       抄送 （默认：无)
       密抄送 （默认：无)

       #application.properties spring邮箱配置
     * spring.mail.host=smtp.qq.com
     * spring.mail.username=发送者@qq.com
     * spring.mail.password=xxxxxxxx
     * spring.mail.default-encoding=UTF-8
     * spring.mail.properties.mail.smtp.ssl.enable=true
     * spring.mail.properties.mail.imap.ssl.socketFactory.fallback=false
     * spring.mail.properties.mail.smtp.ssl.socketFactory.class=com.fintech.modules.base.util.mail.MailSSLSocketFactory
     * spring.mail.properties.mail.smtp.auth=true
     * spring.mail.properties.mail.smtp.starttls.enable=true
     * spring.mail.properties.mail.smtp.starttls.required=true
```

 4. 飞书配置（若接入，*必填）, 需实现：LogpoliceFeiShuProperties接口
```
     * 飞书webhook
```


## 常用配置
1. 推送类型（钉钉/邮件，默认钉钉）
```
        @Override
        public NoticeSendEnum getNoticeSendType() {
            return NoticeSendEnum.DING_DING;
        }
```

2. 推送策略（超时时间/超频次数，默认超时）
```
        @Override
        public NoticeFrequencyType getFrequencyType() {
            return NoticeFrequencyType.SHOW_COUNT;
        }
    
        @Override
        public Long getTimeInterval() {
            return 30;
        }
```
```
        @Override
        public NoticeFrequencyType getFrequencyType() {
            return NoticeFrequencyType.TIMEOUT;
        }
    
        @Override
        public Long getShowCount() {
            return 10;
        }
```

3. 日志数据重置时间，异常白名单
```
        @Override
        public Long getCleanTimeInterval() {
            return 10800;
        }
    
        @Override
        public Set<String> getExceptionWhiteList() {
            return Arrays.stream("java.lang.ArithmeticException".split(",")).collect(Collectors.toSet());
        }
```


## redis接入
1. 修改application.properties 异常redis开关
```
        @Override
        public Boolean getEnableRedisStorage() {
            return true;
        }
    
        @Override
        public String getExceptionRedisKey() {
            return xxx_xxxx_xxxx;
        }
```
2. 需要引入spring-boot-starter-data-redis
```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
```
3. application.properties 新增redis配置
 ```
     spring.redis.database=0
     spring.redis.host=xx.xx.xx.xxx
     spring.redis.port=6379
     spring.redis.password=xxxx
 ```


## 邮件接入
1. 有邮件通知的话需要在``pom.xml``中加入如下依赖
```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-mail</artifactId>
    </dependency>
```
2. 实现LogpoliceMailProperties
```
    public class LogpoliceAcmProperties implements LogpoliceProperties,LogpoliceMailProperties {
        
        ......省略常用配置
        
        @Override
        public String getFrom() {
            return "发送者@qq.com";
        }
    
        @Override
        public String[] getTo() {
            return new String["收件人@qq.com"];
        }
    
        @Override
        public String[] getCc() {
            return new String[0];
        }
    
        @Override
        public String[] getBcc() {
            return new String[0];
        }
    }
```

3. application.properties 新增邮件配置，(163，qq 不同邮箱配置可能有差异)
```
    spring.mail.host=smtp.qq.com
    spring.mail.username=发送者@qq.com
    spring.mail.password=xxxxxxxx
    spring.mail.default-encoding=UTF-8
    spring.mail.properties.mail.smtp.ssl.enable=true
    spring.mail.properties.mail.imap.ssl.socketFactory.fallback=false
    spring.mail.properties.mail.smtp.ssl.socketFactory.class=com.fintech.modules.base.util.mail.MailSSLSocketFactory
    spring.mail.properties.mail.smtp.auth=true
    spring.mail.properties.mail.smtp.starttls.enable=true
    spring.mail.properties.mail.smtp.starttls.required=true
```

注：有任何好的建议可以联系 qq:379198812，感谢支持
