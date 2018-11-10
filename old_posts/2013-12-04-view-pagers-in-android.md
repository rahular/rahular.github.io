---
id: 38
title: Building View Pagers in Android
author: rahular
layout: post
guid: http://rahular.com/?p=38
excerpt: A tutorial on building dynamic views using Fragments
permalink: /view-pagers-in-android/
categories:
  - android
---
In the last post we talked about **ListViews** and how to use them in our apps. Here I want to build a little on the same code base and show how to create **ViewPagers**. (Please keep in mind that we use the **ViewPager** class available in android-support-v4 so that it is compatible with older devices as well) **ViewPagers** are like tabs which you find in almost all web browsers today. They allow users to easily navigate within the apps and allows the developers to categorize their content. An example of an activity using **ViewPagers** is shown below.

  {% include image.html url="../res/app_structure_default_tabs.png" description="" scale="40%" %}
  
The first thing that we need for creating tabs like these are **Fragments**. **Fragments** can be thought of as a piece of UI which can be placed into suitable container (mostly an **Activity**). Many Fragments can be combined to make an **Activity** or a **Fragment** can be reused in multiple activities.

Initially we will just tweak the **ListView** code from the previous blog so that we can plug it directly into our app. For **ListViews**, we created 2 activities called StaticActivity and DynamicActivity. We will now convert them into **Fragments**.

Since an **Activity** can have multiple **Fragments** and each Fragment&#8217;s UI is defined using XML, the compiler can easily get confused if one or more elements have the same IDs. Therefore we need to get a root view of each fragment and then define or bind actions to elements using the root views for disambiguation. The code will make this point more clear.

### StaticActivity.java

(The XML remains same as in the previous blog)

```java
public class StaticActivity extends Fragment {

	ListView listView;
	Context context;

	@Override
	public View onCreateView(LayoutInflater inflater, ViewGroup container,
	Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);

		View rootView = inflater.inflate(R.layout.activity_static, container, false);
		context = getActivity();

		listView = (ListView) rootView.findViewById(R.id.list_view);

		final Intent mainIntent = new Intent(Intent.ACTION_MAIN, null);
		mainIntent.addCategory(Intent.CATEGORY_LAUNCHER);
		final List<ResolveInfo> data = context.getPackageManager()
		.queryIntentActivities(mainIntent, 0);

		final ArrayList<String> stringData = new ArrayList<String>();
		for (int i = 0; i < data.size(); i++) {
			stringData.add(data.get(i).loadLabel(context.getPackageManager())
			.toString());
		}

		ArrayAdapter<String> adapter = new ArrayAdapter<String>(context,
		android.R.layout.simple_list_item_1, stringData);

		listView.setAdapter(adapter);
		listView.setOnItemClickListener(new OnItemClickListener() {

			@Override
			public void onItemClick(AdapterView<?> parent, View view,
			int position, long id) {
				Toast.makeText(context,
				"You clicked on " + stringData.get(position),
				Toast.LENGTH_SHORT).show();
			}
		});

		return rootView;
	}
}
```
  
### DynamicActivity.java

(The XML remains same as in the previous blog)

```java
public class DynamicActivity extends Fragment {

	private EditText etText;
	private ListView listView;
	private Button btnAdd;
	private ArrayList<String> data;
	private Context context;

	public View onCreateView(LayoutInflater inflater, ViewGroup container,
	Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);

		View rootView = inflater.inflate(R.layout.activity_dynamic, container, false);
		context = getActivity();

		etText = (EditText) rootView.findViewById(R.id.et_text);
		btnAdd = (Button) rootView.findViewById(R.id.btn_add);
		listView = (ListView) rootView.findViewById(R.id.list_view);
		final ArrayAdapter<String> adapter = new ArrayAdapter<String>(context,
		android.R.layout.simple_list_item_1);

		data = new ArrayList<String>();

		listView.setAdapter(adapter);
		listView.setOnItemClickListener(new OnItemClickListener() {

			@Override
			public void onItemClick(AdapterView<?> parent, View view,
			int position, long id) {
				Toast.makeText(context,
				"You clicked on " + data.get(position),
				Toast.LENGTH_SHORT).show();
			}
		});

		btnAdd.setOnClickListener(new OnClickListener() {

			@Override
			public void onClick(View v) {
				String text = etText.getText().toString();
				if (text.isEmpty()) {
					Toast.makeText(context, "No data to add",
					Toast.LENGTH_SHORT).show();
					return;
				}
				adapter.add(text);
				data.add(text);
			}
		});

		return rootView;
	}

}
```

