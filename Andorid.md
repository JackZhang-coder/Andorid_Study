## Android基础



#### Activity

* Activity跳转

* Intent

* 注意事项：Manifest声明

* 四种启动模式
  * `standard`： 标准模式 单个栈 活动按顺序进栈出栈
  * `singleTop` ：栈顶若有当前活动 则不会创建
  * `singTask`：栈中若有当前活动，则会弹出栈中该活动上方的活动
  * `singleInstance`：每次创建活动时，会为该活动单独创建一个栈
  
* 生命周期：

   ![img](https://developer.android.google.cn/guide/components/images/activity_lifecycle.png)

  * 四种流程
    * 正常运行结束
    * 从`onPause`处返回：`Activity`不再处于前台 但可见（`Dialogy Activity` ）
    * 从`onStop`处正常返回：`Activity`不再处于前台 且不可见
    * `App process killed`：运行内存不够

* Acitivity之间传递数据

  * A->B

    * 传递基本类型

        ```java
      //发送
      Intent intent = new Intent(MainActivity.this, NewActivity.class);
      intent.putExtra(MAINACTIVITY_INTENT,"Main");
      startActivity(intent);
      
      //接收
      if(getIntent() != null){
          Intent intent = getIntent();
      	String name = intent.getStringExtra(MAINACTIVITY_INTENT);
      }
      ```

    * 传递`bundle`

      ```java
      //发送
      Intent intent = new Intent(MainActivity.this, NewActivity.class);
      Bundle bundle = new Bundle();
      bundle.putString(MAINACTIVITY_INTENT,"Main");
      intent.putExtra(MAINACTIVITY_INTENT,bundle);
      startActivity(intent);
      
      //接收
      if(getIntent() != null){
      	Intent intent = getIntent();
      	Bundle bundle = intent.getBundleExtra(MainActivity.MAINACTIVITY_INTENT);
      	String name = bundle.getString(MainActivity.MAINACTIVITY_INTENT);
      }
      ```

    * 传递序列化对象

      * 定义一个类实现序列化接口

        ```java
        public class User implements Serializable {
            public String title;
        }
        ```

        ```java
        //传递
        User user = new User();
        user.title = "Main";
        intent.putExtra(MAINACTIVITY_INTENT, user);
        startActivity(intent);
        
        //接收
        if(getIntent() != null){
        	Intent intent = getIntent();
            User user = (User) intent.getSerializableExtra(MainActivity.MAINACTIVITY_INTENT);
            assert user != null;
        }
        ```

        

  * B->A ：B页面销毁并返回至A页面

    * A页面打开B页面使用`startActivityForResult(Intent intent, int requestCode)`启动B

      ```java
      startActivityForResult(intent,111); //111 为requestCode
      ```


    * A重写`onActivityResult`：
    
      ```java
      @Override
      protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
      	super.onActivityResult(requestCode, resultCode, data);
      
      	if(requestCode == 111 && resultCode == 555){
      		Toast.makeText(this,"页面返回了",Toast.LENGTH_SHORT).show();
      	}
      }
      ```
    
    * B页面在结束前那通过调用`setResult`来返回数据给A
    
      ```java
      setResult(555);
      finish();
      ```


​      

#### Menu

* 3.0 菜单分水岭 （平板时代）

* 分类

  * `OptionMenu`选项菜单

    * 最多允许一级子菜单  一般在顶部标题栏右侧
    * 一个Activity只有一个
    * 使用步骤
      1. `onCreateOptionsMenu`创建 需返回`true`
      
         ```java
             
         public boolean onCreateOptionsMenu(Menu menu) {
            //将菜单资源加载到menu
            getMenuInflater().inflate(R.menu.optionmenu, menu);
            return true;
         }
         ```
      
      2. `onOptionsItemSelected`菜单选中方
      
         ```java
         // @Override
         public boolean onOptionsItemSelected(@NonNull MenuItem item) {
             //根据按下的菜单键执行相应方法
             switch (item.getItemId()){
         		...
             }
            return super.onOptionsItemSelected(item);
         }
         ```
      
         

  * `ContextMenu` 上下文菜单

    * 长按住某个`View`不放出现

    * 常规模式：以菜单栏样式出现

      1、在`onCreate`中使用`registerForContextMenu`对某个按钮进行注册

      ```java
      registerForContextMenu(findViewById(R.id.contextmenu));
      ```

      2、创建菜单`onCreateContextMenu`

      ```java
      public void onCreateContextMenu(ContextMenu menu, View v, ContextMenu.ContextMenuInfo menuInfo) {
      	//将资源加载到menu
          getMenuInflater().inflate(R.menu.context, menu);
      }
      ```

      

      3、设置菜单项的操作`onContextItemSelected`,需返回`true`

      ```java
      public boolean onContextItemSelected(@NonNull MenuItem item) {
          switch (item.getItemId()){
      		...
          }
          return super.onContextItemSelected(item);
      }
      ```

    * 上下文操作模式：出现位置在顶部,覆盖在应用操作栏上

      1、实现`ActionMode CallBack`

      ```java
      ActionMode.Callback cb = new ActionMode.Callback() {
          //创建 加载菜单资源
          @Override
          public boolean onCreateActionMode(ActionMode mode, Menu menu) {
              getMenuInflater().inflate(R.menu.context,menu);
              return true;
          }
      
          //在创建方法后进行调用
         @Override
         public boolean onPrepareActionMode(ActionMode mode, Menu menu) {
             	Log.e("TAG","准备");
              return false;
         }
      	
         //设置菜单选项操作
         @Override
         public boolean onActionItemClicked(ActionMode mode, MenuItem item) {
              switch (item.getItemId()) {
      			...
              }
             return true;
      	}
      	//菜单结束时调用
          @Override
          public void onDestroyActionMode(ActionMode mode) {
             Log.e("TAG","结束");
          }
      };
      ```
      
      
      
      2、在`Activity`的`onCreate`方法中监听`View`长按事件
      
      ```java
      //为按钮设置监听菜单启动事件
      findViewById(R.id.contextmenu).setOnLongClickListener(new View.OnLongClickListener() {
           @Override
           public boolean onLongClick(View v) {
               startActionMode(cb);
               return false;
      	}
      });
      ```
      
      

  * `PopupMenu` 弹出菜单

    1.  在`Activity`的`onCreate`方法中为启动按钮设置监听  
    2. 实例化`PopupMenu`对象
    3. 加载菜单资源：利用`MenuInflater`将Menu资源加载到`PopupMenu.getMenu()`所返回的对象中
    4. 为`PopupMenu`设置菜单选项操作
    5. 显示菜单		

       ```java
       //设置监听
       final Button popupBtn = findViewById(R.id.popmenu);
       popupBtn.setOnClickListener(new View.OnClickListener(){
        @Override
        public void onClick(View v) {
            //实例化popupMenu对象
        	PopupMenu popupMenu = new PopupMenu(MainActivity.this,popupBtn);
            //加载资源
            getMenuInflater().inflate(R.menu.popmenu,popupMenu.getMenu());
            //设置菜单操作
            popupMenu.setOnMenuItemClickListener(new PopupMenu.OnMenuItemClickListener() {
            	@Override
            	public boolean onMenuItemClick(MenuItem item) {
                	switch (item.getItemId()){
    					...
                    }
                   return false;
               }
         	});
            popupMenu.show();
        }
       });
       ```

    

* **总结**

  > 选项菜单和上下文菜单是通过覆盖父类方法实现的，而弹出式通过调用监听器实现



#### 对话框（Dialog）

* 弹出式对话框 AlertDialog

  1. 实例化一个`Builder`
  2. 设置对话框样式：标题 信息 按钮
  3. 显示对话框

* 自定义对话框

  1. 设计自定义对话框（样式xml文件以及创建一个继承系统`Dialog`的类）
  2. 设计`style`（去标题栏，背景） 因为系统`Dialog`有默认样式
  3. 将布局应用到对话框，并添加功能（按钮等）
  4. 实例化对话框（参数：环境上下文  创建的style）

* PopupWindow 弹窗式对话框

  1、准备弹窗所需要的视图对象并实例化对象

  ```java
  将资源文件转换为视图
  View v = LayoutInflater.from(this).inflate(R.layout.popup_layout,null);
  //实例化 设置宽和高 以及锚点
  PopupWindow popupwindow = new PopupWindow(v,600,160,true);
  ```

  2、设置背景以及能响应外部的点击事件

  ```java
  popupwindow.setBackgroundDrawable(new ColorDrawable(Color.TRANSPARENT));
  popupwindow.setOutsideTouchable(true);
  ```

  3、为弹窗中的文件添加事件

  ```java
  // 这里的v为弹窗视图
  v.findViewById(R.id.choose).setOnClickListener(new View.OnClickListener() {
  	@Override
      public void onClick(View v) {
      	Toast.makeText(MainActivity.this,"选择",Toast.LENGTH_SHORT).show();
      }
  });
  ```

  4、设置动画

  * 创建动画资源  即`Resource type`为`anim`

  ```xml
  //这里的translate代表动画类型为移动 
  <translate
  	android:fromXDelta="0"
  	android:toXDelta="0"
  	android:fromYDelta="1000"
  	android:toYDelta="0"
  	android:duration = "2000"/>
  ```

  * 在`style`中应用动画资源

  ```xml
  //windowEnterAnimation 窗口进入动画
  <style name="translate_anim">
  	<item name="android:windowEnterAnimation">@anim/translate</item>
  </style>
  ```

  * 加载动画`style`

  ```java
  popupwindow.setAnimationStyle(R.style.translate_anim);
  ```

  5、显示动画

  ```java
  //相对于view的位置显示 可设置偏移
  popupwindow.showAsDropDown(view);
  ```

* 数据适配器对话框

  * 显示列表形式的数据->适配器
  * 数组适配器`ArrayAdapter`

  ```java
  //参数1：环境 2：资源文件 指每项数据所呈现的样式 
  	//3：资源文件若为自定义样式 则表示数据在布局中对应位置文本的id 因为ArrayAdapter为文本适配器 4:数据
  ArrayAdapter adapter = new ArrayAdapter(this,R.layout.array_item_layout,R.id.item_text,items);
  ```

  * 绑定适配器并显示

  ```java
  AlertDialog.Builder builder = new AlertDialog.Builder(this)
              .setTitle("请选择")
              .setAdapter(adapter,null);
  builder.show();
  ```




#### Fragment

![fragment lifecycle states and their relation both the fragment's             lifecycle callbacks and the fragment's view lifecycle](https://developer.android.google.cn/images/guide/fragments/fragment-view-lifecycle.png)

* 与`Activity`区别

  * 一个`Activity`可运行多个`Fragment`
  * `Fragment`不能脱离`Activity`存在
  * `Activity`是屏幕主体，而`Fragment`是`Activity`的一个元素

* 创建:

    创建一个`ListFragment`类并继承自`Fragment`类，重写相关方法 ,在`onCreateView`方法中绑定布局视图

    ```java
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
    	View view = inflater.inflate(R.layout.fragment_list,container,false);
    	return view;
    }
    ```

* **静态加载**：通过`xml`文件直接绑定类

    创建一个`Activity`,并在该`Activty`对应的布局中绑定该`ListFragment`

  ```xml
  <fragment
  	...
      android:name="com.example.myfragment.ListFragment"/>
  ```
  
* **动态加载**：`Activity`布局中创建好容器，在代码中实现`Fragment`与容器的绑定

  ```xml
  <LinearLayout
  	android:layout_width="match_parent"
  	android:layout_height="match_parent"
  	android:orientation="horizontal">
  	
      //创建容器
  	<LinearLayout
      	android:orientation="horizontal"
          android:id="@+id/list_container"
          android:layout_width="200dp"
          android:layout_height="match_parent"/>
          
  </LinearLayout>
  ```

  ```java
  //创建fragment类
  ListFragment listfragment = new ListFragment();
  //添加fragment
  getSupportFragmentManager().
  	beginTransaction().
  	add(R.id.list_container,listfragment).commit();
  	//移除
  	remove(R.id.list_container,listfragment).commit();
  	//替换新的fragment
  	replace(R.id.list_container,new LisFragment()).commit();
  ```

* 传值

  * `Activity`->`Fragment`：

    * `setArguments`

        ```java
        //创建fragment时利用setArguments 传递值
        public static ListFragment newInstance(String title){
    		ListFragment fragment = new ListFragment();
    		Bundle bundle = new Bundle();
    		bundle.putString(BUNDLE_TITLE, title);
    		fragment.setArguments(bundle);
    		return fragment;
        }
        ```

    ```java
    //在fragment 的 onCreat方法中利用getArguments获取值
    public void onCreate(@Nullable Bundle savedInstanceState) {
    	super.onCreate(savedInstanceState);
    	if(getArguments() != null){
    		mTitle = getArguments().getString(BUNDLE_TITLE);
    	}
    }
    ```

  * `Fragment`->`Activity`

    1、使用回调 `Callback` 在`fragment`中设置接口

    ```java
    // 设置接口方法
    public void setOnTitleClickListener(onTitleClickListener onTitleClickListener){
    	mOnTitleClickListener = onTitleClickListener;
    }
    
    //定义变量
    private onTitleClickListener mOnTitleClickListener;
    
    //定义接口
    public interface onTitleClickListener{
    	void onClick(String title);
    }    
    ```

    2、在`MainActivity`调用创建的`fragment`类中的设置接口方法，并实现接口

    ```java
    //调用接口设置方法
    listFragment.setOnTitleClickListener(this);
    ...
    @Override
    public void onClick(String title) {
    	setTitle(title);
    }	
    ```

     

#### ViewPager

* 应用背景：原为`android.support.xxx`包中的组件, 后来迁移至`Androidx`下

* 使用：

  * 创建布局：

    ```xml
    <androidx.viewpager.widget.ViewPager
    	android:id="@+id/view_pager"
    	android:layout_width="match_parent"
    	android:layout_height="match_parent"/>
    ```

  * 准备数据源：例如是`ImageView`

    ```java
    mviews = new ArrayList<>();
    for (int index = 0; index < mLayoutIDs.length; ++index){
    	ImageView imageView = new ImageView(this);
    	mviews.add(imageView);
    }
    ```

  * 设置适配器:

    ```java
    mviewPager.setAdapter(mPagerAdapter);
    PagerAdapter mPagerAdapter = new PagerAdapter() {
    	@Override
        //返回pager的数量
    	public int getCount() {
    		return mLayoutIDs.length;
    	}
    	//pager是否可复用
    	@Override
        public boolean isViewFromObject(@NonNull View view, @NonNull Object object) {
        	return view == object;
        }
    
    	@NonNull
        @Override
        //获取每个子pager 并添加到容器中
        public Object instantiateItem(@NonNull ViewGroup container, int position) {
        	View child = mviews.get(position);
            container.addView(child);
            return child;
       }
    	//销毁pager时调用
    	@Override
        public void destroyItem(@NonNull ViewGroup container, int position, @NonNull Object object) {
        	container.removeView(mviews.get(position));
        }
    };
    ```

  * 可设置事件：例如滑动的时候

    ```java
    mviewPager.addOnPageChangeListener(new ViewPager.OnPageChangeListener() {
    	//滚动时
        @Override
        public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
    
    	}
    	
        //当前滑动至当前页面
    	@Override
    	public void onPageSelected(int position) {
        	...
        }
        //滚动状态变化
    	@Override
        public void onPageScrollStateChanged(int state) {
    
    	}
    });
    ```

  * 底部导航实现：与`fragment`配合

    * 创建布局文件（**注：**使用系统资源`id`）

      ```xml
      <?xml version="1.0" encoding="utf-8"?>
      //系统资源id
      <androidx.fragment.app.FragmentTabHost
          xmlns:android="http://schemas.android.com/apk/res/android"
          android:id="@android:id/tabhost"
          android:layout_width="match_parent"
          android:layout_height="match_parent"
          android:orientation="vertical">
      	
          
          <RelativeLayout
              android:layout_width="match_parent"
              android:layout_height="match_parent">
      
              <androidx.viewpager.widget.ViewPager
                  android:id="@+id/view_pager"
                  android:layout_width="match_parent"
                  android:layout_height="match_parent"
                  android:layout_above="@id/tab_divider"
                  />
              
      		//系统资源id tabcontent
              <FrameLayout
                  android:id="@android:id/tabcontent"
                  android:layout_width="match_parent"
                  android:layout_height="match_parent"
                  android:visibility="gone"
                  android:layout_above="@id/tab_divider"/>
              
              <View
                  android:id="@+id/tab_divider"
                  android:layout_width="match_parent"
                  android:layout_height="1dp"
                  android:background="#dfdfdf"
                  android:layout_above="@android:id/tabs"/>
              
              //底部导航栏 系统资源id
              <TabWidget
                  android:id="@android:id/tabs"
                  android:layout_width="match_parent"
                  android:layout_height="60dp"
                  android:layout_alignParentBottom = "true"
                  android:showDividers="none"/>
      
          </RelativeLayout>
      
      </androidx.fragment.app.FragmentTabHost>
      ```

    * 初始化总布局：

      ```java
      mTabHost = findViewById(android.R.id.tabhost);
      mTabHost.setup(this,getSupportFragmentManager(), android.R.id.tabcontent);
      ```

    * 数据与视图绑定：

      ```java
      //data -> view
      for (int index = 0; index < titleIDs.length; ++index){
          View view = getLayoutInflater().inflate(R.layout.main_tab_layout,null,false);
      
          ImageView icon = (ImageView) view.findViewById(R.id.main_tab_icon);
          TextView title = (TextView) view.findViewById(R.id.main_tab_txt);
          View tab = view.findViewById(R.id.tab_bg);
      
          icon.setImageResource(drawableIDs[index]);
          title.setText(getString(titleIDs[index]));
      
          tab.setBackgroundColor(getResources().getColor(R.color.white));
      
          mTabHost.addTab(
          	mTabHost.newTabSpec(getString(titleIDs[index]))
          	.setIndicator(view)
              //需重写相关方法
          	.setContent(this)
          );
      }
      //重写的方法
      @Override
      public View createTabContent(String tag) {
          View view = new View(this);
          view.setMinimumHeight(0);
          view.setMinimumWidth(0);
          return view;
      }
      ```

    * 设置适配器与导航栏

      ```java
      //三个tab做处理
      final Fragment[] fragments = new Fragment[]{
          TestFragment.newFragmentInstance("home"),
          TestFragment.newFragmentInstance("message"),
          TestFragment.newFragmentInstance("me")
      };
      
      final ViewPager viewPager = findViewById(R.id.view_pager);
      //绑定fragment
      viewPager.setAdapter(new FragmentStatePagerAdapter(getSupportFragmentManager()) {
          @NonNull
          @Override
          public Fragment getItem(int position) {
          	return fragments[position];
      	}
      
          @Override
          public int getCount() {
              return fragments.length;
          }
      });
      ```

    * 绑定监听器

      ```java
      viewPager.addOnPageChangeListener(new ViewPager.OnPageChangeListener() {
      	@Override
      	public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
      
      	}
      	//选中状态改变
      	@Override
      	public void onPageSelected(int position) {
      		if(mTabHost != null){
      			mTabHost.setCurrentTab(position);
      		}
      	}
      
      	@Override
      	public void onPageScrollStateChanged(int state) {
      
      	}
      });
      
      mTabHost.setOnTabChangedListener(new TabHost.OnTabChangeListener() {
          @Override
          public void onTabChanged(String tabId) {
              if(mTabHost != null){
                  int position = mTabHost.getCurrentTab();
                  viewPager.setCurrentItem(position);
              }
          }
      });
      ```



## Andorid网络操作



#### 基本请求：Get与Post
* 网络请求`get`与`post`

  * 注意事项：
    * 避免在主线程中IO操作，但`UI`更新要在主线程
    
    * 权限添加
    
      ```xml
      <uses-permission android:name="android.permission.INTERNET"/>
      ```
    
  
* Android9.0（`SDK 29`）对`http`的限制

  * 解决:  添加安全配置文件

    1、在`res`文件夹下创建`xml/network-security-config`文件

    2、增加`cleartextTrafficPermitted`属性

    3、`AndroidManifest.xml`中的`Application`声明
    
    ```xml
    android:networkSecurityConfig="@xml/network_security_config"
    ```
    
  
* `Json`数据解析

* 基本步骤

  * 构建`URL`并构造连接对象：

    ```java
    URL url = new URL(urlString);
    HttpURLConnection connection = (HttpURLConnection) url.openConnection();
    ```

  * 进行设置

    ```java
    connection.setConnectTimeout(30000); //超时
    connection.setRequestMethod("POST"); //请求方式
    connection.setRequestProperty("Content-Type", "application/json"); //希望拿到数据为 json格式
    connection.setRequestProperty("Charset","UTF-8");  // 希望返回的字符集为 UTF-8格式
    connection.setRequestProperty("Accept-Charset","UTF-8"); // 希望接收到的字符集为 UTF-8
    
    //对于post 额外设置
    //设置运行输入，输出：
    connection.setDoOutput(true);
    connection.setDoInput(true);
    
    connection.setUseCaches(false); //设置缓存
    
    //对提交的数据进行封装 转UTF-8码
    String data = "usename" + getEncodeValue("imooc")
                   + "&number=" + getEncodeValue("150088886666");
    
    OutputStream out = connection.getOutputStream();
    out.write(data.getBytes());
    out.flush();
    out.close();
    ```

  * 发起连接：

    ```java
    connection.connect(); 
    ```

  * 获取请求码以及请求结果

    ```java
    if(responseCode == HttpURLConnection.HTTP_OK) {
    	InputStream inputStream = connection.getInputStream();   //获取数据
    	result = streamToString(inputStream);  //将流转换为字符
    } 
    ```

  * 解析数据：`JSON`数据转换为普通对象
  
    * 根据原始数据创建`JSON`对象
  
      ```java
      JSONObject jsonObject = new JSONObject(result);
      ```
  
    * 读出数据
  
      ```java
      int status = jsonObject.getInt("status");
      JSONArray lessons = jsonObject.getJSONArray("data"); //数据
      ```
  
    * 构建转换后的对象
  
      ```java
      LessonResult lessonResult = new LessonResult();
      List<LessonResult.Lesson> lessonList = new ArrayList<>();
      ```
  
    * 对数据进行处理
  
      ```java
       for(int index = 0; index < lessons.length(); ++index){
           JSONObject lesson = (JSONObject) lessons.get(index);
           
           int id = lesson.getInt("id");
           int learner = lesson.getInt("learner");
           String name = lesson.getString("name");
           String smallPic = lesson.getString("picSmall");
           String bigPic = lesson.getString("picBig");
           String description = lesson.getString("description");
      
          LessonResult.Lesson lessonItem = new LessonResult.Lesson();
          lessonItem.setmID(id);
          lessonItem.setmName(name);
          lessonItem.setmSmallPictureUrl(smallPic);
          lessonItem.setmLearnerNumber(learner);
          lessonItem.setmBigPictureUrl(bigPic);
          lessonItem.setmDescription(description);
      
          lessonList.add(lessonItem);
      }
      ```



#### Handler

* 背景：
  * UI线程/主线程/ActivityThread
  * 线程不安全



