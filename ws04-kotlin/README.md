# Develop Kotlin App with AdapterViews

What do you do if the app needs to handle data of a large quantity or dynamically generated? Do you put say 200 student records in TextViews one after another? Obviously, the answer is no here. Let's find out how to do it properly.

## Lab 1 AdapterViews

Spinners you looked at previously is a special case i.e. subclass of AdapterView. In order to get it work, yous had to combine three elements: the View itself (Spinner), the data resource which is typically a Collection (interface), an Adaptor that binds every *single entry* in the Collection with its layout. 

<!-- ![adaptor components](http://cdn.edureka.co/blog/wp-content/uploads/2013/03/adapters-1.png) -->
![simple](.md_images/adapters-1.png)

Now let's look at some more subclasses of AdapterView.

### Simple ListView

Using default options, create a new project and name it 'My Lists'. Then, follow steps below to create a simple list:

1. First of all, define some data to play with. Insert the following into string.xml. 
    
    ```xml
    <string-array name="candidateNames">
        <item>Hillary Clinton</item>
        <item>Bernie Sanders</item>
        <item>Martin O\'Malley</item>
        <item>Lincoln Chafee</item>
        <item>Donald Trump</item>
        <item>Ben Carson</item>
        <item>Marco Rubio</item>
        <item>Jeb Bush</item>
    </string-array>

    <string-array name="candidateDetails">
        <item>Former US Secretary of State Hillary Clinton (New York)</item>
        <item>US Senator Bernie Sanders (Vermont)</item>
        <item>Former Governor Martin O\'Malley (Maryland)</item>
        <item>Former Governor Lincoln Chafee (Rhode Island)</item>
        <item>Businessman Donald Trump (New York)</item>
        <item>Dr. Ben Carson (Florida)</item>
        <item>US Senator Marco Rubio (Florida)</item>
        <item>Former Governor Jeb Bush (Florida)</item>
    </string-array>
    ```
2. Open activity_main.xml, delete the default TextView and drag/drop a ListView from Containers sub-group in Palette. In text mode, rearrange XML so it looks like this
    
    ```xml
    <ListView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_marginBottom="8dp"
        android:layout_marginLeft="8dp"
        android:layout_marginRight="8dp"
        android:layout_marginTop="8dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
    ```
    
3. Open MainActivity.java, insert the following declarations:
    
    ```java
    private ListView listView;
    private String[] candidateNames;
    ```
    
4. In MainActivity.java, insert the following into the `onCreate()` method
    
    ```java
    candidateNames = getResources().getStringArray(R.array.candidateNames);
    
    listView = (ListView) findViewById(R.id.listViewSimple);
    ArrayAdapter<String> arrayAdapter = new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, candidateNames);
    listView.setAdapter(arrayAdapter);
    listView.setOnItemClickListener(
    
            new AdapterView.OnItemClickListener() {
                @Override
                public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                    Toast.makeText(getBaseContext(), candidateNames[position] + ", seriously?", Toast.LENGTH_SHORT).show();
                }
            }
    );
    ```
    
    What the code above does is to associate the ListView declared in the layout file with the data (string array) declared in string resource. ArrayAdapter takes three parameters: context, a layout resource for a single element of data, and the data. Here `android.R.layout.simple_list_item_1` is a system-defined resource layout file that contains only one TextView. You can define you own resource files as you see later on.
    
    > [simple_list_item_1](https://github.com/android/platform_frameworks_base/blob/master/core/res/res/layout/simple_list_item_1.xml) on Github.
    
    Inside `setOnItemClickListener()` block is an anonymous inner class. There're two ways of defining an onClickListener, one is to do `public void doSomething(View v)` and then associate with View's `onClick` attribute in XML, the other is what you see here. Note here instead of declaring `View.onClick()` for Buttons, it declared `AdapterView.OnItemClickListener()` wich is specific for AdapterViews. Here 'parent' is the parent view of single data entry, in our case is the ListView,  view is the View being clicked, position is the 'index' of current view within the adapter, id is the row id the data entry.
    
    The question mark inside angle brackets is Java generic wildcard, which basically means the type of parent passed into the method is an AdapterView of any type.
    
If you run the app, what you'll see is something like this:

![simple](.md_images/simple.png)

The following code can be added after the `setOnItemClickListener()` block, but still within `onCreate()`method:

```java
List<String> candidateNamesNew = new ArrayList<String>(Arrays.asList(candidateNames));
ArrayAdapter<String> arrayAdapterNew = new ArrayAdapter<String>(this, android.R.layout
        .simple_list_item_1, candidateNamesNew);
listView.setAdapter(arrayAdapterNew);
arrayAdapterNew.add("New Someone");
arrayAdapter.notifyDataSetInvalidated();
```

Have a look at the code above, and try to answer the following questions:

* Why is it necessary to convert string array to array list?
* How does array adapter update the actual dataset i.e. candidate data?
* What is the purpose of `notifyDataSetInvalidated()`?

### Complex ListView

Simple ListView is useful for displaying data that can be converted to strings in easy steps. But if you want to have fine control of the presentation of single entries in your ListView, you need to provide customized layout files for your adapter. In this way, you'll make it a 'complex ListView'.

1. First of all, download some images to use later on. Click on [this link](https://github.coventry.ac.uk/300CEM-1718SEPJAN/TEACHING-MATERIALS/blob/master/Additional_resources/candidates_photos.zip) to go to GitHub page and then click 'View Raw' to download some images of US presidential election candidates. Add those to your res/drawable resources folder.
    
2. Create a new Java class called Candidate and insert the following 
    
    ```java
    private String name;
    private String detail;
    private int photo;

    public Candidate(String name, String detail, int photo) {
        this.name = name;
        this.detail = detail;
        this.photo = photo;
    }

    public String getName() {
        return name;
    }

    public String getDetail() {
        return detail;
    }

    public int getPhoto() {
        return photo;
    }

    @Override
    public String toString() {
        return detail;
    }
    ```
    
3. Create a new layout resource file by right-clicking on the layout folder and select New ==> Layout resource file. Name it list_item.xml. 
    
    ![](.md_images/list_res.png)
    
    Change the default orientation of the container LinearLayout from `vertical` to `horizontal`, and insert the following within the LinearLayout
    
    ```xml
    <ImageView
        android:id="@+id/imageView"
        android:layout_width="85dp"
        android:layout_height="85dp"
        android:layout_marginLeft="5dp"
        android:layout_marginTop="5dp"
        android:background="@android:color/darker_gray"
        android:padding="8dp"
        android:scaleType="centerCrop" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="30dp"
        android:layout_marginTop="20dp"
        android:orientation="vertical">

        <TextView
            android:id="@+id/textViewName"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="first + last name"
            android:textSize="16sp" />

        <TextView
            android:id="@+id/textViewDetail"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:text="details of the candidate"
            android:textSize="12sp" />

    </LinearLayout>
    ```
    
    At the moment, your preview window should look like below. This is the layout for a single item in your list.
    
    ![list item](.md_images/listitem.png)
    
4. Create a new activity using the 'Empty Activity' template and name it PhotoListActivity. Open activity_photo_list.xml, insert the following into the container RelativeLayout:
    
    ```xml
    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentLeft="true"
        android:layout_alignParentStart="true"
        android:layout_alignParentTop="true"
        android:layout_marginLeft="8dp"
        android:layout_marginStart="8dp"
        android:layout_marginTop="8dp"
        android:text="2016 Presidential Candidates"
        android:textAppearance="?android:attr/textAppearanceLarge"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintTop_toTopOf="parent"/>

    <ListView
        android:id="@+id/listViewComplex"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/textView"
        android:layout_marginLeft="16dp"
        android:layout_marginRight="8dp"
        android:layout_marginStart="8dp"
        android:layout_marginTop="7dp"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/textView"/>
    ```
    
    This layout is very similar to the simple list example. The only difference is that it has an additional TextView above ListView.
    
5. In PhotoListActivity.java, add the following class member variable declaration/initialization.
    
    ```java
    private ListView listView;
    private String[] candidateNames;
    private String[] candidateDetails;
    public static int[] candidatePhotos = {
            R.drawable.clinton,
            R.drawable.sanders,
            R.drawable.omalley,
            R.drawable.chafee,
            R.drawable.trump,
            R.drawable.carson,
            R.drawable.rubio,
            R.drawable.bush
    };
    private ArrayList<Candidate> candidates = new ArrayList<>();
    ```
    
    Note in the code above an array of integers is declared for drawable resources. (Why is the type int?)
    
6. Insert String array initialization into the `onCreate()` method, just below `setContentView()`, so your `onCreate()` becomes
    
    ```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_photo_list);

        candidateNames = getResources().getStringArray(R.array.candidateNames);
        candidateDetails = getResources().getStringArray(R.array.candidateDetails);
        generateCandidates();
    ```
    
    Insert the following as a class method. This is to create a function that initializes the ArrayList for Candidate class.
    
    ```java
    private void generateCandidates() {

        for (int i = 0; i < candidatePhotos.length; i++) {
            candidates.add(new Candidate(candidateNames[i], candidateDetails[i], candidatePhotos[i]));
        }
    }
    ```
    
7. Insert the following code into `onCreate()` method
    
    ```java
    listView = (ListView) findViewById(R.id.listViewComplex);
        listView.setAdapter(new CandidateAdapter(this, R.layout.list_item, candidates));
        listView.setOnItemClickListener(
        
                new AdapterView.OnItemClickListener() {
                    @Override
                    public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                    
                        Toast.makeText(getBaseContext(), "You clicked " + candidates.get(position), Toast.LENGTH_SHORT).show();
                    }
                }
        );
    ```
    
    The code above initialize a ListView and associates it with a customized Adapter. The onClickListner is the same as in the simple list example. The only thing that is new here is the association of data and ListView through a customized Adapter.
    
    > CandidateAdapter can be used to add more data to the arraylist e.g. `candidateAdapter.add(candidates.get(0)); candidateAdapter.notifyDataSetInvalidated();` But this requires a 'candidateAdapter' object to be created beforehand.
    
8. Create a new class called CandidateAdapter. Open the Java file and replace auto-generated class with the following (don't touch the package declaration!)
    
    ```java
    public class CandidateAdapter extends ArrayAdapter<Candidate> {

        private int resource;
        private ArrayList<Candidate> candidates;
        private Context context;

        public CandidateAdapter(Context context, int resource, ArrayList<Candidate> candidates) {
            super(context, resource, candidates);
            this.resource = resource;
            this.candidates = candidates;
            this.context = context;
        }

        @NonNull
        @Override
        public View getView(int position, View convertView, ViewGroup parent) {
            View v = convertView;
            try {
                if (v == null) {
                    LayoutInflater layoutInflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
                    v = layoutInflater.inflate(resource, parent, false);
                }

                ImageView imageView = (ImageView) v.findViewById(R.id.imageView);
                TextView textViewName = (TextView) v.findViewById(R.id.textViewName);
                TextView textViewDetail = (TextView) v.findViewById(R.id.textViewDetail);

                imageView.setImageResource(candidates.get(position).getPhoto());
                textViewName.setText(candidates.get(position).getName());
                textViewDetail.setText(candidates.get(position).getDetail());

            } catch (Exception e) {
                e.printStackTrace();
                e.getCause();
            }
            return v;
        }

    }
    ```
    
    It's important to understand the code above: the CandidatesAdapter class extends ArrayAdapter of type Candidates. In the constructor, it needs the layout resource name i.e. the file that contains the customized layout XML created earlier. The most important method is `getView()`, where it checks if a convertView (i.e. old view) exists or not. If it doesn’t, it’ll need to inflate it. The reason for doing this is because ListView recycles its rows when they move out of the screen, instead of creating new ones, to save system resources. 
    
    > [How ListView's recycling mechanism works](http://stackoverflow.com/questions/11945563/how-listviews-recycling-mechanism-works)
    
    There're several different ways of getting an LayoutInflater object:
    
    * The code above, `LayoutInflater layoutInflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE)`, is from official [documentation](http://developer.android.com/reference/android/view/LayoutInflater.html).
    * You can use `LayoutInflater inflater = ((Activity)context).getLayoutInflater()`, see an example from [here](http://www.ezzylearning.com/tutorial/customizing-android-listview-items-with-custom-arrayadapter).
    * You can also use `LayoutInflater inflater = (LayoutInflater) CandidatesAdapter.this.getSystemService(Context.LAYOUT_INFLATER_SERVICE)`, see an example from [here](https://github.com/pranayairan/Code-Learn-Android-Example/blob/master/CodeLearnListExample/src/org/codelearn/codelearnlistexample/ListViewWithBaseAdapter.java).
    
    Note here `v.findViewById()` is different from `findViewById()`. `v.findViewById()` will only find sub views i.e. views being contained by 'v'; whereas `findViewById()` will find anything contained in the Activity.
    
    
9. Insert the following into activity_main.xml before the ListView
    
    ```xml
    <Button
        android:id="@+id/complexList"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="8dp"
        android:layout_marginTop="8dp"
        android:onClick="onButtonClick"
        android:text="Complex List"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintTop_toTopOf="parent"/>
    ```
    
    You'll see that ListView and Button overlap at the top-left corner of the screen. Change the following attribute of `ListView` to have more space at the top `app:layout_constraintTop_toBottomOf="@+id/complexList"`
    
10. Open MainActivity.java, insert the following into the class
    
    ```java
    public void onButtonClick(View v){
        startActivity(new Intent(this, PhotoListActivity.class));
    }
    ```
    
If you run the app, click on 'Complex List' button, you'll see the following:

![photo](.md_images/photo.png)

<!-- ### Grid View was here-->

### AdapterView hierarchy

You saw ArrayAdapter already. If you Google online examples you'll see more Adapters such as BaseAdapter, ListAdapter, and SimpleAdapter etc. What are the relationships among these?

Basically, ArrayAdapter is the first concrete Adapter in the tree, above it are interfaces and an abstract class. But sometimes people do declare something like `ListAdapter listAdapter = new ArrayAdapter<String>()`, don't be confused.

<!-- ![AdapterViewHierarchy](http://www.intertech.com/Blog/wp-content/uploads/2014/06/HeirarchyOfAdapter-480x396.png) -->
![AdapterViewHierarchy](.md_images/HeirarchyOfAdapter.png)

A similar hierarchy can be drawn for AdapterView and subclasses. Even though those 'collection' Views are named differently, they are in fact closely related to each other.

<!-- ![HeirarchyOfAdapter](http://www.intertech.com/Blog/wp-content/uploads/2014/06/AdapterViewHierarchy-480x396.png) -->
![HeirarchyOfAdapter](.md_images/AdapterViewHierarchy.png)

> Above images are from a [blog](http://www.intertech.com/Blog/android-adapters-adapterviews/) written by Jim White.
