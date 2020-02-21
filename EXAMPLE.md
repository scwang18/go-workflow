# 部署
## 1.docker部署
### 1.1 安装mysql 

  目前只支持mysql数据库，测试之前先安装好数据库
  1. pull mysql image
  docker pull mysql
  2. 创建目录
  mkdir -p /mysql/data
  mkdir -p /mysql/conf
  mkdir -p /mysql/logs
  mkdir -p /mysql/mysql-files
  3. 创建mysql配置文件
  ```
[client]
default-character-set=utf8mb4
[mysql]
default-character-set=utf8mb4
[mysqld]
pid-file=/var/run/mysqld/mysqld.pid
socket=/var/run/mysqld/mysqld.sock
datadir=/var/lib/mysql

character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
init_connect='SET NAMES utf8mb4'

innodb_use_native_aio=0
innodb_flush_log_at_trx_commit=0
log_queries_not_using_indexes=0
slow_query_log=1

skip-external-locking
skip-name-resolve

symbolic-links=0
max_allowed_packet = 16M
max_connections=10240
key_buffer_size=384M
table_open_cache=512
sort_buffer_size=2M
read_buffer_size=2M
read_rnd_buffer_size=8M
myisam_sort_buffer_size=64M
thread_cache_size=8

log-bin=mysql-bin
binlog_format=mixed
expire_logs_days=10
sync_binlog=0
server-id=1
default-authentication-plugin = mysql_native_password

[mysqldump]
quick
max_allowed_packet=16M

[myisamchk]
key_buffer_size=128M
sort_buffer_size=20M
read_buffer=2M
write_buffer=2M

[mysqlhotcopy]
interactive-timeout
```
  3. 关闭sulinux或者把要挂载的目录添加到白名单
```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
setenforce 0
```

  4. 启动Mysql
```
docker run --name mysql -p 3306:3306 -d -v /mysql/data:/var/lib/mysql -v /mysql/conf:/etc/mysql -v /mysql/mysql-files:/var/lib/mysql-files/ -v /mysql/logs:/var/log/mysql -e MYSQL_ROOT_PASSWORD=scwang  mysql:latest
```

  5. 登录docker
```
docker exec -it mysql /bin/bash
  docker exec -it mysql mysql -uroot -p
  6、修改mysql的访问设置
  # 查看数据库
  show databses;
  # 切换数据库
  use mysql;
  # 查看用户
  select host,user from user;
  # 修改root用户允许登录的机器
  update user set host='%' where user='root';
  # 更新权限
  flush privileges;
  # 确认是否更改完成
  select host,user from user;
  # 创建测试数据库
  create database test
```
 
### 1.2 docker 安装最新版redis服务
  1、拉取镜像
```
  docker pull redis
```  
  2. 创建目录
```  
  mkdir -p /redis/data
  mkdir -p /redis/conf
```  
  3. 创建redis.conf文件
```
#从官网下载
cd /redis/conf
wget http://download.redis.io/redis-stable/redis.conf
修改redis.conf内容

# 修改启动默认配置(从上至下依次)：

bind 127.0.0.1 #注释掉这部分，这是限制redis只能本地访问
protected-mode no #默认yes，开启保护模式，限制为本地访问
daemonize no #默认no，改为yes意为以守护进程方式启动，可后台运行，除非kill进程，改为yes会使配置文件方式启动redis失败
databases 16 #数据库个数（可选）
dir  ./ #输入本地redis数据库存放文件夹（可选）
appendonly yes #redis持久化（可选）
```
  4. 创建redis容器
```  
docker run -d --name redis --restart always -p 6379:6379 -v /redis/conf:/etc/redis -v /redis/data:/data redis redis-server /etc/redis/redis.conf 
```

### 1.3 docker 安装最新版 go-workflow 微服务
```
  docker run  --name goworkflow -e DbType=mysql -e DbLogMode=false -e DbName=test -e DbHost=192.168.3.236 -e DbUser=root -e DbPassword=scwang -e RedisHost=192.168.3.236 -p 8080:8080 registry.cn-hangzhou.aliyuncs.com/mumushuiding/go-workflow:latest
```
## 2 二进制部署

 1. 获取源代码
```
mkdir -p /go-workflow
cd /go-workflow
go get https://github.com/go-workflow/go-workflow
```
 2. 进入根目录，打开config.json文件,修改数据库连接配置
```
cd /go-workflow
vi config.json
```
 3. 编译
```
 $ go build
```
 4. 运行
```
$ go-workflow.exe
```
## 3 K8s部署

请查阅根目录的 k8s.yaml 文件 ， 配置使用了Istio, 未使用Istio的请稍作修改

# 流程存储


