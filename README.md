# ibm_mq

## Getting Started

### Reference Documentation
For further reference, please consider the following sections:

* [Official Apache Maven documentation](https://maven.apache.org/guides/index.html)
* [Spring Boot Maven Plugin Reference Guide](https://docs.spring.io/spring-boot/docs/2.7.3/maven-plugin/reference/html/)
* [Create an OCI image](https://docs.spring.io/spring-boot/docs/2.7.3/maven-plugin/reference/html/#build-image)

## IBM mq setup steps
https://developer.ibm.com/tutorials/mq-jms-application-development-with-spring-boot/

### 1. Run with docker container
```shell
docker run --env LICENSE=accept --env MQ_QMGR_NAME=QM1 --publish 1414:1414 --publish 9443:9443 --detach ibmcom/mq
```

### 2. application.properties
```properties
ibm.mq.queueManager=QM1
ibm.mq.channel=DEV.ADMIN.SVRCONN
ibm.mq.connName=localhost(1414)
ibm.mq.user=admin
ibm.mq.password=passw0rd
```

### 3. pom.xml
```xml
 <dependency>
         <groupId>com.fasterxml.jackson.core</groupId>
         <artifactId>jackson-databind</artifactId>
     </dependency>
     <dependency>
         <groupId>com.ibm.mq</groupId>
         <artifactId>mq-jms-spring-boot-starter</artifactId>
         <version>2.0.0</version>
 </dependency>
```

### 4. JavaApplicationMain
```java
package com.example.demo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.jms.JmsException;
import org.springframework.jms.annotation.EnableJms;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@EnableJms
@RestController
@SpringBootApplication
public class JavaApplication {
    @Autowired
    private JmsTemplate jmsTemplate;

    public static void main(String[] args) {
        SpringApplication.run(JavaApplication.class, args);
    }

    @PostMapping("send")
    String send(@RequestBody Message message) {
        try {
            jmsTemplate.convertAndSend("DEV.QUEUE.1", message.getMessage());
            return "OK";
        } catch (JmsException ex) {
            ex.printStackTrace();
            return "FAIL";
        }
    }

    @GetMapping("recv")
    String recv() {
        try {
            return jmsTemplate.receiveAndConvert("DEV.QUEUE.1").toString();
        } catch (JmsException ex) {
            ex.printStackTrace();
            return "FAIL";
        }
    }
}
```