In this tutorial, for the sake of simplicity, we only have one **Fragment** in an **Activity**. Once these 2 fragments are ready, we need to build a container for them. So we will create a new **Activity** and name in MainActivity. This Activity will hold the fragments and display them as tabs.

### activity_main.xml

Here the XML contains a single **ViewPager** element which will fill the entire screen.

### MainActivity.java

Now we get into some java coding. In MainActivity.java we bind the **ViewPager** element from the UI with a corresponding object so that we can make calls on it. The **ViewPager** needs an adapter so that it can get items at run time based on the current position (which tab is active) and the next position (depends on whether there is a left swipe or a right swipe). This is very similar to the **ListAdapter** we saw in the previous blog.For this app, we also create a simple **ViewPagerAdapter** which supplies the **Fragments** and other required information to the **Activity**. Here is the code :

```java
public class MainActivity extends FragmentActivity implements
ActionBar.TabListener {

	AppSectionsPagerAdapter mAppSectionsPagerAdapter;
	ViewPager mViewPager;

	@SuppressLint("NewApi")
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		mAppSectionsPagerAdapter = new AppSectionsPagerAdapter(
		getSupportFragmentManager());

		final ActionBar actionBar = getActionBar();
		actionBar.setHomeButtonEnabled(false);
		actionBar.setNavigationMode(ActionBar.NAVIGATION_MODE_TABS);

		mViewPager = (ViewPager) findViewById(R.id.pager);
		mViewPager.setAdapter(mAppSectionsPagerAdapter);
		mViewPager
		.setOnPageChangeListener(new ViewPager.SimpleOnPageChangeListener() {
			@Override
			public void onPageSelected(int position) {
				actionBar.setSelectedNavigationItem(position);
			}
		});

		for (int i = 0; i < mAppSectionsPagerAdapter.getCount(); i++) {
			actionBar.addTab(actionBar.newTab()
			.setText(mAppSectionsPagerAdapter.getPageTitle(i))
			.setTabListener(this));
		}
	}

	@Override
	public void onTabUnselected(ActionBar.Tab tab,
	FragmentTransaction fragmentTransaction) {
	}

	@Override
	public void onTabSelected(ActionBar.Tab tab,
	FragmentTransaction fragmentTransaction) {
		mViewPager.setCurrentItem(tab.getPosition());
	}

	@Override
	public void onTabReselected(ActionBar.Tab tab,
	FragmentTransaction fragmentTransaction) {
	}

	public static class AppSectionsPagerAdapter extends FragmentPagerAdapter {

		public AppSectionsPagerAdapter(FragmentManager fm) {
		super(fm);
		}

		@Override
		public Fragment getItem(int i) {
			switch (i) {
				case 0:
				return new StaticActivity();
				case 1:
				return new DynamicActivity();
				default:
				return null;
			}
		}

		@Override
		public int getCount() {
			return 2;
		}

		@Override
		public CharSequence getPageTitle(int position) {
			switch (position) {
			case 0:
				return "Static List";
				case 1:
				return "Dynamic List";
				default:
				return null;
			}
		}
	}
}
```

The result will look something like this :

{% include image.html url="../res/framed_dynamic_list.png" description="Dynamic List" scale="50%" %}

{% include image.html url="../res/framed_static_list.png" description="Static List" scale="50%" %}

That&#8217;s it guys! Have fun :)
