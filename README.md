# EE5415 Android 开发与 BMI 作业源码

[Android Studio 与模拟器操作手册](Android-Studio.md) · [完整 Android Studio 工程包](BMI-Android-Projects.zip)

## 从 BMI 改成其他应用

[把 A02 BMI 改成普通计算器](BMI-to-Calculator.md) 按文件标出要删、要换和要新增的代码，也说明数字键盘、温度换算、平均速度、折扣、第二页和保存怎么接。

[其他 Android 题型的改法](Android-Task-Playbook.md) 覆盖报名预约、搜索列表、待办和记账、答题计分、倒计时、保存设置、加载网络数据。可以按题干关键词查实现步骤和对应作业。

## 作业目录

### [A02-BMI 逐文件注释](A02-BMI.md)

从 activity_main.xml 的四项输入进入 MainActivity。点击按钮先校验，再传到 ReportActivity。BmiCalculator 统一负责公式及年龄分组。用 170 厘米、60 千克、20 岁检查 20.76 和 Normal range。

### [A03-BMI-MVI 逐文件注释](A03-BMI-MVI.md)

从 BmiIntent.Calculate 开始。Activity 把输入封装成事件，BmiStore.dispatch 经 reduce 产生 BmiState，页面观察结果后发出 ReportShown 清空状态。比较同一输入在 MVP、MVVM 版本中的调用顺序。

### [A03-BMI-MVP 逐文件注释](A03-BMI-MVP.md)

先读 BmiView 的展示契约，再看 BmiPresenter 如何接收四个文本并调用模型。模型返回错误字段或 BmiResult，Presenter 回调 Activity 的 showResult，界面定位错误或打开报告。

### [A03-BMI-MVVM 逐文件注释](A03-BMI-MVVM.md)

先读 BmiViewModel。按钮触发 calculate，模型将结果写入 LiveData，Activity 观察后调用 showResult，再 clearResult 消费事件。旋转后要检查有没有重复打开报告。

### [A04-BMI-MVVM 逐文件注释](A04-BMI-MVVM.md)

从 ApiSettingsActivity、ReportActivity、AdviceViewModel 依次追到 DeepSeekClient。页面观察加载、成功和错误状态，线程池执行网络请求，主线程只接收最新请求编号的结果。原作业的服务配置和提示词仅用于复习此作业。

### [A05-BMI-MVVM 逐文件注释](A05-BMI-MVVM.md)

MainActivity 保存身高、年龄和性别到偏好文件。日期和 BMI 写入 SQLite，同一天重新计算会替换旧值。HistoryActivity 用包含本月月初、不包含下月月初的区间查询，BmiChartView 把日期和 BMI 映射成横纵坐标。测试月底跨年、无记录月份和同日重复写入。

Assignment 03 的三个版本均含中英文资源、横竖屏布局和菜单。Assignment 05 按本次核对的存储版提交包整理。

这些课程复习资料可直接在网页阅读。考试中使用资料的范围由课程公告和现场要求决定。
