---
title: "[OLD] Courier App for Android"
date: 2017-05-17T15:34:30-04:00
toc: true
toc_label: "Table of Contents"
categories:
  - blog
tags:
  - coding
---

[This is a post from my old website. Outdated packages and libraries. Viewer discretion is advised ;-)]

## FIRST PART

The idea for a courier app isn’t new. You can find countless examples on Google Play Store. But building one is actually quite handy exercise in integrating Android with Firebase services.

In this series we will cover only parts of building the actual bike courier app. Focus will be on Firebase integration and also some design pattern strategies. Android code can look really ugly really quick if you just put everything within your Activity class.

### App requirements – what should it allow

* Register new users and log in already existing users.
* Place orders.
* Inform couriers that a new order is pending.
* Provide couriers with details on accepted order.
* Update customers on their order’s status with notifications.
* Access to order history.
* Some basic account informations and ability to edit those (i.e. change password or company name).
* Contact information for the company.

### What will we use? APIs & software

We will use pretty much only Google APIs and services: Places, Maps, Firebase Authentication, Firebase Realtime Database and Firebase Cloud Functions.

We will code in Android Studio (2.3.2 at the moment of writing this post).

### How will it look in the end? – Source code

All of the code discussed in this series can be found on my [Github](https://github.com/MiWy/CourierApplication).

### Sidenote !

The code’s quality isn’t the best imaginable. I’m just a beginner as well, so there is a lot of room for optimisation. That being said, if you feel a little bit lost in dealing with official Android/Firebase documentation, or do not know what Firebase is, this series should be able to help you.

## PART 2: Firebase

### What is Firebase

Firebase is a Google platform, a replacement to a traditional server-side solutions. With it, you don't have to own a server and you don't need to spend time configuring one from the scratch. If your needs are small enough, you don't even have to pay for it.

Most apps nowadays need some sort of back-end. It wouldn't be wise to keep all of the users information only on his/hers phone or computer. How would you recover user email if needed? Actually, how else could you enable accounts at all in your app? Send notifications? A need for one central place to store data is pretty common.

Firebase itself is more than just a database service. It can manage your users authentication process, database needs, notifications, custom node.js scripts, store user files, allows you to manage AdMob in your app, analyse user traffic and app usage, it pretty much functions like a ready-to-go server and cloud.

Imagine you have a mobile app and a web app. To connect the two all you have to do is to pin both apps to the same Firebase project. The best part is you don't even need to know any SQL, PHP or other popular server-side languages. Firebase interface is 'clickable', so on the most part you will only have to write client-side code. At some point it will probably be better to learn some code, though. Especially if you would like to automate some processes like responding to some database events (NoSQL and node.js).

### Ok, so how do I connect my app to it?

Easy-peasy. Go to [firebase.google.com](https://firebase.google.com/) and create an account. Then head over to Firebase Console and create a new project (you can attach multiple mobile/web apps to one project). Click **Add Project**:

![image-center](/assets/images/oldblog/firebase-addproject.png){: .align-center}

When you created the project, click on Android icon to add your Android Studio app to your Firebase project:

![image-center](/assets/images/oldblog/firebase-addapptoproject.png){: .align-center}

A window will pop up asking you for some information. At the **Android package name** enter the complete name of your app.

![image-center](/assets/images/oldblog/firebase-addappindow.png){: .align-center}

For the third field you can provide your debug SHA-1 fingerprint (it allows you to use some of the Google Play Services like Google Sign-In). Getting that value can be easily achieved with command-line. For Linux/MacOs machines:

```Gradle
    keytool -exportcert -list -v \
    -alias androiddebugkey -keystore ~/.android/debug.keystore
```

For Windows:

```Gradle
    keytool -exportcert -list -v \
    -alias androiddebugkey -keystore %USERPROFILE%\.android\debug.keystore
```

If you don't feel comfortable with command line, you can also click your way through to this value. Click following things in Android Studio and copy-paste SHA-1:

![image-center](/assets/images/oldblog/firebase-androidstudiowindowsk.png){: .align-center}

When you filled fields click **Register App**. A .json file will be downloaded. You need to put this file into your project in the following place (you can do that by dragging it to the relevant place in Android Studio or simply by navigating to your project folder on the hard-drive and pasting at the right place):

![image-center](/assets/images/oldblog/firebase-astudiojson.png){: .align-center}

Remember to make sure that only you have access to **google-services.json** file, as it contains all necessary information for your app to connect to the Firebase. If you are publishing your code on your Github or somewhere else, do not upload this file.

Lastly, you need to take care of dependencies. You have to edit two .gradle build files:

![image-center](/assets/images/oldblog/firebase-dependencies.png){: .align-center}

In your project-level **build.gradle** add following line of code:

``` Gradle
    buildscript {
      dependencies {
        classpath 'com.google.gms:google-services:3.0.0'
      }
    }
```

In your app-level build.gradle add following line of code **at the bottom **of the file:

```Gradle
    apply plugin: 'com.google.gms.google-services'
```

When you edit either of the build.gradle files a information bar will appear in Android Studio telling you to sync your project. When you've finished editing these files click on **Sync now** button:

![image-center](/assets/images/oldblog/firebase-astudiosync.png){: .align-center}

There you go! You connected your app to Firebase project.

### Easier way

Described method is manual and can seem complicated at the beginning, but I strongly recommend you to at least use it once. Otherwise how will you know exactly which files need to be altered for you to be connected with Firebase?

That being said there is also an easier way to connect your app. In Android Studio click on **Tools >** **Firebase **and then expand one of the features in the newly opened **Assistant **window. Upon expanding click the **Connect to Firebase** button and follow instructions on the screen.

## PART 3

### Do we need to create user accounts

We want users of our app to be able to order bike courier services. Our customers will most likely be companies trying to transport documents from one place to the other. Not that a lot of individuals use this sort of service.

It would be cumbersome for customers to provide all of the relevant data each time they place a new order, like their telephone number or e-mail address. So we should store that data once and reuse it.

One way of doing this is to store data permanently in the user phone. However, information would be lost upon deleting app from the device. Heck, what if user switched his phone to a newer one? We need a solution independent from the user. We need back-end.

### What will we use? Needs and requirements

We will integrate Firebase Authentication and Firebase Realtime Database to our app.

Create two activities: RegisterActivity and LoginActivity. Since user shouldn't see signup screen each time he/she opens the app, let's set AndroidManifest.xml file accordingly:

```XML
    <activity android:name=".RegisterActivity">
    </activity>
    <activity android:name=".LoginActivity">
    <intent-filter>
    <action android:name="android.intent.action.MAIN"/>

    <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
    </activity>
```

This way LoginActivity will be loaded directly after launching the app. We don't cover the details of Activity creation here. Just make sure to provide user with EditViews and Buttons (or alternatives), so that there is a way for them to enter the information needed for signing up and signing in. The Activities can look something like this:

![image-center](/assets/images/oldblog/firebase3-loginscreen.png){: .align-center}

![image-center](/assets/images/oldblog/firebase3-signupscreen.png){: .align-center}

As you can see in RegisterActivity we have four EditText objects: one for e-mail, one for password, one for phone number and one for company name.
First of all we should check whether user is already signed in (what's the point of signing up then?). To do that we need two class variables:

```JAVA    
    private FirebaseAuth mAuth;
    private FirebaseAuth.AuthStateListener mAuthStateListener;
    (...)
    @Override
    protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_register);
    
      mAuth = FirebaseAuth.getInstance();
      // Create an instance of AuthStateListener, which checks whether user is signed in
      mAuthStateListener = new FirebaseAuth.AuthStateListener() {
        @Override
        public void onAuthStateChanged(@NonNull FirebaseAuth firebaseAuth) {
          FirebaseUser user = firebaseAuth.getCurrentUser();
          if(user != null) {
            // User is signed in. Go to MainActivity (we will discuss it in later posts)
            startActivity(new Intent(getApplicationContext(), MainActivity.class));
          } else {
          // User is signed out.
          }
        }
      }
    }
```

(...)

```JAVA
    // Add AuthStateListener to our main FirebaseAuth object at onStart and remove it at onStop().
    @Override
    public void onStart() {
      super.onStart();
      mAuth.addAuthStateListener(mAuthStateListener);
    }

    @Override
    public void onStop() {
      super.onStop();
      mAuth.removeAuthStateListener(mAuthStateListener);
    }
```

If you're unfamiliar with onStart() and onStop() method, check [this article](https://developer.android.com/guide/components/activities/activity-lifecycle.html) on Android activities lifecycle'.
Basically, if user is already signed in, the MainActivity will be loaded. We will discuss that activity in later posts. For now all we want is to prohibit users from creating an account when they're already have one.

Ok, now to the actual signing up process.
We have SignupButton view. Once it is clicked, assuming that every EditText field is filled, it's going to perform the actual signing process in Firebase.

What's important to remember is that Firebase Authentication is a separate process from saving user's data to Firebase Database. First one requires FirebaseAuth.createUserWithEmailAndPassword(String email, String password) call, and the second requires reference to the database.
Let's add that Button listener in our onCreate() method:

```Java 
    private ProgressDialog mProgressDialog;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
      (...)
      mSignUpButton.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
          registerUser();
        }
      });
    }
```

(...)

```Java
    private void registerUser() {
      // Get Strings from EditTexts
      final String mEmail = mEditTextEmail.getText().toString().trim();
      final String mPhone = mEditTextPhone.getText().toString().trim();
      final String mName = mEditTextName.getText().toString().trim();
      String mPass = mEditTextPassword.getText().toString().trim();
      // Preliminary check for whether EditTexts weren't empty. If empty, show Toast to the user with information.
      if(TextUtils.isEmpty(mEmail)){
        Toast.makeText(this,getString(R.string.login_hint_email), Toast.LENGTH_LONG).show();
        return;
      }
      if(TextUtils.isEmpty(mPass)){
        Toast.makeText(this,getString(R.string.login_hint_password),Toast.LENGTH_LONG).show();
        return;
      }
      if(TextUtils.isEmpty(mPhone) || mPhone.length()<9){
        Toast.makeText(this,getString(R.string.hint_phone),Toast.LENGTH_SHORT).show();
        return;
      }
      if(TextUtils.isEmpty(mName)){
        Toast.makeText(this,getString(R.string.hint_name),Toast.LENGTH_SHORT).show();
        return;
      }

      // If the email and password are not empty
      // display a progress dialog to inform the user that his information is being saved in Firebase database
      mProgressDialog.setMessage(getString(R.string.login_progressbar_register));
      mProgressDialog.show();

      //Create a new User in FirebaseAuth
      mAuth.createUserWithEmailAndPassword(mEmail, mPass)
        .addOnCompleteListener(new OnCompleteListener() {
        @Override
        public void onComplete(@NonNull Task task) {
          if(task.isSuccessful()) {
            // Grab a reference to the database (at this point we already 
            // have a user registered in Firebase Authentication part.
            final DatabaseReference mDatabaseReference = FirebaseDatabase.getInstance().getReference();

            // At the ''users'' node create new child with our new users' 
            // UID (unique ID) and fill it with data provided by user in EditTexts
            mDatabaseReference.child("users").child(mAuth.getCurrentUser().getUid())
              .addListenerForSingleValueEvent(new ValueEventListener() {
                @Override
                public void onDataChange(DataSnapshot dataSnapshot) {
                  User mUser = new User();
                  mUser.setName(mName);
                  mUser.setEmail(mEmail);
                  mUser.setRole("customer");
                  mUser.setPhone(mPhone);
                  mUser.setUid(mAuth.getCurrentUser().getUid());
                  mDatabaseReference.child("users").child(
                  mAuth.getCurrentUser().getUid())
                  .setValue(mUser);
                }
               @Override
               public void onCancelled(DatabaseError databaseError) {
               }
             });
             // Once finished start MainActivity (signing up process is done.
             // We have a new User in our Authentication part of Firebase and his details in Firebase Database.
             finish();
             startActivity(new Intent(getApplicationContext(), MainActivity.class));
          }
          // Dismiss the progressDialog.
          mProgressDialog.dismiss();
          // If for some reason signing up couldn't finish with success, catch 
          // exception and inform the user (i.e. when user's password was to weak).
          if(!task.isSuccessful()) {
            try {
              throw task.getException();
            } catch(FirebaseAuthWeakPasswordException e) {
              mEditTextPassword.setError(getString(R.string.login_weak_password));
              mEditTextPassword.requestFocus();
            } catch(Exception e) {
              if(e.toString().contains("WEAK_PASSWORD")) {
              mEditTextPassword.setError(getString(R.string.login_weak_password));
              mEditTextPassword.requestFocus();
              }
            }
          }
        }
      })
      .addOnFailureListener(new OnFailureListener() {
        @Override
        public void onFailure(@NonNull Exception e) {
          if(e instanceof FirebaseAuthWeakPasswordException) {
            mEditTextPassword.setError(getString(R.string.login_weak_password));
            mEditTextPassword.requestFocus();
          } else if(e instanceof FirebaseAuthInvalidCredentialsException) {
            mEditTextEmail.setError(getString(R.string.login_bad_email));
            mEditTextEmail.requestFocus();
          } else if(e instanceof FirebaseAuthUserCollisionException) {
            mEditTextEmail.setError(getString(R.string.login_user_exists));
            mEditTextEmail.requestFocus();
          }
          mProgressDialog.dismiss();
        }
      });
    }
```

FYI, User object is defined as followed:

```Java
    public class User {

    private String mEmail;
    private String mUid;
    private String mName;
    private String mPhone;
    private String mRole;
    private String mInstanceId;

    public User() {
    }

    public User(String email) {
      this.mEmail = email;
    }

    public User(String email, String role) {
      this.mEmail = email;
      this.mRole = role;
    }

    public String getEmail() {
      return mEmail;
    }

    public String getRole() {
      return mRole;
    }

    public void setEmail(String email) {
      mEmail = email;
    }

    public void setRole(String role) {
      mRole = role;
    }

    public String getName() {
      return mName;
    }

    public void setName(String name) {
      mName = name;
    }

    public String getPhone() {
      return mPhone;
    }

    public void setPhone(String phone) {
      mPhone = phone;
    }
```

As you can see, some of the fields get filled with data provided by user in EditTexts. The mRole String gets a default value of 'customer' in RegisterActivity. If we're couriers creating a new account we can change our role in Firebase Database as follows:

![image-center](/assets/images/oldblog/firebase_database_role_changing.png){: .align-center}

We will use User role value later when setting notification functions in Firebase Cloud Functions.

Ok, so now signing in activity. In here we have one Button (main signing in Button), two TextViews with onClickListeners (if user forgot his password or if he doesn't yet have an account) and two EditTexts (login and password input).

```Java
    public class LoginActivity extends AppCompatActivity {

    private EditText mEditTextEmail;
    private EditText mEditTextPass;
    private String mEmail;
    private String mPass;
    private FirebaseAuth mAuth;
    private ProgressDialog mProgressDialog;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_login);

      mAuth = FirebaseAuth.getInstance();
      // If the objects getcurrentuser method is not null
      // then user is already signed in

      // Quicker and worse way of checking whether there is a user signed in.
      // If Activity will be paused and them resumed with onResume(), then there will be no
      // onCreate() call, and there will be no check. So if user is not signed in at that moment
      // Houston we have a problem. Solution from RegisterActivity is better :).
      if(mAuth.getCurrentUser() != null) {
        finish();
        startActivity(new Intent(getApplicationContext(), MainActivity.class));
      }

      mEditTextEmail = (EditText) findViewById(R.id.login_edittext_email);
      mEditTextPass = (EditText) findViewById(R.id.login_edittext_pass);
      Button mButtonLogin = (Button) findViewById(R.id.login_button_login);
      TextView mTextViewNotYet = (TextView) findViewById(R.id.login_textview_not_yet);
      TextView mTextViewForgot = (TextView) findViewById(R.id.login_textview_forgot_pass);
      mProgressDialog = new ProgressDialog(this);

      // Signing in Button's listener.
      mButtonLogin.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
          userLogin();
        }
      });
      // If user doesn't have an account, go to RegisterActivity
      mTextViewNotYet.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
          finish();
          startActivity(new Intent(LoginActivity.this, RegisterActivity.class));
        }
      });
      // If user forgot his password, send a default e-mail with reset instructions.
      mTextViewForgot.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
          mAuth.sendPasswordResetEmail(mEmail).addOnCompleteListener(new OnCompleteListener() {
            @Override
            public void onComplete(@NonNull Task task) {
              // Show Toast based on whether password reset email was sent or not
              if(task.isSuccessful()) {
                Toast toast = Toast.makeText(LoginActivity.this, getString(
                  R.string.login_forgot_pass_toast), Toast.LENGTH_SHORT);
                toast.setGravity(Gravity.CENTER, 0, 0);
                toast.show();
              } else {
                Toast toast = Toast.makeText(LoginActivity.this, getString(
                  R.string.login_forgot_pass_toast_unsucc), Toast.LENGTH_SHORT);
                toast.setGravity(Gravity.CENTER, 0, 0);
                toast.show();
              }
            }
          });
        }
      });
    }

    private void userLogin() {
      mEmail = mEditTextEmail.getText().toString().trim();
      String mPass = mEditTextPass.getText().toString().trim();
      if(TextUtils.isEmpty(mEmail)){
        Toast.makeText(this,getString(R.string.login_hint_email), Toast.LENGTH_LONG).show();
        return;
      }
      if(TextUtils.isEmpty(mPass)){
        Toast.makeText(this,getString(R.string.login_hint_password),Toast.LENGTH_LONG).show();
        return;
      }
      mProgressDialog.setMessage(getString(R.string.login_progressbar_login));
      mProgressDialog.show();
      // Signing the user in
      mAuth.signInWithEmailAndPassword(mEmail, mPass)
        .addOnCompleteListener(LoginActivity.this, new OnCompleteListener() {
          @Override
          public void onComplete(@NonNull Task task) {
            mProgressDialog.dismiss();
            if(task.isSuccessful()) {
              finish();
              startActivity(new Intent(LoginActivity.this, MainActivity.class));
            }
          }
        });
      }
    }
```

And that's it! Check the full source code on GitHub for fully working example.
