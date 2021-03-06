## 简介
1. 通过编写Xml，配置一个动作列表，点击选项时执行指定的shell脚本
2. 为了方便第三方ROM修改爱好者快速定制自己的个性化选项功能
3. 通过界面交互的方式，允许用户在执行脚本前进行一些输入或选择操作
4. 加入Get Set机制（分别执行两段脚本），从而在界面上显示Switch开关

## 定义Action
- 在assets下添加，actions.xml，格式如下：
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<actions>
    <action>...</action>
    <action>...</action>
    ...
</actions>
```

### action基本定义（在1.0.0中实现）
```xml
<!--action
        [root]=[true/false]，是否以root权限执行，默认为false。
        [confirm]=[true/false]，操作是否需要二次确认，避免误操作，默认为false
        [start]=[dir]，执行脚本时的起始位置，将在执行script前cd到起始位置，默认位置为/cache
-->
<action root="true" confirm="false">
    <title>功能标题</title>
    <desc>功能说明</desc>
    <!--script 点击后要执行的脚本（可以直接要执行的文件路径、或要执行的代码）
            内容将支持三种方式
            1.脚本文件或程序路径，如：/system/etc/init.d/00test
            2.要执行的代码内容（不需要#!/xbin/sh等类似标头），如：echo 'hello world！'; echo '执行完毕！';
            3.assets内嵌资源文件，路径以file:///android_asset开头，如：file:///android_asset/scripts/test.sh，执行时会自动提取并运行-->
    <script>echo 'hello world！'</scripts>
</action>
```

### action定义 示例
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<actions>
    <action root="true">
        <title>脚本执行器测试</title>
        <desc>测试脚本执行器，执行内嵌的脚本文件</desc>
        <script>file:///android_asset/scripts/test.sh</script>
    </action>
    <action root="true">
        <title>脚本执行器测试</title>
        <desc>测试脚本执行器，直接执行代码段</desc>
        <script>
            echo '现在，开始执行脚本了！';
            testvalue='1'
            echo 'testvalue=$testvalue';
            echo '好了，代码执行完毕！';
        </script>
    </action>
</actions>
```

### 定义action参数（在1.0.1中实现）
```xml
<action root="true">
    <title>自定义DPI</title>
    <desc>允许你自定义手机DPI，1080P屏幕推荐DPI为400~480，设置太高或太低可能导致界面崩溃！</desc>
    <script>wm density $dpi;</script>
    <!--通过params定义脚本执行参数-->
    <params>
        <param name="dpi" />
    </params>
</action>
```

### 定义和使用参数
#### 1、通过单选列表指定参数
```xml
<action root="true">
    <title>切换状态栏风格</title>
    <desc>选择状态栏布局，[时间居中/默认]</desc>
    <!--可以在script中使用定义的参数-->
    <script>
        echo "mode参数的值：$mode"
        if [ "$mode" = "time_center" ]; then echo '刚刚点了 时间居中' fi;
    </script>
    <!--params 用于在执行脚本前，先通过用户交互的方式定义变量，参数数量不限于一个，但不建议定义太多-->
    <params>
        <!--param 定义单个参数
            [name]=[参数名]，指定参数名称，这是必需的！！！
            [value]=[默认值]，设置参数默认值（默认选中项）
            [val]=[默认值]，设置参数默认值（默认选中项），value的简写，作用等同-->
        <param name="mode" value="default">
            <!--通过option 自定义选项
                [value]=[当前选项的值] 如果不写这个属性，则默认使用显示文字作为值-->
            <option val="default">默认布局</option>
            <option val="time_center">时间居中</option>
        </param>
    </params>
</action>
```

#### 2、通过文本框录入参数
```xml
<action root="true">
    <title>自定义DPI</title>
    <desc>允许你自定义手机DPI，1080P屏幕推荐DPI为400~480，设置太高或太低可能导致界面崩溃！</desc>
    <script>
        wm density $dpi;
        wm size ${width}x${height};
    </script>
    <params>
        <!--如果不在param节点下定义option，则会直接在界面上显示文本输入框，让用户直接输入值，从而达到更灵活的配置
            [name],[value] 同上
            [desc]=[提示信息] 将显示在文本框底部作为提示
            [type]=[int] 限制输入类型，目前支持的类型有：
                            int 整数
                            bool 布尔值，用数字 1 或 0 表示 true 或 false
                            * 如果不设置此项则默认为不限制输入类型
                [readonly]=[readonly] 设置属性值不可修改，不需要则不谢
            [maxlength]=[最大输入长度] 限制可以输入几个字（包括空格和标点），不限制则不写（不推荐）
            注意：文本输入框不支持输入英文双引号（"）-->
        <param name="dpi" desc="请输入DPI，推荐值：400~480" type="int" value="440" maxlength="3" />
        <param name="width" desc="请输入屏幕横向分辨率" type="int" value="1080" maxlength="4" />
        <param name="height" desc="请输入屏幕纵向向分辨率" type="int" value="1920" maxlength="4" />
    </params>
</action>
```

#### 3、为脚本文件或程序传参
- 可参考 https://blog.csdn.net/czyt1988/article/details/79110450
```xml
<action root="true">
    <title>自定义DPI</title>
    <desc>允许你自定义手机DPI，1080P屏幕推荐DPI为400~480，设置太高或太低可能导致界面崩溃！</desc>
    <!--通过这种方式传参，则可以在脚本内通过 $1 $2...按顺序取得参数-->
    <script>/system/etc/init.d/00test $dpi $width $height</script>
    <params>
        <param name="dpi" desc="请输入DPI，推荐值：400~480" type="int" value="440" maxlength="3" />
        <param name="width" desc="请输入屏幕横向分辨率" type="int" value="1080" maxlength="4" />
        <param name="height" desc="请输入屏幕纵向向分辨率" type="int" value="1920" maxlength="4" />
    </params>
</action>
```

