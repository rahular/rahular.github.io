---
id: 36
title: 'Android Views : List View'
author: rahular
layout: post
excerpt: A simple tutorial on Android's List View
guid: http://rahular.com/?p=36
permalink: /android-views-list-view/
categories:
  - android
---
In this post we will see how to implement a <b>List View</b> on Android. A list view is extremely useful when we have to show a lot of information is an orderly manner or when we have to create a menu with large number of options.
  
Android as always does most of the work for us. We just need to provide the data and it does the rest. There are 2 ways to provide data :

* <b>Array Adapter</b> : In this case, the data is already available to us in the form of a list/array/ArrayList
* <b>Cursor</b> <b>Adapter</b> : We use cursors when data is queried from a database.

The cursor can be thought of as a pointer (not in the C sense) to a table containing data which is returned as a result of a query being fired. It contains only those columns which we have specified in our projection.

Here we use Array Adapter for displaying data. But the same process can of displaying data can be extended for Cursor Adapters as well. So let&#8217;s get into some coding.
  
## Static Lists
  
Here, we use a list to display all the apps which are installed in the device on which the app runs.

  
### activity_static.xml
  
First of all we need to set up a layout for anything to show on screen. To simplify things, we just have a list view inside a relative layout in this example.

```java
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:tools="http://schemas.android.com/tools"
android:layout_width="match_parent"
android:layout_height="match_parent"
android:paddingBottom="@dimen/activity_vertical_margin"
android:paddingLeft="@dimen/activity_horizontal_margin"
android:paddingRight="@dimen/activity_horizontal_margin"
android:paddingTop="@dimen/activity_vertical_margin"
tools:context=".StaticActivity" >

<ListView
    android:id="@+id/list_view"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_alignParentLeft="true"
    android:layout_alignParentTop="true" >
</ListView>

</RelativeLayout>
```

    
### StaticActivity.java

The tasks performed by this class are :

* First we bind the list to an object in java as usual.
* Then we get a list of all apps installed on the device using the <b>PackageManager</b> class.
* We then tell the system what data to display as a list by creating an adapter.
* We define a dummy click listener for each item in the list.

NOTE : the data for this list is static in the sense that everything is retrieved before display.

```java
public class StaticActivity extends Activity {

	ListView listView;

	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);

		listView = (ListView) findViewById(R.id.list_view);

		// get a list of apps installed on the device
		final Intent mainIntent = new Intent(Intent.ACTION_MAIN, null);
		mainIntent.addCategory(Intent.CATEGORY_LAUNCHER);
		final List<ResolveInfo> data = getBaseContext().getPackageManager()
		  .queryIntentActivities(mainIntent, 0);
		final ArrayList<String> stringData = new ArrayList<String>();
		for (int i = 0; i < data.size(); i++) {
			stringData
			  .add(data.get(i)
			  .loadLabel(getBaseContext().getPackageManager())
			  .toString());
		}
		ArrayAdapter<String> adapter = new ArrayAdapter<String>(
		getBaseContext(), android.R.layout.simple_list_item_1,
		stringData);

		listView.setAdapter(adapter);
		listView.setOnItemClickListener(new OnItemClickListener() {
			@Override
			public void onItemClick(AdapterView<?> parent, View view,
			int position, long id) {
				Toast.makeText(getBaseContext(),
				"You clicked on " + stringData.get(position),
				Toast.LENGTH_SHORT).show();
			}
		});
	}
}
```
    
## Dynamic Lists

Now let us see how we can add items to a list dynamically at runtime. For this we create another class called <b>DynamicActivity</b> and an associated XML. Here the layout contains an Edit Text, an add button and a list view. Whatever text we type is added to the list by clicking the add button.

### activity_dynamic.xml

```java
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
android:layout_width="match_parent"
android:layout_height="match_parent" >

<LinearLayout
  android:id="@+id/linear_layout"
  android:layout_width="match_parent"
  android:layout_height="wrap_content"
  android:layout_alignParentLeft="true"
  android:layout_alignParentTop="true" >

  <EditText
    android:id="@+id/et_text"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_weight="2"
    android:ems="10" >

    <requestFocus />
    </EditText>

    <Button
      android:id="@+id/btn_add"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:layout_weight="1"
      android:text="@string/btn_add_string" />

</LinearLayout>

<ListView
    android:id="@+id/list_view"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_alignParentLeft="true"
    android:layout_below="@+id/linear_layout" >
</ListView>

</RelativeLayout>
```

### DynamicActivity.java

In this class, as always we perform binding of views from XML to java objects. We then define a click listener for the Add button which will insert data into the list. The rest of the code is similar to Static Activity.

```java
public class DynamicActivity extends Activity {

	private EditText etText;
	private ListView listView;
	private Button btnAdd;
	private ArrayList<String> data;

	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);

		etText = (EditText) findViewById(R.id.et_text);
		btnAdd = (Button) findViewById(R.id.btn_add);
		listView = (ListView) findViewById(R.id.list_view);
		final ArrayAdapter<String> adapter = new ArrayAdapter<String>(
		getBaseContext(), android.R.layout.simple_list_item_1);
		data = new ArrayList<String>();
		listView.setAdapter(adapter);
		listView.setOnItemClickListener(new OnItemClickListener() {

			@Override
			public void onItemClick(AdapterView<?> parent, View view,
			  int position, long id) {
				 Toast.makeText(getBaseContext(),
				 "You clicked on " + data.get(position),
				 Toast.LENGTH_SHORT).show();
			}
		});

		btnAdd.setOnClickListener(new OnClickListener() {

			@Override
			public void onClick(View v) {
				String text = etText.getText().toString();
				if (text.isEmpty()) {
					Toast.makeText(getBaseContext(), "No data to add",
					Toast.LENGTH_SHORT).show();
					return;
				}
				adapter.add(text);
				data.add(text);
			}
		});
	}
}
```

That&#8217;s it guys! Enjoy :)

