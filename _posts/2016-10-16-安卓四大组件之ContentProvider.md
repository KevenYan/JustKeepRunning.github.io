---
layout: post
title: 安卓四大组件之ContentProvider
tags:
- Android
categories: Android四大组件
description: ContentProvider 在android中的作用是对外共享数据，也就是说你可以通过ContentProvider把应用中的数据共享给其他应用访问，其他应用可以通过ContentProvider 对你应用中的数据进行添删改查。
---

#*ContentProvider*
##1.为什么需要内容提供者
	[1]在Android下如何创建一个数据库? 定义一个类继承SqliteOpenHelper
    [2]sqlite3 命令行工具 打开一个数据库  
    [3]chcp 936把当前dos窗体编码方式变成gbk，如果变成utf-8写chcp 65001
    [4]必须拿到哪个类 才能操作数据库 slitedatabase
    [5]chmod  可以改变文件权限  chmod 777 Account.db
    [6]内容提供者可以把私有的数据库内容暴露出来  

	关于数据共享，以前我们学习过文件操作模式，知道通过指定文件的操作模式为Context.MODE_WORLD_READABLE 或Context.MODE_WORLD_WRITEABLE同样也可以对外共享数据。那么，这里为何要使用ContentProvider 对外共享数据呢？是这样的，如果采用文件操作模式对外共享数据，数据的访问方式会因数据存储的方式而不同，导致数据的访问方式无法统一，如：采用xml文件对外共享数据，需要进行xml解析才能读取数据；采用sharedpreferences共享数据，需要使用sharedpreferences API读取数据。
	使用ContentProvider对外共享数据的好处是统一了数据的访问方式。
