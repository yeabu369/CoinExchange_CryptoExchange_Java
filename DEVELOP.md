# Local Development Instructions

This document is suitable for individuals with a certain foundation, requiring a basis in SpringCloud/SpringBoot development, Vue/Npm front-end development, MySQL, MongoDB, Redis, and Linux. Generally speaking, most Java programmers possess these abilities.

Additionally, if you need to connect to a blockchain and perform operations/data retrieval on blockchain wallets, you will also need to have some basic knowledge related to blockchain, including but not limited to: setting up nodes for Bitcoin/Ethereum, accessing blockchain nodes via RPC, Bitcoin wallet basics, Ethereum wallet basics, etc. Generally speaking, you do not need to be an expert in the operation of blockchain, but you should have a certain understanding of the principles of blockchain operation.

This project is separated into front and back ends. If you are a full-stack programmer, you should be able to debug it quickly.

If you have any questions, you can add QQ: 877070886, and I will provide some technical assistance (please avoid asking basic questions).

## About Framework Development

The projects in the 00_Framework folder are a collection of all services, developed using the SpringCloud microservice development model. You can open the entire project in Eclipse, and my development tool version is as follows:

- Eclipse Java EE IDE for Web Developers.
- Version: Photon Release (4.8.0)
- Build id: 20180619-1200

First, install the Lombok plugin for your development tool.

After opening the project in Eclipse, your project directory should look like the following image:
![Development Interface](https://images.gitee.com/uploads/images/2020/0324/104050_1d27dea9_2182501.png "QQ screenshot 20200324104038.png")

When compiling the project, please first compile the core and xxx-core projects, otherwise, you will be prompted for missing classes (this project uses JPA QueryDsl to implement table operations).

Each project has an independent service port (configured in the configuration file), and you can directly call the service through (IP: port), or call the service through the gateway (Cloud project, 7000 port), or call the service through Nginx reverse proxy (this method is used in production environments).

## About Backend Management Web Development

The configuration file is in src/config/api.js, where you can configure the IP and port of your Admin service. Since the backend management web only calls one Admin service, you can directly specify the IP: port of the Admin service.

You also need to modify the access configuration in the src/service/http.js file.

Startup method: You can hot start the project with npm run dev, and compile the deployment file with npm run build.

## About Frontend Web Development

The frontend Web (PC) needs to access different services, such as personal information that needs to call services provided by Ucenter, and transactions that need to call services provided by Exchange-api. Therefore, you need to provide services to the frontend through the gateway.
The configuration file is in src/main.js, where you can implement calls to backend services by modifying Vue.prototype.rootHost and Vue.prototype.host.

## About Database (MySQL)
The database files are in the 00_Framework/sql folder, providing db_patch, which only provides some basic data (menu tree, administrator permissions, etc.). Other database tables will be automatically updated to the database when the jar package runs for the first time, and the configuration items are in:

- #jpa
- spring.jpa.show-sql=true
- spring.data.jpa.repositories.enabled=true
- spring.jpa.hibernate.ddl-auto=update

If you do not want the database tables to dynamically update with Java class entities, you can choose to modify the configuration item: spring.jpa.hibernate.ddl-auto=update

## Environment Configuration

The system relies on Mongodb, Redis, MySQL, Kafka, and Alibaba Cloud OSS, so you need to prepare these basic services.

## Common Compilation Issues (Error)

1. When starting ucenter-api, the following error is prompted:
   Error creating bean with name 'enableRedisKeyspaceNotificationsInitializer' defined in class path resource
   Reason: The annotation @EnableRedisHttpSession is used to enable Redis to manage sessions in a unified manner. When using this method, the Keyspace Notifications feature of Redis needs to be enabled, which is disabled by default. This feature has a parameter to control it, notify-keyspace-events, and its value needs to be changed to Egx.
   Method for modifying the notify-keyspace-events parameter in Tencent Cloud's Redis: Enter the instance -> Parameters, and modify it.

2. When running the jar package, the following error is prompted: Failed to get nested archive for entry BOOT.......
   This error is due to the lack of the spark.xxxx.DB class, which is because the spark-core.jar was not copied to the compressed package during compilation. At this time, you need to modify the pom.xml file by appending:
   ```
   <configuration>
       <includeSystemScope>true</includeSystemScope>
   </configuration>
   ```
   Append the position: <artifactId>spring-boot-maven-plugin</artifactId>, parallel to <executions>
   Then delete the jar package under the target and regenerate it, preferably after running maven clean and then maven install.

3. org.apache.shiro.authz.UnauthorizedException: Subject does not have permission [system:announcement:deletes]
   This error is caused by the operation lacking permission. Check whether the operation method and the database are consistent.
   First step: View the request link through the browser F12, which usually displays 404.
   Second step: Check if the admin_permission table in the database has the link to be accessed.

4. The market.jar startup fails, prompting connection refused errors, etc. First, use netstat -tunlp to view whether port 6005 (i.e., exchange) is being listened to. If not, it means exchange has not fully started.
   The main reason for exchange to start slowly is that it loads slowly during initialization, especially when loading order transaction details from MongoDB, which is a huge amount of data.

## Some Configuration Item Explanations

1. market/application.properties
   - # Whether to issue commissions for secondary referrers (true: open    false: not open)
   - second.referrer.award=false

2. ucenter-api/application.properties
   - # system (used for sending emails)
   - spark.system.work-id=1
   - spark.system.data-center-id=1
   - spark.system.host=
   - spark.system.name=
   - # Referral registration reward, 1 = the referred person must be real-name authenticated to receive the reward, otherwise, there is no limit, and it needs to be consistent with the configuration in the admin module
   - commission.need.real-name=1
   - # Whether to open secondary rewards for referral registration (1 = open, 0 = close)
   - commission.promotion.second-level=0
   - # Prefix for personal promotion links, returned to the client along with the login interface. The client side connects with the promotion code to form a personal promotion link. If there is a promotion registration function, it must be filled in
   - person.promote.prefix=http://www.bizzan.com/#/register?agent=
   - # Transfer interface address
   - transfer.url=
   - transfer.key=
   - transfer.smac=

## Nginx Configuration

Reference URL:
https://blog.csdn.net/qq_36628908/article/details/80243713

Nginx document directory: usr/local/nginx/html/, usr/local/nginx/conf/

Note:
1. api.xxxx.com and www.xxxx.com need to forward different services
2. api.xxxx.com needs to support websocket

===========================
Dependency Environment
yum install -y wget  
yum install -y vim-enhanced  
yum install -y make cmake gcc gcc-c++  
yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel

Download nginx-1.12.2.tar.gz
wget http://nginx.org/download/nginx-1.12.2.tar.gz

Compile and install 
tar -zxvf nginx-1.12.2.tar.gz 
cd nginx-1.12.2

./configure \
--prefix=/usr/local/nginx \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--with-http_stub_status_module \
--with-http_ssl_module \
--http-scgi-temp-path=/var/temp/nginx/scgi \
--with-stream --with-stream_ssl_module

make 
make install