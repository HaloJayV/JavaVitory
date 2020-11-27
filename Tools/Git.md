[TOC]

token： c94cf08881c590a8d7be84d23f5134a1645b5ca5





# Gitee

第一步 ：在本地新建一个文件夹 例如 giteeMy，在这个文件夹 里  右键Git bash here进入控制面板

第二步：选中文件件右键Git bash here进入控制面板，输入命令git init 初始化化文件夹，把这个文件夹变成Git可管理的仓库

第三步：继续输入命令并加入复制的地址->git clone "你的仓库地址"，点击回车。

第四步 弹出用户名和密码 输入即可 最后等待下载 下载完就可以在你的本地看到下载内容

wget https://github.com/alibaba/canal/releases/download/canal-1.1.4/canal.deployer-1.1.4.tar.gz









\1. 准备 Git 仓库

码云：https://gitee.com

## 1.1. 通过网站右上角的「+」号，选择「新建仓库」，进入新建仓库页面

![IMG_256](../../../../Software/Typora/Picture/clip_image002d23d936a-91fe-49c8-8b5d-b8d7001466f1.gif)

## 1.2. 新建仓库

![img](../../../../Software/Typora/Picture/clip_image004607f1b16-7e62-4131-87f5-7f4abc17dc5e.jpg)

## 1.3. 打开项目并点击菜单栏上的【CVS】--》【Import into version control】--》【Create Git Repository】创建本地仓库

![img](../../../../Software/Typora/Picture/clip_image008c71e3ed3-a671-4c65-a95d-9a880746b1ea.jpg)

## 1.4. 在打开的【Create Git Repository】对话框内选择本地仓库的位置，这里我选择项目的根目录。

 

![img](../../../../Software/Typora/Picture/clip_image010d500df72-c9c1-4b2a-9283-5dc2bb30dc7c.jpg)

## 1.5. 右击项目点击【Git】--》【Add】，接着点击【Git】--》【Commit Directory】在打开的窗口中选择要上传到本地仓库的代码并添加注释后提交到本地仓库内。

![img](../../../../Software/Typora/Picture/clip_image01284e30440-d2e8-42b8-9c4f-ee11ea16a5f8.jpg)

## 1.6. 右击项目点击【Git】--》【Repository】--》【Remotes...】。在打开的【Git Remotes】窗口中添加码云的远程仓库。码云的远程仓库地址可以在码云仓库内找到。

![img](../../../../Software/Typora/Picture/clip_image0146603b61c-5446-4d8d-aa4e-8c63025afdda.jpg)

![img](../../../../Software/Typora/Picture/clip_image01648cd4d21-1c9b-4d09-98c7-7ad8ff18bec2.jpg)

![img](../../../../Software/Typora/Picture/clip_image018728b2f64-1e62-41ec-9f93-9230a17f08e9.jpg)

 

## 1.7. 点击【OK】后接着输入码云的账号密码。

![img](../../../../Software/Typora/Picture/clip_image020674fbf44-f6c5-40c8-9d3e-a86d3ac90c08.jpg)

## 1.8. 上传代码到码云，右击项目点击【Git】--》【Repository】--》【Push...】在打开的【Push commits】内可以看到已提交到本地仓库的提交信息。点击【Push】按钮将本地仓库的代码上传到码云上，上传成功后就可以在码云上看到啦。









**1. 准备代码，提交到码云Git库**

**代码中需要包含以下几部分内容：**

**（1）代码中需要包含Dockerfile文件**



![img](../../../../Software/Typora/Picture/646e8872-4825-4932-af9c-400d5fa06897.png)

**文件内容：**

```
FROM openjdk:8-jdk-alpine
VOLUME /tmp
COPY ./target/demojenkins.jar demojenkins.jar
ENTRYPOINT ["java","-jar","/demojenkins.jar", "&"]
```



**（2）在项目pom文件中指定打包类型，包含build部分内容**

![img](../../../../Software/Typora/Picture/a9238d85-0e09-4a61-a354-877a0c635e6e.png)

![img](../../../../Software/Typora/Picture/a6f001d3-1c8e-415b-aec7-eec5c35a0fe3.png)