##2.内容提供者原理
![](http://i.imgur.com/h01ZEp4.png)
##3.实现内容提供者步骤
    [1]定义内容提供者 定义一个类继承contentProvider
    [2]在清单文件里面配置一下  
	为了能让其他应用找到该ContentProvider ， ContentProvider 采用了authorities（主机名/域名）对它进行唯一标识，你可以把 ContentProvider看作是一个网站（想想，网站也是提供数据者），authorities 就是他的域名：
	  <provider
    android:authorities="com.keven.provider"    					    			android:name="com.example.keven.contentprovider.AccountProvider">
       </provider>
    [3] 定义一个urimatcher 

       //定义一个Uri路径适配器
       private static final UriMatcher sURIMatcher = new UriMatcher(UriMatcher.NO_MATCH);

    [4]写一个静态代码块 添加匹配规则 
       static {
        sURIMatcher.addURI("com.keven.provider", "query", QUERYSUCESS);
               }

    [5]按照我们添加的匹配规则 暴露想暴露的方法 

		int code = sURIMatcher.match(uri);
			if (code == QUERYSUCESS) {
            SQLiteDatabase database = mOpenHelper.getReadableDatabase();
            Cursor cursor = database.query("info", projection, selection, selectionArgs, null, null, sortOrder);
            getContext().getContentResolver().notifyChange(uri, null);
            return cursor;

    [6]如果你发现如下log日志 就说明内容提供者写的没有问题
    	>  09-11 02:02:31.142: I/ActivityThread(16636): **Pub **com.itheima.provider: com.itheima.db.AccountProvider
    [7]只要是通过内容提供者暴露出来的数据 其他应用访问的方式都是一样的 就是通过内容解析者

       Uri uri = Uri.parse("content://com.keven.provider/query");
  	   Cursor cursor =* getContentResolver()*.query(uri, new String[]{"name", "money"}, null, null, null);



##4 内容观察者
1. 内容观察不是四大组件  他不需要在清单文件里配置
![](http://i.imgur.com/A0BXajI.png)
2. 定义内容观察者
3. ![](http://i.imgur.com/iVvpfWk.png) 
    #01_使用ContentProvider（内容提供者）共享数据

	ContentProvider 在android中的作用是对外共享数据，也就是说你可以通过ContentProvider把应用中的数据共享给其他应用访问，其他应用可以通过ContentProvider 对你应用中的数据进行添删改查。关于数据共享，以前我们学习过文件操作模式，知道通过指定文件的操作模式为Context.MODE_WORLD_READABLE 或Context.MODE_WORLD_WRITEABLE同样也可以对外共享数据。那么，这里为何要使用ContentProvider 对外共享数据呢？是这样的，如果采用文件操作模式对外共享数据，数据的访问方式会因数据存储的方式而不同，导致数据的访问方式无法统一，如：采用xml文件对外共享数据，需要进行xml解析才能读取数据；采用sharedpreferences共享数据，需要使用sharedpreferences API读取数据。
	使用ContentProvider对外共享数据的好处是统一了数据的访问方式。

	当应用需要通过ContentProvider对外共享数据的步骤：

	第一步需要继承ContentProvider并重写下面方法：
	public class PersonContentProvider extends ContentProvider{
	   public boolean onCreate()
	   public Uri insert(Uri uri, ContentValues values)
	   public int delete(Uri uri, String selection, String[] selectionArgs)
	   public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)
	   public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)
	   public String getType(Uri uri)}

	第二步需要在AndroidManifest.xml使用<provider>对该ContentProvider进行配置，为了能让其他应用找到该ContentProvider ， ContentProvider 采用了authorities（主机名/域名）对它进行唯一标识，你可以把 ContentProvider看作是一个网站（想想，网站也是提供数据者），authorities 就是他的域名：
	<manifest .... >
	    <application android:icon="@drawable/icon" android:label="@string/app_name">
	        <provider android:name=".PersonContentProvider" android:authorities="cn.itcast.providers.personprovider"/>
	    </application>
	</manifest>


代码：
		
	package cn.itcast.db;
	
	import cn.itcast.service.DBOpenHelper;
	import android.content.ContentProvider;
	import android.content.ContentUris;
	import android.content.ContentValues;
	import android.content.UriMatcher;
	import android.database.Cursor;
	import android.database.sqlite.SQLiteDatabase;
	import android.net.Uri;
	
	public class PersonContentProvider extends ContentProvider {
	private static UriMatcher matcher = new UriMatcher(UriMatcher.NO_MATCH);
	private static final int PERSONS = 1;
	private static final int PERSON = 2;
	private DBOpenHelper dbOpenHelper;
	static{
		matcher.addURI("cn.itcast.providers.personprovider", "person", PERSONS);
		matcher.addURI("cn.itcast.providers.personprovider", "person/#", PERSON);
	}

	@Override
	public boolean onCreate() {
		dbOpenHelper = new DBOpenHelper(this.getContext());
		return true;
	}
	
	@Override
	public int delete(Uri uri, String selection, String[] selectionArgs) {
		SQLiteDatabase db = dbOpenHelper.getWritableDatabase();
		int num = 0 ;//已经删除的记录数量
		switch (matcher.match(uri)) {
		case PERSONS:
			num = db.delete("person", selection, selectionArgs);
			break;			
		case PERSON:
			long id = ContentUris.parseId(uri);
			String where = "personid="+ id;
			if(selection!=null && !"".equals(selection)){ // personid=12 and name=?
				where = where + " and "+ selection;
			}
			num = db.delete("person", where, selectionArgs);
			break;
		default:
			throw new IllegalArgumentException("Unkown Uri:"+ uri);
		}
		getContext().getContentResolver().notifyChange(uri, null);
		return num;
	}

	@Override
	public String getType(Uri uri) {//返回当前操作的数据类型
		switch (matcher.match(uri)) {
		case PERSONS://操作的是集合类型数据
			return "vnd.android.cursor.dir/person";
		case PERSON:
			return "vnd.android.cursor.item/person";
		default:
			throw new IllegalArgumentException("Unkown Uri:"+ uri);
		}
	}

	@Override
	public Uri insert(Uri uri, ContentValues values) {
		SQLiteDatabase db = dbOpenHelper.getWritableDatabase();
		long id = 0 ;
		switch (matcher.match(uri)) {
		case PERSONS:
			id = db.insert("person", "personid", values);//得到记录的id
			getContext().getContentResolver().notifyChange(uri, null);
			return ContentUris.withAppendedId(uri, id);//返回代表新增记录的Uri
		case PERSON:
			id = db.insert("person", "personid", values);//得到记录的id
			String strUri = uri.toString();
			Uri personUri = Uri.parse(strUri.substring(0, strUri.lastIndexOf("/")));
			getContext().getContentResolver().notifyChange(personUri, null);
			return ContentUris.withAppendedId(personUri, id);
		default:
			throw new IllegalArgumentException("Unkown Uri:"+ uri);
		}
	}

	@Override
	public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
		SQLiteDatabase db = dbOpenHelper.getReadableDatabase();
		switch (matcher.match(uri)) {
		case PERSONS:
			return db.query("person", projection, selection, selectionArgs, null, null, sortOrder);
			
		case PERSON:
			long id = ContentUris.parseId(uri);
			String where = "personid="+ id;
			if(selection!=null && !"".equals(selection)){ // personid=12 and name=?
				where = where + " and "+ selection;
			}
			return db.query("person", projection, where, selectionArgs, null, null, sortOrder);
			
		default:
			throw new IllegalArgumentException("Unkown Uri:"+ uri);
		}
	}

	@Override
	public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs) {
		SQLiteDatabase db = dbOpenHelper.getWritableDatabase();
		int num = 0 ;//已经修改的记录数量
		switch (matcher.match(uri)) {
		case PERSONS:
			num = db.update("person", values, selection, selectionArgs);
			break;			
		case PERSON:
			long id = ContentUris.parseId(uri);
			String where = "personid="+ id;
			if(selection!=null && !"".equals(selection)){ 
				where = where + " and "+ selection;
			}
			num = db.update("person", values, where, selectionArgs);
			break;
		default:
			throw new IllegalArgumentException("Unkown Uri:"+ uri);
		}
		getContext().getContentResolver().notifyChange(uri, null);//通知数据发生变化
		return num;
	}

	}

