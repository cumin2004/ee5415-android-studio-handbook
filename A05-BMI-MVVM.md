# A05-BMI-MVVM 中文源码讲解

MainActivity 保存身高、年龄和性别到偏好文件。日期和 BMI 写入 SQLite，同一天重新计算会替换旧值。HistoryActivity 用包含本月月初、不包含下月月初的区间查询，BmiChartView 把日期和 BMI 映射成横纵坐标。测试月底跨年、无记录月份和同日重复写入。

## 文件目录

- [app/src/main/java/com/example/bmi/BmiCalculator.java](#file-1)
- [app/src/main/java/com/example/bmi/BmiChartView.java](#file-2)
- [app/src/main/java/com/example/bmi/BmiDatabase.java](#file-3)
- [app/src/main/java/com/example/bmi/BmiModel.java](#file-4)
- [app/src/main/java/com/example/bmi/BmiRecord.java](#file-5)
- [app/src/main/java/com/example/bmi/BmiResult.java](#file-6)
- [app/src/main/java/com/example/bmi/BmiViewModel.java](#file-7)
- [app/src/main/java/com/example/bmi/HistoryActivity.java](#file-8)
- [app/src/main/java/com/example/bmi/MainActivity.java](#file-9)
- [app/src/main/java/com/example/bmi/ReportActivity.java](#file-10)
- [app/build.gradle.kts](#file-11)
- [app/src/main/AndroidManifest.xml](#file-12)
- [app/src/main/res/layout-land/activity_main.xml](#file-13)
- [app/src/main/res/layout-land/activity_report.xml](#file-14)
- [app/src/main/res/layout/activity_history.xml](#file-15)
- [app/src/main/res/layout/activity_main.xml](#file-16)
- [app/src/main/res/layout/activity_report.xml](#file-17)
- [app/src/main/res/menu/menu_context.xml](#file-18)
- [app/src/main/res/menu/menu_options.xml](#file-19)
- [app/src/main/res/values-zh/strings.xml](#file-20)
- [app/src/main/res/values/strings.xml](#file-21)
- [app/src/main/res/values/styles.xml](#file-22)
- [build.gradle.kts](#file-23)
- [settings.gradle.kts](#file-24)

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

### app/src/main/java/com/example/bmi/BmiChartView.java

~~~java
package com.example.bmi; // 声明源码所在包，供同包类型引用。

import android.content.Context; // 引入 Context 类型供本文件使用。
import android.graphics.Canvas; // 引入 Canvas 类型供本文件使用。
import android.graphics.Color; // 引入 Color 类型供本文件使用。
import android.graphics.Paint; // 引入 Paint 类型供本文件使用。
import android.util.AttributeSet; // 引入 AttributeSet 类型供本文件使用。
import android.view.View; // 引入 View 类型供本文件使用。
import java.util.ArrayList; // 引入 ArrayList 类型供本文件使用。
import java.util.List; // 引入 List 类型供本文件使用。
import java.util.Locale; // 引入 Locale 类型供本文件使用。

public class BmiChartView extends View { // 自定义 View，把一个月的 BMI 记录画成折线图。
    private final Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG); // 用抗锯齿画笔绘制线条和文字。
    private List<BmiRecord> records = new ArrayList<>(); // 初始数据为空列表。
    private int days = 31; // 保存本月天数用于计算横坐标。

    public BmiChartView(Context context, AttributeSet attrs) { // 布局从 XML 创建控件时调用此构造器。
        super(context, attrs); // 将上下文和 XML 属性交给父类解析。
    } // 结束当前代码块。

    public void setRecords(List<BmiRecord> values, int daysInMonth) { // 接收当前月份的数据及天数。
        records = new ArrayList<>(values); // 复制输入列表，避免调用者后续修改影响绘制。
        days = daysInMonth; // 记住月份长度。
        StringBuilder description = new StringBuilder("BMI"); // 开始构造读屏描述。
        for (BmiRecord record : records) description.append(String.format(Locale.getDefault(), // 按当前地区格式逐条追加记录。
                ", %s: %.2f", record.date, record.bmi)); // 在描述中列出日期和两位小数的数值。
        setContentDescription(description.toString()); // 为辅助功能提供整张图的文字替代。
        invalidate(); // 请求系统重新绘制控件。
    } // 结束当前代码块。

    private float dp(float value) { return value * getResources().getDisplayMetrics().density; } // 把布局使用的密度单位转换为绘图使用的像素。

    @Override // 覆盖父类或接口声明的方法。
    protected void onDraw(Canvas canvas) { // 系统要求控件绘制时进入。
        super.onDraw(canvas); // 先执行父类的绘制逻辑。
        float left = dp(48), right = getWidth() - dp(18); // 给左侧刻度和右边界留出空间。
        float top = dp(28), bottom = getHeight() - dp(42); // 给图标题和横轴标签预留高度。
        if (right <= left || bottom <= top) return; // 可绘区域为空时结束绘制。
        double low = 18, high = 28; // 空数据时采用默认纵轴区间。
        if (!records.isEmpty()) { // 有记录时改用实际数据范围。
            low = records.get(0).bmi; // 从第一条记录初始化下界。
            high = low; // 初始上界与首条记录相同。
            for (BmiRecord record : records) { // 逐条扫描月份内的记录。
                low = Math.min(low, record.bmi); // 更新最小 BMI。
                high = Math.max(high, record.bmi); // 更新最大 BMI。
            } // 结束当前代码块。
            low = Math.floor(low - 1); // 在最小值下面留一个单位并向下取整。
            high = Math.ceil(high + 1); // 在最大值上方留白并向上取整。
        } // 结束当前代码块。
        paint.setTextSize(dp(12)); // 设置刻度文字尺寸。
        paint.setStrokeWidth(dp(1)); // 设置网格线宽度。
        for (int i = 0; i <= 4; i++) { // 五条水平刻度把区域分成四段。
            float y = bottom - (bottom - top) * i / 4; // 从数据刻度反算向下增长的屏幕纵坐标。
            paint.setColor(0xFFE0E0E0); // 设置浅灰网格色。
            canvas.drawLine(left, y, right, y, paint); // 画当前高度的水平参考线。
            paint.setColor(Color.DKGRAY); // 换深灰色标记坐标。
            paint.setTextAlign(Paint.Align.RIGHT); // 让纵轴标签向绘图区左边缘对齐。
            canvas.drawText(String.format(Locale.getDefault(), "%.1f", low + (high - low) * i / 4), // 用当前刻度对应的 BMI 值生成标签。
                    left - dp(8), y + dp(4), paint); // 把文字放到网格线左边。
        } // 结束当前代码块。
        paint.setColor(Color.DKGRAY); // 换深灰色标记坐标。
        canvas.drawLine(left, top, left, bottom, paint); // 画纵轴。
        canvas.drawLine(left, bottom, right, bottom, paint); // 画横轴。
        paint.setTextAlign(Paint.Align.CENTER); // 横轴日期以刻度位置为中心。
        for (int day : new int[]{1, 8, 15, 22, days}) { // 选出月初、周刻度和月底。
            float x = left + (right - left) * (day - 1) / (days - 1); // 把月内日期映射到左右边界之间的像素。
            canvas.drawText(Integer.toString(day), x, bottom + dp(18), paint); // 在对应横坐标下写日期。
        } // 结束当前代码块。
        canvas.drawText(getResources().getString(R.string.day_axis), (left + right) / 2, // 在横轴中心画轴说明。
                getHeight() - dp(4), paint); // 把轴说明放到控件底端。
        paint.setTextAlign(Paint.Align.LEFT); // 后续图题改用左对齐。
        canvas.drawText("BMI", left, dp(16), paint); // 在图顶标出纵轴含义。
        paint.setColor(0xFF3F51B5); // 切换为折线与数据点的颜色。
        paint.setStrokeWidth(dp(2)); // 设置折线宽度。
        float previousX = 0, previousY = 0; // 保存上一点的坐标用于连线。
        for (int i = 0; i < records.size(); i++) { // 按数据库排序逐条绘制数据。
            BmiRecord record = records.get(i); // 取得本次要画的日期与数值。
            int day = Integer.parseInt(record.date.substring(8)); // 从固定格式日期取出当月日号。
            float x = left + (right - left) * (day - 1) / (days - 1); // 把月内日期映射到左右边界之间的像素。
            float y = bottom - (float) ((record.bmi - low) / (high - low)) * (bottom - top); // 将 BMI 在数据范围中的占比映射为纵坐标。
            if (i > 0) canvas.drawLine(previousX, previousY, x, y, paint); // 从第二条数据起把相邻点连起来。
            canvas.drawCircle(x, y, dp(4), paint); // 画当前数据点。
            previousX = x; // 保存当前横坐标供下一轮连线。
            previousY = y; // 保存当前纵坐标供下一轮连线。
        } // 结束当前代码块。
    } // 结束当前代码块。
} // 结束当前代码块。
~~~

<a id="file-3"></a>

### app/src/main/java/com/example/bmi/BmiDatabase.java

~~~java
package com.example.bmi; // 声明源码所在包，供同包类型引用。

import android.content.ContentValues; // 引入 ContentValues 类型供本文件使用。
import android.content.Context; // 引入 Context 类型供本文件使用。
import android.database.Cursor; // 引入 Cursor 类型供本文件使用。
import android.database.sqlite.SQLiteDatabase; // 引入 SQLiteDatabase 类型供本文件使用。
import android.database.sqlite.SQLiteOpenHelper; // 引入 SQLiteOpenHelper 类型供本文件使用。
import android.database.sqlite.SQLiteException; // 引入 SQLiteException 类型供本文件使用。
import java.text.SimpleDateFormat; // 引入 SimpleDateFormat 类型供本文件使用。
import java.util.ArrayList; // 引入 ArrayList 类型供本文件使用。
import java.util.Calendar; // 引入 Calendar 类型供本文件使用。
import java.util.List; // 引入 List 类型供本文件使用。
import java.util.Locale; // 引入 Locale 类型供本文件使用。

public class BmiDatabase extends SQLiteOpenHelper { // 用 SQLiteOpenHelper 管理 BMI 历史库。
    public BmiDatabase(Context context) { // 创建或打开应用的本地历史库。
        super(context, "bmi_history.db", null, 1); // 指定数据库名称和初始结构版本。
    } // 结束当前代码块。

    @Override // 覆盖父类或接口声明的方法。
    public void onCreate(SQLiteDatabase db) { // 数据库首次建立时初始化表。
        db.execSQL("CREATE TABLE bmi_records (record_date TEXT PRIMARY KEY NOT NULL, " // 日期设为非空主键，使同日只能保留一条记录。
                + "bmi REAL NOT NULL CHECK(bmi > 0))"); // 用实数列保存 BMI，并要求数值大于零。
    } // 结束当前代码块。

    @Override // 覆盖父类或接口声明的方法。
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) { } // 当前版本没有定义跨版本迁移步骤。

    public void save(String date, double bmi) { // 持久化指定日期的一次有效结果。
        ContentValues values = new ContentValues(); // 把列和值装入 ContentValues。
        values.put("record_date", date); // 写入日期主键。
        values.put("bmi", bmi); // 写入 BMI 数值。
        long row = getWritableDatabase().insertWithOnConflict("bmi_records", null, values, // 执行带主键冲突处理的插入。
                SQLiteDatabase.CONFLICT_REPLACE); // 若日期已存在，用本次值替换同日旧记录。
        if (row == -1) throw new SQLiteException("Could not save BMI record"); // 检查插入结果并报告失败。
    } // 结束当前代码块。

    public List<BmiRecord> forMonth(int year, int month) { // 按指定年月读取并排序。
        Calendar start = Calendar.getInstance(); // 准备计算月初边界。
        start.clear(); // 清掉当前时分秒等残留字段。
        start.set(year, month, 1); // 将日期设为传入月份的一号。
        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd", Locale.US); // 固定日期编码形式，使文本顺序与时间顺序一致。
        String from = format.format(start.getTime()); // 生成包含在查询内的起始日期。
        start.add(Calendar.MONTH, 1); // 推进一个月，跨年由 Calendar 处理。
        String until = format.format(start.getTime()); // 生成不包含在区间内的下一月月初。
        List<BmiRecord> records = new ArrayList<>(); // 用列表收集查询结果。
        try (Cursor cursor = getReadableDatabase().query("bmi_records", // 打开可读数据库并自动管理游标。
                new String[]{"record_date", "bmi"}, "record_date >= ? AND record_date < ?", // 只取日期及 BMI，条件为左闭右开的时间区间。
                new String[]{from, until}, null, null, "record_date ASC")) { // 绑定两个日期参数，并按日期升序排序。
            while (cursor.moveToNext()) records.add(new BmiRecord(cursor.getString(0), cursor.getDouble(1))); // 遍历游标，每行转换为一条类型化记录。
        } // 结束当前代码块。
        return records; // 把整月数据交给界面。
    } // 结束当前代码块。
} // 结束当前代码块。
~~~

<a id="file-4"></a>

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

<a id="file-5"></a>

### app/src/main/java/com/example/bmi/BmiRecord.java

~~~java
package com.example.bmi; // 声明源码所在包，供同包类型引用。

public final class BmiRecord { // 将日期与 BMI 封装成不可变记录。
    public final String date; // 保存统一格式的日期文本。
    public final double bmi; // 记录原始 BMI 值。

    public BmiRecord(String date, double bmi) { // 用数据库读出的值初始化记录。
        this.date = date; // 写入日期成员。
        this.bmi = bmi; // 记录计算所得数值。
    } // 结束当前代码块。
} // 结束当前代码块。
~~~

<a id="file-6"></a>

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

<a id="file-7"></a>

### app/src/main/java/com/example/bmi/BmiViewModel.java

~~~java
package com.example.bmi; // 声明源码所在包，供同包类型引用。

import androidx.lifecycle.LiveData; // 引入 LiveData 类型供本文件使用。
import androidx.lifecycle.MutableLiveData; // 引入 MutableLiveData 类型供本文件使用。
import androidx.lifecycle.ViewModel; // 引入 ViewModel 类型供本文件使用。

public class BmiViewModel extends ViewModel { // 用 ViewModel 保存和发布计算结果。
    private final BmiModel model = new BmiModel(); // 持有负责校验与计算的模型。
    private final MutableLiveData<BmiResult> result = new MutableLiveData<>(); // 内部可修改的 LiveData 承载输出。
    public LiveData<BmiResult> getResult() { return result; } // 页面取得只读接口来订阅结果。
    public void calculate(String height, String weight, String age, String gender) { // 接收页面交来的原始表单数据。
        result.setValue(model.calculate(height, weight, age, gender)); // 模型算完后发布结果，触发观察者。
    } // 结束当前代码块。
    public void clearResult() { result.setValue(null); } // 清空已处理结果，防止界面重建时重复打开报告。
} // 结束当前代码块。
~~~

<a id="file-8"></a>

### app/src/main/java/com/example/bmi/HistoryActivity.java

~~~java
package com.example.bmi; // 声明源码所在包，供同包类型引用。

import android.os.Bundle; // 引入 Bundle 类型供本文件使用。
import android.view.MenuItem; // 引入 MenuItem 类型供本文件使用。
import android.view.View; // 引入 View 类型供本文件使用。
import android.view.Gravity; // 引入 Gravity 类型供本文件使用。
import android.widget.LinearLayout; // 引入 LinearLayout 类型供本文件使用。
import android.widget.TextView; // 引入 TextView 类型供本文件使用。
import androidx.appcompat.app.AppCompatActivity; // 引入 AppCompatActivity 类型供本文件使用。
import java.text.SimpleDateFormat; // 引入 SimpleDateFormat 类型供本文件使用。
import java.util.Calendar; // 引入 Calendar 类型供本文件使用。
import java.util.List; // 引入 List 类型供本文件使用。
import java.util.Locale; // 引入 Locale 类型供本文件使用。

public class HistoryActivity extends AppCompatActivity { // 历史页管理月份切换、图形和表格。
    private final Calendar month = Calendar.getInstance(); // 日历对象代表当前正在浏览的月份。
    private BmiDatabase database; // 保存数据库访问对象。

    @Override // 覆盖父类或接口声明的方法。
    protected void onCreate(Bundle savedInstanceState) { // 页面创建时绑定布局和交互逻辑。
        super.onCreate(savedInstanceState); // 先交给父类初始化。
        setContentView(R.layout.activity_history); // 加载历史页 XML。
        if (getSupportActionBar() != null) getSupportActionBar().setDisplayHomeAsUpEnabled(true); // 存在操作栏时提供返回入口。
        month.set(Calendar.DAY_OF_MONTH, 1); // 统一从该月第一天计算查询边界。
        if (savedInstanceState != null) month.setTimeInMillis(savedInstanceState.getLong("month")); // 旋转重建时恢复原先查看的月份。
        database = new BmiDatabase(this); // 创建历史数据库帮助类实例。
        findViewById(R.id.previous_month).setOnClickListener(view -> { // 上个月按钮响应在这里注册。
            month.add(Calendar.MONTH, -1); // 日历后退一个月，允许跨年。
            refresh(); // 按更新后的月份重新查询并绘制。
        }); // 结束回调并完成注册或提交。
        findViewById(R.id.next_month).setOnClickListener(view -> { // 下个月按钮响应在这里注册。
            month.add(Calendar.MONTH, 1); // 日历前进一个月。
            refresh(); // 按更新后的月份重新查询并绘制。
        }); // 结束回调并完成注册或提交。
        refresh(); // 按更新后的月份重新查询并绘制。
    } // 结束当前代码块。

    private void refresh() { // 统一重建本月的标题、计数、图和表。
        List<BmiRecord> records = database.forMonth(month.get(Calendar.YEAR), month.get(Calendar.MONTH)); // 用日历中的年份、从零开始的月份查记录。
        ((TextView) findViewById(R.id.month_label)).setText( // 定位显示当前月份的标题控件。
                new SimpleDateFormat("yyyy-MM", Locale.US).format(month.getTime())); // 把月份格式化为年与两位数月份。
        ((TextView) findViewById(R.id.record_count)).setText(getString(R.string.records_count, records.size())); // 用查询条数填入数量文案。
        findViewById(R.id.empty_history).setVisibility(records.isEmpty() ? View.VISIBLE : View.GONE); // 当前月没有记录时显示空状态提示。
        ((BmiChartView) findViewById(R.id.bmi_chart)).setRecords(records, // 把查询出的记录交给图表。
                month.getActualMaximum(Calendar.DAY_OF_MONTH)); // 用本月实际天数作为横轴上界。
        LinearLayout table = findViewById(R.id.history_rows); // 定位容纳每行记录的竖排容器。
        table.removeAllViews(); // 清空旧月份的行，防止重复追加。
        addRow(table, getString(R.string.date_column), getString(R.string.bmi_column), true); // 先用本地化的列名建立表头。
        for (BmiRecord record : records) { // 逐条扫描月份内的记录。
            addRow(table, record.date, String.format(Locale.getDefault(), "%.2f", record.bmi), false); // 每条记录显示日期与两位小数的 BMI。
        } // 结束当前代码块。
    } // 结束当前代码块。

    private void addRow(LinearLayout table, String date, String bmi, boolean heading) { // 向表格添加一行，参数决定是否为表头。
        LinearLayout row = new LinearLayout(this); // 创建单行容器。
        row.setOrientation(LinearLayout.HORIZONTAL); // 两列采用水平方向排列。
        TextView dateText = new TextView(this); // 建立日期文本控件。
        TextView bmiText = new TextView(this); // 建立数值文本控件。
        dateText.setText(date); // 填入日期或日期列标题。
        bmiText.setText(bmi); // 填入 BMI 或数值列标题。
        dateText.setTextSize(16); // 设定日期文字大小。
        bmiText.setTextSize(16); // 设定数值文字大小。
        bmiText.setGravity(Gravity.END); // 将数值与对应列末端对齐。
        if (heading) { // 表头需要与数据行区分。
            dateText.setTypeface(null, android.graphics.Typeface.BOLD); // 把日期标题设为粗体。
            bmiText.setTypeface(null, android.graphics.Typeface.BOLD); // 把数值标题设为粗体。
        } // 结束当前代码块。
        int space = (int) (8 * getResources().getDisplayMetrics().density); // 把八个密度单位换算为像素。
        row.setPadding(0, space, 0, space); // 为该行添加上下间距。
        row.addView(dateText, new LinearLayout.LayoutParams(0, -2, 1)); // 让第一列按等份权重占用剩余空间。
        row.addView(bmiText, new LinearLayout.LayoutParams(0, -2, 1)); // 第二列使用相同权重。
        table.addView(row); // 把完整的一行加入历史表格。
    } // 结束当前代码块。

    @Override // 覆盖父类或接口声明的方法。
    protected void onSaveInstanceState(Bundle state) { // 系统保存页面临时状态时回调。
        state.putLong("month", month.getTimeInMillis()); // 记录浏览月份，旋转后恢复。
        super.onSaveInstanceState(state); // 让父类保存其它界面状态。
    } // 结束当前代码块。

    @Override // 覆盖父类或接口声明的方法。
    protected void onDestroy() { // 页面销毁时释放资源。
        database.close(); // 关闭该页面持有的数据库帮助类。
        super.onDestroy(); // 执行父类清理。
    } // 结束当前代码块。

    @Override // 覆盖父类或接口声明的方法。
    public boolean onOptionsItemSelected(MenuItem item) { // 接收顶栏菜单点击事件。
        if (item.getItemId() == android.R.id.home) { // 检查操作栏的返回动作。
            finish(); // 关闭当前页面，返回上一层。
            return true; // 返回真，告知调用方本次处理成功。
        } // 结束当前代码块。
        return super.onOptionsItemSelected(item); // 其它条目回退给父类处理。
    } // 结束当前代码块。
} // 结束当前代码块。
~~~

<a id="file-9"></a>

### app/src/main/java/com/example/bmi/MainActivity.java

~~~java
package com.example.bmi; // 声明源码所在包，供同包类型引用。

import android.content.Intent; // 引入 Intent 类型供本文件使用。
import android.content.SharedPreferences; // 引入 SharedPreferences 类型供本文件使用。
import android.app.DatePickerDialog; // 引入 DatePickerDialog 类型供本文件使用。
import android.database.sqlite.SQLiteException; // 引入 SQLiteException 类型供本文件使用。
import android.widget.Button; // 引入 Button 类型供本文件使用。
import android.widget.Toast; // 引入 Toast 类型供本文件使用。
import java.text.SimpleDateFormat; // 引入 SimpleDateFormat 类型供本文件使用。
import java.util.Calendar; // 引入 Calendar 类型供本文件使用。
import java.util.Locale; // 引入 Locale 类型供本文件使用。
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

public class MainActivity extends AppCompatActivity { // 定义使用兼容库的主页面。
    private EditText heightInput, weightInput, ageInput; // 持有三个输入框，供按钮事件取值。
    private RadioGroup genderGroup; // 持有性别单选组，便于读取当前选中的按钮。
    private TextView genderError; // 持有性别错误提示文字，未选时显示。
    private BmiViewModel viewModel; // 保存当前页所属的 ViewModel。
    private BmiDatabase database; // 保存数据库访问对象。
    private Calendar recordDate; // 记录用户当前选定的日期。
    private Button dateButton; // 保留日期按钮引用，用于更新显示。

    private void updateDateButton() { // 用选中日期重新生成按钮文字。
        dateButton.setText(getString(R.string.record_date, // 使用本地化资源组合日期标签。
                new SimpleDateFormat("yyyy-MM-dd", Locale.US).format(recordDate.getTime()))); // 按固定年月日格式写入按钮。
    } // 结束当前代码块。

    @Override // 覆盖父类或接口声明的方法。
    protected void onSaveInstanceState(Bundle state) { // 系统保存页面临时状态时回调。
        state.putLong("record_date", recordDate.getTimeInMillis()); // 保存临时选中的日期供界面重建。
        super.onSaveInstanceState(state); // 让父类保存其它界面状态。
    } // 结束当前代码块。

    @Override // 覆盖父类或接口声明的方法。
    protected void onDestroy() { // 页面销毁时释放资源。
        if (database != null) database.close(); // 只有已初始化数据库时才关闭。
        super.onDestroy(); // 执行父类清理。
    } // 结束当前代码块。
    @Override // 覆盖父类或接口声明的方法。
    protected void onCreate(Bundle savedInstanceState) { // 页面创建时绑定布局和交互逻辑。
        super.onCreate(savedInstanceState); // 先交给父类初始化。
        setContentView(R.layout.activity_main); // 加载主页面布局。控件必须等布局加载后才能查找。
        heightInput = findViewById(R.id.height_input); // 用 XML 中的身高 ID 绑定输入框。
        weightInput = findViewById(R.id.weight_input); // 用 XML 中的体重 ID 绑定输入框。
        ageInput = findViewById(R.id.age_input); // 用 XML 中的年龄 ID 绑定输入框。
        genderGroup = findViewById(R.id.gender_group); // 绑定单选组本身，读取的是组内被选按钮的 ID。
        genderError = findViewById(R.id.gender_error); // 绑定单选错误文字。
        database = new BmiDatabase(this); // 创建历史数据库帮助类实例。
        recordDate = Calendar.getInstance(); // 首次进入时默认使用今天。
        if (savedInstanceState != null) recordDate.setTimeInMillis(savedInstanceState.getLong("record_date")); // 页面重建时恢复之前选定的日期。
        dateButton = findViewById(R.id.record_date_button); // 绑定日期选择按钮。
        updateDateButton(); // 根据当前选定日期更新按钮文案。
        dateButton.setOnClickListener(view -> new DatePickerDialog(this, (picker, year, month, day) -> { // 点击按钮时打开日期选择器。
            recordDate.set(year, month, day); // 把回调中的年月日存入日历。
            updateDateButton(); // 根据当前选定日期更新按钮文案。
        }, recordDate.get(Calendar.YEAR), recordDate.get(Calendar.MONTH), // 用当前选定的年和月初始化选择器。
                recordDate.get(Calendar.DAY_OF_MONTH)).show()); // 补上日期并显示选择器。
        findViewById(R.id.history_button).setOnClickListener(view -> // 监听历史按钮的点击。
                startActivity(new Intent(this, HistoryActivity.class))); // 显式打开历史页。
        if (savedInstanceState == null) { // 首次创建才用偏好设置填表，避免覆盖旋转状态。
            SharedPreferences profile = getSharedPreferences("profile", MODE_PRIVATE); // 打开应用私有的资料偏好文件。
            heightInput.setText(profile.getString("height", "")); // 恢复上次使用的身高。
            ageInput.setText(profile.getString("age", "")); // 恢复上次使用的年龄。
            String gender = profile.getString("gender", ""); // 读取上次保存的性别。
            if ("Male".equals(gender)) genderGroup.check(R.id.gender_male); // 若为男性则恢复相应单选项。
            else if ("Female".equals(gender)) genderGroup.check(R.id.gender_female); // 若为女性则恢复相应单选项。
        } // 结束当前代码块。
        genderGroup.setOnCheckedChangeListener((group, id) -> genderError.setVisibility(View.GONE)); // 一旦用户选择性别，隐藏先前的未选提示。
        registerForContextMenu(findViewById(R.id.bmi_icon)); // 注册图标的长按上下文菜单。
        viewModel = new ViewModelProvider(this).get(BmiViewModel.class); // 通过生命周期管理器取得或复用 ViewModel。
        viewModel.getResult().observe(this, result -> { // 注册观察者，根据状态驱动界面。
            if (result != null) { // 忽略表示已消费的空结果。
                showResult(result); // 将有效或带错误标记的结果交给渲染方法。
                viewModel.clearResult(); // 显示后清空本次事件。
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
            viewModel.calculate(height, weight, age, gender); // 把表单内容交给 ViewModel。
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
        String date = new SimpleDateFormat("yyyy-MM-dd", Locale.US).format(recordDate.getTime()); // 生成用于数据库主键的日期字符串。
        try { // 在这里执行可能抛出异常的操作。
            database.save(date, result.bmi); // 把该日期的 BMI 保存到历史表。
        } catch (SQLiteException error) { // 处理数据库写入错误。
            Toast.makeText(this, R.string.storage_error, Toast.LENGTH_LONG).show(); // 用长提示向用户说明保存失败。
            return; // 在此结束当前方法或回调。
        } // 结束当前代码块。
        getSharedPreferences("profile", MODE_PRIVATE).edit() // 打开资料偏好文件并开始批量编辑。
                .putString("height", heightInput.getText().toString().trim()) // 写入修剪后的身高文本。
                .putString("age", ageInput.getText().toString().trim()) // 写入修剪后的年龄文本。
                .putString("gender", result.gender).apply(); // 保存性别并提交偏好变更。
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

<a id="file-10"></a>

### app/src/main/java/com/example/bmi/ReportActivity.java

~~~java
package com.example.bmi; // 声明源码所在包，供同包类型引用。

import android.os.Bundle; // 引入 Bundle 类型供本文件使用。
import android.content.Intent; // 引入 Intent 类型供本文件使用。
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
        findViewById(R.id.history_button).setOnClickListener(view -> // 监听历史按钮的点击。
                startActivity(new Intent(this, HistoryActivity.class))); // 显式打开历史页。
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

<a id="file-11"></a>

### app/build.gradle.kts

~~~kotlin
plugins { id("com.android.application") } // 应用或声明 Android 构建插件。
android { // 进入安卓模块设置。
    namespace = "com.example.bmi" // 生成资源类的命名空间。
    compileSdk { version = release(33) } // 选择编译时可用的安卓接口。
    defaultConfig { // 统一配置应用标识与支持版本。
        applicationId = "com.example.bmi.storage" // 指定安装包在设备上的唯一标识。
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

<a id="file-12"></a>

### app/src/main/AndroidManifest.xml

~~~xml
<?xml version='1.0' encoding='utf-8'?>
<!-- 声明应用组件以及安卓属性命名空间。 -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 设置应用级主题、图标与备份策略。 -->
    <application android:allowBackup="false" android:label="@string/app_name" android:theme="@style/AppTheme" android:supportsRtl="true">
        <!-- 注册可由 Intent 启动的页面，资源或组件名为 .ReportActivity，是否对其它应用开放为 false。 -->
        <activity android:name=".ReportActivity" android:label="@string/report_title" android:exported="false" />
        <!-- 注册可由 Intent 启动的页面，资源或组件名为 .MainActivity，是否对其它应用开放为 true。 -->
        <activity android:name=".MainActivity" android:label="@string/title" android:exported="true">
            <!-- 组合用于匹配入口的动作和类别。 -->
            <intent-filter>
                <!-- 声明入口动作，资源或组件名为 android.intent.action.MAIN。 -->
                <action android:name="android.intent.action.MAIN" />
                <!-- 声明匹配此入口的类别，资源或组件名为 android.intent.category.LAUNCHER。 -->
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <!-- 注册可由 Intent 启动的页面，资源或组件名为 .HistoryActivity，是否对其它应用开放为 false。 -->
        <activity android:name=".HistoryActivity" android:label="@string/history_title" android:exported="false" />
    </application>
</manifest>
~~~

<a id="file-13"></a>

### app/src/main/res/layout-land/activity_main.xml

~~~xml
<?xml version='1.0' encoding='utf-8'?>
<!-- 让超出屏幕的内容可滚动。 -->
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android" android:layout_width="match_parent" android:layout_height="match_parent" android:fillViewport="true">
    <!-- 按方向依次排列子控件，子控件排列方向为 horizontal。 -->
    <LinearLayout android:layout_width="match_parent" android:layout_height="wrap_content" android:padding="24dp" android:orientation="horizontal">
        <!-- 按方向依次排列子控件，剩余空间权重为 1，子控件排列方向为 vertical。 -->
        <LinearLayout android:layout_width="0dp" android:layout_weight="1" android:layout_height="wrap_content" android:orientation="vertical">
            <!-- 显示标签、摘要或计算结果，显示文字为 @string/height，关联输入框为 @id/height_input。 -->
            <TextView android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/height" android:labelFor="@id/height_input" android:textSize="14sp" />
            <!-- 提供可读取和校验的输入框，控件编号为 @+id/height_input，输入类型为 numberDecimal。 -->
            <EditText android:id="@+id/height_input" android:layout_width="match_parent" android:layout_height="48dp" android:inputType="numberDecimal" android:singleLine="true" android:saveEnabled="true" />
            <!-- 显示标签、摘要或计算结果，显示文字为 @string/weight，关联输入框为 @id/weight_input。 -->
            <TextView android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/weight" android:labelFor="@id/weight_input" android:textSize="14sp" />
            <!-- 提供可读取和校验的输入框，控件编号为 @+id/weight_input，输入类型为 numberDecimal。 -->
            <EditText android:id="@+id/weight_input" android:layout_width="match_parent" android:layout_height="48dp" android:inputType="numberDecimal" android:singleLine="true" android:saveEnabled="true" />
            <!-- 显示标签、摘要或计算结果，显示文字为 @string/age，关联输入框为 @id/age_input。 -->
            <TextView android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/age" android:labelFor="@id/age_input" android:textSize="14sp" />
            <!-- 提供可读取和校验的输入框，控件编号为 @+id/age_input，输入类型为 number。 -->
            <EditText android:id="@+id/age_input" android:layout_width="match_parent" android:layout_height="48dp" android:inputType="number" android:singleLine="true" android:saveEnabled="true" />
        </LinearLayout>
        <!-- 按方向依次排列子控件，剩余空间权重为 1，子控件排列方向为 vertical。 -->
        <LinearLayout android:layout_width="0dp" android:layout_weight="1" android:layout_height="wrap_content" android:layout_marginStart="24dp" android:orientation="vertical">
            <!-- 显示图标或其他图片，控件编号为 @+id/bmi_icon，图像来源为 @drawable/bmi_icon。 -->
            <ImageView android:id="@+id/bmi_icon" android:layout_width="48dp" android:layout_height="48dp" android:layout_gravity="center_horizontal" android:layout_marginBottom="12dp" android:src="@drawable/bmi_icon" android:contentDescription="@string/bmi_icon" />
            <!-- 显示标签、摘要或计算结果，显示文字为 @string/gender。 -->
            <TextView android:layout_width="wrap_content" android:layout_height="wrap_content" android:text="@string/gender" android:layout_marginTop="8dp" />
            <!-- 组织互斥的性别选项，控件编号为 @+id/gender_group，子控件排列方向为 horizontal。 -->
            <RadioGroup android:id="@+id/gender_group" android:layout_width="match_parent" android:layout_height="wrap_content" android:orientation="horizontal">
                <!-- 提供单选组中的一个候选项，控件编号为 @+id/gender_male，显示文字为 @string/male。 -->
                <RadioButton android:id="@+id/gender_male" android:layout_width="wrap_content" android:layout_height="48dp" android:text="@string/male" />
                <!-- 提供单选组中的一个候选项，控件编号为 @+id/gender_female，显示文字为 @string/female。 -->
                <RadioButton android:id="@+id/gender_female" android:layout_width="wrap_content" android:layout_height="48dp" android:text="@string/female" android:layout_marginStart="16dp" />
            </RadioGroup>
            <!-- 显示标签、摘要或计算结果，控件编号为 @+id/gender_error，显示文字为 @string/select_gender，初始可见性为 gone。 -->
            <TextView android:id="@+id/gender_error" android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/select_gender" android:textColor="#B71C1C" android:visibility="gone" />
            <!-- 提供点击操作入口，在页面源码中绑定监听，控件编号为 @+id/record_date_button。 -->
            <Button android:id="@+id/record_date_button" android:layout_width="match_parent" android:layout_height="48dp" android:layout_marginTop="8dp" android:textAllCaps="false" />
            <!-- 提供点击操作入口，在页面源码中绑定监听，控件编号为 @+id/calculate_button，显示文字为 @string/calculate。 -->
            <Button android:id="@+id/calculate_button" android:layout_width="match_parent" android:layout_height="48dp" android:text="@string/calculate" android:textAllCaps="false" android:layout_marginTop="12dp" />
            <!-- 提供点击操作入口，在页面源码中绑定监听，控件编号为 @+id/history_button，显示文字为 @string/view_history。 -->
            <Button android:id="@+id/history_button" android:layout_width="match_parent" android:layout_height="48dp" android:text="@string/view_history" android:textAllCaps="false" />
        </LinearLayout>
    </LinearLayout>
</ScrollView>
~~~

<a id="file-14"></a>

### app/src/main/res/layout-land/activity_report.xml

~~~xml
<?xml version='1.0' encoding='utf-8'?>
<!-- 让超出屏幕的内容可滚动。 -->
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android" android:layout_width="match_parent" android:layout_height="match_parent" android:fillViewport="true">
    <!-- 按方向依次排列子控件，子控件排列方向为 horizontal。 -->
    <LinearLayout android:layout_width="match_parent" android:layout_height="wrap_content" android:padding="24dp" android:orientation="horizontal">
        <!-- 按方向依次排列子控件，剩余空间权重为 1，子控件排列方向为 vertical。 -->
        <LinearLayout android:layout_width="0dp" android:layout_weight="1" android:layout_height="wrap_content" android:orientation="vertical">
            <!-- 显示标签、摘要或计算结果，控件编号为 @+id/bmi_result。 -->
            <TextView android:id="@+id/bmi_result" android:layout_width="match_parent" android:layout_height="wrap_content" android:textSize="32sp" android:textStyle="bold" />
        </LinearLayout>
        <!-- 按方向依次排列子控件，剩余空间权重为 3，子控件排列方向为 vertical。 -->
        <LinearLayout android:layout_width="0dp" android:layout_weight="3" android:layout_height="wrap_content" android:layout_marginStart="24dp" android:orientation="vertical">
            <!-- 显示标签、摘要或计算结果，控件编号为 @+id/profile_summary。 -->
            <TextView android:id="@+id/profile_summary" android:layout_width="match_parent" android:layout_height="wrap_content" android:textSize="18sp" android:layout_marginTop="16dp" />
            <!-- 显示标签、摘要或计算结果，控件编号为 @+id/age_group。 -->
            <TextView android:id="@+id/age_group" android:layout_width="match_parent" android:layout_height="wrap_content" android:textSize="18sp" android:layout_marginTop="8dp" />
            <!-- 显示标签、摘要或计算结果，控件编号为 @+id/bmi_category。 -->
            <TextView android:id="@+id/bmi_category" android:layout_width="match_parent" android:layout_height="wrap_content" android:textSize="22sp" android:layout_marginTop="16dp" />
            <!-- 显示标签、摘要或计算结果，控件编号为 @+id/bmi_note，显示文字为 @string/warning，初始可见性为 gone。 -->
            <TextView android:id="@+id/bmi_note" android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/warning" android:textColor="#B71C1C" android:layout_marginTop="12dp" android:visibility="gone" />
            <!-- 提供点击操作入口，在页面源码中绑定监听，控件编号为 @+id/history_button，显示文字为 @string/view_history。 -->
            <Button android:id="@+id/history_button" android:layout_width="match_parent" android:layout_height="48dp" android:layout_marginTop="20dp" android:text="@string/view_history" android:textAllCaps="false" />
            <!-- 提供点击操作入口，在页面源码中绑定监听，控件编号为 @+id/back_button，显示文字为 @string/back。 -->
            <Button android:id="@+id/back_button" android:layout_width="match_parent" android:layout_height="48dp" android:layout_marginTop="20dp" android:text="@string/back" android:textAllCaps="false" />
        </LinearLayout>
    </LinearLayout>
</ScrollView>
~~~

<a id="file-15"></a>

### app/src/main/res/layout/activity_history.xml

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 让超出屏幕的内容可滚动。 -->
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android" android:layout_width="match_parent" android:layout_height="match_parent" android:fillViewport="true">
    <!-- 按方向依次排列子控件，子控件排列方向为 vertical。 -->
    <LinearLayout android:layout_width="match_parent" android:layout_height="wrap_content" android:orientation="vertical" android:padding="20dp">
        <!-- 按方向依次排列子控件，子控件排列方向为 horizontal。 -->
        <LinearLayout android:layout_width="match_parent" android:layout_height="wrap_content" android:orientation="horizontal" android:gravity="center_vertical">
            <!-- 提供点击操作入口，在页面源码中绑定监听，控件编号为 @+id/previous_month，显示文字为 &lt;。 -->
            <Button android:id="@+id/previous_month" android:layout_width="48dp" android:layout_height="48dp" android:text="&lt;" android:contentDescription="@string/previous_month" />
            <!-- 显示标签、摘要或计算结果，控件编号为 @+id/month_label，剩余空间权重为 1。 -->
            <TextView android:id="@+id/month_label" android:layout_width="0dp" android:layout_weight="1" android:layout_height="wrap_content" android:gravity="center" android:textSize="22sp" />
            <!-- 提供点击操作入口，在页面源码中绑定监听，控件编号为 @+id/next_month，显示文字为 &gt;。 -->
            <Button android:id="@+id/next_month" android:layout_width="48dp" android:layout_height="48dp" android:text="&gt;" android:contentDescription="@string/next_month" />
        </LinearLayout>
        <!-- 显示标签、摘要或计算结果，显示文字为 @string/daily_latest。 -->
        <TextView android:layout_width="match_parent" android:layout_height="wrap_content" android:layout_marginTop="12dp" android:text="@string/daily_latest" android:textSize="14sp" />
        <!-- 显示标签、摘要或计算结果，控件编号为 @+id/record_count。 -->
        <TextView android:id="@+id/record_count" android:layout_width="match_parent" android:layout_height="wrap_content" android:layout_marginTop="8dp" android:textSize="16sp" />
        <!-- 配置 com.example.bmi.BmiChartView 组件的界面职责与尺寸，控件编号为 @+id/bmi_chart。 -->
        <com.example.bmi.BmiChartView android:id="@+id/bmi_chart" android:layout_width="match_parent" android:layout_height="280dp" android:layout_marginTop="12dp" />
        <!-- 显示标签、摘要或计算结果，控件编号为 @+id/empty_history，显示文字为 @string/no_records。 -->
        <TextView android:id="@+id/empty_history" android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/no_records" android:layout_marginTop="12dp" android:textSize="16sp" />
        <!-- 按方向依次排列子控件，控件编号为 @+id/history_rows，子控件排列方向为 vertical。 -->
        <LinearLayout android:id="@+id/history_rows" android:layout_width="match_parent" android:layout_height="wrap_content" android:layout_marginTop="20dp" android:orientation="vertical" />
    </LinearLayout>
</ScrollView>
~~~

<a id="file-16"></a>

### app/src/main/res/layout/activity_main.xml

~~~xml
<?xml version='1.0' encoding='utf-8'?>
<!-- 让超出屏幕的内容可滚动。 -->
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android" android:layout_width="match_parent" android:layout_height="match_parent" android:fillViewport="true">
    <!-- 按方向依次排列子控件，子控件排列方向为 vertical。 -->
    <LinearLayout android:layout_width="match_parent" android:layout_height="wrap_content" android:padding="24dp" android:orientation="vertical">
        <!-- 显示图标或其他图片，控件编号为 @+id/bmi_icon，图像来源为 @drawable/bmi_icon。 -->
        <ImageView android:id="@+id/bmi_icon" android:layout_width="64dp" android:layout_height="64dp" android:layout_gravity="center_horizontal" android:layout_marginBottom="12dp" android:src="@drawable/bmi_icon" android:contentDescription="@string/bmi_icon" />
        <!-- 显示标签、摘要或计算结果，显示文字为 @string/height，关联输入框为 @id/height_input。 -->
        <TextView android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/height" android:labelFor="@id/height_input" android:textSize="14sp" />
        <!-- 提供可读取和校验的输入框，控件编号为 @+id/height_input，输入类型为 numberDecimal。 -->
        <EditText android:id="@+id/height_input" android:layout_width="match_parent" android:layout_height="48dp" android:inputType="numberDecimal" android:singleLine="true" android:saveEnabled="true" />
        <!-- 显示标签、摘要或计算结果，显示文字为 @string/weight，关联输入框为 @id/weight_input。 -->
        <TextView android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/weight" android:labelFor="@id/weight_input" android:textSize="14sp" />
        <!-- 提供可读取和校验的输入框，控件编号为 @+id/weight_input，输入类型为 numberDecimal。 -->
        <EditText android:id="@+id/weight_input" android:layout_width="match_parent" android:layout_height="48dp" android:inputType="numberDecimal" android:singleLine="true" android:saveEnabled="true" />
        <!-- 显示标签、摘要或计算结果，显示文字为 @string/age，关联输入框为 @id/age_input。 -->
        <TextView android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/age" android:labelFor="@id/age_input" android:textSize="14sp" />
        <!-- 提供可读取和校验的输入框，控件编号为 @+id/age_input，输入类型为 number。 -->
        <EditText android:id="@+id/age_input" android:layout_width="match_parent" android:layout_height="48dp" android:inputType="number" android:singleLine="true" android:saveEnabled="true" />
        <!-- 显示标签、摘要或计算结果，显示文字为 @string/gender。 -->
        <TextView android:layout_width="wrap_content" android:layout_height="wrap_content" android:text="@string/gender" android:layout_marginTop="8dp" />
        <!-- 组织互斥的性别选项，控件编号为 @+id/gender_group，子控件排列方向为 horizontal。 -->
        <RadioGroup android:id="@+id/gender_group" android:layout_width="match_parent" android:layout_height="wrap_content" android:orientation="horizontal">
            <!-- 提供单选组中的一个候选项，控件编号为 @+id/gender_male，显示文字为 @string/male。 -->
            <RadioButton android:id="@+id/gender_male" android:layout_width="wrap_content" android:layout_height="48dp" android:text="@string/male" />
            <!-- 提供单选组中的一个候选项，控件编号为 @+id/gender_female，显示文字为 @string/female。 -->
            <RadioButton android:id="@+id/gender_female" android:layout_width="wrap_content" android:layout_height="48dp" android:text="@string/female" android:layout_marginStart="16dp" />
        </RadioGroup>
        <!-- 显示标签、摘要或计算结果，控件编号为 @+id/gender_error，显示文字为 @string/select_gender，初始可见性为 gone。 -->
        <TextView android:id="@+id/gender_error" android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/select_gender" android:textColor="#B71C1C" android:visibility="gone" />
        <!-- 提供点击操作入口，在页面源码中绑定监听，控件编号为 @+id/record_date_button。 -->
        <Button android:id="@+id/record_date_button" android:layout_width="match_parent" android:layout_height="48dp" android:layout_marginTop="8dp" android:textAllCaps="false" />
        <!-- 提供点击操作入口，在页面源码中绑定监听，控件编号为 @+id/calculate_button，显示文字为 @string/calculate。 -->
        <Button android:id="@+id/calculate_button" android:layout_width="match_parent" android:layout_height="48dp" android:text="@string/calculate" android:textAllCaps="false" android:layout_marginTop="12dp" />
        <!-- 提供点击操作入口，在页面源码中绑定监听，控件编号为 @+id/history_button，显示文字为 @string/view_history。 -->
        <Button android:id="@+id/history_button" android:layout_width="match_parent" android:layout_height="48dp" android:text="@string/view_history" android:textAllCaps="false" />
    </LinearLayout>
</ScrollView>
~~~

<a id="file-17"></a>

### app/src/main/res/layout/activity_report.xml

~~~xml
<?xml version='1.0' encoding='utf-8'?>
<!-- 让超出屏幕的内容可滚动。 -->
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android" android:layout_width="match_parent" android:layout_height="match_parent" android:fillViewport="true">
    <!-- 按方向依次排列子控件，子控件排列方向为 vertical。 -->
    <LinearLayout android:layout_width="match_parent" android:layout_height="wrap_content" android:padding="24dp" android:orientation="vertical">
        <!-- 显示标签、摘要或计算结果，控件编号为 @+id/bmi_result。 -->
        <TextView android:id="@+id/bmi_result" android:layout_width="match_parent" android:layout_height="wrap_content" android:textSize="32sp" android:textStyle="bold" />
        <!-- 显示标签、摘要或计算结果，控件编号为 @+id/profile_summary。 -->
        <TextView android:id="@+id/profile_summary" android:layout_width="match_parent" android:layout_height="wrap_content" android:textSize="18sp" android:layout_marginTop="16dp" />
        <!-- 显示标签、摘要或计算结果，控件编号为 @+id/age_group。 -->
        <TextView android:id="@+id/age_group" android:layout_width="match_parent" android:layout_height="wrap_content" android:textSize="18sp" android:layout_marginTop="8dp" />
        <!-- 显示标签、摘要或计算结果，控件编号为 @+id/bmi_category。 -->
        <TextView android:id="@+id/bmi_category" android:layout_width="match_parent" android:layout_height="wrap_content" android:textSize="22sp" android:layout_marginTop="16dp" />
        <!-- 显示标签、摘要或计算结果，控件编号为 @+id/bmi_note，显示文字为 @string/warning，初始可见性为 gone。 -->
        <TextView android:id="@+id/bmi_note" android:layout_width="match_parent" android:layout_height="wrap_content" android:text="@string/warning" android:textColor="#B71C1C" android:layout_marginTop="12dp" android:visibility="gone" />
        <!-- 提供点击操作入口，在页面源码中绑定监听，控件编号为 @+id/history_button，显示文字为 @string/view_history。 -->
        <Button android:id="@+id/history_button" android:layout_width="match_parent" android:layout_height="48dp" android:layout_marginTop="20dp" android:text="@string/view_history" android:textAllCaps="false" />
        <!-- 提供点击操作入口，在页面源码中绑定监听，控件编号为 @+id/back_button，显示文字为 @string/back。 -->
        <Button android:id="@+id/back_button" android:layout_width="match_parent" android:layout_height="48dp" android:layout_marginTop="20dp" android:text="@string/back" android:textAllCaps="false" />
    </LinearLayout>
</ScrollView>
~~~

<a id="file-18"></a>

### app/src/main/res/menu/menu_context.xml

~~~xml
<?xml version='1.0' encoding='utf-8'?>
<!-- 声明可加载到页面的菜单。 -->
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 声明主题属性或菜单条目，控件编号为 @+id/menu_about，菜单标题为 @string/about。 -->
    <item android:id="@+id/menu_about" android:title="@string/about" />
    <!-- 声明主题属性或菜单条目，控件编号为 @+id/menu_wiki，菜单标题为 @string/wiki。 -->
    <item android:id="@+id/menu_wiki" android:title="@string/wiki" />
</menu>
~~~

<a id="file-19"></a>

### app/src/main/res/menu/menu_options.xml

~~~xml
<?xml version='1.0' encoding='utf-8'?>
<!-- 声明可加载到页面的菜单。 -->
<menu xmlns:android="http://schemas.android.com/apk/res/android" xmlns:app="http://schemas.android.com/apk/res-auto">
    <!-- 声明主题属性或菜单条目，控件编号为 @+id/menu_about，菜单标题为 @string/about。 -->
    <item android:id="@+id/menu_about" android:title="@string/about" app:showAsAction="never" />
    <!-- 声明主题属性或菜单条目，控件编号为 @+id/menu_wiki，菜单标题为 @string/wiki。 -->
    <item android:id="@+id/menu_wiki" android:title="@string/wiki" app:showAsAction="never" />
    <!-- 声明主题属性或菜单条目，控件编号为 @+id/menu_exit，菜单标题为 @string/exit。 -->
    <item android:id="@+id/menu_exit" android:title="@string/exit" app:showAsAction="never" />
</menu>
~~~

<a id="file-20"></a>

### app/src/main/res/values-zh/strings.xml

~~~xml
<?xml version='1.0' encoding='utf-8'?>
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
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 record_date。 -->
    <string name="record_date">记录日期：%1$s</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 history_title。 -->
    <string name="history_title">BMI 历史记录</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 view_history。 -->
    <string name="view_history">查看 BMI 历史</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 previous_month。 -->
    <string name="previous_month">上个月</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 next_month。 -->
    <string name="next_month">下个月</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 no_records。 -->
    <string name="no_records">本月暂无 BMI 记录。</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 daily_latest。 -->
    <string name="daily_latest">同一天只保留最新一次结果。</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 records_count。 -->
    <string name="records_count">%1$d 条记录</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 date_column。 -->
    <string name="date_column">日期</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 bmi_column。 -->
    <string name="bmi_column">BMI</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 day_axis。 -->
    <string name="day_axis">日期</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 storage_error。 -->
    <string name="storage_error">无法保存结果，请重试。</string>
</resources>
~~~

<a id="file-21"></a>

### app/src/main/res/values/strings.xml

~~~xml
<?xml version='1.0' encoding='utf-8'?>
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
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 record_date。 -->
    <string name="record_date">Record date: %1$s</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 history_title。 -->
    <string name="history_title">BMI history</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 view_history。 -->
    <string name="view_history">View BMI history</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 previous_month。 -->
    <string name="previous_month">Previous month</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 next_month。 -->
    <string name="next_month">Next month</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 no_records。 -->
    <string name="no_records">No BMI records this month.</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 daily_latest。 -->
    <string name="daily_latest">The latest result is kept for each date.</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 records_count。 -->
    <string name="records_count">%1$d records</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 date_column。 -->
    <string name="date_column">Date</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 bmi_column。 -->
    <string name="bmi_column">BMI</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 day_axis。 -->
    <string name="day_axis">Day</string>
    <!-- 定义供布局和 Java 调用的本地化文案，资源或组件名为 storage_error。 -->
    <string name="storage_error">Could not save the result. Please try again.</string>
</resources>
~~~

<a id="file-22"></a>

### app/src/main/res/values/styles.xml

~~~xml
<?xml version='1.0' encoding='utf-8'?>
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

<a id="file-23"></a>

### build.gradle.kts

~~~kotlin
plugins { id("com.android.application") version "9.4.0" apply false } // 应用或声明 Android 构建插件。
~~~

<a id="file-24"></a>

### settings.gradle.kts

~~~kotlin
pluginManagement { repositories { google(); mavenCentral(); gradlePluginPortal() } } // 声明构建插件的下载仓库。
dependencyResolutionManagement { // 集中设置模块依赖解析规则。
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS) // 限制子模块单独配置仓库。
    repositories { google(); mavenCentral() } // 配置 Google 和 Maven 依赖仓库。
} // 结束当前构建配置块。
rootProject.name = "BMI-Storage" // 设置工程显示名称。
include(":app") // 把应用模块纳入构建。
~~~
