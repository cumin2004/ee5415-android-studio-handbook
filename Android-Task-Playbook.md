# 从 BMI 作业改出其他 Android 应用

这里按题目要求的功能查改法。示例结构沿用你作业里的 Java + XML。下面这些是考前复习方向，实际要做的页面、架构和校验以题面为准。

| 题面出现这些词 | 先看这一节 | 可以复用的作业 |
| --- | --- | --- |
| `register`, `book`, `show confirmation` | [报名或预约表单](#form) | A02 的输入校验和报告页，A05 的日期选择 |
| `display a list`, `filter`, `view details` | [商品、课程或食谱目录](#catalog) | A02 的第二页，A05 的历史页 |
| `add`, `edit`, `delete`, `persist records` | [待办、记账或打卡记录](#crud) | A05 的 SQLite 与历史页 |
| `multiple choice`, `next question`, `score` | [单选答题和计分](#quiz) | A02 的 `RadioGroup` 与报告页 |
| `countdown`, `pause`, `resume`, `reset` | [倒计时或秒表](#timer) | A02 的按钮监听和数字校验 |
| `remember`, `restore on restart`, `settings` | [个人设置和上次输入](#settings) | A05 的 `SharedPreferences` |
| `fetch`, `API`, `JSON`, `loading`, `retry` | [加载网络列表](#network) | A04 的后台请求与界面状态 |
| `convert`, `calculate`, `validate numbers` | [四则运算、温度、速度、折扣](BMI-to-Calculator.md) | A02 的取值和计算流程 |

## 从哪个文件开始改

1. 在纸上区分 **输入、动作、显示内容、需要保存的数据**。例如「记账」的输入是金额和分类，动作是保存和删除，显示是账单和合计，需要保存的是每笔账。
2. 先在 `activity_main.xml` 写屏幕上的控件。`MainActivity.onCreate()` 先 `setContentView(...)`，再用 `findViewById(...)` 绑定同名 ID，最后注册点击监听。
3. 把 A02 的 `showReport()` 换成题目对应的方法，例如 `saveEntry()`、`submitAnswer()` 或 `startTimer()`。进方法后按「读值 → 校验 → 更新数据 → 更新界面」执行。若有第二页，用 A02 的 `Intent`；若题目明确要求 MVP、MVVM 或 MVI，就把业务逻辑接到 A03 的对应层。[A02 源码](A02-BMI.md) · [A03 MVP](A03-BMI-MVP.md) · [A03 MVVM](A03-BMI-MVVM.md) · [A03 MVI](A03-BMI-MVI.md)

<a name="form"></a>

## 1. 报名表单或预约确认

**题目可能怎么写**：填写姓名、类别、人数和日期。各项合法后点 Submit，在新页面显示确认信息。

| 文件 | 从 BMI 改什么 |
| --- | --- |
| `activity_main.xml` | 把身高换成姓名 `EditText`，体重换成人数 `EditText`，性别换成类别 `RadioGroup` 或 `Spinner`，加日期按钮。人数使用 `number` 键盘。 |
| `MainActivity.java` | 把 `readAge()` 换成 `readPartySize()`，按题面验证整数范围；日期可以参照 A05 的 `DatePickerDialog`。把 `showReport()` 改成 `submitBooking()`。 |
| `activity_report.xml`、`ReportActivity.java` | 把 BMI 值、分组和分类改成姓名、人数、类别和日期；保留返回按钮。 |

依次检查姓名去首尾空格后是否为空、人数范围、类别是否已选、日期是否允许。`DatePickerDialog` 回调里的 `month` 从 0 开始，转换成年月日的显示值时加 1；如果把它传给 `Calendar.set(...)`，继续用原来的 0 基月份。校验通过后，用不同的键传值。

```java
Intent intent = new Intent(this, ReportActivity.class); // 指定 A02 已注册的确认页。
intent.putExtra("name", name); // 姓名按字符串传递。
intent.putExtra("party_size", partySize); // 人数用 int，partySize 在校验通过后取原始整数值。
intent.putExtra("date", dateLabel); // 传已经格式化好的日期文字。
startActivity(intent); // 校验全部通过后才跳转。
```

在结果页分别用 `getStringExtra("name")`、`getIntExtra("party_size", -1)` 和 `getStringExtra("date")` 读取；类别同样加一个独立键。验证空姓名、人数超范围、未选日期和正常提交四条路径。[A02 的 `putExtra`](A02-BMI.md#file-2) · [A05 的 `DatePickerDialog`](A05-BMI-MVVM.md)

<a name="catalog"></a>

## 2. 商品、课程或食谱目录

**题目可能怎么写**：显示若干课程，能按标题搜索，点一行进详情页。

1. `activity_main.xml` 改成搜索 `EditText`、`ListView` 和空列表提示。用一个纵向 `LinearLayout` 放它们，让 `ListView` 用剩余空间，避免把 `ListView` 套进原来的 `ScrollView`。题面指定 `RecyclerView` 时，按指定控件写 Adapter。
2. `MainActivity` 准备全量 `allItems` 和当前显示 `visibleItems` 两份数据。每个 `Course` 至少有 `id`、`title` 和 `description`。用最简单的 `ArrayAdapter<Course>` 时，让 `Course.toString()` 返回标题作为行文字。
3. `onCreate()` 把同一个可修改的 `visibleItems` 传给 `ArrayAdapter`，调用 `listView.setAdapter(adapter)`。搜索框用 `TextWatcher.afterTextChanged()` 调 `applyFilter(...)`，按标题筛选并通知 Adapter 刷新。
4. 点击时从当前 Adapter 拿对象，带它的 `id` 或必要字段进详情页。筛选后的位置只对应显示结果。详情页继续沿用 A02 的 Manifest 注册和 `Intent` 取值。

```java
String key = query.trim().toLowerCase(Locale.ROOT); // 清掉首尾空格并统一大小写。
visibleItems.clear(); // 清空上一次搜索结果，保留 Adapter 正在使用的集合。
for (Course item : allItems) { // 从全量数据重新匹配，所以删掉搜索词后能恢复原列表。
    if (item.title.toLowerCase(Locale.ROOT).contains(key)) { // 空关键词会匹配所有标题。
        visibleItems.add(item); // 把原始对象加进显示列表，保留它的 ID 和详情。
    } // 结束当前标题判断。
} // 结束筛选。
adapter.notifyDataSetChanged(); // 让 ListView 用筛选后的数据重画。
```

点行可在 `setOnItemClickListener(...)` 中使用 `adapter.getItem(position)`。检查搜索无结果、清空搜索和筛选后点开详情是否正确。[Android ArrayAdapter](https://developer.android.com/reference/android/widget/ArrayAdapter) · [A02 详情页源码](A02-BMI.md#file-3)

<a name="crud"></a>

## 3. 待办、记账或打卡记录

**题目可能怎么写**：显示记录列表。可以新增、编辑和删除，关闭后重开仍能看到之前的记录。

1. 从 A05 的 `BmiDatabase` 改出 `TaskDatabase`。BMI 以日期定位同一天的记录，同一天可以覆盖。待办和记账通常允许同一天写多条，给每条记录单独的 `_id`。
2. 从 A05 的 `HistoryActivity` 复用「查询 → 更新列表」流程。主页面放列表和 Add 按钮。编辑页放标题输入框、Completed 复选框和 Save。新增时编辑页无 ID；编辑时从 `Intent` 读 `_id`、查询并回填。
3. 数据库按 `insert`、`query`、`update`、`delete` 写四个方法。写入用 `ContentValues`；查询时用 `try (Cursor cursor = ...)`，逐行取出 `_id`、标题和状态后再更新列表。

```sql
-- 每条待办有独立的 ID，允许多条同名任务。
CREATE TABLE tasks (
    _id INTEGER PRIMARY KEY,
    title TEXT NOT NULL,
    done INTEGER NOT NULL DEFAULT 0
);
```

```java
ContentValues values = new ContentValues(); // 准备要写入的一条记录。
values.put("title", title); // title 是已去空白并校验通过的标题。
values.put("done", done ? 1 : 0); // 用 1 和 0 对应已完成和未完成。
if (taskId == null) { // 没有初始 ID 表示新增；taskId 可用 Long 表示是否存在。
    db.insertOrThrow("tasks", null, values); // 新增一行，SQLite 生成 ID。
} else { // 有 ID 时更新对应行。
    int changed = db.update("tasks", values, "_id = ?", new String[]{taskId.toString()}); // 参数化匹配 ID。
    if (changed != 1) return; // 要求更新一行；实际界面在返回前应提示记录已不存在。
} // 结束新增或编辑。
finish(); // 保存成功后返回，主列表重新查询。
```

删除用 `db.delete("tasks", "_id = ?", ...)`，按题面要求确认后执行。主列表可以在 `onResume()` 重新查询，返回后刷新编辑结果。记账改成金额、类别、日期和备注；需要精确货币计算时以整数分保存，并按题面规定小数位。打卡如果明确规定一天只能有一次，可对日期加唯一约束。验证新增两条同名记录、只更新其中一条、删除后重开。[A05 的 `BmiDatabase` 和 `HistoryActivity`](A05-BMI-MVVM.md) · [Android SQLiteDatabase](https://developer.android.com/reference/android/database/sqlite/SQLiteDatabase)

<a name="quiz"></a>

## 4. 单选答题和计分

**题目可能怎么写**：每次展示一题，选择答案后点 Next，最后显示得分，支持 Restart。

1. `activity_main.xml` 保留 A02 的 `RadioGroup`，把性别按钮改成固定数量的选项。新增题目 `TextView`、题号提示和 Next。`Question` 保存 `prompt`、`options` 与 `correctIndex`。
2. `MainActivity` 保存 `questionIndex` 与 `score`。`renderQuestion()` 读取当前题，填入题目和选项，调用 `clearCheck()`，更新 `第几题 / 总题数`。不要把 Android 资源 ID 当选项序号，用 `if` 或 ID 对照表转成 `0`、`1`、`2`。
3. Next 的点击方法用下面的顺序。最后一题直接显示结果并停用 Next，或者用 A02 的 `ReportActivity` 传入 `score` 和 `total`。如果结束后允许返回题目页，保留完成状态，防止再次提交最后一题。Restart 把题号和得分清零，然后重新渲染。

```java
int chosenId = answerGroup.getCheckedRadioButtonId(); // 读取选中的控件 ID。
if (chosenId == -1) { // 还没有选项时留在当前题。
    showAnswerRequired(); // 用你页面中的提示控件提醒先选择。
    return; // 未选择时既不计分也不推进题号。
} // 结束未选校验。
int chosenIndex = indexForButton(chosenId); // 将控件 ID 对应到选项数组下标。
if (chosenIndex == questions.get(questionIndex).correctIndex) score++; // 每次提交当前题最多加一分。
questionIndex++; // 已提交的题推进一次。
if (questionIndex < questions.size()) { // 检查下标，避免最后一题后继续取数组元素。
    renderQuestion(); // 进入下一题并清除上题选择。
} else { // 题目已经全部提交。
    nextButton.setEnabled(false); // 结束后禁止再次提交。
    showScore(); // 显示 score 和 questions.size()。
} // 结束结果分支。
```

如果要求旋转屏幕后恢复，保存题号、分数和当前选项，重建页面时先恢复数据再渲染。验证不选、选错、最后一题、重开一轮。[A02 单选组检查](A02-BMI.md#file-2) · [Android 保存界面状态](https://developer.android.com/topic/libraries/architecture/saving-states)

<a name="timer"></a>

## 5. 倒计时或秒表

**题目可能怎么写**：输入秒数，支持 Start、Pause、Resume 和 Reset。到零后停住。

1. `activity_main.xml` 保留一个只收整数的秒数框，加剩余时间 `TextView` 和按钮。`MainActivity` 把 BMI 校验替换成 `Long.parseLong(...)` 加题面规定的范围检查，例如 `1` 到 `3600` 秒。校验后乘 `1000L` 得毫秒。
2. Activity 维护 `CountDownTimer timer`、`long remainingMs`、`long deadlineMs`。启动时先取消旧计时器，用 `SystemClock.elapsedRealtime() + durationMs` 计算截止点。`onTick` 更新剩余时间，`onFinish` 显示 `0`，设置 `remainingMs = 0` 并清空计时器引用。
3. 暂停先按下方计算准确剩余时间，再 `timer.cancel()` 并清空引用。继续从 `remainingMs` 启动新计时器。重置取消当前计时器，把剩余时间恢复到本次设定值。运行中禁用 Start，或者规定 Start 的行为就是取消后重开。

```java
deadlineMs = SystemClock.elapsedRealtime() + durationMs; // 用单调时钟记录当前这一轮的截止时刻。
// 暂停时执行下面一行，再取消当前 CountDownTimer。
remainingMs = Math.max(0L, deadlineMs - SystemClock.elapsedRealtime()); // 取按下暂停那刻的实际剩余时间。
// 显示秒数时向上取整，最后不足一秒仍显示 1，结束后显示 0。
long displaySeconds = (remainingMs + 999L) / 1000L; // 在题设的正常时长范围内将毫秒转为显示秒数。
```

Activity 销毁时取消旧计时器。题目要求旋转继续时，保存运行状态、截止点和剩余毫秒，重建后用相同的单调时钟补算。秒表的公式改成「已累计时间 + 本轮起点到现在的差」，界面刷新只负责显示，暂停时累计本轮时长。验证连点 Start、暂停后等待、继续、重置和到零。[Android CountDownTimer](https://developer.android.com/reference/android/os/CountDownTimer) · [Android 保存界面状态](https://developer.android.com/topic/libraries/architecture/saving-states)

<a name="settings"></a>

## 6. 个人设置和上次输入

**题目可能怎么写**：用户填昵称，选择是否开启某个选项。点 Save 后关闭应用，重开时显示之前选的值。

在 `activity_main.xml` 放昵称 `EditText`、`Switch` 和 Save。改 A05 `MainActivity` 里 `getSharedPreferences("profile", ...)` 对应的读取与写入位置。把下面两段分别放进 `onCreate()` 的绑定之后，以及 `saveSettings()` 的校验之后。两处文件名、键和类型要匹配。

```java
SharedPreferences prefs = getSharedPreferences("settings", MODE_PRIVATE); // 打开应用私有设置文件。
nameInput.setText(prefs.getString("nickname", "")); // 第一次运行用空字符串。
reminderSwitch.setChecked(prefs.getBoolean("reminder", false)); // 第一次运行默认关闭。
```

```java
String nickname = nameInput.getText().toString().trim(); // 从界面读取昵称并清理空白。
getSharedPreferences("settings", MODE_PRIVATE).edit() // 打开与读取时相同的设置文件。
        .putString("nickname", nickname) // 用字符型存昵称。
        .putBoolean("reminder", reminderSwitch.isChecked()) // 用布尔型存开关。
        .apply(); // 提交改动。
```

这段只保存开关值。题面若要求定时发通知，还需按题意实现通知调度。测试重开后开关的开和关都能恢复，并验证缺省值。[A05 的 `SharedPreferences` 用法](A05-BMI-MVVM.md) · [Android SharedPreferences 文档](https://developer.android.com/training/data-storage/shared-preferences)

<a name="network"></a>

## 7. 加载网络列表、天气或公告

**题目可能怎么写**：用题面给的 HTTPS API 读取 JSON，在页面展示标题或记录列表，提供加载、失败和重试状态。

| 文件 | 从 A04 改什么 |
| --- | --- |
| `AndroidManifest.xml` | 保留或声明 `android.permission.INTERNET`，权限元素是 `<manifest>` 的子元素。 |
| `ApiClient.java` | 参考 A04 `DeepSeekClient` 的 `HttpURLConnection`、连接超时、读取超时和资源关闭。端点、HTTP 方法、请求参数、响应字段按题目给的 API 配置。普通 GET 不写请求体；需要认证时按题面处理。 |
| `ItemsViewModel.java` | 将 A04 的 `AdviceViewModel` 改为加载记录，保留线程池与界面状态更新。若允许重新搜索或重试，保留请求编号，只有最新编号可以发回结果，避免旧页面内容覆盖新结果。 |
| `activity_main.xml`、`MainActivity.java` | 增加加载提示、错误提示、Retry 和列表。观察状态后分别展示 Loading、Success、Empty 或 Error；点击 Retry 重试调用。 |

例如题面返回 `{"items":[{"title":"..."}]}`，可以把解析放在网络线程成功得到正文以后。顶层若是直接数组，改用 `new JSONArray(body)`。

```java
JSONObject root = new JSONObject(body); // 从 API 响应正文解析顶层 JSON 对象。
JSONArray rows = root.getJSONArray("items"); // 用题面规定的字段找到记录数组。
List<String> titles = new ArrayList<>(); // 准备要交给界面的标题列表。
for (int i = 0; i < rows.length(); i++) { // 按响应顺序逐项读取。
    titles.add(rows.getJSONObject(i).getString("title")); // 将每个记录的 title 加进列表。
} // 结束解析，空数组得到空列表。
```

处理顺序是后台请求 → 检查 HTTP 状态 → 解析 JSON → 回到主线程更新界面。按 A04 的写法，用主线程 `Handler.post(...)` 投递回调，在回调内核对请求编号，再调用 `state.setValue(...)`，由 Activity 观察。连接异常、非成功状态码或响应格式错误应进入错误状态，成功但数组为空时显示空列表提示。验证成功、空数组、断网、重试四种结果。[A04 的请求与状态源码](A04-BMI-MVVM.md) · [Android 网络连接文档](https://developer.android.com/develop/connectivity/network-ops/connecting)

本页用于考前复习实现方法。考试时使用资料的范围以课程公告和监考说明为准。[EE5415 考试公告](https://canvas.cityu.edu.hk/courses/70618/discussion_topics/648700)
