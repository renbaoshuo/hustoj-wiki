# 结构简析

### 概述

HUSTOJ 分为两大部分，`core` 和 `web` ，分别对应判题和数据管理两大功能。

`core` 和 `web` 之间数据交换有两种方式：

1、通过数据库，轮询。
2、通过 `wget` 实现的 `HTTP` 请求。

两种方式的选择在判题端的配置文件 `/home/judge/etc/judge.conf` 中， `HTTP_JUDGE=1` 则启用后者，默认为前者。

### Core 解析

`core` 分 `3` 部分，`judged` 、 `judge_client` 、 `sim` 。源码解读见以下链接：
<http://blog.csdn.net/legan/article/details/40746829>
<http://blog.csdn.net/legan/article/details/40789939>

#### judged 解析

##### 简介

`judged` 为服务进程， `d` 即 `daemon`。负责轮询数据库或 `web` 端，提取判题队列。单个主机运行多个 `judged` ，分别负责不同的 `OJ` 判题。

![工作流程图](images/work.jpg)

##### 基本参数

judged 可以接受一个参数作为自己的主目录，默认是 `/home/judge/` 。如： `sudo judged /home/judge/local`

不指定参数将自动以单进程运行。当指定的参数不为/home/judge 时，就会有多个进程出现。

每个主目录可以有自己的 `etc/judge.conf` 数据目录可以共享，runX 目录需要独立。

##### judged 调试模式

judged 接受参数指定目录的情况下，还可以再接受一个 `debug` 作为调试模式开关。如：`sudo judged /home/judge/local debug` 调试模式的 `judged` 将不会进入后台，并且将输出大量调试信息，其调用的judge_client 也工作在debug 模式。
#### judge_client 解析

##### 简介

当发现新任务时产生 `judge_client` 进程。

`judge_client` 进程为实际判题程序，负责准备运行环境、数据，运行并监控目标程序的系统调用，采集运行指标，判断运行结果。

![工作流程图](images/work1.jpg)

##### 后台安全机制

