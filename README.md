# Android 代码规范  
--- 
### 0.Yoosee杂项  
* 字符串与翻译  

    1.Yoosee 新增字符串 与之前工作方式一样 放在英文字符串之下  
    2.不要私自添加字符串，全部需要在UI的管理文件里获取  
    3.在没有翻译之前其他语言文件里面不要再添加相同的key  
    4.对于后续添加的重复字符串，会在UI那里过滤，不出差错的话是不会增加重复字符串的  
    5.翻译之后统一同步到其他语言文件里，注意不要打乱顺序[(已有自动化工具)]()  
* 使用对象包装数据时只能对外提供获取成员变量的方法，而不能将其完全暴露在外面  
* 建议在自定义Entity中重写```toString```方法，打印出关键信息  
* 当一个类代码很多时(一般超过1000行)，成员变量的声明可以不再写在类顶，可写在新增的模块上方，同理接口的声明可不写在类前  

### 1.命名  

#### 1.1文件命名  
##### 1.1.1类文件  
定义应该使用[UpperCamelCase][2]来命名，例如任何类：  

	AndroidActivity, NetworkHelper, UserFragment, PerActivity  

扩展的Android框架组件的任何类应始终与该组件名称结尾。例如：  

	UserFragment, SignUpActivity, RateAppDialog, PushNotificationServer, NumberView  

