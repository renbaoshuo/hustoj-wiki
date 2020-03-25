# 更新日志

- 2020-02-10 更新：新的模板bshark基本可用，如需启用新模板，只需修改/home/judge/src/web/include/db_info.inc.php，设置$OJ_TEMPLATE="bshark";

- 2020-01-31 更新：@melongist 增加了很多页面美化。

- 2020-01-27 更新：题目限时增强为浮点型，3位小数精度，即标称毫秒(ms)。

- 2020-01-26 更新：允许为每个Web添加多个UDP通知对象，每个判题服务器允许使用不同的UDP端口监听消息。阿里云+腾讯云测试通过。

- 2020-01-23 更新：修订了[Moodle集成代码](https://github.com/zhblue/hustoj/blob/master/wiki/MoodleIntegration.md)，实现HUSTOJ给moodle系统作业自动判分。

- 2020-01-20 更新：删除noip模式比赛的多余提交记录，允许自定义“noip”关键词，增加privilege表user_id索引。

- 2019-12-19 更新：增加了judge.conf中的注释，提供了备案号变量$OJ_BEIAN，对系统判题时间分辨率进行了更新优化，提高灵活度。

- 2019-11-23 喜讯：hustoj在首届深度软件开发大赛中获得三等奖。

- 2019-11-21 补丁：修复比赛中Edit按钮后可以选择出题人禁用的语言提交【感谢湘潭大学谢老师的报告】。

- 2019-11-20 更新：在运行结果对比输出（OJ_SHOW_DIFF）中提示每个数据点的结果(AC/WA/TLE...)。

- 2019-11-16 优化：[muzea](https://github.com/muzea) 开发了从github到[gitee](https://gitee.com/zhblue/hustoj)的同步机制，并部署了CI。

- 2019-11-13 更新：在运行时错误(RuntimeError)中显示数据点文件名(infile)

- 2019-10-30 更新：提供$OJ_OI_MODE开关。

- 2019-10-29 更新：加强了OI模式下的限制，控制Web行为。

- 2019-10-3 更新：修订测试deepin15.11安装脚本，补丁：注册页面验证csrf

- 2019-9-23 补丁：修复昵称比赛中不更新问题，以及提醒官方群用户及时更新处理504超时问题。

- 2019-9-21 补丁：修复部分安装脚本不能执行第二次的问题

- 2019-8-6 更新：支持用UDP数据包触发判题轮询，实现Web本地judge秒判。

- 2019-7-26 更新：支持华为鲲鹏服务器，aarch64，感谢深度科技公司和华为云提供鲲鹏服务器。

- 2019-7-6 NOIP：对于标题带有NOIP字样的比赛，比赛结束后才能看到结果。

- 2019-7-4 mark：设置$OJ_MARK="mark"显示得分，$OJ_MARK="percent"显示错误率(WA)或通过率（AC），设置$OJ_MARK=""只显示最终结果。

- 2019-6-24 文档：对项目首页进行分块标注，调整顺序和内容，增加目录。

- 2019-6-12 更新：添加Fortran语言、Matlab(Octave)，修订：比赛结束后编辑时丢失提交统计数据、修复部分RE。

- 2019-5-18 修订：16.04以上版本FB显示异常。 [基于OpenJudger的Windows集成便携版](https://github.com/Azure99/WinHustOJ/releases) [浙传网盘](https://pan.cuz.edu.cn:8443/share/b02149ee631b2776e93590b461)

- 2019-5-17 修订：改善 `ajax` ，减少并发量，降低 web 压力，提高 judge 轮询效率。

- 2019-5-15 修订：修复了部分 TLE 误判为 RE 的情况，主要是在 `Ubuntu18/19` `Deepin15.9/15.10` 以上的版本，估计与 `gcc` `g++` 有关。

- 2019-5-7   更新：muzea 提供了Debian安装包打包(*.deb)，https://github.com/zhblue/hustoj/releases

- 2019-4-13  更新：支持 SQL 判题，基于 `sqlite3` ，支持龙芯 3A3000 （致谢江苏航天龙梦信息技术有限公司提供龙芯主机！）。

- 2019-3-14  更新:主线支持  树莓派(arm)  **龙芯(loongson-2f)**  i386 x86_64 