## 1. 存储流程定义。
  通过 Post 访问： http://localhost:8080/api/v1/workflow/procdef/save

  (参数详解见 ProcessConfig流程定义配置.md)

  Post参数：
  
  {"userid":"11025","name":"请假","company":"A公司","resource":{"name":"发起人","type":"start","nodeId":"sid-startevent","childNode":{"type":"route","prevId":"sid-startevent","nodeId":"8b5c_debb","conditionNodes":[{"name":"条件1","type":"condition","prevId":"8b5c_debb","nodeId":"da89_be76","properties":{"conditions":[[{"type":"dingtalk_actioner_value_condition","paramKey":"DDHolidayField-J2BWEN12__options","paramLabel":"请假类型","paramValue":"","paramValues":["年假"],"oriValue":["年假","事假","病假","调休","产假","婚假","例假","丧假"],"isEmpty":false}]]},"childNode":{"name":"UNKNOWN","type":"approver","prevId":"da89_be76","nodeId":"735c_0854","properties":{"activateType":"ONE_BY_ONE","agreeAll":false,"actionerRules":[{"type":"target_management","level":1,"isEmpty":false,"autoUp":true}]}}},{"name":"条件2","type":"condition","prevId":"8b5c_debb","nodeId":"a97f_9517","properties":{"conditions":[[{"type":"dingtalk_actioner_value_condition","paramKey":"DDHolidayField-J2BWEN12__options","paramLabel":"请假类型","paramValue":"","paramValues":["调休"],"oriValue":["年假","事假","病假","调休","产假","婚假","例假","丧假"],"isEmpty":false}]]},"childNode":{"name":"UNKNOWN","type":"approver","prevId":"a97f_9517","nodeId":"5891_395b","properties":{"activateType":"ALL","agreeAll":true,"actionerRules":[{"type":"target_label","labelNames":"财务","labels":427529103,"isEmpty":false,"memberCount":2,"actType":"and"}],"noneActionerAction":"auto"}}}],"properties":{},"childNode":{"name":"UNKNOWN","type":"approver","prevId":"8b5c_debb","nodeId":"59ba_8815","properties":{"activateType":"ALL","agreeAll":true,"actionerRules":[{"type":"target_label","labelNames":"人事","labels":427529104,"isEmpty":false,"memberCount":1,"actType":"and"}]}}}}}

  如果返回：{"data":"1","ok":true} ，1表示流程实例的id,true表示成功了


  ---------------------------------------------------------------
  或 通过 POST 访问： http://localhost:8080/api/v1/workflow/procdef/saveByToken (后台通过 token 从redis查询用户信息 userinfo，token可以保存在 Authorization 里
  面 或者 reques参数里)

// UserInfo 用户信息

  type UserInfo struct {

    Company string `json:"company"`

    // 用户所属部门

    Department string `json:"department"`

    // 用户ID

    ID string `json:"ID"`
    
    Username   string `json:"username"`

    // 用户的角色

    Roles []string `json:"roles"`

    // 用户负责的部门，用户是哪些部门的主管

    Departments []string `json:"departments"`

  }

  在config.json 里 配置 redis 连接：

  "RedisCluster": "false",  // false表示redis是单点，true表示redis是集群

  "RedisHost": "localhost",

  "RedisPort": "6379",
  
  "RedisPassword": "",

--------------------------------------------------------------

## 2. 查询流程定义

  通过 POST 访问： http://localhost:8080/api/v1/workflow/procdef/findAll

  POST参数： {"name": "请假", "company","pageSize": 1, "pageIndex": 1}  , 四个参数皆可为空

# 启动流程
  通过 POST 访问： http://localhost:8080/api/v1/workflow/process/start

  POST参数：

  {"procName":"请假","title":"请假-张三","userId":"11025","department":"技术中心","company":"A公司","var":{"DDHolidayField-J2BWEN12__duration":"8","DDHolidayField-J2BWEN12__options":"年假"}}

  返回结果：{"data":"1","ok":true}

# 审批

## 1. 审批
  通过POST访问：http://localhost:8080/workflow/task/complete

  POST参数：{"taskID":2,"pass":"true","userID":"11029","company":"A公司","comment": "评论备注","candidate": "王五"}

  参数详解： taskID代表当前任务id，true表示通过，false表示驳回,candidate指定下一步执行人或者审批组如：candidate: "人事组"（一般不指定）

  （注意：整个流程框架，所有关于 userID的值最好是用户名，用户名不可重复）
## 2. 撤回

  通过POST访问：http://localhost:8080/workflow/task/withdraw

  POST参数：{"taskID":2,"userID":"11029","procInstID":1,"company":"A公司"}

   参数详解： taskID为当前任务id

## 3. 任务查询 

  通过POST访问 ：http://localhost:8080/workflow/process/findTask
  
  POST参数：{"userID":"11025","groups":["人事"],"departments":["技术中心"],"company":"A公司","procName": "请假","pageIndex":1,"pageSize":10}

  参数详解： groups 表示用户的所有角色，departments表示用户负责的部门， procName：流程类型,pageIndex表示当前页可不填（默认1)，pageSize表示每页显示条数可不填（默认10）


  （注意：整个流程框架，所有关于 userID的值最好是用户名，用户名不可重复）

### 4. 查询流程审批人与评论
  
  通过GET访问 ：http://localhost:8080/workflow/identitylink/findParticipant?procInstID=12562

  参数详解： procInstID 为流程实例id

# 查询历史流程

## 1. 查询我审批的流程

  通过POST访问： http://localhost:8080/api/v1/workflow/procHistory/findTask

  POST参数：{"userID":"admin","company":"A公司","pageIndex":1,"pageSize":2}

  （注意：整个流程框架，所有关于 userID 的值最好是用户名，用户名不可重复）

## 2. 查询我发起的流程

-----------查询已经结束的流程--------

  通过POST访问： http://localhost:8080/api/v1/workflow/procHistory/startByMyself

  POST参数： {"userID":"admin","company":"A公司","pageIndex":1,"pageSize":2}

-----------查询正在审批的流程--------

  通过POST访问： http://localhost:8080/api/v1/workflow/process/startByMyself

  POST参数： {"userID":"admin","company":"A公司","pageIndex":1,"pageSize":2}

## 3. 查询抄送我的流程

------------------ 审批中 ------------------------

  通过POST访问： http://localhost:8080/api/v1/workflow/process/FindProcNotify

  POST参数：{"userID":"admin","company":"A公司","groups":["人事","产品经理"]}

-----------------  已结束-------------------------

  通过POST访问：http://localhost:8080/workflow/procHistory/FindProcNotify

  POST参数：{"userID":"admin","company":"A公司","groups":["人事","产品经理"]}