##### 1.1.2资源文件  
![资源文件](http://7xp6ld.com1.z0.glb.clouddn.com/QQ%E5%9B%BE%E7%89%8720160831110339.png)  
**进展：**进度条  
**分频器：**listview的分割线  



### 2.代码指南

#### 2.1的Java语言的规则

##### 2.1.1永远不要忽视异常

避免以正确的方式不处理异常。例如：

	public void setUserId(String id) {
   		try {
       		mUserId = Integer.parseInt(id);
    	} catch (NumberFormatException e) { }
	}
这使得任何信息给开发者与用户之间，使其更难调试，也可以离开迷惑用户，如果出现错误。当捕获异常，我们也应该总是错误记录到控制台用于调试目的，如有必要警惕问题的用户。例如：

    public void setCount(String count) {
        try {
            count = Integer.parseInt(id);
        } catch (NumberFormatException e) {
            count = 0;
            Log.e(TAG, "There was an error parsing the count " + e);
            DialogFactory.showErrorMessage(R.string.error_message_parsing_count);
        }
    }
在这里，我们适当处理错误是：

* 示出一个消息给用户通知他们出现了一个错误
* 设置默认值，如果可能的变量
* 抛出一个合适的异常
##### 2.1.2永远赶不上通用异常

捕捉异常一般不应该做的：

	public void openCustomTab(Context context, Uri uri) {
    	Intent intent = buildIntent(context, uri);
    	try {
        	context.startActivity(intent);
    	} catch (Exception e) {
        	Log.e(TAG, "There was an error opening the custom tab " + e);
    	}
	}
为什么？

> 不要这样做。在几乎所有情况下，是不恰当的追赶通用的异常或可抛出（最好不要抛出，因为它包含错误例外）。这是非常危险的，因为它意味着你从来没有想到（包括像ClassCastException异常RuntimeExceptions）异常被夹在应用程序级的错误处理。它掩盖了失败处理代码的性能，这意味着如果有人添加了一个新的类型，我们在调用代码异常时，编译器不会帮你意识到你需要以不同的处理错误。在大多数情况下，你不应该处理不同类型的异常是一样的。 -Android代码风格指南

相反，赶上预期的异常，并采取相应措施：

	public void openCustomTab(Context context, Uri uri) {
    	Intent intent = buildIntent(context, uri);
    	try {
        	context.startActivity(intent);
    	} catch (ActivityNotFoundException e) {
        	Log.e(TAG, "There was an error opening the custom tab " + e);
    	}
	}
##### 2.1.3异常分组

例外情形执行相同的代码，它们应该在为了增加可读性和避免重复代码进行分组。例如，如果你可以这样做：

	public void openCustomTab(Context context, @Nullable Uri uri) {
    	Intent intent = buildIntent(context, uri);
    	try {
        	context.startActivity(intent);
    	} catch (ActivityNotFoundException e) {
        	Log.e(TAG, "There was an error opening the custom tab " + e);
    	} catch (NullPointerException e) {
        	Log.e(TAG, "There was an error opening the custom tab " + e);
    	} catch (SomeOtherException e) {
        	// Show some dialog
    	}
	}
你可以这样做：

	public void openCustomTab(Context context, @Nullable Uri uri) {
    	Intent intent = buildIntent(context, uri);
    	try {
        	context.startActivity(intent);
    	} catch (ActivityNotFoundException e | NullPointerException e) {
        	Log.e(TAG, "There was an error opening the custom tab " + e);
    	} catch (SomeOtherException e) {
        	// Show some dialog
    	}
	}
##### 2.1.4使用的try-catch在抛出异常

使用try-catch语句改善那里的异常发生的代码的可读性。这是因为在那里出现，从而更容易既调试或做出改变到误差的处理方式处理该错误。

##### 2.1.5避免使用终结

> 有没有保证何时终结将被调用，甚至可以说，它会在所有调用。在大多数情况下，你可以做你具有良好的异常处理终结的需要。如果你绝对需要它，定义一个```close（）```方法（或类似）和文档什么时候该方法需要被调用。见```InputStreamfor```一个例子。在这种情况下，它是适当的，但并不需要从终结打印一个短日志信息，只要它不希望洪水日志。 -Android代码风格指南  

**必须使用时再用**

##### 2.1.6导包时导入全名

当声明进口，使用全包声明。例如：

不要这样做：

	import android.support.v7.widget.*;
相反，这样做 ：笑脸：

	import android.support.v7.widget.RecyclerView;
##### 2.1.7不要导入未使用的包

有时，从类删除代码可能意味着一些进口不再需要。如果是这种情况，那么相应的进口应与该代码被删除。

#### 2.2 Java的样式规则

##### 2.2.1现场定义和命名

所有字段应该在文件的顶部声明，遵循以下规则：

* 私人，非静态字段名不应以m开始的。这是对的：

	userSignedIn，userNameText，的AcceptButton

不是这个：

	mUserSignedIn, mUserNameText, mAcceptButton
* 私有，静态字段名并不需要开始一个s。这是对的：

	someStaticField，userNameText

不是这个：

	sSomeStaticField, sUserNameText
* 所有其他领域也开始用小写字母。

	INT numOfChildren; 字符串用户名;

* 静态最终字段（称为常量）是ALL_CAPS_WITH_UNDERSCORES。

	私有静态最终诠释PAGE_COUNT = 0;

不应使用那些没有含义的变量名。例如，

	int e; //number of elements in the list
为什么不干脆给该领域的有意义的名字摆在首位，而不是留下评论！

	int numberOfElements;
这是好多了！

##### 2.2.2避免使用容器类型命名

从上面的领导，我们也应该对集合创建变量时，避免使用容器类型名称。例如，假设我们有一个包含用户id列表一个```ArrayList```：

做：

	List<String> userIds = new ArrayList<>();
别：

	List<String> userIdList = new ArrayList<>();
如果当容器名称在未来改变，这些命名往往被人遗忘 - 而就像视图命名，它不是完全必要的。容器本身的正确的命名应为它是什么的足够信息。

##### 2.2.3避免类似的命名

命名变量，方法和/或类具有类似名称可以把它混淆了其他开发者在阅读你的代码。例如：

	hasUserSelectedSingleProfilePreviously

	hasUserSelectedSignedProfilePreviously
这些区别的区别在乍看之下可能很难理解什么是什么。在一个更清晰的方式命名这些可以使开发人员更容易导航领域在你的代码。

##### 2.2.4多个系列的命名

当Android Studio中自动生成代码的我们，很容易留下东西，因为他们 - 即使它产生可怕的命名参数！例如，这是不是很漂亮：

	public void doSomething(String s1, String s2, String s3)
很难了解这些参数离不开阅读代码。代替：

	public void doSomething(String userName, String userEmail, String userId)
这使得它更容易理解！现在，我们将能够读取的代码有更清晰的了解参数如下🙂

##### 2.2.5 Pronouncable名

在命名字段，方法，他们应该类：

* 是可读的：高效的命名意味着我们就可以看名字并立即理解它，在试图破译什么名字意味着减少认知负荷。

* 要朗读：名称是朗读避免了在你试图发音一个严重命名变量名尴尬的对话。

* 可搜索的：没有什么比试图寻找一种方法或变量在一个类来实现它被拼错了或严重命名更糟。如果我们试图找到搜索用户，那么对于“搜索”搜索的方法应该调出该方法的结果。

* 不要使用[匈牙利表示法][1]：匈牙利表示法违背上述提出的三点，所以绝对不能用！

##### 2.2.6款待缩写为词

即便是专有名词，也应该遵循变量及方法的命名规则例如：

	正确：parseHtml	错误：parseHTML  
	正确：generateXmlFile	错误：generateXMLFile
##### 2.2.7避免理由的变量声明

变量的任何声明不应该使用对齐的任何特殊形式，例如：

这可以：

	private int userId = 8;
	private int count = 0;
	private String username = "hitherejoe";
避免这样做：

	private String username = "hitherejoe";
	private int userId      = 8;
	private int count       = 0;
这将使文字难以阅读。

##### 2.2.8使用空格缩进

缩进可按自己习惯使用Tab或者Space,4空间缩进应使用：

	if (userSignedIn) {
    	count = 1;
	}
而对于跨行代码，8个空格，应使用：

	String userAboutText =
        "This is some text about the user and it is pretty long, can you see!"
##### 2.2.9 if语句

##### 2.2.9.1使用标准括号风格

括号应始终在同一行之前的代码中使用。例如，避免这样做：

	class SomeClass
	{
    	private void someFunction()
    	{
        	if (isSomething)
        	{

        	}
        	else if (!isSomethingElse)
        	{
	
        	}
       		else
        	{

        	}
    	}
	}
取而代之，这样做：

	class SomeClass {
    	private void someFunction() {
        	if (isSomething) {

        	} else if (!isSomethingElse) {
	
        	} else {
	
        	}
    	}
	}
不仅是不是真的有必要的空间，多余的线，但它使块更容易阅读代码时要遵循的。

###### 2.2.9.2如果内嵌子句

有时是有意义的使用单行if语句。例如：

	if (user == null) return false;
然而，它仅适用于简单的操作。像这样的事情会更适合用大括号：

	if (user == null) throw new IllegalArgumentExeption("Oops, user object is required.");
###### 2.2.9.3嵌套如果条件

在可能的情况，如果条件应合并，以避免过分复杂的嵌套。例如：

做：

	if (userSignedIn && userId != null) {
	
	}
尽量避免：

	if (userSignedIn) {
    	if (userId != null) {
	
    	}
	}
这使报表更易于阅读，并从嵌套子句中的不必要的额外行。

###### 2.2.9.4三元运算

在适当情况下，三元运算符可以用来简化操作。

例如，这是很容易阅读：

	userStatusImage = signedIn ? R.drawable.ic_tick : R.drawable.ic_cross;
后者占用更多行：

	if (signedIn) {
    	userStatusImage = R.drawable.ic_tick;
	} else {
    	userStatusImage = R.drawable.ic_cross;
	}
**注意**：有一些时候不应使用三元运算符。如果if从句逻辑是复杂或则应该使用大量字符的标准括号的风格。

##### 2.2.10注解

###### 2.2.10.1注解惯例（没有使用注解习惯的可忽略，暂不要求）

从Android代码风格指南措施：

**@覆盖：** @Override批注必须使用每当一个方法覆盖从超一流的声明或执行。例如，如果您使用@inheritdocs Javadoc标记，并从一个类（不是接口）获得，还必须标注该方法@Overrides父类的方法。

**@SuppressWarnings：**该@SuppressWarnings注释应该只情况下它是不可能消除的警告使用。如果警告通过这个“不可能消除”检验，@SuppressWarnings注释必须使用，以便确保所有警告反映在代码的实际问题。

有关注释指引的更多信息可以在这里找到。

注释应始终尽可能使用。例如，使用@Nullable注释应该在一个字段可以预期为空的情况下使用。例如：

	@Nullable TextView userNameText;

	private void getName(@Nullable String name) { }
###### 2.2.10.2注释风格

所应用的方法或类注解应该总是在声明中定义，每行只有一个：

	@Annotation
	@AnotherAnnotation
	public class SomeClass {
	
 	@SomeAnotation
  	public String getMeAString() {
	
  	}

	}
当使用字段中的注释，你应该确保注释保持在同一条线上，例如：

	@Bind(R.id.layout_coordinator) CoordinatorLayout coordinatorLayout;


	@Inject MainPresenter mainPresenter;
我们这样做是因为它使语句更易于阅读。例如，语句“@注入SomeComponent mSomeName”读为注入此组件使用这个名字。“

##### 2.2.11限制变量的作用域

局部变量的范围应保持在最低限度（有效的Java档案29）。通过这样做，你增加你的代码的可读性和可维护性，减少出错的可能性。每个变量都应该在封闭变量的所有用途最里面块声明。

> 局部变量应当在其第一次使用点声明。几乎每一个局部变量声明应包含一个初始化。如果你还没有足够的信息来合理地初始化变量，你应该直到你推迟的声明。 - Android代码风格指南

##### 2.2.12未使用的元素

所有未使用的**字段**，**进口**，**方法**和**类**应该从代码库被删除，除非后面有什么具体的理由保持它。

##### 2.2.13分类import语句  
> import一般编译器会自动完成这些规则，无需太多关注

由于我们使用Android工作室，所以import应始终自动排序。然而，在这情况下，他们可能没有，那么他们应该被命令如下：

1. Android的import
2. 从第三方import
3. Java和使用javax import
4. 从目前的import项目  

**注意：**

* import应在每个组内按字母顺序排列，用大写字母排在小写字母（如ž前）
* 应该有每个主要的分组（android, com, JUnit, net, org, java, javax）之间的空行  

##### 2.2.14记录

日志记录应用于记录在开发过程中，可能是有用的有用的错误消息和/或其它信息。
日志规则参见[绩效考核]()

在发布版本所有详细和调试日志必须禁用。如果认为有必要信息，警告和错误日志只应保持启用  
**重要信息可以保留，密码不要打印，大量日志打印必须删除**


##### 2.2.15字段排序

在类文件的顶部声明的任何字段应该按以下顺序进行排序：

1. 枚举
2. 常量
3. Dagger Injected注入
4. Butterknife视图绑定
5. 私人全局变量
6. 公共全局变量  

例如：

	public static enum {
    	ENUM_ONE, ENUM_TWO
	}

	public static final String KEY_NAME = "KEY_NAME";
	public static final int COUNT_USER = 0;

	@Inject SomeAdapter someAdapter;

	@BindView(R.id.text_name) TextView nameText;
	@BindView(R.id.image_photo) ImageView photoImage;

	private int userCount;
	private String errorMessage;

	public int someCount;
	public String someString;
使用该排序公约有助于保持字段声明分组，从而增加双方的定位和所述字段的可读性。

##### 2.2.16类成员排序

为了提高代码的可读性，它来组织类成员以逻辑方式是很重要的。以下顺序应该被用来实现这一点：

1. 常量
2. 字段
3. 构造函数
4. 覆盖方法和回调（公共或私营）
5. 公共方法
6. 私有方法
7. 内部类或接口  

例如：

	public class MainActivity extends Activity {

    	private int count;

    	public static newInstance() { }

    	@Override
    	public void onCreate() { }

    	public setUsername() { }

    	private void setupUsername() { }

    	static class AnInnerClass { }

    	interface SomeInterface { }

	}
在Android框架类中使用任何生命周期的方法应该在相应的生命周期排列。例如：  
> Android Studio有插件帮忙完成此项工作，Eclipse后续编码尽量按周期排序

	public class MainActivity extends Activity {

    	// Field and constructors

    	@Override
    	public void onCreate() { }

    	@Override
    	public void onStart() { }

    	@Override
    	public void onResume() { }

    	@Override
    	public void onPause() { }

    	@Override
    	public void onStop() { }

    	@Override
    	public void onRestart() { }

    	@Override
    	public void onDestroy() { }

    	// public methods, private methods, inner classes and interfaces

	}
##### 2.2.17方法参数排序

当定义方法，参数要责令以下约定：

	public Post loadPost(Context context, int postId);


	public void loadPost(Context context, int postId, Callback callback);
**上下文**参数总是第一个，**回调参数**总是最后一个。

##### 2.2.18字符串常量，命名和值

当使用字符串常量，它们应该被声明为final的静态

##### 2.2.19枚举

其中，实际需要，才应使用枚举。如果另一个方法是可行的，那么这应该是接近实现的首选方式。例如：

代替这样的：

	public enum SomeEnum {
    	ONE, TWO, THREE
	}
做这个：

	private static final int VALUE_ONE = 1;
	private static final int VALUE_TWO = 2;
	private static final int VALUE_THREE = 3;
##### 2.2.20参数在Fragment和Activity

当我们通过使用意图或捆绑数据，值必须使用约定的按键定义如下：

**Activity**

将数据传递到一个活动必须在使用到一个KEY一个参考来完成，如定义如下：

	private static final String KEY_NAME = "com.your.package.name.to.activity.KEY_NAME";
**Fragment**

将数据传递到一个片段必须使用参考到EXTRA来完成，如定义如下：

	private static final String EXTRA_NAME = "EXTRA_NAME";
当创建涉及将数据传递片段或活动的新情况下，我们应该提供一个静态方法来获取新的实例，传递数据的方法参数。例如：

**Activity**

	public static Intent getStartIntent(Context context, Post post) {
    	Intent intent = new Intent(context, CurrentActivity.class);
    	intent.putParcelableExtra(EXTRA_POST, post);
    	return intent;
	}
**Fragment**

	public static PostFragment newInstance(Post post) {
    	PostFragment fragment = new PostFragment();
    	Bundle args = new Bundle();
    	args.putParcelable(ARGUMENT_POST, post);
    	fragment.setArguments(args)
    	return fragment;
	}
##### 2.2.21行长度限制

代码不超过100个字符，这使得代码更易读。有时为了达到这个目标，我们可能需要：

* 提取数据到一个局部变量
* 提取逻辑到外部的方法
* 行包代码分开的代码多行单行  
**注意**：对于代码中的注释和import语句不受100个字符的限制。

###### 2.2.21.1换行技巧

当涉及到换行，有一些情况下，我们应该在我们格式化代码的方式是一致的。
**换行**

当我们需要换行：

	int count = countOne + countTwo - countThree + countFour * countFive - countSix
        	+ countOnANewLineBecauseItsTooLong;
如果需要，你可以随时突破后=标志：

	int count =
        	countOne + countTwo - countThree + countFour * countFive + countSix;
**方法链接**

当涉及到链接的方法，每个方法调用应该是在新的一行。

不要这样做：

	Picasso.with(context).load("someUrl").into(imageView);
相反，这样做：

	Picasso.with(context)
        	.load("someUrl")
        	.into(imageView);
**龙参数**

在这种方法中包含长参数的情况下，我们应该换行符在适当情况下。对于声明的方法时的例子，我们应该适合参数的最后一个逗号后换行：

	private void someMethod(Context context, String someLongStringName, String text,
                            long thisIsALong, String anotherString) {               
	}             
并调用方法时，我们应该每个参数的逗号后换行：

	someMethod(context,
        	"thisIsSomeLongTextItsQuiteLongIsntIt",
        	"someText",
        	01223892365463456,
        	"thisIsSomeLongTextItsQuiteLongIsntIt");
##### 2.2.22间距的方法

有只需要在一个类中的方法之间的单行空行，例如：

做这个：

	public String getUserName() {
    	// Code
	}

	public void setUserName(String name) {
    	// Code
	}

	public boolean isUserSignedIn() {
    	// Code
	}
不是这个：

	public String getUserName() {
    	// Code
	}


	public void setUserName(String name) {
    	// Code
	}


	public boolean isUserSignedIn() {
    	// Code
	}
##### 2.2.23评论

###### 2.2.23.1行内注释

在必要时，内部注释应被用来提供一个有意义的描述向读者什么特定的代码片一样。它们只应在的情况下使用，其中该代码可能是复杂的理解。然而，在大多数情况下，码应写入的方式，很容易地理解🙂

**注：**代码注释不必，但应尽量，坚持到100字符规则。

###### 2.2.23.2的javadoc样式注释

虽然方法名称通常应足以通信的方法的功能，它有时可以帮助提供的JavaDoc风格的注释。这有助于读者容易理解的方法的功能，以及正在传递到方法的任何参数的目的。

	/**
 	 * Authenticates the user against the API given a User id.
 	 * If successful, this returns a success result
 	 *
 	 * @param userId The user id of the user that is to be authenticated.
 	 */
###### 2.2.23.3类注释

当创建类的意见，他们应该是有意义的，描述性的，使用链接必要。例如：

	/**
  	  * RecyclerView adapter to display a list of {@link Post}.
  	  * Currently used with {@link PostRecycler} to show the list of Post items.
  	  */
不要让笔者评论，这些都是没有用的，并提供没有真正的有意义的信息时，有许多人，是工作的类。

	/**
  	* Created By Joe 18/06/2016
  	*/  
**注意**：代码双星注释可以由IDE完成，对外提供的方法需使用此类注释，单行或者内部方法使用```//```
##### 2.2.24代码分节

###### 2.2.24.1 Java代码

如果创建的代码'节'，这应该使用下面的方法，这样做：

	public void method() { }

	public void someOtherMethod() { }

	/********* Mvp Method Implementations  ********/

	public void anotherMethod() { }

	/********* Helper Methods  ********/

	public void someMethod() { }
不喜欢这样的：

	public void method() { }

	public void someOtherMethod() { }

	// Mvp Method Implementations

	public void anotherMethod() { }
这使得分节的方法更容易地设在一类。  

**注意**:这个在Mediaplayer类中有实例写法

###### 2.2.24.2字符串文件

在string.xml文件内定义的字符串资源应该是功能部分，例如：

	// User Profile Activity
	<string name="button_save">Save</string>
	<string name="button_cancel">Cancel</string>

	// Settings Activity
	<string name="message_instructions">...</string>
这不仅有助于保持字符串文件整齐，但它使得当他们需要改变更容易找到字符串。

###### 2.2.24.3 RxJava链接

当链接接收业务，各操作符应该是一个新行,例如：

	return dataManager.getPost()
            	.concatMap(new Func1<Post, Observable<? extends Post>>() {
                	@Override
                 	public Observable<? extends Post> call(Post post) {
                     	return mRetrofitService.getPost(post.id);
                 	}
            	})
            	.retry(new Func2<Integer, Throwable, Boolean>() {
                 	@Override
                 	public Boolean call(Integer numRetries, Throwable throwable) {
                     	return throwable instanceof RetrofitError;
                 	}
            	});
这使得更容易理解的操作的呼叫的接收链中的流动。

###### 2.2.25Butterknife

###### 2.2.25.1事件侦听器（暂时不使用此注解，Android Studio新项目可使用）

如果可能的话，使用Butterknife监听绑定。例如，当一个click事件，而不是做这个听：

	mSubmitButton.setOnClickListener(new View.OnClickListener() {
    	public void onClick(View v) {
        	// Some code here...
    	}
  	};
做这个：

	@OnClick(R.id.button_submit)
	public void onSubmitButtonClick() { }
#### 2.3 XML样式规则

##### 2.3.1使用自 -closing标签

当在一个XML布局视图没有任何孩子的意见，应使用自结束标记。

做：

	<ImageView
    	android:id="@+id/image_user"
    	android:layout_width="90dp"
    	android:layout_height="90dp" />
别：

	<ImageView
    	android:id="@+id/image_user"
    	android:layout_width="90dp"
    	android:layout_height="90dp">
	</ImageView>
##### 2.3.2资源命名

所有的资源名称和ID应该使用小写和下划线，例如这样写：

	tx_username, activity_main, frg_user, error_message_network_connection
造成这种情况的主要原因是一致，也可以更容易地搜索的布局文件中的观点，当涉及到改变该文件的内容。

###### 2.3.2.1 ID命名

所有的ID应该使用他们已经被宣布为元素的名称作为前缀。

这里使用统一缩写iv_,tx_,layout_,rl,ll,frg_,btn_
例如：

	<TextView
    	android:id="@+id/tx_username"
    	android:layout_width="wrap_content"
    	android:layout_height="wrap_content" />
意见指出，通常只需要一个每次布局，如工具栏，可以简单地给它的视图类型的ID。例如toolbar。

###### 2.3.2.2字符串（字符串还不一定全部由UI编写）

所有的字符串名称应该与他们正在页面功能描述前缀开头。

需要注意的字符串资源的两个重要的事情：

* 字符串资源不应该在整个屏幕上重复使用。这可能会导致问题，当涉及到改变一个字符串为特定的屏幕。通过让每个屏幕使用一个字符串节省未来的并发症。

* 字符串资源应始终在字符串文件中定义，并在布局或类文件从来没有硬编码。

###### 2.3.2.3风格和主题

当定义两种风格和主题，就应该使用UpperCamelCase命名。例如：

	AppTheme.DarkBackground.NoActionBar
	AppTheme.LightBackground.TransparentStatusBar

	ProfileButtonStyle
	TitleTextStyle
##### 2.3.3属性排序

订购排序不仅外观整洁，但它可以帮助寻找布局文件中的属性时，使其更快。作为基本规则，

1. 视图ID
2. 样式
3. 布局的宽度和布局高度
4. 其他layout_属性，按字母顺序排序
5. 其余的属性，按字母顺序排序  

例如：

	<Button
    	android:id="@id/button_accept"
    	style="@style/ButtonStyle"
    	android:layout_width="wrap_content"
    	android:layout_height="wrap_content"
    	android:layout_alignParentBottom="true"
    	android:layout_alignParentStart="true"
    	android:padding="16dp"
    	android:text="@string/button_skip_sign_in"
    	android:textColor="@color/bluish_gray" />
**注：**此格式可以通过在Android Studio中的格式功能进行 -

	ctrl + shift + L

这样做很容易通过XML属性，当谈到在更改布局文件进行浏览。

#### 2.4测试样式规则

##### 2.4.1单元测试  

**--------------以下内容暂时忽略-------------**

任何单元测试类应写入匹配测试的目标类，其次是试验的后缀名。例如：

类	测试类
DataManager的	DataManagerTest
UserProfilePresenter	UserProfilePresenterTest
PreferencesHelper	PreferencesHelperTest
所有的测试方法应与被注解@Test注释的方法应使用以下模板命名：

	@Test
	public void methodNamePreconditionExpectedResult() { }
因此，例如，如果我们要检查注册（）方法无效的电子邮件地址失败，测试将如下所示：

	@Test
	public void signUpWithInvalidEmailFails() { }
测试应着重测试只是什么方法名称赋予，如果在你的测试方法进行测试附带条件那么这应该被移动到它自己的单独测试。

如果我们正在测试类包含许多不同的方法，那么测试应该在多个测试类分裂 - 这有助于保持测试更易于维护，更容易找到。例如，一个DatabaseHelper类可能需要被分成多个测试类如：

	DatabaseHelperUserTest
	DatabaseHelperPostsTest
	DatabaseHelperDraftsTest
##### 2.4.2测试的Espresso

每个咖啡测试类一般针对一个活动，所以赋予它的名称应与该活动的目标，再接着测试。例如：

类	测试类
主要活动	MainActivityTest
ProfileActivity	ProfileActivityTest
DraftsActivity	DraftsActivityTest
当使用咖啡API，方法应该在新的生产线被链接以使报表更具可读性，例如：

	onView(withId(R.id.text_title))
        	.perform(scrollTo())
        	.check(matches(isDisplayed()))
这种风格不仅链接调用帮助我们保持每行少于100个字符，但它也可以很容易地读取发生在java测试事件链。

### 3.Gradle

#### 3.1依赖

##### 3.1.1版本

在适用的情况，即在多个依赖性共享版本应该被定义为依赖范围内的变量。例如：

	final SUPPORT_LIBRARY_VERSION = '23.4.0'

	compile "com.android.support:support-v4:$SUPPORT_LIBRARY_VERSION"
	compile "com.android.support:recyclerview-v7:$SUPPORT_LIBRARY_VERSION"
	compile "com.android.support:support-annotations:$SUPPORT_LIBRARY_VERSION"
	compile "com.android.support:design:$SUPPORT_LIBRARY_VERSION"
	compile "com.android.support:percent:$SUPPORT_LIBRARY_VERSION"
	compile "com.android.support:customtabs:$SUPPORT_LIBRARY_VERSION"
这使得因为我们只需要一次更改版本号为多个依赖很容易在将来更新的依赖。

##### 3.1.2分组

> 在适用时，相关性应该由包名进行分组，与该组之间在空间中。例如：

	compile "com.android.support:percent:$SUPPORT_LIBRARY_VERSION"
	compile "com.android.support:customtabs:$SUPPORT_LIBRARY_VERSION"

	compile 'io.reactivex:rxandroid:1.2.0'
	compile 'io.reactivex:rxjava:1.1.5'

	compile 'com.jakewharton:butterknife:7.0.1'
	compile 'com.jakewharton.timber:timber:4.1.2'

	compile 'com.github.bumptech.glide:glide:3.7.0'
	compile，testCompile以及androidTestCompile   
> Grandle添加依赖时备注功能，依赖关系也应被分组到它们的相应部分。例如：

	// App Dependencies
	compile "com.android.support:support-v4:$SUPPORT_LIBRARY_VERSION"
	compile "com.android.support:recyclerview-v7:$SUPPORT_LIBRARY_VERSION"

	// Instrumentation test dependencies
	androidTestCompile "com.android.support:support-annotations:$SUPPORT_LIBRARY_VERSION"

	// Unit tests dependencies
	testCompile 'org.robolectric:robolectric:3.0'
这两种方法都可以很容易地找到特定的相关要求时，因为它使依赖声明既干净整洁 ：raised_hands：

##### 3.1.3独立依赖

其中，相关性仅分别用于应用程序或测试的目的，请务必使用它们只编译```compile，testCompile```或者```androidTestCompile```。例如，如果只需要单元测试的robolectric依赖，就应该使用被添加：

	testCompile 'org.robolectric:robolectric:3.0'  

----
[1]:http://baike.baidu.com/link?url=o1siDxcmodTwabRQ55DhZb5MMZ7Q3_iRCQpH_4_E_sp-FqrxSW18onfUt2UrgQFmhzEyk-KnQYfZpGT94UCwvK  
[2]:http://baike.baidu.com/view/1165629.htm?fromtitle=%E9%A9%BC%E5%B3%B0%E5%91%BD%E5%90%8D%E6%B3%95&fromid=7560610&type=syn
