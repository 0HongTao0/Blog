---

title: Android View

date: 2018-07-18 16:20:47

## tags:

---

##### 自定义控件的核心

- UI 界面的绘制（自定义 View，自绘 UI）

- 业务逻辑的控制（View 控件的事件监听，数据绑定）

##### 自定义控件实现

- 继承系统控件：如：继承 TextView 添加一些事件监听

- 组合系统控件：如：继承 LinearLayout，通过 LayoutInflater 动态加载布局文件到此 LinearLayout 中形成一个新控件。

- 自定义绘制控件：如：继承 View ，通过重写 onDraw() 方法，在 Canvas 中绘制所需的 UI 界面

  以上实现方式，即可单独使用，又可以联合使用。


<!--more-->


##### 介绍几个重要的类

- LayoutInflater：一个用于动态加载布局的类。

  获得 LayoutInflater 的方式

  ```java
  //通过 Context 获取，其实也是通过系统服务获取，只是 Android 封装了一层方便使用
   LayoutInflater inflater = LayoutInflater.from(context);
  // 通过 SystemService （系统服务） 获取
  LayoutInflater inflater =(LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
  ```

  如何使用 LayoutInflater 动态加载布局

  ```java
  // resource : XNL layout 布局的资源 id
  // root ： 动态加载布局的父布局
  // attachToRoot ： 是否要将加载的布局添加到 root（父容器）中
  View view =  inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot);
  ```

  关于 attachToRoot 的说明

| attachToRoot | root     | 作用                                   |
|:------------:|:--------:|:------------------------------------:|
| *            | null     | 不起作用                                 |
| true         | not null | 将加载的布局指定父布局为（root） 根节点的属性都有效         |
| false        | not null | 加载的布局没有指定父布局，根节点的属性失效。需要手动 addView（） |
| 不填           | not null | 不填默认就是 attachToRoot = true           |

- AttributeSet ：XML 资源文件中定义属性的集合。如：在资源文件中定义的 layout_height layout_width 等属性，可以在 java 代码中获取到属性对应的值，进而对控件的逻辑进行实现扩展。自定义 AttrbuteSet 与 获取 AttrbuteSet 值
  - 在 res 资源文件夹的 values 下创建 attrs.xml 文件

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <resources>
        <!--name：指定自定义属性所对应的自定义控件。NewCalendar 是自定义的控件-->
        <declare-styleable name="NewCalendar">
            <!--name：自定义属性的名字-->
            <!--format：自定义属性值的属性-->
            <attr name="dateFormat" format="string"/>
        </declare-styleable>
    </resources>
    ```

  - 在布局 XML 文件中使用

    ```xml
    <xyz.awqingnian.study.view.NewCalendar
        app:dateFormat="yyyy M"
        android:id="@+id/main_nc_calendar"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    </xyz.awqingnian.study.view.NewCalendar>
    ```

  - 在 java 代码中取出对应的自定义属性的值

    ```java
    // 获取对应控件的属性
    TypedArray typedArray = getContext().obtainStyledAttributes(attrs,R.styleable.NewCalendar);
    // R.styleable.NewCalendar_dateFormat 此值是 Android 通过 attrs.xml 拼接生成的值
    String format = typedArray.getString(R.styleable.NewCalendar_dateFormat);
    // 回收 typeArray ，让下一个调用者使用。在 typedArray.recycle() 后不能再使用 typedArray 的东西
    typedArray.recycle();
    ```

##### 继承系统控件与自定义绘制控件实现步骤（两者通常都是一起实现的）：

- 继承 Android 已有的控件

- 重写 onDraw() 方法

- 给自定义控件设置数据，并在自定义控件的 onDraw() 方法中进行逻辑操作

下面一个简单的例子：通过设置 isToday 的数据，在此自定义控件中的 onDraw() 方法内画个圆圈

```java
/**
 * 继承 Android 系统已有的控件（AppCompatTextView）
 */
public class CalendarDayTextView extends android.support.v7.widget.AppCompatTextView {
    /**
     * 自定义控件的数据部分（当然也可以通过 set 方法进行设置）
     */
    public boolean isToday = false;
    private Paint mPaint = new Paint();

