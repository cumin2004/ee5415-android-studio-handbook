# 把 Assignment 02 的 BMI 改成普通计算器

这份考前改造指南从已有的 Java + XML BMI 项目出发。四则运算版按两个输入框、四个操作按钮和同页显示结果来讲。题目如果规定数字键盘、第二页、保存或不同的小数位，按对应章节调整。先用课程里的 A02 项目对照文件和方法名。[A02 中文源码讲解](A02-BMI.md)

## 目录

- [从哪几个文件开始](#files)
- [改 XML：两个数字框、四个按钮、一个结果框](#layout)
- [改 Java：绑定控件和替换计算方法](#java)
- [题目要求数字键盘怎么办](#keypad)
- [题目换成其他计算公式怎么办](#formulas)
- [题目要求第二页、保存或架构怎么办](#extensions)
- [改完怎么检查](#checks)

<a id="files"></a>

## 1. 从哪几个文件开始

| 文件 | 在 BMI 里先搜 | 改四则运算时做什么 |
| --- | --- | --- |
| `res/layout/activity_main.xml` | `height_input`、`weight_input`、`age_input`、`gender_group`、`calculate_button` | 把身高和体重框改成第一个数和第二个数；移除年龄和性别控件；把一个 Calculate 按钮换成四个运算按钮；加一个结果 `TextView`。 |
| `MainActivity.java` | `onCreate()`、`readAge()`、`readPositiveNumber()`、`showReport()` | 改控件绑定；移除年龄和性别逻辑；把正数校验换成有限数字校验；把跳转报告页换成计算后调用 `resultText.setText(...)`。 |
| `BmiCalculator.java` | `calculate(heightCm, weightKg)` | 四则运算版的四个公式写在新 `calculate(char operator)`。如果题目要求业务逻辑独立，把公式放进单独的数学类，再由 Activity 调用。 |
| `res/values/strings.xml` | `app_name`、BMI 输入提示和错误文案 | 把应用名改为 Calculator，并添加下节列出的新字符串。旧的 BMI 字符串可以等项目跑通再清理。 |
| `ReportActivity.java`、`activity_report.xml`、`AndroidManifest.xml` | `ReportActivity` | 同页展示时去掉 `startActivity()`。清理旧报告页时，删除它在 Manifest 中的 `<activity>` 声明及其 Java、布局，再检查还有没有其他地方引用 BMI 计算类或旧资源。有第二页要求时按第 6 节改。 |

如果直接在 A02 项目副本上改，保留项目原有的 `package`、Gradle 和 SDK 设置。如果按题目新建 Java 的 Views 工程，用新工程自己的 `package`，并在模板的 `super.onCreate(...)` 和 `setContentView(...)` 之后绑定控件。

<a id="layout"></a>

## 2. 改 XML

`activity_main.xml` 保留现有的 `ScrollView` 和它里面的纵向 `LinearLayout`。只替换 `LinearLayout` 内原来的 BMI 表单控件，保留外层布局的宽、高和间距。下面是两个输入框。`numberSigned` 允许输入负号，`numberDecimal` 允许小数，实际合法性仍由 Java 检查。

```xml
<!-- 第一个数对应 Java 中的 firstInput。 -->
<EditText
    android:id="@+id/first_input"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:hint="@string/first_number"
    android:inputType="numberSigned|numberDecimal" />

<!-- 第二个数对应 Java 中的 secondInput。做除法时它是除数。 -->
<EditText
    android:id="@+id/second_input"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:hint="@string/second_number"
    android:inputType="numberSigned|numberDecimal" />
```

用下方的 `<Button>` 写一个加法按钮，再复制三次，按表改 ID 和字符串引用。每个控件都必须有自己的 ID。

```xml
<!-- 加法按钮。其余三个按钮复用相同布局参数。 -->
<Button
    android:id="@+id/add_button"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="@string/add" />

<!-- 把这个结果框放在四个按钮之后，Java 会更新其文字。 -->
<TextView
    android:id="@+id/result_text"
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
```

| 按钮 | `android:id` | `android:text` |
| --- | --- | --- |
| 加 | `@+id/add_button` | `@string/add` |
| 减 | `@+id/subtract_button` | `@string/subtract` |
| 乘 | `@+id/multiply_button` | `@string/multiply` |
| 除 | `@+id/divide_button` | `@string/divide` |

在 `strings.xml` 的 `<resources>` 内加下列资源。如果已有 `app_name`，直接更新它的文本。

```xml
<!-- 两个输入框的英文提示。 -->
<string name="first_number">First number</string>
<string name="second_number">Second number</string>
<!-- 四个按钮的文字。 -->
<string name="add">+</string>
<string name="subtract">−</string>
<string name="multiply">×</string>
<string name="divide">÷</string>
<!-- 在输入框或结果框显示的错误。 -->
<string name="invalid_number">Enter a valid number</string>
<string name="division_by_zero">Cannot divide by zero</string>
<string name="result_out_of_range">Result is out of range</string>
<!-- 保留两位小数的结果格式。按题目的显示要求调整 2。 -->
<string name="calculator_result">Result: %1$.2f</string>
```

<a id="java"></a>

## 3. 改 Java

在 `MainActivity.java` 保留原来的 `onCreate()` 声明、`super.onCreate(...)` 和 `setContentView(R.layout.activity_main)`。把身高、体重、年龄、性别相关的字段和绑定换成下面这两段。继续使用 A02 已有的 `EditText` 和 `TextView` 导入。

把这三行放在 `MainActivity` 类里、`onCreate()` 外。

```java
private EditText firstInput; // 存第一个操作数的输入框。
private EditText secondInput; // 存第二个操作数的输入框。
private TextView resultText; // 存用于显示答案的文字框。
```

把这七行放在 `onCreate()` 的 `setContentView(...)` 后，替换原来的 `findViewById(...)` 绑定和 `showReport()` 监听。XML 的按钮可以显示 `×` 和 `÷`，Java 用 `*` 和 `/` 传递操作符。

```java
firstInput = findViewById(R.id.first_input); // 绑定 XML 中的第一个输入框。
secondInput = findViewById(R.id.second_input); // 绑定第二个输入框。
resultText = findViewById(R.id.result_text); // 绑定用于显示结果的文字框。
findViewById(R.id.add_button).setOnClickListener(view -> calculate('+')); // 点加号执行加法。
findViewById(R.id.subtract_button).setOnClickListener(view -> calculate('-')); // 点减号执行减法。
findViewById(R.id.multiply_button).setOnClickListener(view -> calculate('*')); // 点乘号执行乘法。
findViewById(R.id.divide_button).setOnClickListener(view -> calculate('/')); // 点除号执行除法。
```

在类里用下面两个方法替换 `readAge()`、`readPositiveNumber()` 和 `showReport()`。BMI 的 `number > 0` 必须去掉，否则 `-3 + 5`、`0 × 6` 都会被拒绝。除法单独检查第二个数是不是零。

```java
private Double readNumber(EditText input) { // 读取任意一个操作数，可返回 null 表示无效。
    input.setError(null); // 清掉这个输入框上一次计算产生的错误。
    try { // 文本无法转换时，转到下方的统一错误提示。
        double number = Double.parseDouble(input.getText().toString().trim()); // 读文字、去首尾空格，再解析小数。
        if (!Double.isNaN(number) && !Double.isInfinite(number)) { // 接受零和负数，排除非有限值。
            return number; // 解析成功就把数值返回给计算方法。
        } // 结束数字范围检查。
    } catch (NumberFormatException ignored) { // 空输入或非法文本由下方提示处理。
    } // 结束解析错误处理。
    input.setError(getString(R.string.invalid_number)); // 在当前错误输入框上标注原因。
    return null; // 让调用方停止本次计算。
} // 结束取值方法。

private void calculate(char operator) { // 四个按钮都走这里，operator 决定算哪一种。
    resultText.setText(""); // 清掉上一笔结果，避免输入有错时仍显示旧答案。
    Double first = readNumber(firstInput); // 校验第一个数。
    Double second = readNumber(secondInput); // 校验第二个数，两栏都能独立提示错误。
    if (first == null || second == null) return; // 校验完成前不参与运算。

    double result; // 存本次四则运算的结果。
    switch (operator) { // 根据按钮传入的运算符选择公式。
        case '+': // 求和。
            result = first + second; // 第一个数加第二个数。
            break; // 加法分支结束，进入统一显示流程。
        case '-': // 求差。
            result = first - second; // 从第一个数减去第二个数。
            break; // 减法分支结束。
        case '*': // 求积。
            result = first * second; // 两个数相乘。
            break; // 乘法分支结束。
        case '/': // 求商。
            if (second == 0.0) { // 正零和负零都不能做除数。
                secondInput.setError(getString(R.string.division_by_zero)); // 在第二个输入框提示错误。
                return; // 提前退出，避免除以零。
            } // 结束除数检查。
            result = first / second; // 用 double 相除，保留小数部分。
            break; // 除法分支结束。
        default: // 防止以后有按钮传来未定义的操作符。
            throw new IllegalArgumentException("Unknown operator: " + operator); // 直接指出代码接线错误。
    } // 结束运算符分支。

    if (Double.isInfinite(result) || Double.isNaN(result)) { // 检查超大输入造成的计算溢出。
        resultText.setText(R.string.result_out_of_range); // 在结果区显示范围提示。
        return; // 当前结果无法正常显示。
    } // 结束结果范围检查。
    resultText.setText(getString(R.string.calculator_result, result)); // 格式化结果并在本页显示。
} // 结束四则运算方法。
```

移除当前类里未再使用的 `Intent`、`View` 和 `RadioGroup` 导入。先运行并点通四个按钮，再按需要清理旧报告页。若题目只要一个 Calculate 按钮加一个运算选择器，在该按钮点击时读取选中的操作符，再调用同一个 `calculate(...)`。

<a id="keypad"></a>

## 4. 题目要求数字键盘怎么办

看到 **numeric keypad、digit buttons、equals、clear**，就按题目搭数字键盘。把两个 `EditText` 换成一个用于显示当前输入的 `TextView`，在布局加 `0` 到 `9`、小数点、四个运算符、`=` 和 `C`。BMI 的 `findViewById` 和监听器写法仍然适用，需要更换的是取数方式。

| 用户按键 | 在 `MainActivity` 里保存或修改什么 |
| --- | --- |
| 数字或小数点 | 用 `StringBuilder` 保存当前输入的数字文本。小数点已经存在时忽略第二个点；空输入时点小数点可先写成 `0.`。 |
| `+ − × ÷` | 有当前输入时，解析成 `firstNumber`，把按钮对应的字符记为 `pendingOperator`，然后清空当前输入，开始接收第二个数。 |
| `=` | 有待执行的操作符和第二个数时，取 `firstNumber`、`pendingOperator` 和第二个数执行同样的四则分支。显示结果，并按题目规定决定下一次按键从结果继续还是开始新运算。 |
| `C` | 同时清空当前输入、第一操作数、待执行运算符和显示状态。 |

题面如果要求完整表达式和乘除优先级，例如 `2 + 3 × 4` 得 `14`，需要先收集整串表达式，再按优先级计算。上表对应的是两个操作数一次运算的要求。

<a id="formulas"></a>

## 5. 题目换成其他计算公式怎么办

先在题面标出输入、单位、合法范围、公式、显示格式，再顺着 XML → Java 读取 → 校验 → 公式 → 展示逐层改。下表给的是替换位置和规则，阈值和单位以题面为准。

| 变体 | XML 改哪里 | Java 改哪里 | 用一组数核对 |
| --- | --- | --- | --- |
| **温度换算** | 一个温度框保留带符号小数键盘，加两个方向按钮或选择器。 | 保留 `readNumber()`；在计算分支做 `c * 9.0 / 5.0 + 32.0` 或 `(f - 32.0) * 5.0 / 9.0`。结果文案带 `°C` 或 `°F`。 | `-40 °C → -40 °F`，`32 °F → 0 °C`。 |
| **平均速度** | 两个输入框分别标记 `Distance (km)` 和 `Duration (min)`，一个 Calculate 按钮。 | 按题面检查距离范围，时长必须大于零；按 `distance / (minutes / 60.0)` 计算。题目要求分类时，用原始速度与阈值比较，再格式化显示。 | `10 km / 20 min → 30.00 km/h`；`0 min` 报错。 |
| **折扣或小费** | 两框标记金额和百分比，结果区按题意显示优惠金额、小费金额或最终应付。 | 百分数 `15` 先除以 `100.0`。折后价用 `price * (1.0 - rate / 100.0)`；小费总额用 `bill * (1.0 + rate / 100.0)`。范围和货币舍入按题面。 | `100` 打 `15%` 折扣得 `85.00`；加 `15%` 小费应付 `115.00`。 |

<a id="extensions"></a>

## 6. 题目要求第二页、保存或架构怎么办

- **第二页 `ReportActivity`**。沿用 A02 的 `Intent` 和 Manifest 声明。先在首页校验并计算，把结果写入 `putExtra("result", result)`；第二页用 `getDoubleExtra("result", Double.NaN)` 读取。还需显示两个操作数和运算符时，为它们单独定义键，两页的键名和类型必须一样。把 `activity_report.xml` 的年龄、性别和 BMI 分类控件改为题目要求的明细，保留 `finish()` 返回按钮。[A02 的页面跳转源码](A02-BMI.md#file-2)
- **保存上一次输入**。按 A05 中 `getSharedPreferences(...)` 的做法，给新程序定义明确的键，比如 `first_input`、`second_input`、`operator`。成功后用 `edit().putString(...).apply()` 保存，在 `onCreate()` 绑定控件后用 `getString(...)` 恢复。如果题目说要保存多笔、能按日期浏览，再看 A05 的 SQLite 写入和查询。[A05 的偏好设置与历史记录源码](A05-BMI-MVVM.md)
- **指定 MVP、MVVM 或 MVI**。先看题干要求哪一种。MVP 把计算接到 `BmiPresenter` 对应的计算入口，结果通过 View 接口回传；MVVM 从 ViewModel 的 `calculate()` 更新 `LiveData`；MVI 从 `Calculate` 事件进入 `dispatch`，在 `reduce` 里计算并返回新状态。三个版本都要换输入字段、业务公式和结果模型。[MVP](A03-BMI-MVP.md) · [MVVM](A03-BMI-MVVM.md) · [MVI](A03-BMI-MVI.md)

<a id="checks"></a>

## 7. 改完怎么检查

| 操作 | 预期 |
| --- | --- |
| `-3` 和 `5` 点加号 | `2.00`，验证负数可输入。 |
| `2` 和 `7` 点减号 | `-5.00`，验证操作数顺序。 |
| `0` 和 `6` 点乘号 | `0.00`，验证零可参与乘法。 |
| `2` 和 `4` 点除号 | `0.50`，验证没有做整数除法。 |
| `3` 和 `0` 点除号 | 第二个输入框报错，旧结果清空。 |
| 任意一栏留空再点按钮 | 对应输入框报错，旧结果清空。 |

如果 `R.id` 标红，先看 XML 能否解析，再逐字对照 ID 和字符串名。点按钮无反应，检查监听器是否在 `setContentView(...)` 之后注册。出现启动崩溃时，打开 Logcat 找 `FATAL EXCEPTION` 后第一条指向自己 `MainActivity.java` 的行。[Android Studio 与模拟器手册](Android-Studio.md) · [Android 的数字键盘类型](https://developer.android.com/reference/android/text/InputType)

本页供考前复习编写过程。考场资料使用范围以课程公告和监考说明为准。[EE5415 考试公告](https://canvas.cityu.edu.hk/courses/70618/discussion_topics/648700)
