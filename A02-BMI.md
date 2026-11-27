# A02-BMI 中文源码讲解

从 activity_main.xml 的四项输入进入 MainActivity。点击按钮先校验，再传到 ReportActivity。BmiCalculator 统一负责公式及年龄分组。用 170 厘米、60 千克、20 岁检查 20.76 和 Normal range。

## 文件目录

- [app/src/main/java/com/example/bmi/BmiCalculator.java](#file-1)
- [app/src/main/java/com/example/bmi/MainActivity.java](#file-2)
- [app/src/main/java/com/example/bmi/ReportActivity.java](#file-3)
- [app/build.gradle.kts](#file-4)
- [app/src/main/AndroidManifest.xml](#file-5)
- [app/src/main/res/drawable/result_card.xml](#file-6)
- [app/src/main/res/drawable/result_card_warning.xml](#file-7)
- [app/src/main/res/layout/activity_main.xml](#file-8)
- [app/src/main/res/layout/activity_report.xml](#file-9)
- [app/src/main/res/values/strings.xml](#file-10)
- [app/src/main/res/values/styles.xml](#file-11)
- [build.gradle.kts](#file-12)
- [settings.gradle.kts](#file-13)

<a id="file-1"></a>

### app/src/main/java/com/example/bmi/BmiCalculator.java

~~~java
package com.example.bmi; // 使用与两个 Activity 相同的包，让它们可以引用本类。

public final class BmiCalculator { // 定义只处理计算规则的工具类，界面代码放在 Activity 中。
    private static final double[][] BOYS = { // 定义男生查表数据。每行对应一岁，每列是一个分类上界。
            {12.8, 13.1, 18.8, 21.4}, // 6 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {13.0, 13.3, 19.8, 23.0}, // 7 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {13.2, 13.6, 20.9, 24.6}, // 8 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {13.5, 13.8, 21.8, 26.0}, // 9 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {13.8, 14.1, 22.7, 27.3}, // 10 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {14.1, 14.5, 23.6, 28.3}, // 11 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {14.4, 14.8, 24.3, 29.2}, // 12 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {14.7, 15.1, 25.0, 30.0}, // 13 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {15.0, 15.4, 25.5, 30.6}, // 14 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {15.3, 15.8, 26.1, 31.2}, // 15 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {15.6, 16.1, 26.5, 31.7}, // 16 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {15.9, 16.3, 27.0, 32.1}, // 17 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {16.1, 16.6, 27.4, 32.4} // 18 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
    }; // 结束男生查表数据。

    private static final double[][] GIRLS = { // 定义女生查表数据，结构与男生表相同。
            {12.6, 12.8, 18.3, 20.5}, // 6 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {12.8, 13.1, 19.1, 21.8}, // 7 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {13.1, 13.4, 20.1, 23.1}, // 8 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {13.4, 13.7, 21.0, 24.4}, // 9 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {13.7, 14.1, 21.9, 25.6}, // 10 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {14.1, 14.4, 22.7, 26.6}, // 11 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {14.4, 14.8, 23.4, 27.5}, // 12 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {14.8, 15.2, 24.0, 28.3}, // 13 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {15.1, 15.5, 24.6, 28.9}, // 14 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {15.4, 15.8, 25.0, 29.4}, // 15 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {15.7, 16.1, 25.4, 29.7}, // 16 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {15.9, 16.3, 25.7, 30.0}, // 17 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
            {16.1, 16.5, 25.9, 30.3} // 18 岁的一行。四列依次是严重偏低、偏低、可接受和超重各区间的上界。
    }; // 结束女生查表数据。

    private BmiCalculator() { } // 禁止创建工具类对象，外部直接调用静态方法。

    public static double calculate(double heightCm, double weightKg) { // 接收厘米和千克，返回未格式化的 BMI 数值。
        if (!(heightCm > 0) || !(weightKg > 0) // 检查身高体重是否都为正，非数字也不能通过。
                || Double.isInfinite(heightCm) || Double.isInfinite(weightKg)) { // 同时排除正无穷，避免其进入换算。
            throw new IllegalArgumentException("Height and weight must be finite and positive"); // 用参数异常让调用方决定怎样提示。
        } // 结束输入范围检查。
        double heightM = heightCm / 100.0; // 将厘米除以一百，换成公式所需的米。
        double bmi = weightKg / (heightM * heightM); // 体重除以身高米数的平方，得到原始 BMI。
        if (!(bmi > 0) || Double.isInfinite(bmi)) { // 再校验计算结果，处理极端输入导致的数值异常。
            throw new IllegalArgumentException("Values are outside the supported range"); // 抛出范围错误，交给调用页面处理。
        } // 结束输出范围检查。
        return bmi; // 返回原始数值，显示时再格式化。
    } // 结束 BMI 计算方法。

    public static boolean isAdult(int age) { // 按作业规则验证年龄，合法时返回是否走成人分支。
        if (age < 6 || age > 120) { // 只接受六至一百二十周岁，包含两个端点。
            throw new IllegalArgumentException("Age must be from 6 to 120"); // 拒绝超出范围的整岁输入。
        } // 结束范围校验。
        return age > 18; // 十八岁仍走非成人分支，十九岁开始走成人分支。
    } // 结束年龄判断方法。

    public static String adultCategory(double bmi) { // 按作业规定的成人阈值，给原始 BMI 分类。
        if (!(bmi > 0) || Double.isInfinite(bmi)) { // 分类前检查 BMI 是有意义的有限正数。
            throw new IllegalArgumentException("BMI must be finite and positive"); // 无效值不能进入分类分支。
        } // 结束分类前校验。
        if (bmi < 18.5) return "Underweight"; // 严格小于十八点五走体重偏低类别。
        if (bmi < 23.0) return "Normal range"; // 前面的分支已排除低值，此处分段包含十八点五，不含二十三。
        if (bmi < 25.0) return "Overweight"; // 此处分段包含二十三，不含二十五。
        return "Obese"; // 剩下的有效值都大于或等于二十五。
    } // 结束成人分类方法。

    public static String childCategory(double bmi, int age, String gender) { // 按非成人年龄和性别查表，再用作业规定的四个上界分类。
        if (age < 6 || age > 18) { // 先保护数组索引，只接受六至十八岁。
            throw new IllegalArgumentException("Child age must be from 6 to 18"); // 用参数异常表示年龄无法查表。
        } // 结束查表年龄检查。
        double[][] chart; // 声明本次分类使用的阈值表。
        if ("Male".equals(gender)) { // 用常量字符串调用相等比较，即使传入空值也不会抛空指针。
            chart = BOYS; // 选男生阈值表。
        } else if ("Female".equals(gender)) { // 次选女生对应的字符串。
            chart = GIRLS; // 选女生阈值表。
        } else { // 其它字符串无法选择图表。
            throw new IllegalArgumentException("Gender must be Male or Female"); // 抛出参数异常通知调用方。
        } // 结束查表选择。
        double value = Math.round(bmi * 10.0) / 10.0; // 按原作业规则先把 BMI 舍入到一位小数再查表。
        double[] row = chart[age - 6]; // 表的首行对应六岁，因此年龄减六得到下标。
        if (value <= row[0]) return "Severely Underweight"; // 不超过第一列上界，归入严重偏低类别。
        if (value <= row[1]) return "Underweight"; // 上一个分支已排除第一段，此处包含第二列上界。
        if (value <= row[2]) return "Acceptable Weight"; // 第三个区间包含正常范围的上界。
        if (value <= row[3]) return "Overweight"; // 第四个区间包含超重一段的上界。
        return "Severely Overweight"; // 大于第四列上界，归入严重超重类别。
    } // 结束非成人分类方法。

    public static boolean isSevere(String category) { // 判断当前分类是否需要显示特殊样式与提示。
        return "Severely Underweight".equals(category) // 严重偏低需要提示。
                || "Severely Overweight".equals(category); // 严重超重也需要提示，其他分类返回假。
    } // 结束严重程度检测。
} // 结束计算工具类。
~~~

<a id="file-2"></a>

### app/src/main/java/com/example/bmi/MainActivity.java

~~~java
package com.example.bmi; // 指定包名。它要和项目命名空间及该文件的目录对应。

import android.app.Activity; // 使用系统的 Activity 作为输入页面基类。
import android.content.Intent; // 使用 Intent 指定结果页并携带输入参数。
import android.os.Bundle; // 使用 Bundle 接收安卓传给页面的启动或恢复状态。
import android.view.View; // 使用 View 中的可见性常量控制错误文字。
import android.widget.EditText; // 使用 EditText 读身高、体重和年龄。
import android.widget.RadioGroup; // 使用 RadioGroup 让性别选项互斥。
import android.widget.TextView; // 使用 TextView 显示尚未选择性别的提示。

public class MainActivity extends Activity { // 定义输入页面。它负责接收和校验输入，随后打开结果页。
    private EditText heightInput; // 持有身高输入框，约定其单位为厘米。
    private EditText weightInput; // 持有体重输入框，约定其单位为千克。
    private EditText ageInput; // 持有年龄输入框，要求整岁。
    private RadioGroup genderGroup; // 持有性别单选组，便于读取当前选中的按钮。
    private TextView genderError; // 持有性别错误提示文字，未选时显示。

    @Override // 标明下面的方法覆盖系统生命周期方法。
    protected void onCreate(Bundle savedInstanceState) { // 系统创建页面时进入这里。
        super.onCreate(savedInstanceState); // 先让父类完成页面初始化。
        setContentView(R.layout.activity_main); // 加载主页面布局。控件必须等布局加载后才能查找。
        heightInput = findViewById(R.id.height_input); // 用 XML 中的身高 ID 绑定输入框。
        weightInput = findViewById(R.id.weight_input); // 用 XML 中的体重 ID 绑定输入框。
        ageInput = findViewById(R.id.age_input); // 用 XML 中的年龄 ID 绑定输入框。
        genderGroup = findViewById(R.id.gender_group); // 绑定单选组本身，读取的是组内被选按钮的 ID。
        genderError = findViewById(R.id.gender_error); // 绑定单选错误文字。
        genderGroup.setOnCheckedChangeListener((group, checkedId) -> genderError.setVisibility(View.GONE)); // 选项变化后隐藏未选择的错误提示。
        findViewById(R.id.calculate_button).setOnClickListener(view -> showReport()); // 找到计算按钮并注册事件，点击后才校验和跳转。
    } // 结束页面初始化。

    private Integer readAge() { // 返回包装类型，以便用空值标记年龄无效。
        try { // 把文本转整数及数值范围检查放在同一个异常处理里。
            int age = Integer.parseInt(ageInput.getText().toString().trim()); // 读取文字，去除首尾空白，再转成整数。
            BmiCalculator.isAdult(age); // 复用计算类里的年龄范围校验。此处只需要检查，不使用它的布尔结果。
            return age; // 校验通过就返回整岁。
        } catch (IllegalArgumentException invalid) { // 格式不对或超范围会抛出此类异常，并进入同样的提示分支。
            ageInput.setError(getString(R.string.invalid_age)); // 从字符串资源取错误文案，显示在年龄框上。
            return null; // 把空值返回给调用方，阻止跳转。
        } // 结束年龄错误处理。
    } // 结束年龄读取方法。

    private Double readPositiveNumber(EditText input) { // 接收任何数字输入框，复用同一套正数校验。
        try { // 尝试把文本解析成小数。
            double number = Double.parseDouble(input.getText().toString().trim()); // 先取当前文字并去空白，再用双精度类型解析。
            if (number > 0 && !Double.isInfinite(number)) return number; // 只接受大于零的有限数。非数字值不会通过大于零的判断。
        } catch (NumberFormatException ignored) { } // 文本不能解析时，继续执行下面统一的错误提示。
        input.setError(getString(R.string.invalid_input)); // 在对应输入框显示非正数或非法输入提示。
        return null; // 用空值通知调用方这个字段无效。
    } // 结束通用数字读取方法。

    private void showReport() { // 计算按钮入口，集中处理校验、打包参数和页面跳转。
        heightInput.setError(null); // 清掉上一次身高错误，重新检查本次提交值。
        weightInput.setError(null); // 清掉上一次体重错误。
        ageInput.setError(null); // 清掉上一次年龄错误。
        genderError.setVisibility(View.GONE); // 清掉旧的性别提示，按本次选中状态重新决定是否显示。
        Double height = readPositiveNumber(heightInput); // 使用通用方法读取身高。
        Double weight = readPositiveNumber(weightInput); // 对体重复用同一方法。
        Integer age = readAge(); // 使用专用方法读取整数年龄。
        int checkedGender = genderGroup.getCheckedRadioButtonId(); // 取得被选按钮的资源 ID。无选项时本组返回负一。
        if (checkedGender == -1) genderError.setVisibility(View.VISIBLE); // 没有选项时显示单独的性别错误文字。
        if (height == null || weight == null || age == null || checkedGender == -1) return; // 任一项无效就返回，避免空值被拆箱或错误参数进入详情页。
        String gender = checkedGender == R.id.gender_male ? "Male" : "Female"; // 将单选按钮的 ID 转成结果页和计算类共用的字符串。
        try { // 跳转之前先验证这组身高体重能否得到有效的计算结果。
            BmiCalculator.calculate(height, weight); // 调用计算类，也覆盖极端数值引起的上溢或下溢。
        } catch (IllegalArgumentException invalid) { // 参数超出计算支持范围时，在输入页处理。
            heightInput.setError(getString(R.string.invalid_input)); // 本项目把这类计算错误显示在身高框。
            return; // 有计算错误时留在输入页。
        } // 结束结果范围检查。
        Intent intent = new Intent(this, ReportActivity.class); // 构造显式 Intent，目标是本应用里的结果页。
        intent.putExtra("height", height.doubleValue()); // 以 height 为键传 double，详情页必须用相同键和类型取值。
        intent.putExtra("weight", weight.doubleValue()); // 以 weight 为键传 double。
        intent.putExtra("age", age.intValue()); // 以 age 为键传 int。
        intent.putExtra("gender", gender); // 以 gender 为键传字符串。
        startActivity(intent); // 参数填齐后才打开结果页。
    } // 结束计算按钮流程。
} // 结束输入页面类。
~~~

<a id="file-3"></a>

### app/src/main/java/com/example/bmi/ReportActivity.java

~~~java
package com.example.bmi; // 使用与输入页、计算类相同的包。

import android.app.Activity; // 使用系统 Activity 作为详情页基类。
import android.graphics.Color; // 用 Color 把颜色字符串解析为文字颜色。
import android.os.Bundle; // 用 Bundle 接收系统传给页面的状态。
import android.view.View; // 用 View 控制提示文字是否占用布局空间。
import android.widget.LinearLayout; // 用 LinearLayout 引用结果卡片容器。
import android.widget.TextView; // 用 TextView 读取和更新结果页文字。

public class ReportActivity extends Activity { // 定义该应用的第二个页面。
    @Override // 标明下面覆盖安卓的生命周期方法。
    protected void onCreate(Bundle savedInstanceState) { // 结果页创建时进入这里，读取参数并展示。
        super.onCreate(savedInstanceState); // 先交给父类初始化。
        setContentView(R.layout.activity_report); // 加载结果页 XML，之后才可依 ID 绑定结果控件。
        double height = getIntent().getDoubleExtra("height", 0); // 按 height 键读取 double 身高，缺失时返回零交给校验。
        double weight = getIntent().getDoubleExtra("weight", 0); // 按 weight 键读取 double 体重，同样用零作为缺失默认值。
        int age = getIntent().getIntExtra("age", 0); // 按 age 键读取 int 年龄。
        String gender = getIntent().getStringExtra("gender"); // 按 gender 键读字符串。该键和输入页对应。
        double bmi; // 留出计算值变量，供后面格式化显示。
        String category; // 留出分类文字变量，供界面和警告分支使用。
        boolean adult; // 留出成人标志，供选择分类函数和显示用户资料使用。
        try { // 一起处理数据计算、年龄校验及分类中的异常。
            bmi = BmiCalculator.calculate(height, weight); // 复用同一计算方法得到原始 BMI。
            adult = BmiCalculator.isAdult(age); // 确定走成人还是非成人分类，顺便校验年龄。
            category = adult // 按成人标志从两种分类函数选一种。
                    ? BmiCalculator.adultCategory(bmi) // 成人分支只按 BMI 数值判断。
                    : BmiCalculator.childCategory(bmi, age, gender); // 非成人分支还按年龄与性别查表。
        } catch (IllegalArgumentException invalid) { // 捕获非法来源或超范围参数。
            finish(); // 关闭无法正确呈现数据的详情页。
            return; // 立即中止方法，避免继续更新界面。
        } // 结束详情页数据校验。
        TextView result = findViewById(R.id.bmi_result); // 按布局 ID 找到用于显示 BMI 数字的控件。
        result.setText(getString(R.string.bmi_value, bmi)); // 把 BMI 套入字符串资源中的两位小数占位符。
        TextView profile = findViewById(R.id.profile_summary); // 按布局 ID 找到显示年龄、性别和组别的文字。
        profile.setText(getString(R.string.profile_summary, age, // 准备把年龄等三个参数传给格式化资源。
                gender == null ? "" : gender, getString(adult ? R.string.adult : R.string.child))); // 空性别显示为空，成人标志则决定组别文案。
        TextView categoryText = findViewById(R.id.bmi_category); // 找到用于显示分类名称的控件。
        categoryText.setText(category); // 把计算类返回的英文分类显示在界面上。
        TextView note = findViewById(R.id.bmi_note); // 找到用于显示严重提示的控件。
        LinearLayout card = findViewById(R.id.result_card); // 找到可以替换背景的结果卡片容器。
        boolean severe = BmiCalculator.isSevere(category); // 用计算类判定是否需要警告样式。
        if (severe) { // 只有严重分类才执行下面的样式更新。
            card.setBackgroundResource(R.drawable.result_card_warning); // 把默认卡片背景换成警告背景资源。
            result.setTextColor(Color.parseColor("#B91C1C")); // 把 BMI 数字显示为较深的红色。
            categoryText.setTextColor(Color.parseColor("#991B1B")); // 把分类名称也显示为红色。
            note.setText(category.equals("Severely Underweight") // 根据当前严重分类，从两个警告资源中选一个。
                    ? R.string.severe_underweight_warning // 严重偏低使用对应的警告文案。
                    : R.string.severe_overweight_warning); // 其它严重情况用严重偏高警告文案。
            note.setTextColor(Color.parseColor("#991B1B")); // 应用红色提示文字样式。
        } else { // 进入不需要警告的分支。
            note.setVisibility(View.GONE); // 隐藏警告控件，并移除它占用的空间。
        } // 结束警告样式判断。
        findViewById(R.id.back_button).setOnClickListener(view -> finish()); // 点击返回时关闭当前详情页，回到原来的输入页。
    } // 结束结果页创建和展示流程。
} // 结束报告页面类。
~~~

<a id="file-4"></a>

### app/build.gradle.kts

~~~kotlin
plugins { id("com.android.application") } // 应用或声明 Android 构建插件。
android { // 进入安卓模块设置。
    namespace = "com.example.bmi" // 生成资源类的命名空间。
    compileSdk { version = release(33) } // 选择编译时可用的安卓接口。
    defaultConfig { // 统一配置应用标识与支持版本。
        applicationId = "com.example.bmi" // 指定安装包在设备上的唯一标识。
        minSdk = 21 // 限制最低可安装系统版本。
        targetSdk = 33 // 指定适配的目标接口版本。
        versionCode = 1 // 定义整数版本号。
        versionName = "1.0" // 定义可读版本名。
    } // 结束当前构建配置块。
    compileOptions { // 设置 Java 编译选项。
        sourceCompatibility = JavaVersion.VERSION_11 // 规定源码语言级别。
        targetCompatibility = JavaVersion.VERSION_11 // 规定目标字节码版本。
    } // 结束当前构建配置块。
} // 结束当前构建配置块。
~~~

<a id="file-5"></a>

### app/src/main/AndroidManifest.xml

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 声明应用清单，定义安卓属性使用的命名空间。 -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 配置应用名称资源、主题和从右向左布局支持。此练习关闭自动备份。 -->
    <application android:allowBackup="false" android:label="@string/app_name"
        android:theme="@style/AppTheme"
        android:supportsRtl="true">
        <!-- 声明详情页。它只在本应用内打开，因此不对其他应用导出。 -->
        <activity android:name=".ReportActivity" android:exported="false" />
        <!-- 声明首页为可由启动器访问的 Activity。 -->
        <activity android:name=".MainActivity" android:exported="true">
            <!-- 以下两个条件共同把首页注册成桌面启动入口。 -->
            <intent-filter>
                <!-- 标记为应用的主要入口动作。 -->
                <action android:name="android.intent.action.MAIN" />
                <!-- 允许系统启动器显示应用图标。 -->
                <category android:name="android.intent.category.LAUNCHER" />
            <!-- 结束启动入口描述。 -->
            </intent-filter>
        <!-- 结束主页面声明。 -->
        </activity>
    <!-- 结束应用配置。 -->
    </application>
<!-- 结束清单。 -->
</manifest>
~~~

<a id="file-6"></a>

### app/src/main/res/drawable/result_card.xml

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 用矩形 shape 定义普通结果卡的可复用背景。 -->
<shape xmlns:android="http://schemas.android.com/apk/res/android" android:shape="rectangle">
    <!-- 填充浅蓝底色。 -->
    <solid android:color="#EFF6FF" />
    <!-- 设置十六密度像素的圆角。 -->
    <corners android:radius="16dp" />
    <!-- 用一密度像素宽的浅蓝描边。 -->
    <stroke android:width="1dp" android:color="#93C5FD" />
<!-- 结束普通结果卡背景。 -->
</shape>
~~~

<a id="file-7"></a>

### app/src/main/res/drawable/result_card_warning.xml

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 用矩形 shape 定义严重结果的警告卡背景。 -->
<shape xmlns:android="http://schemas.android.com/apk/res/android" android:shape="rectangle">
    <!-- 使用浅红底色，让警告卡有独立的视觉样式。 -->
    <solid android:color="#FEF2F2" />
    <!-- 保持与普通卡相同的圆角尺寸。 -->
    <corners android:radius="16dp" />
    <!-- 用两密度像素宽的红色描边强调警告状态。 -->
    <stroke android:width="2dp" android:color="#EF4444" />
<!-- 结束警告结果卡背景。 -->
</shape>
~~~

<a id="file-8"></a>

### app/src/main/res/layout/activity_main.xml

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 根布局 ScrollView 占满界面，允许内容滚动。fillViewport 让短内容也可撑满视口。 -->
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent" android:layout_height="match_parent" android:fillViewport="true">
    <!-- ScrollView 包含一个直接子布局。纵向排列子控件，内容高度随控件变化，内边距为二十四密度像素。 -->
    <LinearLayout android:layout_width="match_parent" android:layout_height="wrap_content"
        android:orientation="vertical" android:padding="24dp">
        <!-- 主标题使用字符串资源、较大的可缩放字号、粗体和主题深蓝色。 -->
        <TextView android:layout_width="match_parent" android:layout_height="wrap_content"
            android:text="@string/title" android:textSize="28sp" android:textStyle="bold" android:textColor="#172554" />
        <!-- 副标题显示需要填写的四项资料，顶部留出八密度像素间距。 -->
        <TextView android:layout_width="match_parent" android:layout_height="wrap_content"
            android:layout_marginTop="8dp" android:text="@string/subtitle" android:textSize="16sp" />
        <!-- 身高标签用 labelFor 关联身高输入框，便于辅助服务知道字段含义。 -->
        <TextView android:layout_width="match_parent" android:layout_height="wrap_content"
            android:layout_marginTop="32dp" android:text="@string/height" android:labelFor="@id/height_input" />
        <!-- 身高输入框使用 height_input ID、允许小数的数字键盘、单行输入和下一项动作。 -->
        <EditText android:id="@+id/height_input" android:layout_width="match_parent" android:layout_height="56dp"
            android:inputType="numberDecimal" android:singleLine="true" android:imeOptions="actionNext" />
        <!-- 体重标签关联体重输入框，并与上一栏留出十六密度像素间距。 -->
        <TextView android:layout_width="match_parent" android:layout_height="wrap_content"
            android:layout_marginTop="16dp" android:text="@string/weight" android:labelFor="@id/weight_input" />
        <!-- 体重输入框使用 weight_input ID，允许小数。键盘动作尝试转到下一项。 -->
        <EditText android:id="@+id/weight_input" android:layout_width="match_parent" android:layout_height="56dp"
            android:inputType="numberDecimal" android:singleLine="true" android:imeOptions="actionNext" />
        <!-- 年龄标签关联年龄框；界面资源文字给出岁作为单位。 -->
        <TextView android:layout_width="match_parent" android:layout_height="wrap_content"
            android:layout_marginTop="16dp" android:text="@string/age" android:labelFor="@id/age_input" />
        <!-- 年龄框使用 age_input ID 和整数键盘，键盘动作标为完成。 -->
        <EditText android:id="@+id/age_input" android:layout_width="match_parent" android:layout_height="56dp"
            android:inputType="number" android:singleLine="true" android:imeOptions="actionDone" />
        <!-- 为下方性别选项显示说明文字。 -->
        <TextView android:layout_width="match_parent" android:layout_height="wrap_content"
            android:layout_marginTop="16dp" android:text="@string/gender" />
        <!-- 单选组使用 gender_group ID，把内部的单选按钮横向排列并保持互斥。 -->
        <RadioGroup android:id="@+id/gender_group" android:layout_width="match_parent"
            android:layout_height="wrap_content" android:orientation="horizontal">
            <!-- 男性按钮的资源 ID 为 gender_male。Java 用被选按钮的 ID 判断结果。 -->
            <RadioButton android:id="@+id/gender_male" android:layout_width="wrap_content"
                android:layout_height="48dp" android:text="@string/male" />
            <!-- 女性按钮使用 gender_female ID，距前一个选项有起始边距。 -->
            <RadioButton android:id="@+id/gender_female" android:layout_width="wrap_content"
                android:layout_height="48dp" android:layout_marginStart="24dp" android:text="@string/female" />
        <!-- 结束单选组。两个按钮都应放在这个组内。 -->
        </RadioGroup>
        <!-- 性别错误使用 gender_error ID，初始不可见且不占位置，由 Java 切换。 -->
        <TextView android:id="@+id/gender_error" android:layout_width="match_parent"
            android:layout_height="wrap_content" android:text="@string/select_gender"
            android:textColor="#B91C1C" android:visibility="gone" />
        <!-- 计算按钮使用 calculate_button ID，与 Java 注册点击监听的位置对应。 -->
        <Button android:id="@+id/calculate_button" android:layout_width="match_parent" android:layout_height="56dp"
            android:layout_marginTop="24dp" android:text="@string/calculate" android:textAllCaps="false" />
        <!-- 用一个非交互的文本控件显示公式，文字从字符串资源加载。 -->
        <TextView android:layout_width="match_parent" android:layout_height="wrap_content"
            android:layout_marginTop="24dp" android:text="@string/formula" android:textSize="14sp" />
    <!-- 结束竖排表单布局。 -->
    </LinearLayout>
<!-- 结束滚动容器。 -->
</ScrollView>
~~~

<a id="file-9"></a>

### app/src/main/res/layout/activity_report.xml

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 结果页根容器可滚动，避免小屏幕或大字号遮住内容。 -->
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent" android:layout_height="match_parent" android:fillViewport="true">
    <!-- 用竖排 LinearLayout 装标题、资料、结果卡和返回按钮。 -->
    <LinearLayout android:layout_width="match_parent" android:layout_height="wrap_content"
        android:orientation="vertical" android:padding="24dp">
        <!-- 顶部标题使用结果页字符串资源，设为粗体并放大字号。 -->
        <TextView android:layout_width="match_parent" android:layout_height="wrap_content"
            android:text="@string/report_title" android:textSize="26sp" android:textStyle="bold" />
        <!-- 资料摘要先留空，由 ReportActivity 填入年龄、性别和组别。 -->
        <TextView android:id="@+id/profile_summary" android:layout_width="match_parent"
            android:layout_height="wrap_content" android:layout_marginTop="12dp" android:textSize="16sp" />
        <!-- 结果卡是一个包含三条 TextView 的竖排容器。Java 可用 result_card ID 更换背景。 -->
        <LinearLayout android:id="@+id/result_card" android:layout_width="match_parent"
            android:layout_height="wrap_content" android:layout_marginTop="16dp"
            android:orientation="vertical" android:padding="20dp" android:background="@drawable/result_card">
            <!-- 用 bmi_result ID 让 Java 填入格式化的 BMI 数值。默认显示为粗体蓝色。 -->
            <TextView android:id="@+id/bmi_result" android:layout_width="match_parent" android:layout_height="wrap_content"
                android:textSize="36sp" android:textColor="#1E40AF" android:textStyle="bold" />
            <!-- 用 bmi_category ID 接收分类文字，并设置与数值之间的间距。 -->
            <TextView android:id="@+id/bmi_category" android:layout_width="match_parent" android:layout_height="wrap_content"
                android:layout_marginTop="8dp" android:textColor="#172554" android:textSize="21sp" android:textStyle="bold" />
            <!-- 用 bmi_note ID 接收警告文字，Java 在正常分支中隐藏此控件。 -->
            <TextView android:id="@+id/bmi_note" android:layout_width="match_parent" android:layout_height="wrap_content"
                android:layout_marginTop="12dp" android:lineSpacingExtra="2dp" android:textSize="14sp" />
        <!-- 结束结果卡内部容器。 -->
        </LinearLayout>
        <!-- 返回按钮 ID 为 back_button，点击行为由结果页的 Java 监听器实现。 -->
        <Button android:id="@+id/back_button" android:layout_width="match_parent" android:layout_height="56dp"
            android:layout_marginTop="20dp" android:text="@string/back" android:textAllCaps="false" />
    <!-- 结束结果页内部纵向布局。 -->
    </LinearLayout>
<!-- 结束结果页滚动容器。 -->
</ScrollView>
~~~

<a id="file-10"></a>

### app/src/main/res/values/strings.xml

~~~xml
<!-- 声明可通过 R.string 或 XML 引用的字符串资源。 -->
<resources>
    <!-- 应用启动器看到的名称，供清单引用。 -->
    <string name="app_name">BMI</string>
    <!-- 主页面标题。 -->
    <string name="title">BMI Calculator</string>
    <!-- 主页面输入说明。 -->
    <string name="subtitle">Enter height, weight, age and gender</string>
    <!-- 使用厘米作为身高字段标签的显示单位。 -->
    <string name="height">Height (cm)</string>
    <!-- 使用千克作为体重字段标签的显示单位。 -->
    <string name="weight">Weight (kg)</string>
    <!-- 提示年龄以整岁为单位。 -->
    <string name="age">Age (years)</string>
    <!-- 性别字段的标题。 -->
    <string name="gender">Gender</string>
    <!-- 男性按钮标签。代码另用 Male 字符串作为分类参数。 -->
    <string name="male">Male</string>
    <!-- 女性按钮标签。代码另用 Female 字符串作为分类参数。 -->
    <string name="female">Female</string>
    <!-- 未选单选项时显示的提示。 -->
    <string name="select_gender">Please select a gender</string>
    <!-- 执行计算按钮的文字。 -->
    <string name="calculate">Calculate BMI</string>
    <!-- 身高或体重不是有效正数时显示的错误。 -->
    <string name="invalid_input">Enter a number greater than 0</string>
    <!-- 年龄非整数或超出六至一百二十区间时显示的错误。 -->
    <string name="invalid_age">Enter a whole-number age from 6 to 120</string>
    <!-- 结果页标题。 -->
    <string name="report_title">Your BMI Report</string>
    <!-- 第一个参数是浮点数，保留两位小数。 -->
    <string name="bmi_value">BMI: %1$.2f</string>
    <!-- 三个参数依次是整数年龄、性别字符串和年龄组字符串。 -->
    <string name="profile_summary">Age: %1$d   •   Gender: %2$s   •   %3$s</string>
    <!-- 成人年龄组显示名。 -->
    <string name="adult">Adult</string>
    <!-- 非成人年龄组显示名。 -->
    <string name="child">Non-adult</string>
    <!-- 作业原有的严重偏低提示文案。 -->
    <string name="severe_underweight_warning">Warning: BMI is severely below the recommended range. Please seek advice from a healthcare professional.</string>
    <!-- 作业原有的严重偏高提示文案。 -->
    <string name="severe_overweight_warning">Warning: BMI is severely above the recommended range. Please seek advice from a healthcare professional.</string>
    <!-- 返回输入页按钮的文字。 -->
    <string name="back">Back to input</string>
    <!-- 显示计算公式，提醒身高在运算时用米。 -->
    <string name="formula">BMI = weight (kg) / height (m)²</string>
<!-- 结束字符串资源。 -->
</resources>
~~~

<a id="file-11"></a>

### app/src/main/res/values/styles.xml

~~~xml
<!-- 声明本文件内的资源。 -->
<resources>
    <!-- 定义应用主题，继承系统的浅色无标题栏主题。 -->
    <style name="AppTheme" parent="android:style/Theme.Material.Light.NoActionBar">
        <!-- 选用系统无衬线字体族。 -->
        <item name="android:fontFamily">sans</item>
        <!-- 设置控件强调色，例如选择状态的颜色。 -->
        <item name="android:colorAccent">#2563EB</item>
        <!-- 设置屏幕顶部状态栏的底色。 -->
        <item name="android:statusBarColor">#1E40AF</item>
        <!-- 设置系统底部导航栏的白色背景。 -->
        <item name="android:navigationBarColor">#FFFFFF</item>
        <!-- 让支持此属性的系统在浅色导航背景上使用较深的图标。 -->
        <item name="android:windowLightNavigationBar">true</item>
        <!-- 设置页面窗口默认白色背景。 -->
        <item name="android:windowBackground">#FFFFFF</item>
    <!-- 结束应用主题。 -->
    </style>
<!-- 结束样式资源。 -->
</resources>
~~~

<a id="file-12"></a>

### build.gradle.kts

~~~kotlin
plugins { id("com.android.application") version "9.4.0" apply false } // 应用或声明 Android 构建插件。
~~~

<a id="file-13"></a>

### settings.gradle.kts

~~~kotlin
pluginManagement { repositories { google(); mavenCentral(); gradlePluginPortal() } } // 声明构建插件的下载仓库。
dependencyResolutionManagement { // 集中设置模块依赖解析规则。
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS) // 限制子模块单独配置仓库。
    repositories { google(); mavenCentral() } // 配置 Google 和 Maven 依赖仓库。
} // 结束当前构建配置块。
rootProject.name = "BMI" // 设置工程显示名称。
include(":app") // 把应用模块纳入构建。
~~~
