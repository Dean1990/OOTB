#### **该项目停止维护**

由于项目一直在引用添加第三方包，变的越来越大，而且有一部分包是新项目用不到的，违背了当初的开始这个项目的初衷。我把最初的一部分代码和积累下来的工具类提取出来，开始了一个新项目 [OotbLite](https://github.com/Dean1990/ootbLite) ，该项目不再会引用多余的第三方包，只提供参考。

#### 简介

这是一个Android开发的工具集合,或者叫CodeBase。  [![](https://jitpack.io/v/Dean1990/OOTB.svg)](https://jitpack.io/#Dean1990/OOTB)

#### 配置

##### 1. build.gradle(Project)

```groovy
allprojects {
	repositories {
	    ...
	    maven { url "https://jitpack.io" }
	    ...
	}
}
```

##### 2. build.gradle(Module)

```groovy
dependencies {
	...
	compile 'com.github.Dean1990:OOTB:2.4.9.1'
	...
}
```

如果你的项目中使用的引用与该项目中的引用冲突，请使用 exclude 排除，比如 org.greenrobot:eventbus 不同版本的冲突

```groovy
dependencies {
	...
    compile ('com.github.Dean1990:OOTB:2.4.9.1',{
            exclude group: 'org.greenrobot',module:'eventbus'
        })
    ...
}
```

如果与其他引用冲突，也可采用此法，具体操作[百度](https://www.baidu.com/s?wd=gradle%20exclude)

##### 3. application类

```java
@Override
public void onCreate() {
    super.onCreate();
    ...
    //放在使用OOTB的最前面
    OotbConfig.init(this,true);
    //网络请求设置
    OotbConfig.setRequestServer(Constants.serviceUrl, null, new UserResult(),new DefaultLoadingDialog());
    ...
}
```

##### 4. 权限

并不要求所有权限都声明，按需添加

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.CAMERA" />
```

##### 5.集成中可能遇到问题

- 因为集成的东西比较多，会出现一个打包时超过方法上限的问题，解决办法引用：

  ```groovy
  implementation 'com.android.support:multidex:1.0.0'
  ```

  Application继承 MultiDexApplication类

  ```java
  public class App extends MultiDexApplication
  ```

  设置app 的 build.gradle 文件 设置 `multiDexEnabled = true`

  ```groovy
  android {
      compileSdkVersion 28
      defaultConfig {
          applicationId "com.deanlib.xxxx"
          minSdkVersion 17
          targetSdkVersion 28
          versionCode 1
          versionName "1.0"
          multiDexEnabled = true
   	}
  }
  ```

- Rajava需要在app 的 build.gradle文件设置 `packagingOptions {exclude 'META-INF/rxjava.properties'}`

  ```groovy
  android {
      defaultConfig {...}
      packagingOptions {exclude 'META-INF/rxjava.properties'}
  }
  ```


以上内容就可以正常使用OOTB，以下为扩展

##### 6. 假数据测试

###### 有时，接口还没有开发好，但需要使用假数据调试

```java
OotbConfig.setRequestFalseData(false, 1000);
```
###### 然后修改网络请求的request实现类的parse方法的参数值

```java
@Override
public Object parse(String s) {

    s = "{\"msg\":\"Success\",\"result\":[{\"imagesId\":123,\"createTime\":{\"date\":17,\"hours\":15,\"seconds\":48,\"month\":6,\"timezoneOffset\":-480,\"year\":117,\"minutes\":21,\"time\":1500276108000,\"day\":1},\"resource\":\"我\",\"isRecommended\":12,\"id\":1,\"title\":\"小猪快跑\",\"content\":\"我\",\"createDate\":\"2017-07-17\"}],\"code\":\"200\"}";

    JSONObject object = JSON.parseObject(s);

    return object;
}
```

##### 7. 定义全局的网络请求参数

```java
public class UserReqParam implements IRequestParam {
		@Override
	    public RequestParams disposeParam(RequestParams requestParams) {
	    
	        //可以在这个方法里设置一些全局网络请求参数
	        if (requestParams!=null){
	            if (UserManager.isUserLogin()){
	                DLogUtils.d("User Header>>>token:"+UserManager.user.token);
	                requestParams.addHeader("token",UserManager.user.token);
	                requestParams.setReadTimeout(30000);
	            }
	
	        }
	
	        return requestParams;
	    }
	}
```

##### 8. 定义网络请求返回值

```java
public class UserResult extends Result {
  //例如接口数据标准是这样的 {"code": "200","msg": "this is msg","data": {"xxx":"xxx",...}}
  //需要public声明其中的'code'和'msg'
  public String code;
  public String msg;
  
  public UserResult() {
    //指定请求成功的状态码
    //例如接口规定是“200”，或者是“1”，或者是什么什么...(http协议无关)
    super("200");
  }
  //多种构造函数可使用
  public UserResult(String successCode,HashMap<String, String> resultCodeMap) {
    super(successCode,resultCodeMap);
  }
  
  @Override
  public String getResultCode() {
     return code;//指定数据请求的代表状态码，及上面声明的字段code
  }

  @Override
  public String getResultMsg() {
     return msg;//指定数据请求的代表服务器信息，及上面声明的字段msg
  }

  /**
  * 请求结果 自行分析处理
  *
  * @param code
  * @return 返回true，表示分析由用户自己完成，返回false，表示分析由用户和request一起完成
  */
  @Override
  public boolean onResultParse(String code) {
    //可以在这里设置一些全局的网络请求结果的相应操作
    //例如接口规定 code返回123时登录过期,即可如下操作
    if ("123".equals(code)){
      UserManager.deleteUser();
    }
    return super.onResultParse(code);
  }
}
```
当返回结果不是以“返回码”做为数据正确与否的判断时，返回的数据没有规律可言的情况下，可以放弃定义网络请求返回值，OotbConfig.setRequestServer方法Request.Result参数传null，又或是当大部分返回数据是有规律的，只有个别数据不守规矩，可以在继承Request抽象类时，在构造方法中调用 super(Conext context,boolean resultDeal) 方式，设置resultDeal为false，来屏蔽返回数据的自动处理部分（自动处理部分可以通过“返回码”决定数据的取舍）。

##### 9. 定义网络请求加载框

```java
public class UserLoadingDialog extends ILoadingDialog {

  @Override
  public Dialog showLoadingDialog(Activity activity,String msg) {

    //自定义加载框
    View view = LayoutInflater.from(activity).inflate(R.layout.layout_loading, null);
    ImageView imgLoading = view.findViewById(R.id.imgLoading);

    Animation animation = AnimationUtils.loadAnimation(activity, R.anim.round_rotate);
    animation.setInterpolator(new LinearInterpolator());
    imgLoading.setImageResource(R.mipmap.loading);
    imgLoading.startAnimation(animation);

    return new AlertDialog.Builder(activity, R.style.TransparentBGDialog).setCancelable(false).setView(view).show();
  }

  @Override
  public void dismissLoadingDialog() {
    //网络请求结束，关闭加载框时会调用该方法
  }

}
```

由于加载框的结束时间（dismiss）是在Request.RequestCallback 的所有调用方法之后，及 onFinished() 之后，所以在Request.RequestCallback 的所有调用方法中，都不建议使用Activity.finish() 等关闭Activity的方法（虽然不会造成App 闪退，日志会报错“android.view.WindowLeaked”），因为加载框是附着在 Activity之上的（从 public Request(Context context) 传过去的，如果context 传值并非 Activity ，不会出现任何加载框，请忽略该模块讲解内容），所以Request 提供了dismissDialog() 方法，并且建议在Activity的基类中使用。

```java
@Override
public void finish() {
    Request.dismissDialog();
    super.finish();
}
```

##### 10.定义一个接口类

```java
public class TestReq extends Request {
	int id;
    public TestReq(Context context,int id) {
        super(context);
      	this.id = id;
    }

    @Override
    public String getName() {
        return getClass().getSimpleName();//补充这个返回值内容，有益于日志的显示
    }

    @Override
    public RequestParams params() {
      	//接口地址和参数
      	RequestParams params = new RequestParams(Request.SERVER+"/test.html");
      	params.addBodyParameter("id",id+"");
        return params;
    }

    @Override
    public Entity parse(String json) {
		//做近一步解析处理
		//json = "{\"code\":\"12\",\"msg\":\"this is msg\",\"data\":{\"eid\":1112,\"name\":\"this is test\"}}";
        Data data = JSON.parseObject(json,Data.class);
      	if(data!=null)
        	return data.result;
      	return null;
    }

  	//定义具体的类继承自定义的Result,用于fastjson的解析
    static class Data extends UserResult{
        public Entity data;
    }
}
```

Tip: 在第7步中定义UserResult时，可以利用泛型技术

```java
public class UserResult<T> extends Result {
  public String code;
  public String msg;
  public T data;
```

这样做可以省掉内部静态类Data，在parse方法中这样写

```java
@Override
public Entity parse(String json) {
    UserResult<Entity> o = 
       JSON.parseObject(json, new UserResult<Entity>(){}.getEntityType());
    return o.data;
}
```

这里的getEntityType()方法是基类Result的方法，用于获取带泛型的Result类，不能直接使用getClass()，因为它会擦除泛型，并且注意“{}”大括号的使用，大括号是必须的。

一直想把定义接口时的声明内部静态类和parse方法的实现去掉，从Request基类中一步实现，返回需要的类型的数据，可悲是还没有实现。

##### 11.使用接口类请求数据

```java
new TestReq(this,1).execute(new Request.RequestCallback<Entity>() {
    @Override
    public void onSuccess(Entity o) {
        //请求成功执行
        if(o!=null)
      	   Toast.makeText(getApplicationContext(),
                o.getEid()+":"+o.getName(),Toast.LENGTH_SHORT).show();
    }
    @Override
    public void onError(Throwable ex, boolean isOnCallback) {
        //请求失败执行
        ex.printStackTrace();
    }
    @Override
    public void onCancelled(Callback.CancelledException cex) {
        //请求被取消执行，Request的execute是会返回Callback.Cancelable的句柄
      	//如果需要取消，可以调用Callback.Cancelable.cancel()方法，在这个请求没有完成之前。
	}
	@Override
    public void onFinished() {
		//请求完成时执行(无论是成功或失败或者取消)
    }
});
```



#### 文件目录

* data
  * db
    * DB.java（数据库管理）
  * io
    * DefaultLoadingDialog.java（提供的默认加载框）
    * Request.java（网络请求的抽象类）
      * ILoadingDialog.java（加载框自定义接口）
      * IRequestParam.java（请求参数自定义接口）
      * RequestCallback.java（网络请求的回调）
      * Result.java（返回参数自定义接口）
  * FileUtils.java（文件操作）
  * ImageUtils.java（图片操作）
  * MediaUtils.java（多媒体相关）
  * PersistenceUtils.java（持久化操作,有时感觉这个类有点鸡肋）

* entity

  * BaseEntiy.java（基础实体类）

* manager
  * NetworkManager.java（网络管理）
  * PermissionManager.java（权限管理）
  * ​

* utils
  * AppUtils.java（应用相关）
  * DeviceUtils.java（设备相关）
  * DLogUtils.java（日志操作）
  * FormatUtils.java（格式化操作）
  * MathUtils（数值计算操作）
  * PinyinUtils.java（拼音相关）
  * PopupUtils（弹出界面相关）
  * TextUtils.java（文本操作）
  * ValidateUtils.java（数据验证相关）
  * VersionUtils.java（应用版本相关）
  * WifiAutoConnectUtils（自动连接WIFI工具）

* widget

  - CustomScrollView （带滚动监听，可替换低版本中ScrollView）

  * GridViewForScrollView.java（支持嵌套在ScrollView中的GridView）
  * LazyViewPager.java（懒加载的ViewPager）
  * ListViewForScrollView.java（支持嵌套在ScrollView中的ListView）
  * loopviewpager
    * LoopPagerAdapterWrapper.java（支持类）
    * LoopViewPager.java（循环的ViewPager）

OotbConfig.java（配置文件，使用ootb需要先调用该类中的init函数）

#### 引用

>'org.xutils:xutils:3.5.0'
>
>>感谢xUtils，多年来一直在用这个框架，所以这个工具有很大一部分功能是对xUtils的再包装。

>'com.alibaba:fastjson:1.2.41'
>
>>JSON操作

>'com.github.bumptech.glide:glide:3.7.0'
>
>>图片加载

>'com.jakewharton.rxbinding:rxbinding:0.4.0'

>'com.tbruyelle.rxpermissions:rxpermissions:0.9.3@aar'
>
>>以上两个用来权限申请

>'com.github.PhilJay:MPAndroidChart:v3.0.2'
>
>>图表绘制

>'cn.bingoogolapple:bga-qrcodecore:1.1.7@aar'
>
>>扫码包装

>'cn.bingoogolapple:bga-zbar:1.1.7@aar'
>
>>扫码zbar核心

>'cn.bingoogolapple:bga-zxing:1.1.7@aar'
>
>>扫码zxing核心

>'org.greenrobot:eventbus:3.0.0'
>
>>消息传递

>'q.rorbin:VerticalTabLayout:1.2.5'
>
>>竖向的TabLayout

>'com.zhy:flowlayout-lib:1.0.3'
>
>>横向流式布局

>'se.emilsjolander:stickylistheaders:2.7.0'
>
>>带标题(header)悬停的ListView

>'com.github.Dean1990:BothwayListview:-SNAPSHOT'
>
>>双向滑动的ListView

>'com.belerweb:pinyin4j:2.5.1'
>
>>拼音相关

>'com.scwang.smartrefresh:SmartRefreshLayout:1.0.3'
>
>>上下拉加载

>'org.jsoup:jsoup:1.11.3'
>
>>Jsoup HTML操作

> 'com.jakewharton.rxbinding2:rxbinding:2.1.1'
>
> > 点击绑定 防抖 如果能和 butterknife 合二为一就好，这是下一步应该考虑的问题

> 'io.reactivex:rxjava:1.0.14'
> 'io.reactivex:rxandroid:1.0.1'
>
> > RxJava 异步

> 'jp.wasabeef:glide-transformations:3.2.0'
> 'jp.co.cyberagent.android.gpuimage:gpuimage-library:1.4.1'
>
> > Glide的辅助类，圆角，圆形，多边，滤镜，应有尽有

> 'com.kevin:loopview:1.4.1'
>
> > 循环播放的Banner