    public CalendarDayTextView(Context context) {
        super(context);
        initControl();
    }

    public CalendarDayTextView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        initControl();
    }

    public CalendarDayTextView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initControl();
    }

    private void initControl() {
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setColor(Color.BLUE);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        // 自定义控件的数据逻辑操作部分
        if (isToday) {
            canvas.translate(getWidth() / 2, getHeight() / 2);
            canvas.drawCircle(0, 0, getWidth() / 2, mPaint);
        }
    }
}
```

##### 组合系统控件实现步骤：

- 新建布局文件，并将需要组合的控件添加进去并进行排版操作。（calendar_view.xml）

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_margin="16dp"
    android:orientation="vertical">

    <RelativeLayout
        android:id="@+id/calendar_header"
        android:layout_width="match_parent"
        android:layout_height="30dp">
        <!--详细设计省略-->
    </RelativeLayout>

    <LinearLayout
        android:id="@+id/calender_ll_week_header"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:orientation="horizontal">
    <!--详细设计省略-->
  </LinearLayout>

    <GridView
        android:id="@+id/calender_gv_date"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:numColumns="7"></GridView>
</LinearLayout>
```

- 创建 java 文件，继承 ViewGroup 的子类(如：LinearLayout ，RelativeLayout 等)，主要是在此布局中加载步骤一的布局文件，所以要用到 ViewGroup 的子类 装载子控件。(NewCalendar.java)

```java
public class NewCalendar extends LinearLayout {
    private ImageView mIvPrev;
    private ImageView mIvNext;
    private TextView mTvDate;
    private GridView mGvDate;
    private Calendar mCalendar = Calendar.getInstance();
    private String displayFormat = "yyyy M";
    public NewCalendarListener mNewCalendarListener;

    public NewCalendar(Context context) {
        super(context);
    }

    public NewCalendar(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        initControl(context, attrs);
    }

    public NewCalendar(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initControl(context, attrs);
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {

    }

    private void initControl(Context context, AttributeSet attrs) {
        binView(context);
        bindViewEvent();
        renderCalender();
    }

    /**
     * 加载布局文件，并且 findViewById 找到布局文件中各控件
     *
     * @param context
     */
    private void binView(Context context) {
          // 获取 LayoutInflater
        LayoutInflater inflater = LayoutInflater.from(context);
          // 通过 inflater 加载 R.layout.calendar_view 到 this（LinearLayout） 中
        View view = inflater.inflate(R.layout.calendar_view, this);
                // 通过 view 找到对应 res ID 的控件
        mIvPrev = view.findViewById(R.id.calendar_iv_prev);
        mIvNext = view.findViewById(R.id.calendar_iv_next);
        mTvDate = view.findViewById(R.id.calendar_tv_date);
        mGvDate = view.findViewById(R.id.calender_gv_date);
    }

    private void bindViewEvent() {
        //······ 具体逻辑操作省略 ······
    }

    private void renderCalender() {
        //······ 具体逻辑操作省略 ······
    }

    /**
     * 日历控件的 Adapter
     */
    private class CalendarAdapter extends ArrayAdapter<Date> {

        LayoutInflater mLayoutInflater;

        public CalendarAdapter(@NonNull Context context, int resource, List<Date> dates) {
            super(context, resource, dates);
            mLayoutInflater = LayoutInflater.from(context);
        }

        @NonNull
        @Override
        public View getView(int position, @Nullable View convertView, @NonNull ViewGroup parent) {
           //······ 具体逻辑操作省略 ······
            return convertView;
        }
    }
}
```

- 使用方法，直接在资源文件中像普通控件一样使用

```xml
<xyz.awqingnian.study.view.NewCalendar
    android:id="@+id/main_nc_calendar"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
</xyz.awqingnian.study.view.NewCalendar>
```

#### 上面具体代码实现就是一个自定义日历控件。

#### 具体代码可参考[GitHub](https://github.com/0HongTao0/CalendarView)

![CalendarViewPicture](https://github.com/0HongTao0/CalendarView/blob/master/CalendarViewTest.jpg?raw=true)