# 2. 安装JAVA 运行环境

第一步：上传或下载安装包

cd/usr/local

jdk-8u121-linux-x64.tar.gz

 

第二步：解压安装包

tar -zxvf jdk-8u121-linux-x64.tar.gz

 

第三步：建立软连接

ln -s /usr/local/jdk1.8.0_121/ /usr/local/jdk

 

第四步：修改环境变量

vim /etc/profile

export JAVA_HOME=/usr/local/jdk

export JRE_HOME=$JAVA_HOME/jre

export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib

export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin

通过命令source /etc/profile让profile文件立即生效

source /etc/profile

 

第五步、测试是否安装成功

②、使用java -version，出现版本

 

# 3. 安装maven

第一步：上传或下载安装包

cd/usr/local

apache-maven-3.6.1-bin.tar.gz

 

第二步：解压安装包

tar -zxvf apache-maven-3.6.1-bin.tar.gz

 

第三步：建立软连接

ln -s /usr/local/apache-maven-3.6.1/ /usr/local/maven

 

第四步：修改环境变量

vim /etc/profile

export MAVEN_HOME=/usr/local/maven

export PATH=$PATH:$MAVEN_HOME/bin

通过命令source /etc/profile让profile文件立即生效

source /etc/profile

 

第五步、测试是否安装成功

mvn –v

 

# 4. 安装git

yum -y install git

# 5. 安装docker

参考文档：

https://help.aliyun.com/document_detail/60742.html?spm=a2c4g.11174283.6.548.24c14541ssYFIZ

 

第一步：安装必要的一些系统工具

yum install -y yum-utils device-mapper-persistent-data lvm2

 

第二步：添加软件源信息

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

 

第三步：更新并安装Docker-CE

yum makecache fast

yum -y install docker-ce

 

第四步：开启Docker服务

service docker start

 

第五步、测试是否安装成功

docker -v

# 6. 安装Jenkins

第一步：上传或下载安装包

cd/usr/local/jenkins

jenkins.war

 

第二步：启动

nohup java -jar /usr/local/jenkins/jenkins.war >/usr/local/jenkins/jenkins.out &

 

第二步：访问

http://ip:8080

 

# 7. 初始化 Jenkins 插件和管理员用户

## 7.1访问jenkins

http://ip:8080

 

## 7.2解锁jenkins

获取管理员密码

![img](../../../../Software/Typora/Picture/clip_image0229d3bd13f-f015-47a5-9cd1-e480f73bbed4.jpg)

 

![img](../../../../Software/Typora/Picture/clip_image0240a50a057-c856-4cc7-84bf-e661a06241eb.jpg)

![img](../../../../Software/Typora/Picture/clip_image02607409e96-13ea-4d2a-b7df-fc4208025a09.jpg)

 



## 注意：配置国内的镜像

官方下载插件慢 更新下载地址

cd {你的Jenkins工作目录}/updates #进入更新配置位置

sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json && sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json

这是直接修改的配置文件，如果前边Jenkins用sudo启动的话，那么这里的两个sed前均需要加上sudo

重启Jenkins，安装插件



## 7.3选择“继续”

![img](../../../../Software/Typora/Picture/clip_image028b17121a0-8176-4112-8889-98a8c733ab96.jpg)

 

## 7.4选择“安装推荐插件”

![img](../../../../Software/Typora/Picture/clip_image03043b21963-7524-4fd5-946b-6677ebc63a3b.jpg)

 

## 7.5插件安装完成，创建管理员用户

![img](../../../../Software/Typora/Picture/clip_image032099e16d1-a4e3-4b82-9899-8c0a824e60b9.jpg)

## 7.6保存并完成

![img](../../../../Software/Typora/Picture/clip_image034f0507710-95d3-4c5b-81c5-f4503c102c9d.jpg)

## 7.7进入完成页面

![IMG_256](../../../../Software/Typora/Picture/clip_image036529b702b-e5ca-44f7-bb1a-34eb9d10339b.gif)

# 8. 配置 Jenkins 构建工具

 

