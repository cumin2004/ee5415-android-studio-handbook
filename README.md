# Android Studio 上机准备与排错手册

适用于 EE5415 的 Windows 机房环境和 BMI 类 Android 小应用。按 **新建工程 → 跑通模拟器 → 完成主功能 → 验证 → 生成 APK → 按题面提交** 的顺序练习。手册中的操作步骤供考前复习。考试查阅材料时，以老师的最新通知和现场说明为准。

## 目录

1. [考场规则和官方入口](#01-考场规则和官方入口)
2. [机房启动与前十分钟](#02-机房启动与前十分钟)
3. [从空项目新建 Java 应用](#03-从空项目新建-java-应用)
4. [SDK Manager 和版本选择](#04-sdk-manager-和版本选择)
5. [模拟器创建启动与操作](#05-模拟器创建启动与操作)
6. [BMI 类应用怎么拆](#06-bmi-类应用怎么拆)
7. [第二页面和保存数据](#07-第二页面和保存数据)
8. [在已交的 BMI 作业里怎么查](#08-在已交的-bmi-作业里怎么查)
9. [Android Studio 日常操作](#09-android-studio-日常操作)
10. [按症状排错](#10-按症状排错)
11. [验收用例](#11-验收用例)
12. [APK 源码和提交](#12-apk-源码和提交)
13. [三小时怎么分配](#13-三小时怎么分配)
14. [考前演练与原始文档](#14-考前演练与原始文档)

## 01 考场规则和官方入口

- **考试时间**为 2026 年 11 月 28 日 09:00–12:10，地点 **YEUNG–B7701**。前十分钟配置，之后三小时考试。须使用机房电脑。个人笔记本只能作为机房电脑故障时的备选，经监考明确允许后才能用。[考试最新公告](https://canvas.cityu.edu.hk/courses/70618/announcements/648700) · [原考试公告](https://canvas.cityu.edu.hk/courses/70618/discussion_topics/646154)
- 可以查看 **Canvas 课件**，下载自己以前提交的作业，使用 **Google Search** 查编程资料。11 月 27 日重发规则允许在 **GitHub 网页**上查看，禁止下载或克隆整个 GitHub 项目。自行准备的材料能否查阅，按现场说明确认。[考试最新公告](https://canvas.cityu.edu.hk/courses/70618/announcements/648700)
- 禁止使用 AI 代码生成或 AI 搜索答案、USB、远程桌面及外部协助。Google 页面若显示 **AI Overview**，可通过 **Web** 筛选进入普通网页结果，再点开原始文档。[原考试公告](https://canvas.cityu.edu.hk/courses/70618/discussion_topics/646154) · [Google 搜索说明](https://support.google.com/websearch/answer/14901683)

## 02 机房启动与前十分钟

1. 登录机房 Windows，打开 Android Studio。如果桌面没有快捷方式，在文件资源管理器地址栏输入 `C:\Program Files\Android\Android Studio\bin\studio64.exe`，按 Enter。[机房配置公告](https://canvas.cityu.edu.hk/courses/70618/discussion_topics/643542)
2. 确认 Canvas 登录能用，能打开课程页面和作业。若需要手机验证，开考前遵照监考安排处理。
3. 如果 Android Studio 显示首次启动向导，按机房配置公告与现有 SDK 设置操作。能识别已安装的 SDK 时，保留机房配置。[机房配置公告](https://canvas.cityu.edu.hk/courses/70618/discussion_topics/643542)
4. 打开 **Device Manager**，看看是否已有可用虚拟设备并确认能启动。按监考允许的范围做好配置，正式项目按题目要求建立。
5. 如果登录、SDK、模拟器或磁盘权限有问题，记下具体错误和机位，及时告诉监考。可用现有配置时，先验证工具链能运行。

## 03 从空项目新建 Java 应用

如果题目允许自行选择技术，下面用 **Java + XML Views**，和课程 BMI 作业一致。题面指定的语言、界面技术、名称、包名或 SDK 要求优先。

1. 欢迎页点 **New Project**。已有项目开着时，可用 **File → New → New Project**，或用 **Find Action** 搜索 *New Project*。
2. 选 **Phone and Tablet → Empty Views Activity**。选带 **Views** 的模板。选择完成后，应看到 Activity 源码和 `res/layout` 中的 XML 文件。若模板页面提供 **Language**，选择 **Java**。
3. 填 **Name**、**Package name**、**Save location**、**Minimum SDK**。无指定时用清楚的英文名和简单的本地目录，确保有写入权限。保留模板自带的 Gradle 配置与默认选项。
4. 点 **Finish**，等 Gradle Sync 完成。左边 **Android** 视图通常将源码归在 **app → java** 或 **kotlin+java**，布局归在 **app → res → layout**。如果找不到真实路径，可把左上视图改成 **Project**。
5. 在顶部选应用运行配置 **app** 和目标模拟器，点 **Run ▶**。默认页面在模拟器出现后，再开始改布局。先记录项目实际存在哪个文件夹，交源码时要找到项目根目录。

| 文件 | 平时用来做什么 |
| --- | --- |
| `app/src/main/AndroidManifest.xml` | 应用和 Activity 声明。可查哪个页面是启动页。 |
| `app/src/main/java/<包路径>/MainActivity.java` | 主页面加载布局、绑定控件、处理按钮和数据。 |
| `app/src/main/res/layout/activity_main.xml` | 主页面的 XML 布局、控件及其 ID。 |
| `app/src/main/res/values/strings.xml` | 存放界面文字资源。 |
| `app/build.gradle` 或 `app/build.gradle.kts` | 应用模块的 SDK 版本、构建选项和依赖。 |
| `settings.gradle` 或 `settings.gradle.kts` | 项目级设置和模块列表。 |

官方步骤和运行界面可对照文档。[新建工程](https://developer.android.com/studio/projects/create-project) · [构建与运行](https://developer.android.com/studio/run)

## 04 SDK Manager 和版本选择

打开 **Tools → SDK Manager**，先看顶部的 **Android SDK Location**，再检查以下两个标签页。

| 标签页 | 检查什么 |
| --- | --- |
| **SDK Platforms** | 确认工程编译所用的 Android 平台已安装，版本对应工程的 `compileSdk`。 |
| **SDK Tools** | 确认 **Android SDK Platform-Tools** 和 **Android Emulator** 可用。若构建提示缺少特定 **Build-Tools** 版本，按报错和监考指示处理。 |

- `compileSdk` 是用哪个版本的 Android API 编译。缺这个平台时，先确认 SDK 位置和安装状态。
- `minSdk` 是应用允许安装的最低 API。模拟器系统镜像的 API 必须等于或高于它。
- `targetSdk` 是应用声明适配的 API 级别，会影响部分系统行为。它和模拟器 API 可以不同。
- 若 Gradle Sync 报 JDK 不匹配，用 **File → Settings → Build, Execution, Deployment → Build Tools → Gradle** 检查 **Gradle JDK**。优先用这个 Android Studio 安装及新工程配置匹配的 JDK。安装、升级工具或变更机房配置按监考指示执行。[SDK Manager](https://developer.android.com/studio/intro/update) · [Gradle 所用的 JDK](https://developer.android.com/build/jdks)

## 05 模拟器创建启动与操作

### 有现成设备时

1. 打开右侧 **Device Manager**，或选 **View → Tool Windows → Device Manager**。欢迎页也可用 **More Actions → Virtual Device Manager**。
2. 打开 **Virtual** 标签页，选一个 API 满足 `minSdk` 的设备。点击该行的 **▶** 启动。等它显示 Android 主屏，再从 Android Studio 的设备列表选中该设备。
3. 点击工具栏 **Run ▶** 安装并启动应用。只改过源码以后再点一次 **Run**，确认正在运行的是最新版本。

### 需要创建时

1. **Device Manager → + → Create Virtual Device**。
2. 在硬件列表选 **Phone** 中普通手机预设，再点 **Next**。
3. 选**已经装好**且满足最低 API 的系统镜像。普通输入和计算应用通常可以用机房已有的通用 Android 镜像。若页面只显示待下载镜像，先问监考该用哪个或是否有现成设备。
4. 确认虚拟设备名称和 API 后点 **Finish**，回到 Device Manager 点 **▶**。第一次启动可能比后续慢。启动进入主屏后再运行工程。

### 运行中的常用操作

| 目的 | 从哪里操作 |
| --- | --- |
| 找正在运行的屏幕 | **View → Tool Windows → Running Devices**，或看独立 emulator 窗口。 |
| 模拟用户输入 | 点输入框后用电脑键盘或屏幕键盘打字。屏幕键盘挡住按钮时，先执行 **Back** 收起键盘。 |
| 回到应用上一页 | 用模拟器系统 **Back**，也可以点击应用内实现的返回按钮。 |
| 看详情页数据 | 先在主页面算出结果再跳转，核对每个字段是否和输入一致。 |
| 验证重开后保存 | 保存后从最近任务退出应用再重开。如果要明确验证进程重启，可在允许使用终端的环境下执行下面的命令。 |
| 检查旋转 | 用模拟器的旋转按钮，在题目要求时验证横竖屏、滚动区域和数据是否正确。 |

终端命令中的 `com.example.myapp` 要换成自己的应用 ID。命令前的注释为 PowerShell 语法。

```powershell
# 查看已连接的模拟器或设备，确认设备状态是 device。
adb devices

# 停止自己的应用进程，随后从模拟器桌面重新打开应用以验证保存。
adb shell am force-stop com.example.myapp
```

如果没有 `adb` 命令，可用 **SDK Manager** 找 Android SDK 路径，再看其中的 `platform-tools`。安装、启动和普通功能测试也能在 Android Studio 内完成。[虚拟设备管理](https://developer.android.com/studio/run/managing-avds) · [Android Debug Bridge](https://developer.android.com/tools/adb)

## 06 BMI 类应用怎么拆

### 先把英文题面拆成可测试的要求

| 题面用词 | 需要做的事 |
| --- | --- |
| **Input / accept** | 记下字段数量、单位、数字或文字类型、是否允许小数。 |
| **Validate / reject** | 列出空白、非法格式、超范围等情况，并决定在界面哪里提示错误。 |
| **Calculate / convert** | 先把公式和单位写清楚。看是否有整数除法、除以零或四舍五入要求。 |
| **Display / classify** | 记下单位、精度、类别和边界是否包含等号。 |
| **Navigate / persist** | 题目要求时，再增加页面跳转或保存。 |

### 按这个顺序实现

1. **搭布局。** 在 Android Studio 打开布局 XML。需要编辑 XML 时切到 **Code** 或 **Split**。普通表单可以用纵向 `LinearLayout`，从上到下放字段标签、输入框、计算按钮和结果文字。每个要绑定或交互的控件用唯一 ID。界面放不下时再根据需要加入滚动容器。
2. **绑定控件。** Activity 的 `onCreate` 先调用 `setContentView`，再用 `findViewById` 找输入框、按钮和输出文字。XML 的 `@+id/...` 和 Java 中的 `R.id...` 要对应到同一个名字。
3. **接按钮事件。** 用 `setOnClickListener` 注册点击事件。先考虑点下按钮需要读哪些输入，最后需要更新哪些文字或跳转哪个页面。
4. **校验输入。** 读取字符串并清理前后空白。空白、非法数字、题目不允许的零或负数要单独处理。解析或计算失败时显示清楚的字段提示。如果界面已有旧结果，按题意清空或标记，避免误把旧结果视为当前结果。
5. **计算和分类。** 先统一单位，再代入公式，用与题目精度匹配的数值类型。按题意判断原值或舍入值，测试每个类别边界的左右两侧。
6. **显示结果。** 结果要有数值、单位及题目要求的类别。把小数位格式化放到显示阶段，避免在中间计算过早丢精度。
7. **跑一次。** 测一组能手算出来的正常输入，然后马上试一组非法输入。单页两个用例都通过后，再做后续要求。

官方文档可按 API 名称搜索。[布局资源](https://developer.android.com/guide/topics/resources/layout-resource) · [View 接口](https://developer.android.com/reference/android/view/View) · [输入框类型](https://developer.android.com/develop/ui/views/touch-and-input/keyboard-input/style)

## 07 第二页面和保存数据

### 题目要求第二个页面

1. 用 Android Studio 的 Activity 模板创建页面。创建完先确认 Activity 有源码、若需要有布局文件，并在 `AndroidManifest.xml` 检查页面声明。
2. 主页面准备 **explicit Intent（显式 Intent）**，指定目标 Activity，用 `putExtra` 逐项添加要传的数据。
3. 详情页用 `getIntent` 取得 Intent，再用对应类型的 `getDoubleExtra`、`getIntExtra` 或 `getStringExtra` 读取。发送方与接收方的**键名和数据类型**要一致。
4. 各页面显示后检查返回。详情页遇到缺失数据时，根据题目给出的行为处理。

### 题目要求重开后保留数据

| 要保留的东西 | 可以参考的做法 |
| --- | --- |
| 少量字段、用户设置、上一次输入 | 参考课程里的 `SharedPreferences`。用同一文件名和键保存、读取，并给空记录设置合理默认值。 |
| 多条有日期或需要筛选的历史记录 | 对照题面指定技术。Assignment 05 中的 `BmiDatabase.java` 可用来复习表结构和查询写法。 |
| 只要求跨页面交接当前数据 | 用题目允许的页面参数传递方式，先把发送、接收和返回做对。 |

验证保存时，先写入明显可识别的值，再退出并重开 App。确认界面显示的是保存过的值，需要时也测首次启动的空数据场景。[Intent 文档](https://developer.android.com/guide/components/intents-filters) · [SharedPreferences 文档](https://developer.android.com/training/data-storage/shared-preferences)

## 08 在已交的 BMI 作业里怎么查

从 **Canvas → EE5415 → Assignments** 打开自己已经提交的 **Assignment 02** 或 **Assignment 05**，进入提交详情找原附件。解压后在下面的文件里搜索方法名。老师的机房练习建议是用机房模板先创建新工程，按需要参考相关文件。[Assignment 02](https://canvas.cityu.edu.hk/courses/70618/assignments/334548) · [Assignment 05](https://canvas.cityu.edu.hk/courses/70618/assignments/340070) · [机房练习建议](https://canvas.cityu.edu.hk/courses/70618/discussion_topics/643550)

| 想查的功能 | 作业及文件 | 搜索词 |
| --- | --- | --- |
| XML 布局、输入框、按钮 ID | A02 · `BMI_Source/activity_main.xml` | `EditText`、`calculate_button` |
| 按钮响应和输入校验 | A02 · `BMI_Source/MainActivity.java` | `setOnClickListener`、`parseDouble` |
| 公式和分段判断 | A02 · `BMI_Source/BmiCalculator.java` | `calculate` |
| 第二页传值和收值 | A02 · `BMI_Source/MainActivity.java` 与 `ReportActivity.java` | `putExtra`、`getDoubleExtra` |
| 保存个人输入 | A05 · 模块下的 `MainActivity.java` | `getSharedPreferences("profile"` |
| 多条记录与查询 | A05 · 模块下的 `BmiDatabase.java` | `SQLiteOpenHelper`、`insertWithOnConflict` |

## 09 Android Studio 日常操作

| 操作 | Windows 默认键位或菜单 |
| --- | --- |
| 找菜单命令 | **Ctrl+Shift+A**，搜索英文动作名。例如 *Device Manager*、*Sync Project* 或 *Logcat*。 |
| 找类或文件 | 按两次 **Shift** 打开 **Search Everywhere**，输入文件名或方法名。 |
| 当前文件查找 | **Ctrl+F**。 |
| 在项目中查找文字 | **Ctrl+Shift+F**。 |
| 查看可用的普通补全 | **Ctrl+Space**。考试中按规则使用常规 IDE 功能，避开 Gemini、Copilot 等 AI 功能。 |
| 按缩进和格式整理代码 | **Ctrl+Alt+L**。 |
| 选择设备并运行 | 工具栏选运行配置 **app**、模拟器，再点 **Run ▶**。 |
| 看构建失败原因 | **View → Tool Windows → Build**，展开失败任务，找第一处具体错误。 |
| 看运行时错误 | **View → Tool Windows → Logcat**，选模拟器，用当前应用包名缩小范围。 |

机房若调整过快捷键，使用菜单或 **Find Action**。[Android Studio 官方快捷键](https://developer.android.com/studio/intro/keyboard-shortcuts) · [Logcat](https://developer.android.com/studio/debug/logcat)

## 10 按症状排错

| 症状 | 先检查 | 接着做 |
| --- | --- | --- |
| **Gradle Sync failed** | Build 输出最早一条具体报错。 | 对照是 SDK、JDK、依赖下载还是路径权限问题，再处理相应项。 |
| **SDK location not found** | SDK Manager 里的 SDK 位置。 | 让 Android Studio 使用机房实际的 SDK 目录，再同步。 |
| **Failed to find target with hash string android-XX** | 指定平台是否装在 SDK Manager 的 SDK Platforms。 | 优先使用题面及机房配置要求指向的版本，缺平台时告知监考。 |
| **No devices** | Device Manager 中设备是否已启动，顶部运行目标是否选对。 | 等 Android 主屏，再检查设备列表。必要时点 **Stop** 后重启模拟器。 |
| **模拟器一直黑屏或卡在启动画面** | 区分首次启动耗时和快照损坏。 | 用 **Device Manager → 设备菜单 → Cold Boot**。**Wipe Data** 会清空该虚拟设备的数据，应先确认可删除，机房按监考安排处理。 |
| **INSTALL_FAILED_OLDER_SDK** | 模拟器 API 是否低于项目的 `minSdk`。 | 选 API 足够的已安装镜像。题目规定最低版本时按规定处理。 |
| **Cannot resolve symbol R / R.id** | XML 是否有错误、控件 ID 是否一致，Java 是否误用了 `android.R`。 | 先修正最早的 XML 或资源错误，再重新构建。 |
| **点按钮没有反应** | Activity 用的是否是当前布局，监听器是否绑到了正确 ID。 | 逐个验证点击、读取、校验、设置输出哪一步没有走到。 |
| **点按钮立刻崩溃** | Logcat 选择设备和应用包名，可用 `package:mine is:crash` 缩小范围。 | 找异常名称，以及堆栈中第一条指向自己 Activity 的行。 |
| **NumberFormatException** | 是否直接解析了空字段、含单位的文字或不支持的小数格式。 | 在转数字之前清理和校验输入，失败时给对应字段提示。 |
| **NullPointerException 在操作控件时出现** | 查布局加载顺序和当前布局中该 ID 是否存在。 | 确认先加载布局，再绑定和操作控件。多个布局版本要分别检查。 |
| **结果相差 10、100 或 60 倍** | 单位或转换方向。 | 用一组能手算的数值验证每个中间步骤。 |
| **详情页显示默认值或空值** | 发送与接收的键名、参数类型和目标 Activity。 | 对照实际字段逐项修正，再从主页面重新计算并跳转。 |
| **保存后重开没有数据** | 文件名、键、值类型、写入操作是否提交。 | 确认保存时机，并用退出重开检查持久化。 |

构建失败看 **Build**，App 在设备上运行后出错看 **Logcat**。两种问题分别定位。[构建疑难](https://developer.android.com/studio/run) · [模拟器快照](https://developer.android.com/studio/run/emulator-snapshots) · [Logcat 调试](https://developer.android.com/studio/debug/logcat)

## 11 验收用例

| 用例 | 验收方式 |
| --- | --- |
| 一组正常值 | 先手算期望值，再在模拟器对照数值、单位、类别和小数位。 |
| 一组小数 | 如果题目允许小数，检查两字段输入、计算和显示。 |
| 空白或非法输入 | 验证错误落在对应字段，应用不崩溃。 |
| 零、负数、超范围 | 按题目判定合法性。分母为零的情况要单独试。 |
| 精确分界、略低和略高 | 对每个分段边界确认等号属于哪个类别。 |
| 改值后连续算两次 | 第二次必须对应新输入，错误输入时检查旧结果如何处理。 |
| 详情、返回、重开 | 仅在题目要求这些功能时逐项确认。 |

## 12 APK 源码和提交

1. 先从正式题面确认交付物、文件名、有无签名或截图要求及提交入口。
2. 如果题目要 **debug APK**，用 **Build → Generate Bundle(s) / APK(s) → Generate APK(s)**。旧版本菜单可能叫 **Build Bundle(s) / APK(s) → Build APK(s)**。构建成功弹窗可点 **Locate**，也可以在项目的 `app/build/outputs/apk/debug/` 找 `app-debug.apk`。
3. 若 Android Studio 菜单暂时不可用，且机房允许用终端，可以在工程根目录运行下面命令。项目必须已有 Gradle Wrapper 和可用的 SDK。

```powershell
# 在有 Gradle Wrapper 的项目根目录生成应用模块的调试 APK。
.\gradlew.bat :app:assembleDebug
```

4. 如果题目要**工程源码**，打包前打开项目根目录，确认有 `app/src/main`、模块和项目构建文件、以及模板生成的 Gradle Wrapper。按题目要求放入 ZIP。若只要求可重建源码，可以排除构建缓存和含机器绝对路径的 `local.properties`。APK 或截图按题面放在指定位置。
5. 用资源管理器打开 ZIP，看里面是否有工程所需文件，并检查 APK 文件修改时间是否对应最后一次构建。
6. 按老师指定入口提交。若通过 Canvas，上传完还要完成正式提交步骤，在页面确认状态和附件名称。留一点时间应对重新上传。

[生成调试或发布 APK](https://developer.android.com/build/build-for-release) · [Gradle 构建命令](https://developer.android.com/build/building-cmdline)

## 13 三小时怎么分配

这是练习用分配，根据题面难度和交付要求调整。考场前十分钟单独用于设置。

| 计时 | 达成目标 |
| --- | --- |
| 0–10 分钟 | 圈出题目必做功能、语言与布局要求、交付文件。 |
| 10–75 分钟 | 新建工程，在模拟器跑通默认页，并实现第一条完整功能流程。 |
| 75–125 分钟 | 按题面补第二页、保存或其他要求，做完一项就实跑一次。 |
| 125–150 分钟 | 跑边界和非法输入，复核界面、单位和功能。 |
| 150–180 分钟 | 生成最新版 APK，整理源码和题面指定附件，提交并核对状态。 |

## 14 考前演练与原始文档

用一个平均速度或分账应用做练习。给自己设一个最小验收目标，例如“输入两项正确得数，空输入能报错”。做完后再加详情页与保存。最后从同一个项目生成 APK，确认自己能在文件资源管理器找到它。

| 需要复习 | 原始文档 |
| --- | --- |
| 创建项目、设备和运行 | [新建工程](https://developer.android.com/studio/projects/create-project) · [虚拟设备](https://developer.android.com/studio/run/managing-avds) · [运行应用](https://developer.android.com/studio/run) |
| 配置 SDK 与 JDK | [SDK Manager](https://developer.android.com/studio/intro/update) · [Gradle JDK](https://developer.android.com/build/jdks) |
| XML、输入、传参、存储 | [布局资源](https://developer.android.com/guide/topics/resources/layout-resource) · [输入框类型](https://developer.android.com/develop/ui/views/touch-and-input/keyboard-input/style) · [Intent](https://developer.android.com/guide/components/intents-filters) · [SharedPreferences](https://developer.android.com/training/data-storage/shared-preferences) |
| 日志和安装包 | [Logcat](https://developer.android.com/studio/debug/logcat) · [APK 构建](https://developer.android.com/build/build-for-release) |
| 机房配置与规则 | [B7701 配置](https://canvas.cityu.edu.hk/courses/70618/discussion_topics/643542) · [最新考试公告](https://canvas.cityu.edu.hk/courses/70618/announcements/648700) |