在UNIX 中有上百个[系统调用](http://www.ibm.com/developerworks/cn/linux/kernel/syscall/part1/appendix.html)，有一大部分是在用户程序运行过程中不需要的，比如说`mkdir` , `mount` 等，还有一部分会对系统造成安全隐患的，比如`fork` , `kill` , `exec` 等，还有一些比如 `socket` 等会造成敏感信息，比如测试数据的泄漏等。因为以上情况的存在，所以需要在运行用户程序的时候对用户加以限制，`linux` 下的 `ptrace` 在这里是一个非常好用的工具，它可以在用户态和内核态之间切换之前和之后，将进程暂停，以方便控制进程的处理，控制进程通过 `ptrace` 可以读取到当前进程想要去做什么，这样就可以在用户程序造成破坏之前将程序中止。限制非法系统调用，最好的办法是使用白名单机制，只允许程序使用一个小集合里的调用，对于其它调用，即使它是安全的，也不会被允许，比如mkdir。由于Pascal,Java,C/C++的机制有些区别，因此，三种不同语言的白名单各不相同。

##### judge_client 调试模式

`judge_client [工作主目录] [调试]`

如：`judge_client 2001 5 /home/judge/demo debug`

将在 `/home/judge/demo/run5` 目录中对 `2001` 号提交进行重判，并打开调试模式，输出大量调试信息，运行后不删除中间结果。

这个模式可以帮助调试题目数据，发现数据问题和了解提交 RE 的详细错误原因。

#### sim 简介

当配置为启用抄袭检查时，judge_client 将调用 `sim`，判断相似性结果，并写回数据库或 `web` 端。

`sim` 为第三方应用程序， 可进行语法分析判断文本相似度， 通过检验的程序将由 `judge_client` 复制进题目数据的 `ac` 目录，成为新的参考样本。

#### 配置文件 `judge.conf` 注释

配置                       |                      注释
-------------------------  |  ------------------------------------------
`OJ_HOST_NAME=127.0.0.1`   |  如果用 mysql 连接读取数据库，数据库的主机地址。
`OJ_USER_NAME=root`        |  数据库帐号。
`OJ_PASSWORD=root`         |  数据库密码。
`OJ_DB_NAME=jol`           |  数据库名称。
`OJ_PORT_NUMBER=3306`      |  数据库端口。
`OJ_RUNNING=4`             |  `judged` 会启动 `judge_client` 判题，这里规定最多同时运行几个 `judge_client`。
`OJ_SLEEP_TIME=5`          |  `judged` 通过轮询数据库发现新任务，轮询间隔的休息时间，单位为秒。
`OJ_TOTAL=1`               |  老式并发处理中总的 `judged` 数量。
`OJ_MOD=0`                 |  老式并发处理中，本 `judged` 负责处理 `solution_id` 按照 `TOTAL` 取模后余数为几的任务。
`OJ_JAVA_TIME_BONUS=2`     |  Java 等虚拟机语言获得的额外运行时间。
`OJ_JAVA_MEMORY_BONUS=512` |  Java 等虚拟机语言获得的额外内存。
`OJ_SIM_ENABLE=0`          |  是否使用sim 进行代码相似度的检测。
`OJ_HTTP_JUDGE=0`          |  是否使用 `HTTP` 方式连接数据库，如果启用，则前面的 `HOST_NAME` 等设置忽略。
`OJ_HTTP_BASEURL=http://127.0.0.1/`  |  使用 `HTTP` 方式连接数据库的基础地址，就是 `OJ` 的首页地址。
`OJ_HTTP_USERNAME=admin`   |  使用 `HTTP` 方式所用的用户帐号（`HTTP_JUDGE` 权限），该帐号登录时不能启用 `VCODE` 图形验证码，但可以登录成功后启用。
`OJ_HTTP_PASSWORD=admin`   |  使用 `HTTP` 方式所用的用户帐号的密码
`OJ_OI_MODE=0`             |  是否启用 `OI（信息学奥林匹克竞赛）` 模式，即无论是否出错都继续判剩余的数据，在 `ACM` 比赛中一旦出错就停止运行。
`OJ_SHM_RUN=0`             |  是否使用 `/dev/shm` 的共享内存虚拟磁盘来运行答案，如果启用能提高判题速度，但需要较多内存。
`OJ_USE_MAX_TIME=1`        |  是否使用所有测试数据中最大的运行时间作为最后运行时间，如果不启用则以所有测试数据的总时间作为超时判断依据。
O`J_LANG_SET=0,1,2,3,4`    |  判哪些语言的题目

### Web 解析

`web` 分两大部分，`前端` 和 `admin` 目录下的管理程序。

#### 前端

无非是数据库的 CRUD 操作，关键功能是将用户提交的程序源码加入数据库的任务队列（`solution` 表、`souce_code` 表）。

#### 管理程序

提供具有 `administrator` 等高级权限的账号管理试题、账号等方面的功能。其中 `FPS` 导入
导出程序主要为 `XML` 格式的数据处理。

特别的， `judged` 可以多重启动， 通过增加基准目录参数指定启动位置（ 默认`/home/judge`），从而确定judge.conf 的位置，并确定其他参数。因此不但可以一个 `web` 服务器下挂多个判题服务器，也可以一台物理机器上同时启动任意多个相互独立的OJ 系统。实际使用中，使用开源的 `ispcp` 虚拟主机管理系统搭建多 `Web` 环境与 `hustoj` 协同工作取
得了良好效果。

#### 比赛方面

1. 比赛根据数据通过率排名，而不只看 `AC` 数量

2. 数据库`solution` 表 `pass_rate` 字段表示该条通过率。

3. 把`contestrank.php` 中的 `solved` 字段变成浮点对待。

4. 修改积分方式，按照希望的方式积分。可能需要给 TM 增加字段 `$p_wa_best_rate` 记录每题最大通过率。

#### 配置文件 `db_info.inc.php` 注释

配置                                  |                      注释
------------------------------------  |  ------------------------------------------
`static $DB_HOST="localhost";`        |  数据库的服务器地址。
`static $DB_NAME="jol";`              |  数据库名。
`static $DB_USER="root";`             |  数据库用户名。
`static $DB_PASS="root";`             |  数据库密码。
`static $OJ_NAME="HUSTOJ";`           |  OJ 的名字，将取代页面标题等位置 `HUSTOJ` 字样。
`static $OJ_HOME="./";`               |  OJ 的首页地址。
`static $OJ_ADMIN="root@localhost";`  |  管理员email。
`static $OJ_DATA="/home/judge/data";` |  测试数据所在目录，实际位置。
`static $OJ_BBS="discuss";`           |  论坛的形式，`discuss3` 为自带的简单论坛，`bbs` 为外挂论坛，参考 `bbs.php` 代码。
`static $OJ_ONLINE=false;`            |  是否使用在线监控，需要消耗一定的内存和计算，因此如果并发大建议关闭。
`static $OJ_LANG="en";`               |  默认的语言，中文为 `cn` 。
`static $OJ_SIM=true;`                |  是否显示相似度检测的结果。
`static $OJ_DICT=true;`               |  是否启用在线英字典。
`static $OJ_LANGMASK=1008;`           |  `1mC` `2mCPP` `4mPascal` `8mJava` `16mRuby` `32mBash` 用掩码表示的OJ 接受的提交语言，可以被比赛设定覆盖。1008 为只使用 `C` `CPP` `Pascal` `Java`。
`static $OJ_EDITE_AREA=true;`         |  是否启用高亮语法显示的提交界面，可以在线编程，无须IDE。
`static $OJ_AUTO_SHARE=false;`        |  `true`: 自动分享代码，启用的话，做出一道题就可以在该题的 `Status` 中看其他人的答案。
`static $OJ_CSS="hoj.css";`           |  默认的css,可以选择 `dark.css` 和 `gcode.css` , 具有有限的界面制定效果。
`static $OJ_SAE=false;`               |  是否是在新浪的云平台运行web 部分
`static $OJ_VCODE=true;`              |  是否启用图形登录、注册验证码。
`static $OJ_APPENDCODE=false;`        |  是否启用自动添加代码，启用的话，提交时会参考$OJ_DATA 对应目录里是否有 `append.c` 一类的文件，有的话会把其中代码附加到对应语言的答案之后，巧妙使用可以指定 `main` 函数而要求学生编写main 部分调用的函数。
`static $OJ_MEMCACHE=false;`          |  是否使用 `memcache` 作为页面缓存，如果不启用则用 `/cache` 目录
`static $OJ_MEMSERVER="127.0.0.1";`   |  `memcached` 的服务器地址
`static $OJ_MEMPORT=11211;`           |  `memcached` 的端口
`static $OJ_RANK_LOCK_PERCENT=0;`     |  比赛封榜时间的比率，如 5 小时比赛设为 `0.2` 则最后 1 小时封榜。
`static $OJ_SHOW_DIFF=false;`         |  显示 `WrongAnswer` 时的对比

### Core 与 Web 的连接方式解析

#### 简化ER 图

![](images/c2web.jpg)

#### 数据库连接（默认）

1. `Web` 插入 `Solution` 表。
2. `core` 轮询 `solution` 表，发现新记录。
3. `core` 更新 `solution` 表 `result` 等字段
4. `Web` 端轮询 `soltuion` 显示 `result` 等字段。

#### HTTP 方式连接

1. `Web` 插入 `Solution` 表
2. `core` 访问 `Web` 端 `admin/problem_judge.php` ，发现新纪录
3. `core` 向 `Web` 端 `admin/problem_judge.php` 提交数据，`problem_judge.php` 更新 `solution` 表 `result` 等字段。
4. `Web` 端轮询 `soltuion` 显示 `result` 等字段。

### 数据库解析

#### 数据库关系图

![](images/db.jpg)

#### 数据库表分析（by `夏夏`）

序号  |         表名        |         作用           |  备注
----  |  -----------------  |  -------------------  |  --------------------
1     |  `compileinfo`      |  记录编译错误的记录     |
2     |  `contest`          |  竞赛表                | 
3     |  `contest_problem`  |  竞赛题目              |
4     |  `loginlog`         |  登入日志              |  记录正确与错误的登入日志 
5     |  `mail`             |  消息列表              | 
6     |  `news`             |  新闻表                | 
7     |  `online`           |  用户在线数据统计       | 
8     |  `privilege`        |  权限授予              |
9     |  `problem`          |  题目表                |
10    |  `reply`            |  论坛（帖子及回复）表   |
11    |  `runtimeinfo`      |  运行错误信息           |
12    |  `sim`              |  相似度检测表           |  用于防作弊
13    |  `solution`         |  程序运行结果记录       | 
14    |  `source_code`      |  提交的源码             |
15    |  `topic`            |  论坛帖子表             |
16    |  `users`            |  用户信息               |
17    |  `custominput`      |  用于在线IDE            |

#### 数据库表详解

**`compileinfo` - 记录编译错误的提交号（`id`）及原因**

字段名       |  类型  |  长度  |  是否允许为空  |  备注
------------- | ----- | ---| - | -----
`solution_id` | `int` | 11 | N | 主键（提交id，即RunID）
`error`       | `text` | - | Y | 编译错误原因

**`contest` - 竞赛表**

字段名         | 类型       | 长度 | 是否允许为空 | 备注
------------- | ---------- | ---- | ----------- | ---
`contest_id`  | `int`      | 11   | N           | 竞赛id（主键）
`title`       | `varchar`  | 255  | Y           | 竞赛标题
`start_time`  | `datetime` | -    | Y           | 开始时间(年月日时分)
`end_time`    | `datatime` | -    | Y           | 结束时间(年月日时分)
`defunct`     | `char`     | 1    | N           | 是否屏蔽（Y/N）
`description` | `text`     | -    | Y           | 描述（在此版本中未用）
`private`     | `tinyint`  | 4    | -           | 公开/内部（0/1）
`langmask`    | `int`      | 11   | -           | 语言

**`contest_problem` - 竞赛题目**

字段名       |  类型   |  长度 | 是否允许为空 | 备注
------------ | ------ | ----- | ----------  | ---
`problem_id` | `int`  | 11    | N           | 题目id
`contest_id` | `int`  | 11    | Y           | 竞赛id
`title`      | `char` | 200   | N           | 标题
`num`        | `int`  | 11    | N           | 竞赛题目编号

**`loginlog` - 登入日志（不管是否登入成功都记录）**

字段名     | 类型        | 长度 | 是否允许为空 | 备注
---------  | ---------- | ---- | ----------- | ---
`user_id`  | `varchar`  | 20   | N           | 用户id
`password` | `varchar`  | 40   | Y           | 密码（不一定正确）
`ip`       | `varcahr`  | 100  | Y           | 登录的ip
`time`     | `datetime` | -    | Y           | 登入时间

**`mail` - 站内消息系统**

字段名      | 类型        | 长度 | 是否允许为空 | 备注
----------- | ---------- | ---- | ----------- | ---
`mail_id`   | `int`      | 11   | N           | 消息编号 
`to_user`   | `archar`   | 20   | N           | 接收者
`from_user` | `varchar`  | 20   | N           | 发送者
`title`     | `varchar`  | 200  | N           | 标题
`content`   | `text`     | -    | Y           | 内容
`new_mail`  | `tinyint`  | 1    | N           | 新消息（1/0）
`reply`     | `tinyint`  | 4    | Y           | 回复
`in_date`   | `datetime` | -    | Y           | 时间
`defunct`   | `char`     | 1    | N           | 是否屏蔽（Y/N）

**news - 新闻（首页显示）**

字段名        | 类型       | 长度 | 是否允许为空 | 备注
------------ | ---------- | --- | ----------- | -----
`news_id`    | `int`      | 11  | N           | 新闻编号（主键）
`user_id`    | `varchar`  | 20  | N           | 用户账号
`title`      | `varchar`  | 200 | N           | 新闻标题
`content`    | `text`     | -   | N           | 内容
`time`       | `datetime` | -   | N           | 更新时间
`importance` | `tinyint`  | 4   | N           | 关键字
`defunct`    | `char`     | 1   | N           | 是否屏蔽（Y/N）

**`online`**

字段名       | 类型      | 长度 | 是否允许为空 | 备注
----------- | --------- | ---- | ----------- | ------------
`hash`      | `varchar` | 32   | N           | 主键
`ip`        | `varchar` | 20   | N           | IP 地址
`ua`        | `varchar` | 255  | N           | 浏览器发出的浏览器相关的标识字符串
`refer`     | `varchar` | 255  | Y           |浏览器发出的一个表示访问的上个页面的网址。
`lastmove`  | `int`     | 10   | N           | 最后一次修改时间
`firsttime` | `int`     | 10   | Y           | 第一次访问时间
`uri`       | `varchar` | 255  | Y           | 统一资源指示器，包括URL（统一资源定位符） 和URN（统一资源名称）两种

**`privilege` - 用户分组**

字段名      | 类型   | 长度 | 是否允许为空 | 备注
---------- | ------ | ---- | ----------- | -----------
`user_id`  | `char` | 20   | N           | 用户帐号
`rightstr` | `char` | 30   | N           | 分组
`defunct`  | `char` | 1    | N           | 是否屏蔽（Y/N）

**`problem` - 题目表**

字段名          | 类型        | 长度 | 是否允许为空 | 备注
--------------- | ---------- | ---- | ----------- | ---------------
`problem_id`    | `int`      | 11   | N           | 题目编号，主键
`title`         | `varchar`  | 200  | N           | 标题
`description`   | `text`     | -    | Y           | 题目描述
`inupt`         | `text`     | -    | Y           | 输入说明
`output`        | `text`     | -    | Y           | 输出说明
`sample_input`  | `text`     | -    | Y           | 输入参照
`sample_output` | `text`     | -    | Y           | 输出参照
`spj`           | `char`     | 1    | N           | 是否为特别题目
`hint`          | `text`     | -    | Y           | 提示
`source`        | `varchar`  | 100  | Y           | 来源
`in_date`       | `datetime` | -    | Y           | 加入时间
`time_limit`    | `int`      | 11   | N           | 限时（秒）
`memory_limit`  | `int`      | 11   | N           | 空间限制（MByte）
`defunct`       | `char`     | 1    | N           | 是否屏蔽（Y/N）
`accepted`      | `int`      | 11   | Y           | 总ac 次数
`submit`        | `int`      | 11   | Y           | 总提交次数
`solved`        | `int`      | 11   | Y           | 解答（未用）

**`reply` - 论坛（帖子及回复）**

字段名       | 类型       | 长度 | 是否允许为空 | 备注
----------- | ---------- | ---- | ----------- | -----------
`rid`       | `int`      | 11   | N           | 帖子序号（ 主键）
`author_id` | `varchar`  | 20   | N           | 作者帐号
`time`      | `datetime` | -    | N           | 发布时间
`content`   | `text`     | -    | N           | 帖子内容
`topic_id`  | `int`      | 11   | N           | 帖子分组
`status`    | `int`      | 2    | N           | 状态（0：正常，1：锁定，2：删除）
`ip`        | `varchar`  | 30   | N           | 发帖子者ip

**`runtimeinfo` - 运行错误信息**

字段名         | 类型   | 长度 | 是否允许为空 | 备注
------------- | ------ | ---- | ----------- | -----------
`solution_id` | `int`  | 11   | N           | 运行id（主键）
`error`       | `text` | -    | Y           | 错误记录

**`sim` 相似度检测**

字段名         | 类型   | 长度 | 是否允许为空 | 备注
------------- | ------ | ---- | ----------- | -----------
`s_id`        | `int`  | 11   | N           | 提交号 `soltiotn_id`（主键）
`sim_s_id`    | `int`  | 11   | Y           | 与 `s_id` 相似的`soltion_id`
`sim`         | `int`  | 11   | Y           | 相似度（50-100）

**`solution` 程序运行结果记录**

字段名         | 类型       | 长度 | 是否允许为空 | 备注
------------- | ---------- | ---- | ----------- | -----------
`solution_id` | `int`      | 11   | N           | 运行id（主键）
`problem_id`  | `int`      | 11   | N           | 问题id
`user_id`     | `char`     | 20   | N           | 用户id
`time`        | `int`      | 11   | N           | 用时（秒）
`memory`      | `int`      | 11   | N           | 所用空间
`in_date`     | `datetime` | -    | N           | 加入时间
`result`      | `smallint` | 6    | N           | 结果（4：AC）
`language`    | `tinyint`  | 4    | N           | 语言
`ip`          | `char`     | 15   | N           | 用户ip
`contest_id`  | `int`      | 11   | Y           | 所属于竞赛组
`valid`       | `tinyint`  | 4    | N           | 是否有效
`num`         | `tinyint`  | 4    | N           | 题目在竞赛中的顺序号
`code_lenght` | `int`      | 11   | N           | 代码长度
`judgetime`   | `datetime` | -    | Y           | 判题时间
`pass_rate`   | `decimal`  | 2    | N           | 通过百分比（OI模式下可用）

**`source_code` - 源代码**

字段名         | 类型   | 长度 | 是否允许为空 | 备注
------------- | ------ | ---- | ----------- | -----------
`solution_id` | `int`  | 11   | N           | 运行id（主键）
`source`      | `text` | -    | N           | 源代码

**`topic` - 论坛帖子主题**

字段名       | 类型        | 长度  | 是否允许为空 | 备注
----------- | ----------- | ----- | ----------- | -----------
`tid`       | `int`       | 11    | N           | 帖子编号（ 主键）
`title`     | `varbinary` | 60    | N           | 标题
`status`    | `int`       | 2     | N           | 状态（0：未锁定，1：锁定）
`top_level` | `int`       | 2     | N           | 置顶等级（0，1：题目置顶，2：分区置顶，3：总置顶）
`cid`       | `int`       | 11    | Y           | 竞赛编号
`pid`       | `int`       | 11    | N           | 竞赛中题目编号
`author_id` | `varchar`   | 20    | N           | 作者id

**`users` - 用户**

字段名         | 类型       | 长度 | 是否允许为空 | 备注
------------- | ---------- | ---- | ----------- | -----------
`user_id`     | `varchar`  | 20   | N           | 用户id（主键）
`email`       | `varchar`  | 100  | Y           | 用户E-mail
`submit`      | `int`      | 11   | Y           | 用户提交次数
`solved`      | `int`      | 11   | Y           | 成功次数
`defunct`     | `char`     | 1    | N           | 是否屏蔽（Y/N）
`ip`          | `varchar`  | 20   | N           | 用户注册ip
`accesstime`  | `datetime` | -    | Y           | 用户注册时间
`volume`      | `int`      | 11   | N           | 页码（表示用户上次看到第几页）
`language`    | `int`      | 11   | N           | 语言
`password`    | `varchar`  | 32   | Y           | 密码（加密）
`reg_time`    | `datetime` | -    | Y           | 用户注册时间
`nick`        | `varchar`  | 100  | N           | 昵称
`school`      | `varchar`  | 100  | N           | 用户所在学校

**`custominput` - IDE**

字段名         | 类型   | 长度 | 是否允许为空 | 备注
------------- | ------ | ---- | ----------- | -----------
`solution_id` | `int`  | 11   | N           | 用户id（主键）
`Input_text`  | `text` | -    | -           | 输入测试数据

### LiveCD 解析

#### LiveCD 的实现
通过 `uck` 工具解压出 `Ubuntu LiveCD` 的 `chroot` 环境，并在其中删除 `oo` 、 `gnome` 等大型程序释放空间，然后用 `apt` 工具安装基础环境，安装配置 `lxde` 和 `hustoj` 。再使用 `uck` 重新打包形成 `iso`。

#### 升级方式

利用 `Github` 的 SVN 服务，用 SVN 客户端分别升级 `core` 和 `web` ,再编译 `core` ，并通过 `web` 提供可能的数据库升级。

`LiveCD` 中的升级脚本为 `update-hustoj` ，可以用 `which` 命令查找其实际位置。