#02_Uri介绍

	Uri代表了要操作的数据，Uri主要包含了两部分信息：1》需要操作的ContentProvider ，2》对ContentProvider中的什么数据进行操作，一个Uri由以下几部分组成：

	ContentProvider（内容提供者）的scheme已经由Android所规定， scheme为：content://
	主机名（或叫Authority）用于唯一标识这个ContentProvider，外部调用者可以根据这个标识来找到它。
	路径（path）可以用来表示我们要操作的数据，路径的构建应根据业务而定，如下:
	要操作person表中id为10的记录，可以构建这样的路径:/person/10
	要操作person表中id为10的记录的name字段， person/10/name
	要操作person表中的所有记录，可以构建这样的路径:/person
	要操作xxx表中的记录，可以构建这样的路径:/xxx
	当然要操作的数据不一定来自数据库，也可以是文件、xml或网络等其他存储方式，如下:
	要操作xml文件中person节点下的name节点，可以构建这样的路径：/person/name
	如果要把一个字符串转换成Uri，可以使用Uri类中的parse()方法，如下：
	Uri uri = Uri.parse("content://cn.itcast.provider.personprovider/person")


##03_UriMatcher类使用介绍

	因为Uri代表了要操作的数据，所以我们经常需要解析Uri，并从Uri中获取数据。Android系统提供了两个用于操作Uri的工具类，分别为UriMatcher 和ContentUris 。掌握它们的使用，会便于我们的开发工作。
	UriMatcher类用于匹配Uri，它的用法如下：
	首先第一步把你需要匹配Uri路径全部给注册上，如下：
	//常量UriMatcher.NO_MATCH表示不匹配任何路径的返回码
	UriMatcher  sMatcher = new UriMatcher(UriMatcher.NO_MATCH);
	//如果match()方法匹配content://cn.itcast.provider.personprovider/person路径，返回匹配码为1
	sMatcher.addURI(“cn.itcast.provider.personprovider”, “person”, 1);//添加需要匹配uri，如果匹配就会返回匹配码
	//如果match()方法匹配content://cn.itcast.provider.personprovider/person/230路径，返回匹配码为2
	sMatcher.addURI(“cn.itcast.provider.personprovider”, “person/#”, 2);//#号为通配符
	switch (sMatcher.match(Uri.parse("content://cn.itcast.provider.personprovider/person/10"))) { 
	   case 1
	    break;
	   case 2
	    break;
	   default://不匹配
	    break;
	}
	注册完需要匹配的Uri后，就可以使用sMatcher.match(uri)方法对输入的Uri进行匹配，如果匹配就返回匹配码，匹配码是调用addURI()方法传入的第三个参数，假设匹配content://cn.itcast.provider.personprovider/person路径，返回的匹配码为1



##04_使用ContentProvider共享数据

	ContentProvider类主要方法的作用：
	public boolean onCreate()
	该方法在ContentProvider创建后就会被调用， Android开机后， ContentProvider在其它应用第一次访问它时才会被创建。
	public Uri insert(Uri uri, ContentValues values)
	该方法用于供外部应用往ContentProvider添加数据。
	public int delete(Uri uri, String selection, String[] selectionArgs)
	该方法用于供外部应用从ContentProvider删除数据。
	public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)
	该方法用于供外部应用更新ContentProvider中的数据。
	public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)
	该方法用于供外部应用从ContentProvider中获取数据。
	public String getType(Uri uri)
	该方法用于返回当前Url所代表数据的MIME类型。如果操作的数据属于集合类型，那么MIME类型字符串应该以vnd.android.cursor.dir/开头，例如：要得到所有person记录的Uri为content://cn.itcast.provider.personprovider/person，那么返回的MIME类型字符串应该为：“vnd.android.cursor.dir/person”。如果要操作的数据属于非集合类型数据，那么MIME类型字符串应该以vnd.android.cursor.item/开头，例如：得到id为10的person记录，Uri为content://cn.itcast.provider.personprovider/person/10，那么返回的MIME类型字符串应该为：“vnd.android.cursor.item/person”。
	
	