#### 4、为内嵌脚本传参
```xml
<action root="true">
    <title>自定义DPI</title>
    <desc>允许你自定义手机DPI，1080P屏幕推荐DPI为400~480，设置太高或太低可能导致界面崩溃！</desc>
    <!--通过file:///android_asset协议调用内置脚本时，会自动按顺序传递参数到脚本，直接在脚本内通过 $1 $2...按顺序取得参数即可-->
    <script>file:///android_asset/scripts/test.sh</script>
    <params>
        <param name="dpi" desc="请输入DPI，推荐值：400~480" type="int" value="440" maxlength="3" />
        <param name="width" desc="请输入屏幕横向分辨率" type="int" value="1080" maxlength="4" />
        <param name="height" desc="请输入屏幕纵向向分辨率" type="int" value="1920" maxlength="4" />
    </params>
</action>
```

### Action的Desc设置动态值（在1.0.4中实现）
- 为Action的desc节点增加了动态取值特性，通过[sh]=[使用普通权限执行的脚本代码]或[su]=[以ROOT权限执行的脚本代码]，执行脚本过程中输出的内容将作为显示内容
- 当然，除了设置[su]和[sh]执行脚本外，依然可以设置固定值，在脚本执行出错时，将继续显示固定值
```xml
<action root="true">
    <title>调整DPI</title>
    <desc su="echo '快速调整手机DPI，不需要重启，当前设置：`wm density`';">快速调整手机DPI，不需要重启</desc>
    <script>
        wm size reset;
        wm density $dpi;
        busybox killall com.android.systemui;
    </script>
    <params>
        <param name="dpi" value="440"></param>
    </params>
</action>
```


### Action的Param设置动态值（在1.0.4中实现）
- Param的value也允许通过自定义脚本来设置默认值了，新增了一个属性在param节点上
- 设置方式：[value-sh]=[要执行的脚本行]，执行脚本过程中输出的内容将作为显示内容
- 类似于上面设置Desc的方式，[value-sh]和[value]可以共存，当[value-sh]的脚本执行失败，将继续使用[value]的值
```xml
<action root="true">
    <title>调整DPI</title>
    <desc su="echo '快速调整手机DPI，不需要重启，当前设置：`wm density`';">快速调整手机DPI，不需要重启</desc>
    <script>
        wm size reset;
        wm density $dpi;
        busybox killall com.android.systemui;
    </script>
    <params>
        <param name="dpi" value="440" value-sh="echo '410';"></param>
    </params>
</action>
```

### Action的Param设置动态Options（在1.0.5中实现）
- 现在允许更灵活的定义Param的option列表了，通过使用脚本代码输出内，即可实现
- 设置方式：在param上定义[options-sh]=[脚本]，同样支持file:///assets_file 方式的脚本文件
- 脚本的执行过程中的输出内容，每一行将作为一个选项。如果你需要将选项的value（值）和label（显示文字）分开，用“|”分隔value和label即可，如：echo '380|很小（380）';
```xml
<action root="true">
    <title>调整DPI</title>
    <desc su="echo '快速调整手机DPI，不需要重启，当前设置：';echo `wm density`;" polling="2000">快速调整手机DPI，不需要重启</desc>
    <script>
        wm size reset;
        wm density $dpi;
        busybox killall com.android.systemui;
    </script>
    <params>
        <param name="dpi" value="440" options-sh="echo '380|很小（380）';echo '410|较小（410）';echo '440|适中（440）';echo '480|较大（480）';" />
    </params>
</action>
```


## 定义Switch（在1.0.3中实现）
#### 定义Switch列表
- 在assets目录下，添加 switchs.xml，格式如下
- 如果你不定义，界面上将不会显示开关栏目（在1.0.4中改进的）
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<switchs>
    <switch>...</switch>
    <switch>...</switch>
    ...
</switchs>
```

#### switch定义规范
```xml
<!--
    [root]=[true/false] 是否使用ROOT权限执行脚本
    [confirm]=[true/false]执行脚本前是否需要确认
-->
<switch root="true">
    <!--开关标题-->
    <title>流畅模式</title>
    <!--开关说明，同样支持动态设置内容，可参考Action的Desc设置-->
    <desc>在正常负载的情况下优先使用大核，大幅提高流畅度，但会降低续航能力</desc>
    <!--在应用启动时获取状态，用于设定开关显示状态，确保执行时间不会太长，避免启动时界面未响应-->
    <getstate>file:///android_asset/switchs/booster_get.sh</getstate>
    <!--设置状态，直接脚本时通过$state读取参数，file:///assets_file 方式的脚本文件，则通过 $1 获取参数-->
    <setstate>file:///android_asset/switchs/booster_set.sh</setstate>
</switch>
```

```xml
<switch root="true">
    <title>模拟全面屏</title>
    <desc>显示设置中的“全面屏”选项</desc>
    <getstate>
        if [ `grep qemu\.hw\.mainkeys= /system/build.prop|cut -d'=' -f2` = 1 ]; then
            echo 0;
        else
            echo 1;
        fi;
    </getstate>
    <setstate>
        busybox mount -o remount,rw -t auto /system;
        if [ $state == 1 ];then
            sed -i 's/^qemu\.hw\.mainkeys.*/qemu.hw.mainkeys=0/g' /system/build.prop
            echo '全面屏手势开启成功'
        else
            sed -i 's/^qemu\.hw\.mainkeys.*/qemu.hw.mainkeys=1/g' /system/build.prop
            echo '全面屏手势关闭成功'
        fi
        sync;
        echo '重启后生效！'
    </setstate>
</switch>
```