![img](../../../../Software/Typora/Picture/clip_image038b68b06b0-01b0-457f-abec-6f8c2d8acae4.jpg)

 

 

## 8.1全局工具配置

![img](../../../../Software/Typora/Picture/clip_image040a1386eb1-b3db-44d8-b962-7dad4c1b2b22.jpg)

### 8.1.1配置jdk

JAVA_HOME：/usr/local/jdk

![1574673821(../../../../Software/Typora/Picture/clip_image042dd40c860-0d6a-430d-b6ec-0095a04089db.gif)](../../../../Software/Typora/Picture/clip_image042dd40c860-0d6a-430d-b6ec-0095a04089db.gif)

### 8.1.2配置maven

MAVEN_HOME：/usr/local/maven

![img](../../../../Software/Typora/Picture/clip_image04479ae7da4-e7c7-4ec8-8fbd-569318555dce.jpg)

### 8.1.2配置git

查看git安装路径：which git

![img](../../../../Software/Typora/Picture/clip_image046b6ee379c-aafc-4d6c-abf0-7ec5b65857b7.jpg)

# 9. 构建作业

## 9.1点击创建一个新任务，进入创建项目类型选择页面

![img](../../../../Software/Typora/Picture/clip_image04868733363-ef28-4957-aa37-eca37ebe52fb.jpg)

 

填好信息点击“确认”

## 9.2配置“General”

![img](../../../../Software/Typora/Picture/clip_image0500a0bc0cb-7ef0-40c5-917b-b0f8cd652d8d.jpg)

## 9.3配置“源码管理”

填写源码的git地址

![img](../../../../Software/Typora/Picture/clip_image052b5cd2b8b-c46a-4b83-bcd9-d62f1781d531.jpg)

添加git用户，git的用户名与密码

![img](../../../../Software/Typora/Picture/clip_image054986c8515-033a-49ca-a1b0-558689acc83a.jpg)

选择添加的用户，上面的红色提示信息消失，说明连接成功，如下图

![img](../../../../Software/Typora/Picture/clip_image05654e63831-ee2d-4db3-afd2-a72a7683e646.jpg)

## 9.4构建作业

**到源码中找到docker脚本**

选择“执行shell”

![img](../../../../Software/Typora/Picture/clip_image058b6310024-b362-4ea4-a7b0-a9339b95b0a1.jpg)



保存上面的构建作业

![img](../../../../Software/Typora/Picture/clip_image0605c6b906b-2155-407e-89f0-30924768e50c.jpg)



## 9.5构建

构建作业之后，就可以执行构建过程了。

### 9.5.1执行构建过程

![img](../../../../Software/Typora/Picture/clip_image062bd6c535f-a7fb-49cf-b48a-ee92b130e38c.jpg)

### 9.5.2构建结构

第一列是 "上次构建状态显示"，是一个圆形图标，一般分为四种：

![IMG_256](../../../../Software/Typora/Picture/clip_image064e3ce22b3-cc2e-404a-bdad-39f3fd583372.gif)

蓝色：构建成功；

![IMG_257](../../../../Software/Typora/Picture/clip_image06634836aaf-46cb-455a-bc3d-675caed3b7df.gif)

黄色：不确定，可能构建成功，但包含错误；

![IMG_258](../../../../Software/Typora/Picture/clip_image068b897acea-f1bf-4026-8905-89170b186054.gif)

红色：构建失败；

![IMG_259](../../../../Software/Typora/Picture/clip_image07006392f2b-79c3-4a06-a7ac-410e12cc72d2.gif)

灰色：项目从未构建过，或者被禁用；

如上显示蓝色，表示构建成功。

注意：手动触发构建的时间与自动定时构建的时间互不影响。

### 9.5.3查看控制台输出

![img](../../../../Software/Typora/Picture/clip_image0721812a15b-d8f0-40ff-8aad-ec356b7780cb.jpg)

日志内容：

![img](../../../../Software/Typora/Picture/clip_image0747ddb4907-eb94-4574-97c1-72c4e46d4f83.jpg)

![img](../../../../Software/Typora/Picture/clip_image07674be6a28-b272-40ae-9529-46b1196468ef.jpg)







