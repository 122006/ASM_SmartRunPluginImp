## [ASM_SmartRunPluginImp 线程调度插件](src/main/groovy/com/by122006/buildsrc/ASM_SmartRunPluginImp.groovy)

适用于任何需要线程切换的程序。

你可以对方法进行简单注解，使方法在调用时自动运行在指定线程。

* 使用 **注解** 进行方法线程设定

         @UIThread
         public void xx(xx) {

         }
   * 使用`@UIThread`设置方法运行于主线程
   * 使用`@BGThread`设置方法运行于后台线程
   * 更多功能设置请参考 **高级功能**

* 同类对比

    1. 无需对类名、方法名进行额外字符附加，与正常方法调用 **写法完全相同**
    2. 无需预编译生成build类
    3. 占用编译时间只有部分相似插件的**5%**
    4. 消耗的方法量和原生线程调用写法完成相同
    5. 支持返回值，方法等待，方法超时等功能

* 插件引入

    Step 1. 在你的根目录项目`build.gradle`文件中加入以下仓库目录及插件依赖

	    allprojects {
		    repositories {
		    	...
		    	maven { url 'https://jitpack.io' }
		    }
		    dependencies {
		        ...
                classpath 'com.github.122006.ASM_SmartRunPluginImp:ASMPlugin:版本号'
            }
	    }
    Step 2. 在需要使用插件的module的`build.gradle`文件中开头增加以下内容

	    apply plugin: 'smartrun'

    以及增加工具类依赖

	    dependencies {
	        ...
	        compile 'com.github.122006.ASM_SmartRunPluginImp:Utils:版本号'
	    }

    当前版本号：[![](https://jitpack.io/v/122006/ASM_SmartRunPluginImp.svg)](https://jitpack.io/#122006/ASM_SmartRunPluginImp)


* 使用 **方法** 进行方法线程设定

    > 适用于Lambda表达式等无法增加方法注解的地方

         public void xx(xx) {
            xxx//任意代码
            ThreadUtils.toBgThread();
            xxx//任意代码
         }
   * 使用`ThreadUtils.toUIThread();`设置方法运行于主线程
   * 使用`ThreadUtils.toBgThread();`设置方法运行于后台线程
   * 该方法调用只是标记，没有实际调用意义，只要在方法中存在，**无论逻辑是否运行该方法**
   * 注解设置优先于方法设置

* 高级功能

    >`BGThread`：后台线程 `UIThread`：主线程，下同

    >eg. @BGThread(Style = Async, OutTime = 2000, Result = Skip)

    * **支持跨线程返回**

        >@BGThread(OutTime = 2000, Result = Wait)

        1. `Result`：是否等待返回 （等待：`Wait` 跳过：`Skip`）
        2. 使用该方法后，调用线程会被挂起（如果调用线程为`UIThread`，其不会被挂起），等待方法返回、或者超时于`OutTime`后，返回方法返回值的默认值并恢复原线程运行
        3. 只适用于`BGThread`。`UIThread`不应该被等待返回
        4. 默认：不等待方法返回值(`Result = Skip`)。如果设置需要返回，默认超时`OutTime`为2000
        5. 具体返回类型与方法返回值有关。如果方法返回值为`void`且设置需要返回，原线程依然会被堵塞直到方法返回，即使不会有任何返回值

    * 需要返回值时支持超时

        1. 见上条

    * 设置同步异步线程运行方案

        >@BGThread(Style = Async)

        1. `Style`：当前方法的运行模式 （异步：`Async` 同步：`Sync`）
        2. 如果调用者线程和方法线程均为 `BGThread`，`Sync`会直接调用方法，`Async`会在新线程中调用方法
        3. `Sync`将 **忽略** 设置的超时及返回值（`OutTime`、`Result`）
        4. 默认：异步(`Style = Async`)
        4. 只适用于`BGThread`。`UIThread`均为同步任务

    * **支持父类/接口继承**

        1. 继承顺序：按照代码顺序依次搜寻所有父类和接口中已注明的线程信息内容，优先搜索父类而不是接口
        2. 只会继承本插件相关的注解信息，不影响其他注解
        3. 因此，你可以在接口中注解线程信息，使应用到的所有子类都使用同一规则

* 其他注意事项

   * **同继承参数类型方法调用不正确**：如果有多个同名方法，且参数被继承或者为基础类型及基础类型包装器，可能会调用错误的方法(eg:xx(int)和xx(Integer))

   * **方法调用结构变动**：方法正常调用会被插入处理方法，如果检索方法调用堆栈会检索到处理方法而不是原调用方法。且实际运行方法也会被重命名。不影响正常使用

   * 混淆时报错：
        1. 使用4.7及以上的混淆器
        2. 混淆配置文件中加入行 `-dontpreverify`

   * **不完全兼容预编译的相关第三方库**
        eg. 含有DataBinding相关代码的文件
