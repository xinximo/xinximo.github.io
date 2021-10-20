---
title: 'kafka安装(for Mac)并实例生产者消费者,且打包jar文件服务器运行'
date: 2021-10-11 16:26:29
tags: [kafka, Mac]
top: true
---
# kafka安装for Mac
## 1.brew安装
```
brew install kafka
```
## 2.本地调试kafka
安装完毕后,生成安装路径和配置路径:
```
 /usr/local/Cellar/kafka/2.7.0/bin
 /usr/local/etc/kafka/
```
启动zookeeper依赖:
```
cd /usr/local/Cellar/kafka/2.7.0/bin

zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties
```
启动kafka:
```
cd /usr/local/Cellar/kafka/2.7.0/bin

kafka-server-start /usr/local/etc/kafka/server.properties
```
<!--more-->
创建一个topic:
```
cd /usr/local/Cellar/kafka/2.7.0/bin

kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```
```
--create #创建主题命令
--zookeeper localhost:2181 #指定zookeeper
--replication-factor 1 #指定副本个数
--partitions 1 #指定分区个数
--topic test #主题名称为test
```
查看topic:
```
kafka-topics --list --zookeeper localhost:2181
```
创建一个生产者:
```
kafka-console-producer --broker-list localhost:9092 --topic test
```
创建一个消费者:
```
kafka-console-consumer --bootstrap-server localhost:9092 --topic test --from-beginning
```
生产者发送消息即可在消费者接受消息
## 3.连接远程服务器kafka接收消息
修改Kafka配置文件以下三个参数,listeners和host.name写上kafka broker主机的地址,这个地址不配置会造成远程无法访问:
```
vim /usr/local/etc/kafka/server.properties

zookeeper.connect=localhost:2181
listeners=PLAINTEXT://远程服务器IP:9092
host.name=远程服务器IP
```
启动消费者接收消息:topic为remote
```
kafka-console-consumer --bootstrap-server 远程服务器IP:9092 --topic remote
```

# 实例生产者消费者
## 1.配置项目文件pom.xml安装依赖
```
<dependencies>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>2.7.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_2.12</artifactId>
            <version>2.7.0</version>
        </dependency>
</dependencies>
```
在pom.xml文件dependencies中加入以上kafka依赖并在项目命令行中执行以下命令安装:
```
mvn clean install
```
## 2.IDEA中实例生产者代码
```
package kafka;

import java.util.*;
import java.util.Properties;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;

public class SendMessages {
    //发送消息的topic,此处注意该topic需要在服务器已被创建
    public static final String WebhookChannelTopic = "test-webhook";
    private final KafkaProducer<String, String> producer;
   
    public SendMessages() {
        Properties props = new Properties();
        //指定代理服务器的地址,直接部署在对应的服务器所以直接本地访问
        props.put("bootstrap.servers", "localhost:9092");
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        producer = new KafkaProducer<String, String>(props);

    }
    public void produce() {
    	//指定发送100W条数据
        int count = 1000000;
        ExecutorService executor = Executors.newFixedThreadPool(4);
        for (int i = 0; i < count; i++) {
            final String key = String.valueOf(i);
            final String kafkaMsg = "test-webhook-" + i;
            executor.submit(new Runnable() {

                @Override
                public void run() {
                    producer.send(new ProducerRecord<String, String>(WebhookChannelTopic, kafkaMsg));
                }
            });

        }
    }
    public static void main(String[] args) {
        SendMessages a = new SendMessages();
        a.produce();
        System.out.println("success");
    }
}
```

# 打包jar文件在服务器运行程序
## 1.开始打包(方法一)
![开始打包](https://img-blog.csdnimg.cn/2021040611514776.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpbnhpbW8=,size_16,color_FFFFFF,t_70)
## 2.生成配置文件
选择主类运行文件并生成配置文件MANIFEST.MF在项目的src路径下:
![生成配置文件](https://img-blog.csdnimg.cn/2021040611555273.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpbnhpbW8=,size_16,color_FFFFFF,t_70)
![jar文件](https://img-blog.csdnimg.cn/2021040611584255.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpbnhpbW8=,size_16,color_FFFFFF,t_70)
## 3.生成jar文件
构建项目并打包生成jar文件:
![构建项目](https://img-blog.csdnimg.cn/20210406134959868.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpbnhpbW8=,size_16,color_FFFFFF,t_70)
![生成jar文件](https://img-blog.csdnimg.cn/20210406135135476.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpbnhpbW8=,size_16,color_FFFFFF,t_70)
![生成成功](https://img-blog.csdnimg.cn/20210406135307499.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpbnhpbW8=,size_16,color_FFFFFF,t_70)
## 4.利用org.apache.maven.plugins插件打包(方法二)
在项目的pom.xml文件中```<project></project>```内插入以下代码(mainClass中填入自己运行类的包名和文件名):
![包名](https://img-blog.csdnimg.cn/20210408142713759.png)
```
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.1.0</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>kafka.SendMessages</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```
执行打包命令,打包成功后会生成jar文件在项目的target目录下:
```
mvn clean package
```

## 5.scp到服务器运行
cd到jar文件所在目录执行scp命令:
```
scp xxx.jar root@xxx.xx.xx.xx:~
```
运行程序:
```
java -jar xxx.jar
```

## 6.测试消息发送成功
可以在服务器起一个本地消费者:
```
# cd到服务器kafka的执行目录,具体步骤参考本地测试部分

./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-webhook --from-beginning
```
