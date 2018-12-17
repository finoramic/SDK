# android-sdk
Following are the steps to integrate any android app with Finoramic.

1. Download the SDK from here.

2. To add SDK to your android project follow below steps in android studio\
    File > New > New Module > Import .JAR/.AAR Package > Next > Choose path to .AAR file > Finish
    
3. Add finoramic repository and dependencies to your app-level build.gradle.
```
    dependencies {
      implementation project(':finoramic-android-sdk')
      implementation 'com.google.android.gms:play-services-location:16.0.0'
      implementation 'com.google.android.gms:play-services-auth:16.0.1'
      implementation 'com.google.firebase:firebase-auth:16.0.5'
    }
```

4. Build your project now for the SDK to import

5. Add the following variables above the `onCreate` method
```
    private static final String FINORAMIC_CLIENT_ID = "";
    private static final int SIGN_IN_REQUEST_CODE = 1;
    private static final String GOOGLE_CLIENT_ID = "";
    private static final String[] extraScopes = {};
    private static final int SMS_REQUEST_CODE = 2;
```

6. In your sign-in activity’s `onCreate` method, initialize `FinoramicSdk` and add SignIn Button.
```
    protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_main);
      
      // Add the following part
      // Initialize Finoramic SDK
      FinoramicSdk.init(this, FINORAMIC_CLIENT_ID);
      // Add SignIn Button
      Button signInButton = findViewById(R.id.button_login);
      signInButton.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
        signIn();
        }
      });
    }
```

7. In the button’s `onClick`, handle sign-in button taps by creating a sign-in intent with the `getSignInIntent` method and starting the activity with `startActivityForResult`. Pass the activity’s context, google client id and any extra scopes that you may have in the `getSignInIntent` method.
```
    private void signIn() {
      Intent signInIntent = FinoramicSdk.getSignInIntent(this, GOOGLE_CLIENT_ID, extraScopes);
      startActivityForResult(signInIntent, SIGN_IN_REQUEST_CODE);
    }
```
  Starting the intent prompts the user to select a Google account to sign in with. If you requested scopes beyond profile, email and birthday, the user is also prompted to grant access to the requested resources.
    
8. After the user signs in, you can get a `GoogleSignInAccount` object for the user in the activity's `onActivityResult` method. Note that this google account you get is in the String format. You need to convert it to a `GoogleSignInAccount` if you need it.
```
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
      super.onActivityResult(requestCode, resultCode, data);

      // Result returned from launching the intent from FinoramicSdk.getSignInIntent(...)
      if (requestCode == SIGN_IN_REQUEST_CODE) {
        if (resultCode == Activity.RESULT_OK) {
          String gAccount = data.getStringExtra(“account”);
          if (gAccount != null) {
            // Signed in successfully. You can update UI here
          } else {
            // Handle unsuccessful sign-in
          }
        }
      }
    }
```

9. For SMS, ask for the following permissions from the user:
    * READ_SMS
    * FINE_LOCATION
```
    String[] permissions = {Manifest.permission.READ_SMS, Manifest.permission.ACCESS_FINE_LOCATION};
    ActivityCompat.requestPermissions(this, permissions, SMS_REQUEST_CODE);
```

10. After permissions are granted, call the `sendSMS` method from SDK and pass the context into it.
```
    @Override
    public void onRequestPermissionsResult(int requestCode, String permissions[], int[] grantResults) {
        if(requestCode == SMS_REQUEST_CODE){
            if(grantResults.length >0 && grantResults[0] == PackageManager.PERMISSION_GRANTED){
                FinoramicSdk.sendSMS(this);
            } else {
                // Handle if access is denied    
            }
        }
    }

```

11. To fetch the finoramic userId call `getFinoramicUserId` and pass the context into it. This returns the finoramic userId which is a String.
```
    FinoramicSdk.getFinoramicUserId(this);
```

12. To update the client userId call `updateClientUserId` 	and pass clientUserId of String type.
```
    FinoramicSdk.updateClientUserId(clientUserId);
```