##05_使用ContentResolver操作ContentProvider中的数据
	
	当外部应用需要对ContentProvider中的数据进行添加、删除、修改和查询操作时，可以使用ContentResolver 类来完成，要获取ContentResolver 对象，可以使用Activity提供的getContentResolver()方法。 ContentResolver 类提供了与ContentProvider类相同签名的四个方法：
	public Uri insert(Uri uri, ContentValues values)
	该方法用于往ContentProvider添加数据。
	public int delete(Uri uri, String selection, String[] selectionArgs)
	该方法用于从ContentProvider删除数据。
	public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)
	该方法用于更新ContentProvider中的数据。
	public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)
	该方法用于从ContentProvider中获取数据。

	这些方法的第一个参数为Uri，代表要操作的ContentProvider和对其中的什么数据进行操作，假设给定的是： Uri.parse(“content://cn.itcast.providers.personprovider/person/10”)，那么将会对主机名为cn.itcast.providers.personprovider的ContentProvider进行操作，操作的数据为person表中id为10的记录。


	使用ContentResolver对ContentProvider中的数据进行添加、删除、修改和查询操作：
	ContentResolver resolver =  getContentResolver();
	Uri uri = Uri.parse("content://cn.itcast.provider.personprovider/person");
	//添加一条记录
	ContentValues values = new ContentValues();
	values.put("name", "itcast");
	values.put("age", 25);
	resolver.insert(uri, values);		
	//获取person表中所有记录
	Cursor cursor = resolver.query(uri, null, null, null, "personid desc");
	while(cursor.moveToNext()){
		Log.i("ContentTest", "personid="+ cursor.getInt(0)+ ",name="+ cursor.getString(1));
	}
	//把id为1的记录的name字段值更改新为liming
	ContentValues updateValues = new ContentValues();
	updateValues.put("name", "liming");
	resolver.update(updateIdUri, updateValues, null, null);
	


#06_监听ContentProvider中数据的变化


	如果ContentProvider的访问者需要知道ContentProvider中的数据发生变化，可以在ContentProvider 发生数据变化时调用getContentResolver().notifyChange(uri, null)来通知注册在此URI上的访问者，例子如下：

	public class PersonContentProvider extends ContentProvider {
	public Uri insert(Uri uri, ContentValues values) {
		db.insert("person", "personid", values);
		getContext().getContentResolver().notifyChange(uri, null);
	}
	}

	如果ContentProvider的访问者需要得到数据变化通知，必须使用ContentObserver对数据（数据采用uri描述）进行监听，当监听到数据变化通知时，系统就会调用ContentObserver的onChange()方法：

	getContentResolver().registerContentObserver(Uri.parse("content://cn.itcast.providers.personprovider/person"),
        		true,//true表示只要发出通知的Uri以方法第一个参数开头都能被监听到，否侧监听uri必须与发出通知的uri完全匹配才能被监听到
                                           new PersonObserver(new Handler()));
	public class PersonObserver extends ContentObserver{
	public PersonObserver(Handler handler) {
		super(handler);
	}
	public void onChange(boolean selfChange) {
	    //此处可以进行相应的业务处理
	}
	}




##7读取联系人数据
  
    public class ContactUtils {

	// 给调用者返回一个集合
	public static List<ContactInfo> getContacts(Context context) {

		List<ContactInfo> contactInfos = new ArrayList<ContactInfo>();

		// 1 查询raw_contacts 表 查询 contact_id 列
		// 定义 查询raw_contacts 表的 uri
		Uri uri = Uri.parse("content://com.android.contacts/raw_contacts");
		// 拿到内容提供者的解析器
		ContentResolver resolver = context.getContentResolver();
		Cursor cursor = resolver.query(uri, new String[] { "contact_id" },
				null, null, null);
		while (cursor.moveToNext()) {
			ContactInfo info = new ContactInfo();
			String id = cursor.getString(0);
			if (id != null) {
				System.out.println("raw_contact-id~~~~" + id);
				info.setId(id);

				// 2 根据这个id 去查询 data 表
				Uri datauri = Uri.parse("content://com.android.contacts/data"); // 定义
																				// data表的uri
				// 查询实际上 是data表和 mimetype 表的视图 视图中 定义mimetype_id 叫 mimetype
				Cursor dataCursor = resolver.query(datauri, new String[] {
						"mimetype", "data1" }, "raw_contact_id=?",
						new String[] { id }, null);
				while (dataCursor.moveToNext()) {
					String mimetype = dataCursor.getString(0); // 获取mimetype_id
					String data = dataCursor.getString(1); // 获取data1 列的内容

					// 3 根据 mimetype 类型 区分数据
					if ("vnd.android.cursor.item/email_v2".equals(mimetype)) {
						// 说明是data 是email 数据
						System.out.println("email~" + data);
						info.setEmail(data);

					} else if ("vnd.android.cursor.item/name".equals(mimetype)) {
						System.out.println("name~" + data);
						info.setName(data);

					} else if ("vnd.android.cursor.item/phone_v2"
							.equals(mimetype)) {
						System.out.println("phone~" + data);
						info.setPhone(data);
					}

				}
				dataCursor.close();
				contactInfos.add(info);

			}

		}
		cursor.close();

		return contactInfos;

	  }

    }




##08_窃听用户发出的短信

	用户使用系统自带的短信程序发送短信，程序会通过ContentProvider把短信保存进数据库，并且发出一个数据变化通知，使用ContentObserver对数据变化进行监听，在用户发送短信时，就会被ContentObserver窃听到短信：
注册监听：

	getContentResolver().registerContentObserver(Uri.parse("content://sms"),  true, new SmsObserver(new Handler()));
	
监听类：

	private final class SmsObserver extends ContentObserver{
	public SmsObserver(Handler handler) {
		super(handler);
	}

	public void onChange(boolean selfChange) {//查询发送箱中的短信(处于正在发送状态的短信放在发送箱)
		Cursor cursor = getContentResolver().query(Uri.parse("content://sms/outbox"),null, null, null, null); 
		while(cursor.moveToNext()){
			StringBuilder sb = new StringBuilder();
			sb.append("_id=").append(cursor.getInt(cursor.getColumnIndex("_id")));
			sb.append(",address=").append(cursor.getString(cursor.getColumnIndex("address")));
			sb.append(";body=").append(cursor.getString(cursor.getColumnIndex("body")));
			sb.append(";time=").append(cursor.getLong(cursor.getColumnIndex("date")));
			Log.i("ReceiveSendSMS", sb.toString());
	             } }    	
	}





