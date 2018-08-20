# ELK
> 找了不少使用 docker-elk 搭建的博客, 英文的阅读吃力不说, 镜像源也是慢的让人头皮发麻, 因此重新编排了一个`docker-compose`,源都是从 https://hub.docker.com/ 上找的, 即使拉的国内镜像源应该也能很好的支持了吧?

### 环境
* Docker          `18.06.0-ce`
* docker-compose  `1.22.0`
给每个容器最少分配 1G 的内存

### 软件版本
* logstash:         `5.*`
* elasticsearch:    `5.*`
* kibana:           `5.*`

### 启动前的配置
在各个目录下都有对应的 config 配置, 根据各自的情况自行处理

拿默认的 logstash/confg/test.conf 中的配置举例:
```bash
input {
      file {
          #这里的路径指的是 logstash 容器中的路径, 外部接入需要使用 volume 进行目录映射 
            path => "/logs/input/*"
      }

        # 在 logstash 容器中的输入
      stdin {}
      
      # 因为做了本地5000端口和容器5000端口进行绑定, 所以可以用 nc 工具测试一下
      # echo "Test Logstash TCP Input Plugin" | nc localhost 5000
      tcp {
            type => "tcp"
            port => 5000
            mode => "server"
      }
}

output {
      file {
          #这里的路径指的是 logstash 容器中的路径, 外部接入需要使用 volume 进行目录映射
            path => "/logs/output/%{+yyyy-MM-dd-HH}/%{host}.log"
      }
      stdout {
            codec => rubydebug
      }
      elasticsearch {
	    hosts => "elasticsearch:9200"
        # 这里设置的 index 在 kibana 中会用到
            index => "file-log-%{+YYYY.MM}"  
	}
}
```


### 启动容器
执行
```
git clone https://github.com/gaopengfei123123/docker-elk.git && cd docker-elk
docker-compose up -d --build
```
等一会看到执行成功的提示
```
Creating docker-elk_elasticsearch_1 ... done
Creating docker-elk_logstash_1      ... done
Creating docker-elk_kibana_1        ... done
```

在本地浏览器输入 `http://localhost:5601/` 进入 kibana 界面

**注意**, 第一次启动时有可能会出现提示 `elasticsearch not found` 这类的问题, 可以先等个一两分钟刷新一下就好了, 如果还是不行就谷歌或者提 issue 解决一下


同目录下输入
```bash
docker-compose stop
```
则停止所有服务

### 测试一下
在 `logs/input/` 目录下新增个 test.log 文件, 然后输入点东西验证一下, 或者命令行执行`echo "Test Logstash TCP Input Plugin" | nc localhost 5000` 通过 tcp 发送日志

```
docker-compose logs -f
```
查看各容器日志输出


### TODO

* 引入 kafka 做缓冲 
* 搭建 es 集群