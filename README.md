#Android性能编码规范
<br/><br/><br/><br/>

[](id:目录)
##目录
<br/>

[前言](#前言)

[1.编码之初](#编码之初)

[2.禁止（避免）操作](#禁止)

[3.优化操作](#优化操作)

[4.优化策略](#优化策略)

[5.性能优化心得](#优化心得)

[参考资料](#参考资料)  

[知识详述](#详述)[](id:详述h)

[](id:编码之初)

<br/><br/><br/>
[](id:前言)
##前言
<br/>

#####&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一个一般事实:只有当发现“严重”的性能问题时，我们才会开始着手进行性能优化，此时虽然可以针对性的解决程序严重性能问题。但在继续优化过程中，面对无数细小的“不良”代码，却又力不从心。相比得到的些微性能改善，庞大的工作量不得不令人放弃。

######&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;但同样不得不承认的是，无数细小的不良代码所累加的性能问题是严重的。面对这样一个问题，也许最佳的解决办法便是从编码之初上着手进行。

######&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;传统的编码规范仅只是为了阅读规定了代码的编写格式，无数的性能优化博客则更多的是一种性能优化策略。一个应用的性能更多的是依靠程序员自身积累及习惯。

######&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本文目的是为了从编码初始硬性的对某些将会影响程序性能的操作进行规范，杜绝使用一定会引起性能问题的代码，以及给出更优的建议代码。

######&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;值得注意的是，为了保证新的编码规范不会引起开发者强烈的反感与束缚，规范中并未涉及过于复杂与细化的编码操作，更多的是一种在原来基础上更优的代码替换。同时避免出现泛泛而谈的理论点，而是具体到固定代码如何做。本文并未对操作进行优先级划分，明确的优先级划分一定程度会引起开发者的惰性心理。每一种优化都很重要，笔者已经做了一定排序，排在前面的是你应该先注意的。

<br/><br/><br/><br/><br/><br/><br/><br/>


### 1.编码之初

- #####对于布局内容的数量要求
		
		单个Activity显示的视图一般情况少于20，层数少于4
		对于Adapter控件，如ListView ，item的布局层数一般情况为2，不得超过3.
		

#####**Activity加载中，背景的加载极其耗时，对其进行优化效果明显且工作简单**

- #####将Acitivity 中的Window 的背景图设置为空。

		getWindow().setBackgroundDrawable(null); 
		android的默认背景不为空。

- ######将Activity的背景放到Activity的Theme中设置。同时避免fragment和activity背景重复设置。

		Theme设置属性
		<item name="android:windowBackground">src_image</item>

  [原理详解](#背景置于主题)

- ######采用硬件加速

		androidmanifest.xml中application添加 
		android:hardwareAccelerated="true"。
		需要注意的是：android 3.0以上才可以使用。
		
- #####考虑使用Webp代替传统png图片。对于某些使用JPEG即可实现的效果，尽量采用JPEG。

		png虽能提供无损的图片，但相对于JPEG过大。Webp是既保持png优点，又能减少图片大小的新型格式。
		但你需要提前知道的是：编译器并不能预览webp格式的图片，所以在预览布局时，并不能显示你的素材。
		不过手机上是可以正常显示的。
		我的建议：在类似于表情库这样量大，且不需要预览的素材使用webp格式。
		还有背景这样体积大，但不影响布局预览的素材。

<br/><br/><br/><br/><br/><br/>
[](id:禁止)
### 2.禁止（避免）操作

**核心：少的对象创建，意味着少的GC操作。  杜绝引起内存溢出、内存抖动的操作行为**





- #####禁止在单例模式中引用Activity的context。使用Application。

		如果在某个Activity中使用一下代码 就会造成该Activity一直被 Singleton 引用着，不能释放。
		Singleton instance = Singleton.getInstance(this); // 禁止操作
		
		是使用 getApplicationContext()  这样就能避免内存泄露。
		Singleton instance = Singleton.getInstance(getApplicationContext()); //建议操作
		


- #####禁止使用枚举

		枚举将造成大量的内存浪费


- #####禁止使用异步回调，

		异步回调被执行的时间不确定,很有可能发生在activity已经被销毁之后,
		这不仅仅很容易引起crash,还很容易发生内存泄露。


- #####禁止static引用资源耗费过多的实例

		例如：context  , Activity
		
		对于某些不得不出现static引用context的情况，在onDestroy()方法中，解除Activity与static的绑定关系,
		从而去除static对Activity的引用，使Context能够被回收；
		
		
- #####禁止内部的Getters/Setters

		对于类的成员我们需要提供Get和Set方法
		但在类内部，应该避免使用Get和Set方法
		
- #####禁止在非常复杂的布局上使用动画

- #####避免在循环（for、while、listView - getView方法、onDraw）里创建对象

- #####避免在onDraw里创建对象 

		对于onDraw中 Paint 我们可以这样优化
		
		private Paint paint = new Paint();
		
		public on Draw(){
			paint.setColor(mBorderColor);
		}
[原理详解](#禁止onDraw创建对象)	
		

- #####避免使用static成员对象

		static生命周期过长，对于需要传递的对象，使用(Intent)和(Handler)
- #####避免使用浮点数

		浮点数会比整型慢两倍

- #####避免Timer.schedule，对于延时操作，可用以下方式代替
		
		ScheduledExecutorService, 
		handler.postDelayed, 
		handler.postAtTime , 
		handler.sendMessageDelayed ,  
		View.postDelayed，      
		AlarmManager


- #####避免加载过大图片。压缩或者使用对象池后再使用


- #####慎用异常
		
		原因：创建一个异常时,需收集一个栈记录(stack track),用于描述异常是在何处创建的。
    构建这些此栈时需要为运行时栈做一份快照,这一部分开销很大。

		
- #####避免使用递归


- #####避免使用轮询

- #####避免长周期内部类、匿名内部类长时间持有外部类对象导致相关资源无法释放。如：Handler, Thread , AsyncTask
[返回目录](#目录)

[](id:优化操作)
### 2.优化操作
<br/>

  <最优替换/>
  
- #####当数据量在100以内时，使用ArrayMap代替HashMap

- #####为了避免自动装箱，当数量在1000以下时，使用如下容器
		
		a)SparseBoolMap <bool , obj>
		b)SparseIntMap <int , obj>
		c)SparseLongMap <long , obj>
		d)LongSparseMap <long ,obj>

- #####字符串拼接用StringBuilder或StringBuffer

		//这种string第一次初始化的情况下，下面得效率更高
		String str1 = "abc"+“def”+"hij";
		//非并发情况 ， StringBuilder效率更优
		StringBuilder str2 = str3 + str1 + "builder" ;
		//并发情况使用 StringBuffer
		StringBuffer str2 = str1 + "buffer" ;
  
- #####文件、网络IO缓存，使用有缓存机制的输入流

		BufferedInputStream替代InputStream
		BufferedReader替代Reader
		BufferedReader替代BufferedInputStream. 
 
 
- #####用两个平行的基本类型数组int[] int[]，代替一个对象Array(int , int)

		两个平行数组一定比一个对象数组的效率高。
		但是如果是建立一个供第三方调用的API接口，需要牺牲一定效率保证接口友好
 
- #####考虑使用Webp代替传统png图片。对于某些使用JPEG即可实现的效果，尽量采用JPEG。

		png虽能提供无损的图片，但相对于JPEG过大。Webp是既保持png优点，又能减少图片大小的新型格式。
- #####在使用线程池的情况中，除需要设置优先级的线程使⽤用new Thread创建外,其余线程创建使用new Runnable。

		因为⼦子类会有⾃自⼰己的属性创建需要更多开销。

- #####在使用Factory或类似Factory模式的情况。
		少用new关键字创建对象，使用new，构造函数链中得所有构造函数都会被自动调用。
		
		public static Credit createCredit(){
			return new Credit();
		}
		改写为：
		private static Credit BaseCredit = new Credit();
		public static Credit createCredit(){
			return (Credit)BaseCredit.clone();
		}
		你必须要注意的：clone是浅拷贝。
 
 <优化操作/>
 
 - #####尽量使用局部变量  
 
 	[原理详解](#禁止onDraw创建对象)	

 
 - #####for循环要求 

		禁止在for循环的第二个条件中调用任何方法，应这样做

		int size = array.length;
		for(int i = 0; i< size;i++)
		替代：
		for(int i =0;i < array.length;i++)
		
		在不需要使用下标的情况下，建议使用for_each循环
		
		
 - #####如果没有特殊需求，使用基本数据类型，而非对象类型。

		基本类似指：int , double , char等。
		
		
- #####静态方法代替 虚拟对象执行方法（虚拟对象执行方法new Object1().tool1();）

		如果不需要访问某对象的字段，将方法设置为静态，调用会加速15%到20%。
		
		
- #####对于使用超过两次的对象成员， 将成员缓存到本地。
		反复使用的变量，保存到本地成为临时变量活成员变量后进行操作。尤其是在循环中
		例：多次比较目标时间和当前时间差。		
		
- #####当new的对象并不是100%一定会被用到时，在使用时创建,有效减少不必要的对象生成

		Object ob = new Object();
		int value;
		if(i>0)
			value = ob.getVlaue();
		
		改写为：
		int value;
		
		if(i>0){
			Object ob = new Object();   //用到时加载
			value = ob.getVlaue();
		}

- #####不在使用的变量，手动置为null
		通常对于对象成员如此使用，局部变量不需要
		this.object = null；
		

 
 - #####非架构层的代码上，如果返回直接类型足够，那就不返回接口类型（？？？？）

		例：如果在一个类的内部两个方法传递，或两个实现类的传递对象
		返回HashMap足够，不要返回map
		相对于通过具体的引用调用函数，通过接口引用来调用函数会花费超过两倍的时间。
		


- #####常量用 static final修饰		
<缓存/> 

- #####消息缓存，从handler消息池中取预存的Message

		handler.sendMessage(handler.obtainMessage(0, object)); 
   
- #####尽量使用对象池机制

		对象池机制可以有效避免内存抖动提升性能
		优化：我们可以对对象进行预加载，有效提高程序首次运行速度
		警告：为避免内存泄露，需要保证所有对象和外部对象没有引用关系
  
- #####使用对象池时，在使用结束后，需要保证对象池中对象和外部对象没有引用关系
		
		通常，我们可以通过 objcet = null ; 来去掉对象的引用。
		


- #####禁止将View添加到没有清除机制的容器里

		如：WeakHashMap，没有清除机制，易引起内存溢出

<图片/>

- #####对于不同目的的图片需求（Bitmap），使用不同的图片解码格式

		itmap output = Bitmap.createBitmap(scaledSrcBmp.getWidth(),
		scaledSrcBmp.getHeight(),Config.ARGB_8888);
		
	- ARGB_8888		32Bit		(这是一种高质量的图片格式，电脑上普通采用的格式。它也是Android手机上一个BitMap的)
	
	- RGB_565			16Bit	(对于没有透明和半透明颜色的图片来说，该格式的图片能够达到比较的呈现效果，相对于ARGB_8888来说也能减少一半的内存开销。因此它是一个不错的选择。另外我们通过android.content.res.Resources来取得一个张图片时，它也是以该格式来构建BitMap的 
从 Android4.0 开始，该选项无效。即使设置为该值，系统任然会采用  ARGB_8888 来构造图片)
	
	- ARGB_4444		16Bit		(这种格式的图片，看起来质量太差，已经不推荐使用)
	
	- ALPHA_8		8Bit  		(此时图片只有alpha值，没有RGB值， )
	
		
			
		
- #####对于图片缩放，提供一下几种方式和其各自优缺点。**后期再补充**

		/* 
		*1.  Android自带缩放API ,使用方便，但需要一次性讲图片读入内存，对于过大图片容易引起内存溢出
		*/
		createScaleBitmap(inBmp , 64 , 128);
		/*
		*2.	 inSimpleSize可以等比例缩放图片，参数表示 1/n.同时避免把原图加载到内存中  
		*/
		mBitmapOptions.inSimpleSize = 4 ;  //原图 1/4 
		mBitmap = BitmapFactory.decodeFile(fileName,mBitmapOptions);

  

- #####Bitmap使用结束后，recycle（）释放内存

		Bitmap.recycle();



<布局/>

- #####慎用layout_weight属性，用相对布局替换线性布局亦可实现相同效果

- #####避免多个线性布局嵌套，使用相对布局减少层级

- #####对于TextView和ImageView组成的Layout，直接使用TextView替换

		<TextView
    		android:id="@+id/nameText"
    		android:layout_width="wrap_content"
    		android:layout_height="wrap_content"
    		android:text="暴打小女孩"
    		android:layout_marginBottom="center"
    		android:gravity="center"
  	 		android:drawableTop="@drawable/icon"/>  //将图片置于上方  
  	
  	 		
- #####默认不会显示的布局使用 viewstub 标签（但是并没有发现使用viewstub和GONE在效率上的区别，还是更倾向于使用GONE）

		<ViewStub
			android:id="@+id/network_error_layout"
			android:layout_width="match_parent"
			android:layout_height="match_parent"
			android:layout="@layout/network_error" />
			
		//非显示的转换ViewStub 获取
		View viewStub = findViewById(R.id.network_error_layout);
		viewStub.setVisibility(View.VISIBLE); // ViewStub被展开后的布局所替换
		networkErrorView = findViewById(R.id.network_error_layout); // 获取 展开后的布局    		
- #####对于两次以上相同的infalte操作，用成员变量代替局部变量，避免重复加载

		class test{
			private View view = null;
			public void getView(){
				view = findViewById(R.id.network_error_layout);
			}
			publi void showView(){
				view.setVisiblity(View.VIWIBLE);
			}
		}
		
- #####对于重复出现超过2-3次的子布局，用 include 实现复用。
		
		<include layout="@layout/foot.xml" />

- #####当<include>复用的布局中子View对所依赖的根节点要求不高时，使用 merge 作为根节点

		要求不高标准：非复杂结构布局，无Background,padding等属性，且子View数量较少

		<merge xmlns:android="http://schemas.android.com/apk/res/android"
			android:layout_width="match_parent"
			android:layout_height="match_parent" >
			<Button
				android:id="@+id/button"
				android:layout_width="match_parent"
				android:layout_height="@dimen/dp_40"
				android:layout_above="@+id/text"/>
			<TextView
				android:id="@+id/text"
				android:layout_width="match_parent"
				android:layout_height="@dimen/dp_40"
				android:layout_alignParentBottom="true"
				android:text="@string/app_name" />
		</merge>


[返回目录](#目录)


<br/><br/><br/>

[](id:优化策略)
###性能优化策略

- #####减少过渡绘制，可以极大提高动画效率
- #####使用简单的动画效果，如：位置移动，慎用改变内容的动画效率

		动画的绘制过程：创建DisplayList → 渲染DisplayList → 更新到屏幕。
		（DisplayList:DisplayList帮助完成把XML布局文件转换成GPU能识别并绘制的对象）
		
		不改变内容，DsiplayList不会重建，提高动画效率

- #####捆绑非及时的网络请求，统一执行。
- #####网络数据的预取：预先判断此次请求后，后续零散请求是否很有可能马上被触发，对此类数据进行预取。
- #####回退机制：对于轮询式的网络请求，服务器端判断此次请求和上次请求数据是否发生变化，负责不传输
[返回目录](#目录)


[](id:优化心得)
##性能优化心得



#####&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在Android系统 ，一个大多数人存在的误区：手机变卡 = 内存不足。所以才依靠第三方的软件不停的清理手机内存。依照这个惯性心理，我们在做性能优化过程中，优化的方向变成了尽量少的使用内存资源。

#####&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	其实这是一个不那么准确的误区。

#####&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;手机卡不卡和内存大小没有关系，直接影响手机流畅度的是CPU，只有当CPU超负荷运行时，手机才会变卡。（当然，大部分CPU超负荷运行的时候，内存也满载，这是引起误解的原因，但有时即使你清理了内存，手机依然卡）

#####&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;那么如何优化CPU呢？其实还是在内存上做，但重点不一样了，我们不能因为怕占内存而把所有数据存到本地存储（有点极端了，只是举一个例子），用一次取一次，这是极其耗时的。内存我们是一定要用的，数据存于内存，CPU读取快，应用运行便流畅。避免大量占用内存的原因不是怕内存满载，而是要避免 GC 。

#####&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;世上没有免费得午餐，对象创建后总是要回收的---GC。那么GC是由谁来做的呢？CPU。最重要的是，当进行GC时，其他所有线程都会被暂停，虽然系统已经尽力让GC的时间变短，但当大量的GC操作发生时，依然会影响到用户体验。

#####&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;所以，如何让CPU的使用率更高，不做无用功。如何更高效的利用内存，避免大量的GC，就是需要我们优化的地方了。

[返回目录](#目录)

[](id:参考资料)

<br/><br/>
##参考资料


[Android性能优化典范（一)](http://www.cnblogs.com/hanyonglu/p/4244035.html)

[Android性能优化典范（二)](http://hukai.me/android-performance-patterns-season-2/)

[Android性能优化典范（三)](http://hukai.me/android-performance-patterns-season-3/)

[Android App 性能优化实践](http://mp.weixin.qq.com/s?__biz=MzA3MDMyMjkzNg==&mid=210885796&idx=1&sn=7538666062c2b41e7d46e5d41d2a49d4&scene=5#rd)
[性能优化总纲——性能问题及性能调优方式](http://www.trinea.cn/android/performance/)

[性能优化第一篇——数据库性能优化](http://www.trinea.cn/android/database-performance/)

[性能优化第二篇——布局优化](http://www.trinea.cn/android/layout-performance/)

[性能优化第三篇——Java(Android)代码优化](http://www.trinea.cn/android/java-android-performance/)

[性能优化第四篇——移动网络优化](http://www.trinea.cn/android/mobile-performance-optimization/)

[性能优化实例](http://www.trinea.cn/android/android-performance-demo/)

<br/>

[返回目录](#目录)


<br/><br/>



[](id:详述)
##相关知识详述   

[](id:背景置于主题)

- #####为什么将背景设置在主题可以减少加载时间？
		回答这个问题，我们先要知道 activity的画面是如何绘制到屏幕上的？
		
		Resterization栅格化是绘制那些Button，Shape，Path，String，Bitmap等组件最基础的操作。它把那些组件拆分到不同的像素上进行显示。这是一个很费时的操作，GPU的引入就是为了加快栅格化的操作。
		
		CPU负责把UI组件计算成Polygons，Texture纹理，然后交给GPU进行栅格化渲染。
		
		然而每次从CPU转移到GPU是一件很麻烦的事情，所幸的是OpenGL ES可以把那些需要渲染的纹理Hold在GPU Memory里面，在下次需要渲染的时候直接进行操作。所以如果你更新了GPU所hold住的纹理内容，那么之前保存的状态就丢失了。
		
		在Android里面那些由主题所提供的资源，例如Bitmaps，Drawables都是一起打包到统一的Texture纹理当中，然后再传递到GPU里面，这意味着每次你需要使用这些资源的时候，都是直接从纹理里面进行获取渲染的。


[](id:禁止onDraw创建对象)

- #####为什么禁止onDraw创建对象？

		首先onDraw()方法是执行在UI线程的，在UI线程尽量避免做任何可能影响到性能的操作。虽然分配内存的操作并不需要花费太多系统资源，但是这并不意味着是免费无代价的。设备有一定的刷新频率，导致View的onDraw方法会被频繁的调用，如果onDraw方法效率低下，在频繁刷新累积的效应下，效率低的问题会被扩大，然后会对性能有严重的影响。
		
		如果在onDraw里面执行内存分配的操作，会容易导致内存抖动，GC频繁被触发，虽然GC后来被改进为执行在另外一个后台线程(GC操作在2.3以前是同步的，之后是并发)，可是频繁的GC的操作还是会影响到CPU，影响到电量的消耗。


- #####为什么要尽量使用局部变量？

		Android系统里面有一个Generational Heap Memory的模型，系统会根据内存中不同的内存数据类型分别执行不同的GC操作。例如，最近刚分配的对象会放在Young Generation区域，这个区域的对象通常都是会快速被创建并且很快被销毁回收的，
		同时这个区域的GC操作速度也是比Old Generation区域的GC操作速度更快的。

　　![image](http://images.cnitblog.com/blog/56846/201501/231407350162907.png)












[返回目录](#目录)


		
