---
title: "Java Tomcat在IDEA上的部署过程和解决乱码问题"
date: 2021-03-21
author: pp
---

### IDEA集成Tomcat服务器

**第一步**

![image-20210321143805901](2021-03-21-ljp-java-tomcat-idea-deploy.assets\image-20210321143805901.png)

**第二步**

![image-20210321143937853](2021-03-21-ljp-java-tomcat-idea-deploy.assets\image-20210321143937853.png)

**第三步**

![image-20210321144246719](2021-03-21-ljp-java-tomcat-idea-deploy.assets\image-20210321144246719.png)



### Tomcat日志乱码问题

![image-20210321144746615](2021-03-21-ljp-java-tomcat-idea-deploy.assets\image-20210321144746615.png)

这是因为Tomcat默认UTF-8，而Windows cmd默认gbk。需要修改Tomcat配置文件。

在tomcat-9.0.29/conf/logging.properties中，修改ConsoleHandler.encoding如下。

![image-20210321144938135](2021-03-21-ljp-java-tomcat-idea-deploy.assets\image-20210321144938135.png)

