## HUSTOJ FAQ

1．如何进入后台？

以管理员身份登录，点击 `Admin/管理` 进入后台。

2．如何添加题目？

进入后台，点击左侧NewProblem。

3．如何添加测试数据？

添加题目时，可以在 `test input` `test output` 添加一组测试数据，大规模的数据（10KB+）和更多的数据，可以在添加完题目后，通过 `ftp/sftp`，上传到题目对应目录，通常是 `/home/judge/data/题号` 。命名规则是输入数据以 `.in` 结尾，输出数据以 `.out` 结尾，主文件名相同。

4．如何编辑题目？

后台中点击 `Problem List` ，找到需要编辑的题目，点击 `Edit` 。编辑时不能修改测试数据，测试数据请使用 `ftp` 工具修改。

5．如何启用题目？

题目添加后，默认是停用状态，以防比赛提前漏题，后台中点击 `ProblemList` ，找到题目，点击 `Resume` 启用题目，或者组织比赛，比赛中的题目将自动启用。

6．如何组织比赛？

在题目列表 `ProblemList` 中选择使用的题目， 在 `PID` 一栏打钩， 点击 `CheckToNewContest` 按钮，进入到比赛添加页面，输入比赛名称，设定 `比赛时间`、`语言类型`、`访问权限`，然后提交即可。

也可以使用管理菜单中的 `NewContest` ，需要手动输入题目编号，用英文逗号分隔。

7．如何修改、删除比赛？

点击比赛列表 ContestList，选择 `Edit` 或 `Delete`。

8．如何修改公告信息？

点击 `SetMessage`。

9．如何修改用户密码？

点击 `ChangePassWord`。

10．如何重新判题？

点击 `Rejudge` ，输入题号或运行编号。

11．如何增加用户权限？

`Addprivilege` `administrator` 为管理员，`source_browser` 为代码审查，`contest_creator`为比赛组织者。

通常给使用系统的老师分配代码审查和比赛组织者权限即可。

12．如何导入、导出题目？

使用 ImportProblem ，上传 FPS 文件。

使用 ExportProblem ，输入起始编号，结束编号，或题号列表，如果输入了列表，起始结束将不起作用。

13．如何更新数据库结构？

系统升级中，有对数据库的修改，这些修改不能通过 SVN 实现自动更新，如果发现升级 web/core 代码后系统报错，可以执行 update database 操作，进行数据库升级。因为脚本中有测试代码，所以重复执行不会造成影响。

14．如何下载新题目？

+ 访问 `FreeProblemSet` ，查看 `Downloads` 列表。<https://github.com/zhblue/freeproblemset/>
+ 访问 [TK题库](http://tk.hustoj.com/) 下载题目


15．如何让判题程序忽略行尾的空白字符？

在 `judge_client.cc` 头部增加宏定义 `IGNORE_ESOL` 。

16．如何上传多组数据？
 
加好题目后在题目列表找 `TestData` ，点击上传。主文件名一样的 `*.in` `*.out`，如 `test1.in` `test1.out`
