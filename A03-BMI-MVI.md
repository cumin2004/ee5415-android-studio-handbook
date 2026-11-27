# A03-BMI-MVI 中文源码讲解

从 BmiIntent.Calculate 开始。Activity 把输入封装成事件，BmiStore.dispatch 经 reduce 产生 BmiState，页面观察结果后发出 ReportShown 清空状态。比较同一输入在 MVP、MVVM 版本中的调用顺序。

## 文件目录

- [app/src/main/java/com/example/bmi/BmiCalculator.java](#file-1)
- [app/src/main/java/com/example/bmi/BmiIntent.java](#file-2)
- [app/src/main/java/com/example/bmi/BmiModel.java](#file-3)
- [app/src/main/java/com/example/bmi/BmiResult.java](#file-4)
- [app/src/main/java/com/example/bmi/BmiState.java](#file-5)
- [app/src/main/java/com/example/bmi/BmiStore.java](#file-6)
- [app/src/main/java/com/example/bmi/MainActivity.java](#file-7)
- [app/src/main/java/com/example/bmi/ReportActivity.java](#file-8)
- [app/build.gradle.kts](#file-9)
- [app/src/main/AndroidManifest.xml](#file-10)
- [app/src/main/res/layout-land/activity_main.xml](#file-11)
- [app/src/main/res/layout-land/activity_report.xml](#file-12)
- [app/src/main/res/layout/activity_main.xml](#file-13)
- [app/src/main/res/layout/activity_report.xml](#file-14)
- [app/src/main/res/menu/menu_context.xml](#file-15)
- [app/src/main/res/menu/menu_options.xml](#file-16)
- [app/src/main/res/values-zh/strings.xml](#file-17)
- [app/src/main/res/values/strings.xml](#file-18)
- [app/src/main/res/values/styles.xml](#file-19)
- [build.gradle.kts](#file-20)
- [settings.gradle.kts](#file-21)

<a id="file-1"></a>

### app/src/main/java/com/example/bmi/BmiCalculator.java

~~~java
package com.example.bmi; // 声明源码所在包，供同包类型引用。

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
    }; // 结束数组初始化。

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
    }; // 结束数组初始化。

    private BmiCalculator() { } // 禁止创建工具类对象，外部直接调用静态方法。

    public static double calculate(double heightCm, double weightKg) { // 接收厘米和千克，返回未格式化的 BMI 数值。
        if (!(heightCm > 0) || !(weightKg > 0) // 检查身高体重是否都为正，非数字也不能通过。
                || Double.isInfinite(heightCm) || Double.isInfinite(weightKg)) { // 同时排除正无穷，避免其进入换算。
            throw new IllegalArgumentException("Height and weight must be finite and positive"); // 用参数异常让调用方决定怎样提示。
        } // 结束当前代码块。
        double heightM = heightCm / 100.0; // 将厘米除以一百，换成公式所需的米。
        double bmi = weightKg / (heightM * heightM); // 体重除以身高米数的平方，得到原始 BMI。
        if (!(bmi > 0) || Double.isInfinite(bmi)) { // 分类前检查 BMI 是有意义的有限正数。
            throw new IllegalArgumentException("Values are outside the supported range"); // 抛出范围错误，交给调用页面处理。
        } // 结束当前代码块。
        return bmi; // 返回原始数值，显示时再格式化。
    } // 结束当前代码块。

    public static boolean isAdult(int age) { // 按作业规则验证年龄，合法时返回是否走成人分支。
        if (age < 6 || age > 120) { // 只接受六至一百二十周岁，包含两个端点。
            throw new IllegalArgumentException("Age must be from 6 to 120"); // 拒绝超出范围的整岁输入。
        } // 结束当前代码块。
        return age > 18; // 十八岁仍走非成人分支，十九岁开始走成人分支。
    } // 结束当前代码块。

    public static String adultCategory(double bmi) { // 按作业规定的成人阈值，给原始 BMI 分类。
        if (!(bmi > 0) || Double.isInfinite(bmi)) { // 分类前检查 BMI 是有意义的有限正数。
            throw new IllegalArgumentException("BMI must be finite and positive"); // 无效值不能进入分类分支。
        } // 结束当前代码块。
        if (bmi < 18.5) return "Underweight"; // 严格小于十八点五走体重偏低类别。
        if (bmi < 23.0) return "Normal range"; // 前面的分支已排除低值，此处分段包含十八点五，不含二十三。
        if (bmi < 25.0) return "Overweight"; // 此处分段包含二十三，不含二十五。
        return "Obese"; // 剩下的有效值都大于或等于二十五。
    } // 结束当前代码块。

    public static String childCategory(double bmi, int age, String gender) { // 按非成人年龄和性别查表，再用作业规定的四个上界分类。
        if (age < 6 || age > 18) { // 先保护数组索引，只接受六至十八岁。
            throw new IllegalArgumentException("Child age must be from 6 to 18"); // 用参数异常表示年龄无法查表。
        } // 结束当前代码块。
        double[][] chart; // 声明本次分类使用的阈值表。
        if ("Male".equals(gender)) { // 用常量字符串调用相等比较，即使传入空值也不会抛空指针。
            chart = BOYS; // 选男生阈值表。
        } else if ("Female".equals(gender)) { // 次选女生对应的字符串。
            chart = GIRLS; // 选女生阈值表。
        } else { // 前面的条件不满足时执行本分支。
            throw new IllegalArgumentException("Gender must be Male or Female"); // 抛出参数异常通知调用方。
        } // 结束当前代码块。
        double value = Math.round(bmi * 10.0) / 10.0; // 按原作业规则先把 BMI 舍入到一位小数再查表。
        double[] row = chart[age - 6]; // 表的首行对应六岁，因此年龄减六得到下标。
        if (value <= row[0]) return "Severely Underweight"; // 不超过第一列上界，归入严重偏低类别。
        if (value <= row[1]) return "Underweight"; // 上一个分支已排除第一段，此处包含第二列上界。
        if (value <= row[2]) return "Acceptable Weight"; // 第三个区间包含正常范围的上界。
        if (value <= row[3]) return "Overweight"; // 第四个区间包含超重一段的上界。
        return "Severely Overweight"; // 大于第四列上界，归入严重超重类别。
    } // 结束当前代码块。

    public static boolean isSevere(String category) { // 判断当前分类是否需要显示特殊样式与提示。
        return "Severely Underweight".equals(category) // 严重偏低需要提示。
                || "Severely Overweight".equals(category); // 严重超重也需要提示，其他分类返回假。
    } // 结束当前代码块。
} // 结束当前代码块。
~~~

<a id="file-2"></a>

### app/src/main/java/com/example/bmi/BmiIntent.java

~~~java
package com.example.bmi; // 声明源码所在包，供同包类型引用。
public abstract class BmiIntent { // 定义用户操作的抽象类型，Store 通过子类型识别事件。
    public static final class Calculate extends BmiIntent { // 计算事件携带提交时的四项原始输入。
        public final String height, weight, age, gender; // 用不可重新赋值的字段保存一次输入快照。
        public Calculate(String height, String weight, String age, String gender) { // 从页面接收身高、体重、年龄及性别字符串。
            this.height = height; this.weight = weight; this.age = age; this.gender = gender; // 逐项保存事件参数，交给归约函数处理。
        } // 结束当前代码块。
    } // 结束当前代码块。
    public static final class ReportShown extends BmiIntent { } // 报告显示完毕后派发此事件，消耗已处理的结果。
} // 结束当前代码块。
~~~

<a id="file-3"></a>

### app/src/main/java/com/example/bmi/BmiModel.java

~~~java
package com.example.bmi; // 声明源码所在包，供同包类型引用。
public class BmiModel { // 封装表单校验和 BMI 计算，界面只处理返回值。
    public BmiResult calculate(String heightText, String weightText, String ageText, String gender) { // 把表单文本转换为带数值或错误字段的结果对象。
        double height, weight; // 预留解析后的身高和体重。
        int age; // 年龄要求整岁，因此使用整数。
        try { // 在这里执行可能抛出异常的操作。
            height = Double.parseDouble(heightText.trim()); // 清除前后空白后解析身高。
            if (!(height > 0) || Double.isInfinite(height)) return BmiResult.error("height"); // 拒绝非正数、无穷及非数字值，标记身高字段。
        } catch (NumberFormatException e) { return BmiResult.error("height"); } // 解析失败直接返回身高错误。
        try { // 在这里执行可能抛出异常的操作。
            weight = Double.parseDouble(weightText.trim()); // 把体重文本转为小数。
            if (!(weight > 0) || Double.isInfinite(weight)) return BmiResult.error("weight"); // 验证体重是有限正数。
        } catch (NumberFormatException e) { return BmiResult.error("weight"); } // 无法解析时只标记体重字段。
        try { // 在这里执行可能抛出异常的操作。
            age = Integer.parseInt(ageText.trim()); // 把年龄解析为整数。
            BmiCalculator.isAdult(age); // 复用计算类里的年龄范围校验。此处只需要检查，不使用它的布尔结果。
        } catch (IllegalArgumentException e) { return BmiResult.error("age"); } // 捕获整数格式错误和超范围年龄。
        if (!"Male".equals(gender) && !"Female".equals(gender)) return BmiResult.error("gender"); // 性别字段只接受单选按钮对应的两个内部值。
        try { // 在这里执行可能抛出异常的操作。
            double bmi = BmiCalculator.calculate(height, weight); // 复用公共公式，输入厘米和千克。
            String category = BmiCalculator.isAdult(age) // 根据年龄选择下面的成人或儿童分支。
                    ? BmiCalculator.adultCategory(bmi) // 成人分支只按 BMI 数值判断。
                    : BmiCalculator.childCategory(bmi, age, gender); // 非成人分支还按年龄与性别查表。
            return new BmiResult(bmi, age, gender, category, null); // 成功结果包含数值与分类，错误字段留空。
        } catch (IllegalArgumentException e) { return BmiResult.error("height"); } // 底层公式拒绝非法参数时返回身高错误标记。
    } // 结束当前代码块。
} // 结束当前代码块。
~~~

<a id="file-4"></a>

### app/src/main/java/com/example/bmi/BmiResult.java

~~~java
package com.example.bmi; // 声明源码所在包，供同包类型引用。
public final class BmiResult { // 定义在计算层和界面之间传递的结果对象。
    public final double bmi; // 记录原始 BMI 值。
    public final int age; // 保留整岁供报告显示。
    public final String gender; // 保留性别作为页面参数。
    public final String category; // 保存计算类返回的分类标识。
    public final String errorField; // 校验失败时指出哪个输入控件需要显示提示。
    public BmiResult(double bmi, int age, String gender, String category, String errorField) { // 统一初始化成功和失败两种结果。
        this.bmi = bmi; // 记录计算所得数值。
        this.age = age; // 记录本次计算的年龄。
        this.gender = gender; // 记录本次计算的性别。
        this.category = category; // 记录分类结果。
        this.errorField = errorField; // 记录可能的错误字段。
    } // 结束当前代码块。
    public static BmiResult error(String field) { // 提供创建失败结果的入口。
        return new BmiResult(0, 0, "", "", field); // 其它字段使用占位值，界面应先判断错误字段。
    } // 结束当前代码块。
} // 结束当前代码块。
~~~

<a id="file-5"></a>

### app/src/main/java/com/example/bmi/BmiState.java

~~~java
package com.example.bmi; // 声明源码所在包，供同包类型引用。
public final class BmiState { // 把本轮结果包装为可观察的界面状态。
    public final BmiResult result; // 空结果表示当前没有待处理的报告。
    public BmiState(BmiResult result) { this.result = result; } // 创建新状态时固定本轮结果。
} // 结束当前代码块。
~~~

<a id="file-6"></a>

### app/src/main/java/com/example/bmi/BmiStore.java

~~~java
package com.example.bmi; // 声明源码所在包，供同包类型引用。

import androidx.lifecycle.LiveData; // 引入 LiveData 类型供本文件使用。
import androidx.lifecycle.MutableLiveData; // 引入 MutableLiveData 类型供本文件使用。
import androidx.lifecycle.ViewModel; // 引入 ViewModel 类型供本文件使用。

public class BmiStore extends ViewModel { // 用 ViewModel 承载 MVI 的状态和事件处理。
    private final BmiModel model = new BmiModel(); // 持有负责校验与计算的模型。
    private final MutableLiveData<BmiState> state = new MutableLiveData<>(new BmiState(null)); // 以空结果初始化状态流。
    public LiveData<BmiState> getState() { return state; } // 向观察者暴露只读的状态接口。
    public void dispatch(BmiIntent intent) { state.setValue(reduce(intent)); } // 接收事件，先归约再发布新状态。
    private BmiState reduce(BmiIntent intent) { // 把用户事件变换为下一个状态。
        if (intent instanceof BmiIntent.Calculate) { // 判断是否为请求计算的事件。
            BmiIntent.Calculate input = (BmiIntent.Calculate) intent; // 转换成计算事件以读取四个字段。
            return new BmiState(model.calculate(input.height, input.weight, input.age, input.gender)); // 调用模型，用返回的结果创建状态。
        } // 结束当前代码块。
        return new BmiState(null); // 清空已消费的结果，防止重复跳转。
    } // 结束当前代码块。
} // 结束当前代码块。
~~~

<a id="file-7"></a>

### app/src/main/java/com/example/bmi/MainActivity.java

~~~java
package com.example.bmi; // 声明源码所在包，供同包类型引用。

import android.content.Intent; // 引入 Intent 类型供本文件使用。
import android.net.Uri; // 引入 Uri 类型供本文件使用。
import android.os.Bundle; // 引入 Bundle 类型供本文件使用。
import android.view.ContextMenu; // 引入 ContextMenu 类型供本文件使用。
import android.view.Menu; // 引入 Menu 类型供本文件使用。
import android.view.MenuItem; // 引入 MenuItem 类型供本文件使用。
import android.view.View; // 引入 View 类型供本文件使用。
import android.widget.EditText; // 引入 EditText 类型供本文件使用。
import android.widget.RadioGroup; // 引入 RadioGroup 类型供本文件使用。
import android.widget.TextView; // 引入 TextView 类型供本文件使用。
import androidx.appcompat.app.AlertDialog; // 引入 AlertDialog 类型供本文件使用。
import androidx.appcompat.app.AppCompatActivity; // 引入 AppCompatActivity 类型供本文件使用。
import androidx.lifecycle.ViewModelProvider; // 引入 ViewModelProvider 类型供本文件使用。
public class MainActivity extends AppCompatActivity  { // 定义 MVI 版输入页。
    private EditText heightInput, weightInput, ageInput; // 持有三个输入框，供按钮事件取值。
    private RadioGroup genderGroup; // 持有性别单选组，便于读取当前选中的按钮。
    private TextView genderError; // 持有性别错误提示文字，未选时显示。
    private BmiStore store; // 保存页面使用的 Store。
    @Override // 覆盖父类或接口声明的方法。
    protected void onCreate(Bundle savedInstanceState) { // 页面创建时绑定布局和交互逻辑。
        super.onCreate(savedInstanceState); // 先交给父类初始化。
        setContentView(R.layout.activity_main); // 加载主页面布局。控件必须等布局加载后才能查找。
        heightInput = findViewById(R.id.height_input); // 用 XML 中的身高 ID 绑定输入框。
        weightInput = findViewById(R.id.weight_input); // 用 XML 中的体重 ID 绑定输入框。
        ageInput = findViewById(R.id.age_input); // 用 XML 中的年龄 ID 绑定输入框。
        genderGroup = findViewById(R.id.gender_group); // 绑定单选组本身，读取的是组内被选按钮的 ID。
        genderError = findViewById(R.id.gender_error); // 绑定单选错误文字。
        genderGroup.setOnCheckedChangeListener((group, id) -> genderError.setVisibility(View.GONE)); // 一旦用户选择性别，隐藏先前的未选提示。
        registerForContextMenu(findViewById(R.id.bmi_icon)); // 注册图标的长按上下文菜单。
        store = new ViewModelProvider(this).get(BmiStore.class); // 按页面作用域取得 Store，支持配置变化。
        store.getState().observe(this, state -> { // 订阅状态，仅在页面生命周期允许时接收更新。
            if (state.result != null) { // 只有存在新结果才进行显示或报错。
                showResult(state.result); // 把事件结果交给页面渲染。
                store.dispatch(new BmiIntent.ReportShown()); // 通知 Store 已消费本次结果。
            } // 结束当前代码块。
        }); // 结束回调并完成注册或提交。
        findViewById(R.id.calculate_button).setOnClickListener(view -> { // 绑定计算按钮，点击后读取当时的输入。
            heightInput.setError(null); // 清掉上一次身高错误，重新检查本次提交值。
            weightInput.setError(null); // 清掉上一次体重错误。
            ageInput.setError(null); // 清掉上一次年龄错误。
            genderError.setVisibility(View.GONE); // 清掉旧的性别提示，按本次选中状态重新决定是否显示。
            String height = heightInput.getText().toString(); // 读取厘米文本。
            String weight = weightInput.getText().toString(); // 读取千克文本。
            String age = ageInput.getText().toString(); // 读取年龄文本，稍后在模型中转整数。
            int selected = genderGroup.getCheckedRadioButtonId(); // 取得选中按钮的资源编号。
            String gender = selected == R.id.gender_male ? "Male" // 男选项映射成模型识别的固定字符串。
                    : selected == R.id.gender_female ? "Female" : ""; // 女选项映射为 Female，未选则传空串。
            store.dispatch(new BmiIntent.Calculate(height, weight, age, gender)); // 把输入包装成 Calculate 事件交给 Store。
        }); // 结束回调并完成注册或提交。
    } // 结束当前代码块。
    public void showResult(BmiResult result) { // 统一渲染计算结果，先检查字段错误。
        if (result.errorField != null) { // 有错误时保持在输入页。
            switch (result.errorField) { // 根据错误字段定位对应控件。
                case "height": heightInput.setError(getString(R.string.invalid_input)); break; // 在身高框显示错误后退出此分支。
                case "weight": weightInput.setError(getString(R.string.invalid_input)); break; // 在体重框显示错误后退出此分支。
                case "age": ageInput.setError(getString(R.string.invalid_age)); break; // 年龄框显示专用整数范围说明。
                case "gender": genderError.setVisibility(View.VISIBLE); break; // 未选性别时显示组旁的错误文字。
            } // 结束当前代码块。
            return; // 在此结束当前方法或回调。
        } // 结束当前代码块。
        Intent intent = new Intent(this, ReportActivity.class); // 构造显式 Intent，目标是本应用里的结果页。
        intent.putExtra("bmi", result.bmi); // 把算好的 BMI 传给报告页。
        intent.putExtra("age", result.age); // 传递年龄，供报告显示年龄组。
        intent.putExtra("gender", result.gender); // 传递性别供报告页本地化。
        intent.putExtra("category", result.category); // 传递分类标识供报告页选文字和样式。
        startActivity(intent); // 参数填齐后才打开结果页。
    } // 结束当前代码块。
    @Override // 覆盖父类或接口声明的方法。
    public boolean onCreateOptionsMenu(Menu menu) { // 系统请求构造顶栏菜单时进入。
        getMenuInflater().inflate(R.menu.menu_options, menu); // 把菜单资源中的条目加入当前菜单。
        return true; // 返回真，告知调用方本次处理成功。
    } // 结束当前代码块。
    @Override // 覆盖父类或接口声明的方法。
    public boolean onOptionsItemSelected(MenuItem item) { // 接收顶栏菜单点击事件。
        return handleMenu(item.getItemId()) || super.onOptionsItemSelected(item); // 共用处理方法优先，未处理时让父类接手。
    } // 结束当前代码块。
    @Override // 覆盖父类或接口声明的方法。
    public void onCreateContextMenu(ContextMenu menu, View view, ContextMenu.ContextMenuInfo info) { // 系统构建长按菜单时回调。
        super.onCreateContextMenu(menu, view, info); // 先保留父类菜单行为。
        getMenuInflater().inflate(R.menu.menu_context, menu); // 加载长按菜单资源。
    } // 结束当前代码块。
    @Override // 覆盖父类或接口声明的方法。
    public boolean onContextItemSelected(MenuItem item) { // 接收长按菜单条目的点击。
        return handleMenu(item.getItemId()) || super.onContextItemSelected(item); // 共用处理方法未处理时交回父类。
    } // 结束当前代码块。
    private boolean handleMenu(int id) { // 集中用菜单条目编号识别操作。
        if (id == R.id.menu_about) { // 判断关于菜单。
            new AlertDialog.Builder(this).setTitle(R.string.about) // 建立带标题的关于对话框。
                    .setMessage(R.string.about_message).setPositiveButton(R.string.ok, null).show(); // 加入说明和关闭按钮后显示。
            return true; // 返回真，告知调用方本次处理成功。
        } // 结束当前代码块。
        if (id == R.id.menu_wiki) { // 判断是否打开百科链接。
            String language = getResources().getConfiguration().locale.getLanguage(); // 读取当前资源语言。
            String url = "zh".equals(language) ? "https://zh.wikipedia.org/wiki/身体质量指数" // 中文环境选择中文百科页面。
                    : "https://en.wikipedia.org/wiki/Body_mass_index"; // 其它语言选择英文百科页面。
            startActivity(new Intent(Intent.ACTION_VIEW, Uri.parse(url))); // 交给系统可处理网页地址的应用打开。
            return true; // 返回真，告知调用方本次处理成功。
        } // 结束当前代码块。
        if (id == R.id.menu_exit) { // 判断退出操作。
            finish(); // 关闭当前页面，返回上一层。
            return true; // 返回真，告知调用方本次处理成功。
        } // 结束当前代码块。
        return false; // 没有匹配的菜单项，允许调用方继续处理。
    } // 结束当前代码块。
} // 结束当前代码块。
~~~

<a id="file-8"></a>

### app/src/main/java/com/example/bmi/ReportActivity.java

~~~java
package com.example.bmi; // 声明源码所在包，供同包类型引用。

import android.os.Bundle; // 引入 Bundle 类型供本文件使用。
import android.view.View; // 引入 View 类型供本文件使用。
import android.widget.TextView; // 引入 TextView 类型供本文件使用。
import androidx.appcompat.app.AppCompatActivity; // 引入 AppCompatActivity 类型供本文件使用。

public class ReportActivity extends AppCompatActivity { // 定义结果页，读取已经生成的计算结果。
    @Override // 覆盖父类或接口声明的方法。
    protected void onCreate(Bundle savedInstanceState) { // 页面创建时绑定布局和交互逻辑。
        super.onCreate(savedInstanceState); // 先交给父类初始化。
        setContentView(R.layout.activity_report); // 加载结果页 XML，之后才可依 ID 绑定结果控件。
        double bmi = getIntent().getDoubleExtra("bmi", 0); // 读取启动参数里的 BMI 数值。
        int age = getIntent().getIntExtra("age", 0); // 按 age 键读取 int 年龄。
        String gender = getIntent().getStringExtra("gender"); // 按 gender 键读字符串。该键和输入页对应。
        String category = getIntent().getStringExtra("category"); // 读取内部分类字符串。
        TextView result = findViewById(R.id.bmi_result); // 按布局 ID 找到用于显示 BMI 数字的控件。
        TextView categoryText = findViewById(R.id.bmi_category); // 找到用于显示分类名称的控件。
        result.setText(getString(R.string.bmi_value, bmi)); // 把 BMI 套入字符串资源中的两位小数占位符。
        ((TextView) findViewById(R.id.profile_summary)).setText(getString(R.string.profile_summary, // 用资源模板构造资料摘要。
                age, getString("Male".equals(gender) ? R.string.male : R.string.female))); // 显示年龄，并按当前语言显示性别文字。
        ((TextView) findViewById(R.id.age_group)).setText(age > 18 ? R.string.adult : R.string.child); // 十八岁以上显示成人组，否则显示非成人组。
        categoryText.setText(categoryResource(category)); // 将内部分类映射为当前语言的字符串资源。
        if (BmiCalculator.isSevere(category)) { // 判断结果是否进入严重提示分支。
            findViewById(R.id.bmi_note).setVisibility(View.VISIBLE); // 让原本隐藏的警告文字可见。
            result.setTextColor(0xFFB71C1C); // 把数值改为警告色。
            categoryText.setTextColor(0xFFB71C1C); // 分类名称也使用相同的警告色。
        } // 结束当前代码块。
        findViewById(R.id.back_button).setOnClickListener(view -> finish()); // 点击返回时关闭当前详情页，回到原来的输入页。
    } // 结束当前代码块。
    private int categoryResource(String category) { // 把不随语言变化的分类转成字符串资源编号。
        switch (category) { // 按分类标识选择资源。
            case "Underweight": return R.string.underweight; // 选择偏低类别的本地化文案。
            case "Normal range": return R.string.normal; // 选择正常类别的本地化文案。
            case "Overweight": return R.string.overweight; // 选择超重类别的本地化文案。
            case "Obese": return R.string.obese; // 选择肥胖类别的本地化文案。
            case "Acceptable Weight": return R.string.acceptable; // 选择儿童可接受范围的文案。
            case "Severely Underweight": return R.string.severe_underweight; // 选择严重偏低类别文案。
            case "Severely Overweight": return R.string.severe_overweight; // 选择严重超重类别文案。
            default: throw new IllegalArgumentException("Unknown BMI category"); // 未知分类属于程序数据错误，抛出异常。
        } // 结束当前代码块。
    } // 结束当前代码块。
} // 结束当前代码块。
~~~

<a id="file-9"></a>

### app/build.gradle.kts

~~~kotlin
plugins { id("com.android.application") } // 应用或声明 Android 构建插件。
android { // 进入安卓模块设置。
    namespace = "com.example.bmi" // 生成资源类的命名空间。
    compileSdk { version = release(33) } // 选择编译时可用的安卓接口。
    defaultConfig { // 统一配置应用标识与支持版本。
        applicationId = "com.example.bmi.mvi" // 指定安装包在设备上的唯一标识。
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

dependencies { // 列出模块用到的依赖。
    implementation("androidx.appcompat:appcompat:1.6.1") // 加入此行指定的库供应用编译与运行。
    implementation("androidx.lifecycle:lifecycle-viewmodel:2.6.1") // 加入此行指定的库供应用编译与运行。
    implementation("androidx.lifecycle:lifecycle-livedata:2.6.1") // 加入此行指定的库供应用编译与运行。
} // 结束当前构建配置块。
~~~

<a id="file-10"></a>

### app/src/main/AndroidManifest.xml

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 声明应用组件以及安卓属性命名空间。 -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 设置应用级主题、图标与备份策略。 -->
    <application android:allowBackup="false" android:label="@string/app_name" android:theme="@style/AppTheme" android:supportsRtl="true">
        <!-- 注册可由 Intent 启动的页面，资源或组件名为 .ReportActivity，是否对其它应用开放为 false。 -->
        <activity android:name=".ReportActivity" android:label="@string/report_title" android:exported="false"/>
        <!-- 注册可由 Intent 启动的页面，资源或组件名为 .MainActivity，是否对其它应用开放为 true。 -->
        <activity android:name=".MainActivity" android:label="@string/title" android:exported="true">
            <!-- 组合用于匹配入口的动作和类别。 -->
            <intent-filter>
                <!-- 声明入口动作，资源或组件名为 android.intent.action.MAIN。 -->
                <action android:name="android.intent.action.MAIN"/>
                <!-- 声明匹配此入口的类别，资源或组件名为 android.intent.category.LAUNCHER。 -->
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>
</manifest>
~~~

<a id="file-11"></a>

### app/src/main/res/layout-land/activity_main.xml

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 让超出屏幕的内容可滚动。 -->
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android" android:layout_width="match_parent" android:layout_height="match_parent" android:fillViewport="true">
    <!-- 按方向依次排列子控件，子控件排列方向为 horizontal。 -->
    <LinearLayout android:layout_width="match_parent" android:layout_height="wrap_content" android:padding="24dp" android:orientation="horizontal">
        <!-- 按方向依次排列子控件，剩余空间权重为 1，子控件排列方向为 vertical。 -->
        <LinearLayout android:layout_width="0dp" android:layout_weight="1" android:layout_height="wrap_content" android:orientation="vertical">
            <!-- 显示标签、摘要或计算结果，显示文字为 @string/height，关联输入框为 @id/height_input。 -->
            <TextView android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/height" android:labelFor="@id/height_input" android:textSize="14sp"/>
            <!-- 提供可读取和校验的输入框，控件编号为 @+id/height_input，输入类型为 numberDecimal。 -->
            <EditText android:id="@+id/height_input" android:layout_width="match_parent" android:layout_height="48dp" android:inputType="numberDecimal" android:singleLine="true" android:saveEnabled="true"/>
            <!-- 显示标签、摘要或计算结果，显示文字为 @string/weight，关联输入框为 @id/weight_input。 -->
            <TextView android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/weight" android:labelFor="@id/weight_input" android:textSize="14sp"/>
            <!-- 提供可读取和校验的输入框，控件编号为 @+id/weight_input，输入类型为 numberDecimal。 -->
            <EditText android:id="@+id/weight_input" android:layout_width="match_parent" android:layout_height="48dp" android:inputType="numberDecimal" android:singleLine="true" android:saveEnabled="true"/>
            <!-- 显示标签、摘要或计算结果，显示文字为 @string/age，关联输入框为 @id/age_input。 -->
            <TextView android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/age" android:labelFor="@id/age_input" android:textSize="14sp"/>
            <!-- 提供可读取和校验的输入框，控件编号为 @+id/age_input，输入类型为 number。 -->
            <EditText android:id="@+id/age_input" android:layout_width="match_parent" android:layout_height="48dp" android:inputType="number" android:singleLine="true" android:saveEnabled="true"/>
        </LinearLayout>
        <!-- 按方向依次排列子控件，剩余空间权重为 1，子控件排列方向为 vertical。 -->
        <LinearLayout android:layout_width="0dp" android:layout_weight="1" android:layout_height="wrap_content" android:layout_marginStart="24dp" android:orientation="vertical">
            <!-- 显示图标或其他图片，控件编号为 @+id/bmi_icon，图像来源为 @drawable/bmi_icon。 -->
            <ImageView android:id="@+id/bmi_icon" android:layout_width="48dp" android:layout_height="48dp" android:layout_gravity="center_horizontal" android:layout_marginBottom="12dp" android:src="@drawable/bmi_icon" android:contentDescription="@string/bmi_icon"/>
            <!-- 显示标签、摘要或计算结果，显示文字为 @string/gender。 -->
            <TextView android:layout_width="wrap_content" android:layout_height="wrap_content" android:text="@string/gender" android:layout_marginTop="8dp"/>
            <!-- 组织互斥的性别选项，控件编号为 @+id/gender_group，子控件排列方向为 horizontal。 -->
            <RadioGroup android:id="@+id/gender_group" android:layout_width="match_parent" android:layout_height="wrap_content" android:orientation="horizontal">
                <!-- 提供单选组中的一个候选项，控件编号为 @+id/gender_male，显示文字为 @string/male。 -->
                <RadioButton android:id="@+id/gender_male" android:layout_width="wrap_content" android:layout_height="48dp" android:text="@string/male"/>
                <!-- 提供单选组中的一个候选项，控件编号为 @+id/gender_female，显示文字为 @string/female。 -->
                <RadioButton android:id="@+id/gender_female" android:layout_width="wrap_content" android:layout_height="48dp" android:text="@string/female" android:layout_marginStart="16dp"/>
            </RadioGroup>
            <!-- 显示标签、摘要或计算结果，控件编号为 @+id/gender_error，显示文字为 @string/select_gender，初始可见性为 gone。 -->
            <TextView android:id="@+id/gender_error" android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/select_gender" android:textColor="#B71C1C" android:visibility="gone"/>
            <!-- 提供点击操作入口，在页面源码中绑定监听，控件编号为 @+id/calculate_button，显示文字为 @string/calculate。 -->
            <Button android:id="@+id/calculate_button" android:layout_width="match_parent" android:layout_height="48dp" android:text="@string/calculate" android:textAllCaps="false" android:layout_marginTop="12dp"/>
        </LinearLayout>
    </LinearLayout>
</ScrollView>
~~~

<a id="file-12"></a>

### app/src/main/res/layout-land/activity_report.xml

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 让超出屏幕的内容可滚动。 -->
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android" android:layout_width="match_parent" android:layout_height="match_parent" android:fillViewport="true">
    <!-- 按方向依次排列子控件，子控件排列方向为 horizontal。 -->
    <LinearLayout android:layout_width="match_parent" android:layout_height="wrap_content" android:padding="24dp" android:orientation="horizontal">
        <!-- 按方向依次排列子控件，剩余空间权重为 1，子控件排列方向为 vertical。 -->
        <LinearLayout android:layout_width="0dp" android:layout_weight="1" android:layout_height="wrap_content" android:orientation="vertical">
            <!-- 显示标签、摘要或计算结果，控件编号为 @+id/bmi_result。 -->
            <TextView android:id="@+id/bmi_result" android:layout_width="match_parent" android:layout_height="wrap_content" android:textSize="32sp" android:textStyle="bold"/>
        </LinearLayout>
        <!-- 按方向依次排列子控件，剩余空间权重为 2，子控件排列方向为 vertical。 -->
        <LinearLayout android:layout_width="0dp" android:layout_weight="2" android:layout_height="wrap_content" android:layout_marginStart="24dp" android:orientation="vertical">
            <!-- 显示标签、摘要或计算结果，控件编号为 @+id/profile_summary。 -->
            <TextView android:id="@+id/profile_summary" android:layout_width="match_parent" android:layout_height="wrap_content" android:textSize="18sp" android:layout_marginTop="16dp"/>
            <!-- 显示标签、摘要或计算结果，控件编号为 @+id/age_group。 -->
            <TextView android:id="@+id/age_group" android:layout_width="match_parent" android:layout_height="wrap_content" android:textSize="18sp" android:layout_marginTop="8dp"/>
            <!-- 显示标签、摘要或计算结果，控件编号为 @+id/bmi_category。 -->
            <TextView android:id="@+id/bmi_category" android:layout_width="match_parent" android:layout_height="wrap_content" android:textSize="22sp" android:layout_marginTop="16dp"/>
            <!-- 显示标签、摘要或计算结果，控件编号为 @+id/bmi_note，显示文字为 @string/warning，初始可见性为 gone。 -->
            <TextView android:id="@+id/bmi_note" android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/warning" android:textColor="#B71C1C" android:layout_marginTop="12dp" android:visibility="gone"/>
            <!-- 提供点击操作入口，在页面源码中绑定监听，控件编号为 @+id/back_button，显示文字为 @string/back。 -->
            <Button android:id="@+id/back_button" android:layout_width="match_parent" android:layout_height="48dp" android:layout_marginTop="20dp" android:text="@string/back" android:textAllCaps="false"/>
        </LinearLayout>
    </LinearLayout>
</ScrollView>
~~~

<a id="file-13"></a>

### app/src/main/res/layout/activity_main.xml

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 让超出屏幕的内容可滚动。 -->
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android" android:layout_width="match_parent" android:layout_height="match_parent" android:fillViewport="true">
    <!-- 按方向依次排列子控件，子控件排列方向为 vertical。 -->
    <LinearLayout android:layout_width="match_parent" android:layout_height="wrap_content" android:padding="24dp" android:orientation="vertical">
        <!-- 显示图标或其他图片，控件编号为 @+id/bmi_icon，图像来源为 @drawable/bmi_icon。 -->
        <ImageView android:id="@+id/bmi_icon" android:layout_width="64dp" android:layout_height="64dp" android:layout_gravity="center_horizontal" android:layout_marginBottom="12dp" android:src="@drawable/bmi_icon" android:contentDescription="@string/bmi_icon"/>
        <!-- 显示标签、摘要或计算结果，显示文字为 @string/height，关联输入框为 @id/height_input。 -->
        <TextView android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/height" android:labelFor="@id/height_input" android:textSize="14sp"/>
        <!-- 提供可读取和校验的输入框，控件编号为 @+id/height_input，输入类型为 numberDecimal。 -->
        <EditText android:id="@+id/height_input" android:layout_width="match_parent" android:layout_height="48dp" android:inputType="numberDecimal" android:singleLine="true" android:saveEnabled="true"/>
        <!-- 显示标签、摘要或计算结果，显示文字为 @string/weight，关联输入框为 @id/weight_input。 -->
        <TextView android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/weight" android:labelFor="@id/weight_input" android:textSize="14sp"/>
        <!-- 提供可读取和校验的输入框，控件编号为 @+id/weight_input，输入类型为 numberDecimal。 -->
        <EditText android:id="@+id/weight_input" android:layout_width="match_parent" android:layout_height="48dp" android:inputType="numberDecimal" android:singleLine="true" android:saveEnabled="true"/>
        <!-- 显示标签、摘要或计算结果，显示文字为 @string/age，关联输入框为 @id/age_input。 -->
        <TextView android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/age" android:labelFor="@id/age_input" android:textSize="14sp"/>
        <!-- 提供可读取和校验的输入框，控件编号为 @+id/age_input，输入类型为 number。 -->
        <EditText android:id="@+id/age_input" android:layout_width="match_parent" android:layout_height="48dp" android:inputType="number" android:singleLine="true" android:saveEnabled="true"/>
        <!-- 显示标签、摘要或计算结果，显示文字为 @string/gender。 -->
        <TextView android:layout_width="wrap_content" android:layout_height="wrap_content" android:text="@string/gender" android:layout_marginTop="8dp"/>
        <!-- 组织互斥的性别选项，控件编号为 @+id/gender_group，子控件排列方向为 horizontal。 -->
        <RadioGroup android:id="@+id/gender_group" android:layout_width="match_parent" android:layout_height="wrap_content" android:orientation="horizontal">
            <!-- 提供单选组中的一个候选项，控件编号为 @+id/gender_male，显示文字为 @string/male。 -->
            <RadioButton android:id="@+id/gender_male" android:layout_width="wrap_content" android:layout_height="48dp" android:text="@string/male"/>
            <!-- 提供单选组中的一个候选项，控件编号为 @+id/gender_female，显示文字为 @string/female。 -->
            <RadioButton android:id="@+id/gender_female" android:layout_width="wrap_content" android:layout_height="48dp" android:text="@string/female" android:layout_marginStart="16dp"/>
        </RadioGroup>
        <!-- 显示标签、摘要或计算结果，控件编号为 @+id/gender_error，显示文字为 @string/select_gender，初始可见性为 gone。 -->
        <TextView android:id="@+id/gender_error" android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/select_gender" android:textColor="#B71C1C" android:visibility="gone"/>
        <!-- 提供点击操作入口，在页面源码中绑定监听，控件编号为 @+id/calculate_button，显示文字为 @string/calculate。 -->
        <Button android:id="@+id/calculate_button" android:layout_width="match_parent" android:layout_height="48dp" android:text="@string/calculate" android:textAllCaps="false" android:layout_marginTop="12dp"/>
    </LinearLayout>
</ScrollView>
~~~

<a id="file-14"></a>

### app/src/main/res/layout/activity_report.xml

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 让超出屏幕的内容可滚动。 -->
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android" android:layout_width="match_parent" android:layout_height="match_parent" android:fillViewport="true">
    <!-- 按方向依次排列子控件，子控件排列方向为 vertical。 -->
    <LinearLayout android:layout_width="match_parent" android:layout_height="wrap_content" android:padding="24dp" android:orientation="vertical">
        <!-- 显示标签、摘要或计算结果，控件编号为 @+id/bmi_result。 -->
        <TextView android:id="@+id/bmi_result" android:layout_width="match_parent" android:layout_height="wrap_content" android:textSize="32sp" android:textStyle="bold"/>
        <!-- 显示标签、摘要或计算结果，控件编号为 @+id/profile_summary。 -->
        <TextView android:id="@+id/profile_summary" android:layout_width="match_parent" android:layout_height="wrap_content" android:textSize="18sp" android:layout_marginTop="16dp"/>
        <!-- 显示标签、摘要或计算结果，控件编号为 @+id/age_group。 -->
        <TextView android:id="@+id/age_group" android:layout_width="match_parent" android:layout_height="wrap_content" android:textSize="18sp" android:layout_marginTop="8dp"/>
        <!-- 显示标签、摘要或计算结果，控件编号为 @+id/bmi_category。 -->
        <TextView android:id="@+id/bmi_category" android:layout_width="match_parent" android:layout_height="wrap_content" android:textSize="22sp" android:layout_marginTop="16dp"/>
        <!-- 显示标签、摘要或计算结果，控件编号为 @+id/bmi_note，显示文字为 @string/warning，初始可见性为 gone。 -->
        <TextView android:id="@+id/bmi_note" android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/warning" android:textColor="#B71C1C" android:layout_marginTop="12dp" android:visibility="gone"/>
        <!-- 提供点击操作入口，在页面源码中绑定监听，控件编号为 @+id/back_button，显示文字为 @string/back。 -->
        <Button android:id="@+id/back_button" android:layout_width="match_parent" android:layout_height="48dp" android:layout_marginTop="20dp" android:text="@string/back" android:textAllCaps="false"/>
    </LinearLayout>
</ScrollView>
~~~

<a id="file-15"></a>

### app/src/main/res/menu/menu_context.xml

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 声明可加载到页面的菜单。 -->
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 声明主题属性或菜单条目，控件编号为 @+id/menu_about，菜单标题为 @string/about。 -->
    <item android:id="@+id/menu_about" android:title="@string/about"/>
    <!-- 声明主题属性或菜单条目，控件编号为 @+id/menu_wiki，菜单标题为 @string/wiki。 -->
    <item android:id="@+id/menu_wiki" android:title="@string/wiki"/>
</menu>
~~~

<a id="file-16"></a>

### app/src/main/res/menu/menu_options.xml

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 声明可加载到页面的菜单。 -->
<menu xmlns:android="http://schemas.android.com/apk/res/android" xmlns:app="http://schemas.android.com/apk/res-auto">
    <!-- 声明主题属性或菜单条目，控件编号为 @+id/menu_about，菜单标题为 @string/about。 -->
    <item android:id="@+id/menu_about" android:title="@string/about" app:showAsAction="never"/>
    <!-- 声明主题属性或菜单条目，控件编号为 @+id/menu_wiki，菜单标题为 @string/wiki。 -->
    <item android:id="@+id/menu_wiki" android:title="@string/wiki" app:showAsAction="never"/>
    <!-- 声明主题属性或菜单条目，控件编号为 @+id/menu_exit，菜单标题为 @string/exit。 -->
    <item android:id="@+id/menu_exit" android:title="@string/exit" app:showAsAction="never"/>
</menu>
~~~

<a id="file-17"></a>

### app/src/main/res/values-zh/strings.xml

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 集中声明可由资源编号引用的项目。 -->
<resources>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 app_name。 -->
    <string name="app_name">BMI</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 title。 -->
    <string name="title">BMI 计算器</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 height。 -->
    <string name="height">身高（厘米）</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 weight。 -->
    <string name="weight">体重（公斤）</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 age。 -->
    <string name="age">年龄（岁）</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 gender。 -->
    <string name="gender">性别</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 male。 -->
    <string name="male">男</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 female。 -->
    <string name="female">女</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 calculate。 -->
    <string name="calculate">计算 BMI</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 invalid_input。 -->
    <string name="invalid_input">请输入大于 0 的数字</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 invalid_age。 -->
    <string name="invalid_age">请输入 6 至 120 岁的整数年龄</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 select_gender。 -->
    <string name="select_gender">请选择性别</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 report_title。 -->
    <string name="report_title">BMI 报告</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 bmi_value。 -->
    <string name="bmi_value">BMI：%1$.2f</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 profile_summary。 -->
    <string name="profile_summary">年龄：%1$d   性别：%2$s</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 adult。 -->
    <string name="adult">成人</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 child。 -->
    <string name="child">非成人</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 back。 -->
    <string name="back">返回</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 about。 -->
    <string name="about">关于 BMI</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 wiki。 -->
    <string name="wiki">BMI 维基</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 exit。 -->
    <string name="exit">退出</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 about_message。 -->
    <string name="about_message">BMI = 体重（公斤）/ 身高（米）²</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 ok。 -->
    <string name="ok">确定</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 underweight。 -->
    <string name="underweight">体重过轻</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 normal。 -->
    <string name="normal">正常范围</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 overweight。 -->
    <string name="overweight">体重过重</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 obese。 -->
    <string name="obese">肥胖</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 acceptable。 -->
    <string name="acceptable">正常范围</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 severe_underweight。 -->
    <string name="severe_underweight">严重偏瘦</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 severe_overweight。 -->
    <string name="severe_overweight">严重超重</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 warning。 -->
    <string name="warning">警告：BMI 严重超出建议范围，请咨询医生。</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 bmi_icon。 -->
    <string name="bmi_icon">BMI 图标</string>
</resources>
~~~

<a id="file-18"></a>

### app/src/main/res/values/strings.xml

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 集中声明可由资源编号引用的项目。 -->
<resources>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 app_name。 -->
    <string name="app_name">BMI</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 title。 -->
    <string name="title">BMI Calculator</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 height。 -->
    <string name="height">Height (cm)</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 weight。 -->
    <string name="weight">Weight (kg)</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 age。 -->
    <string name="age">Age (years)</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 gender。 -->
    <string name="gender">Gender</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 male。 -->
    <string name="male">Male</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 female。 -->
    <string name="female">Female</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 calculate。 -->
    <string name="calculate">Calculate BMI</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 invalid_input。 -->
    <string name="invalid_input">Enter a number greater than 0</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 invalid_age。 -->
    <string name="invalid_age">Enter a whole-number age from 6 to 120</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 select_gender。 -->
    <string name="select_gender">Please select a gender</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 report_title。 -->
    <string name="report_title">BMI Report</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 bmi_value。 -->
    <string name="bmi_value">BMI: %1$.2f</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 profile_summary。 -->
    <string name="profile_summary">Age: %1$d   Gender: %2$s</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 adult。 -->
    <string name="adult">Adult</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 child。 -->
    <string name="child">Non-adult</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 back。 -->
    <string name="back">Back</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 about。 -->
    <string name="about">About BMI</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 wiki。 -->
    <string name="wiki">BMI Wiki</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 exit。 -->
    <string name="exit">Exit</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 about_message。 -->
    <string name="about_message">BMI = weight (kg) / height (m)²</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 ok。 -->
    <string name="ok">OK</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 underweight。 -->
    <string name="underweight">Underweight</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 normal。 -->
    <string name="normal">Normal range</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 overweight。 -->
    <string name="overweight">Overweight</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 obese。 -->
    <string name="obese">Obese</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 acceptable。 -->
    <string name="acceptable">Acceptable Weight</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 severe_underweight。 -->
    <string name="severe_underweight">Severely Underweight</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 severe_overweight。 -->
    <string name="severe_overweight">Severely Overweight</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 warning。 -->
    <string name="warning">Warning: BMI is severely outside the recommended range. Please seek medical advice.</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 bmi_icon。 -->
    <string name="bmi_icon">BMI icon</string>
</resources>
~~~

<a id="file-19"></a>

### app/src/main/res/values/styles.xml

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 集中声明可由资源编号引用的项目。 -->
<resources>
    <!-- 组合应用主题属性，资源或组件名为 AppTheme，继承的主题为 Theme.AppCompat.Light.DarkActionBar。 -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- 声明主题属性或菜单条目，资源或组件名为 colorPrimary。 -->
        <item name="colorPrimary">#3F51B5</item>
        <!-- 声明主题属性或菜单条目，资源或组件名为 colorPrimaryDark。 -->
        <item name="colorPrimaryDark">#303F9F</item>
        <!-- 声明主题属性或菜单条目，资源或组件名为 colorAccent。 -->
        <item name="colorAccent">#3F51B5</item>
        <!-- 声明主题属性或菜单条目，资源或组件名为 android:windowLightNavigationBar。 -->
        <item name="android:windowLightNavigationBar">true</item>
        <!-- 声明主题属性或菜单条目，资源或组件名为 android:navigationBarColor。 -->
        <item name="android:navigationBarColor">#FFFFFF</item>
    </style>
</resources>
~~~

<a id="file-20"></a>

### build.gradle.kts

~~~kotlin
plugins { id("com.android.application") version "9.4.0" apply false } // 应用或声明 Android 构建插件。
~~~

<a id="file-21"></a>

### settings.gradle.kts

~~~kotlin
pluginManagement { repositories { google(); mavenCentral(); gradlePluginPortal() } } // 声明构建插件的下载仓库。
dependencyResolutionManagement { // 集中设置模块依赖解析规则。
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS) // 限制子模块单独配置仓库。
    repositories { google(); mavenCentral() } // 配置 Google 和 Maven 依赖仓库。
} // 结束当前构建配置块。
rootProject.name = "BMI-MVI" // 设置工程显示名称。
include(":app") // 把应用模块纳入构建。
~~~
