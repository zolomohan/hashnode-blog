## Google OAuth using Firebase in React Native

OAuth is an open standard for access delegation, commonly used as a way for Internet users to grant websites or applications to access information on other websites without [giving them the passwords](https://www.coursehero.com/file/56229675/oaithtxt).

In this tutorial, we will learn how to authenticate users with their Google accounts using the authentication module in Firebase in a Non-Expo React Native application.

To learn more about Firebase, refer [here](https://en.wikipedia.org/wiki/Firebase).

### Prerequisites

We will not cover the basics of React and React Native in this tutorial. If you are not comfortable with the basics, we highly recommended going over this [tutorial](https://reactnative.dev/docs/tutorial) before you continue further.

### Overview

1. [Development environment](#development-environment)
2. [Cloning the starter code](#cloning-the-starter-code)
3. [Setting up the Firebase project](#setting-up-the-firebase-project)
4. [Setting up Firebase Authentication](#setting-up-firebase-authentication)
5. [Sign-in](#sign-in)
6. [Display authenticated screen](#display-authenticated-screen)
7. [Sign out](#sign-out)
8. [Recap](#lets-recap)

### Development environment

> **IMPORTANT** - We will not be using [Expo](https://expo.io/) in our project.

You can follow [this documentation](https://reactnative.dev/docs/environment-setup) to set up the environment and create a new React app.

Make sure you're following the React Native CLI Quickstart and not the Expo CLI Quickstart.

![env_setup.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617216225646/qsG3Ev2Zq.png)

### Cloning the starter code
To focus more on the authentication module, You can clone the starter code from this [repository](https://github.com/zolomohan/react-native-firebase-google-auth-starter) on GitHub. 

Follow the Repository's `README` for instructions.

For the final code, refer to this [GitHub Repository](https://github.com/zolomohan/react-native-firebase-google-auth).

Folder Structure of the repository:

![folder_structure.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617216245021/kxJjY3cE7.png)

I've set up 2 screens in the `screens/` directory:

1. `Authentication.js`: Screen with a Google Sign-in button to initiate the sign-in process.

2. `Authenticated.js`: The user can see this screen only if he is logged in.

![screens.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617216732940/851xvDc5J.jpeg)

### Setting up the Firebase project

Head to the [Firebase console](https://console.firebase.google.com/u/0/), sign in to your account, and then create a new project.

![firebase_new.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617216262351/RlcH2Ohqk.png)

You'll see the dashboard after you create a new project.

![new_dashboard.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617216273425/f2r8Tcu48.png)

Click on the Android icon to add an Android app to your Firebase project.

![register_app.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617216284783/nLo3nrg30.png)

You'll need the package name of the application to register the application.

You can find the package name in the `AndroidManifest.xml` file which is located in `android/app/src/main/`.

![package_name.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617216297768/ypMyJKDat.png)

You will also need the `Debug signing certificate SHA-1`. You can get that by running the following command in the project directory.

```bash
cd android && ./gradlew signingReport
```

This will generate the signing certificate of the application. 

You will get an output something similar to this:

```bash
Task :app:signingReport
Variant: debugUnitTest
Config: debug
Store: C:\Users\Mohan\.android\debug.keystore
Alias: AndroidDebugKey
MD5: 5F:BB:9E:98:5E:E7:E6:29:19:28:61:4F:42:B9:74:AB
SHA1: 9E:61:75:0E:5C:F4:EB:B4:EB:9D:B3:13:5F:50:D6:AB:2E:4E:12:0D
SHA-256: 6C:BB:49:66:18:B9:7F:74:49:B5:56:D0:24:43:6A:1B:41:91:97:A3:2E:7C:4A:6E:59:40:8F:5C:74:6F:CC:93
Valid until: Friday, December 23, 2050
```

> Make sure you are copying the `SHA1` from `Task :app:signingReport` and not from any other `Task`.

Copy the `SHA1` value and paste it into the Firebase console.

Now, proceeding to the next step, you will have to download the `google-services.json` file. You should place this file in the `android/app` directory.

This file contains configurations that'll enable your application to access firebase services.

![download_services.json.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617216312307/SSp7EIWO1.png)

After adding the file, proceed to the next step. It will ask you to add some configurations to the `build.gradle` files.

First, add the `google-services` plugin as a dependency inside of your `android/build.gradle` file:

```gradle
buildscript {
  dependencies {
    // ... other dependencies
    classpath 'com.google.gms:google-services:4.3.3'
  }
}
```

Then, execute the plugin by adding the following to your `android/app/build.gradle` file:

```Gradle
apply plugin: 'com.android.application'
apply plugin: 'com.google.gms.google-services'
```

You need to perform some additional steps to configure `Firebase` for `iOS`. Follow [this documentation](https://rnfirebase.io/#3-ios-setup) to set it up.

Finally, we should install the `@react-native-firebase/app` package in our app to complete the set up for Firebase.

```bash
npm install @react-native-firebase/app
```

### Setting up Firebase Authentication

Head over to the `Authentication` section in the dashboard and click on the `Get Started` button. This will enable the authentication module in your project.

![auth_get_starterd.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617216334171/PoZzOeszF.png)

Next, you should enable phone authentication in the sign-in methods. Once you've enabled it, press `Save`.

![google-enable.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617216362986/TINqCgBsK.png)

Now, let's head to the application and install the auth module.

Let's install the `@react-native-firebase/auth` package in our app.

```bash
npm install @react-native-firebase/auth
```

Let's declare the dependency for the authentication module in the `android/app/build.gradle` file using the [Firebase Android BoM](https://firebase.google.com/docs/android/learn-more?authuser=0#bom)

```gradle
dependencies {
    // Add these lines
    implementation platform('com.google.firebase:firebase-bom:26.3.0')
    implementation 'com.google.firebase:firebase-auth'
}
```

With this, the firebase authentication module is set up in our application.

### Sign-in

The [`google-signin`](https://github.com/react-native-google-signin/google-signin) library is a wrapper around the official Google sign-in library.

We should use this library to create a credential, and then sign-in with Firebase.

First, we must initialize the Google SDK using your `webClientId` which can be found in the `google-services.json` file in `android/app` as the `client/oauth_client/client_id` property.

![oauth_id.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617216415935/m_2W8m7A4.png)

In `App.js`, let's import the `google-signin` library and the Firebase `auth` module.

```JSX
import auth from '@react-native-firebase/auth';
import { GoogleSignin } from '@react-native-community/google-signin';
```

You should call the `GoogleSignin.configure` method with the `webClientId` to initialize the SDK. You should do this outside the `App()` function.

```JSX
GoogleSignin.configure({
  webClientId:
    '260759292128-4h94uja4bu3ad9ci5qqagubi6k1m0jfv.apps.googleusercontent.com',
});
```

Now, that we have initialized the Google SDK, let's work on authenticating the user.

In the starter code, I've set up a function called `onGoogleButtonPress` in the `App.js` file. This function is passed down to the `Authentication` screen as a `prop`, and then, it is set as the `onPress` property of the Google sign-in button. 

Thus, this function in the `App.js` file will be called when the Google sign-in button is pressed by the user.

Let's write the code to sign-in the user in the `onGoogleButtonPress` function.

```JSX
async function onGoogleButtonPress() {
  // Sign-in Process here
}
```

First, we should get the user's `idToken` from Google using the `GoogleSignin.signIn()` method. It's an asynchronous function, so let's use the `await` keyword to wait for the promise to get resolved.

```JSX
// Get the users ID token
const { idToken } = await GoogleSignin.signIn();
```

Now, we should create a Google credential using the `idToken`.

```JSX
const googleCredential = auth.GoogleAuthProvider.credential(idToken);
```

With the Google credential that we have created for the user, we should use the `signInWithCredential` method from the Firebase auth module to sign-in the user into the app.

```JSX
return auth().signInWithCredential(googleCredential);
```

This is the complete code for the `onGoogleButtonPress` function.

```JSX
async function onGoogleButtonPress() {
  const { idToken } = await GoogleSignin.signIn();
  const googleCredential = auth.GoogleAuthProvider.credential(idToken);
  return auth().signInWithCredential(googleCredential);
}
```

### Display authenticated screen

The `onAuthStateChanged` event will be triggered whenever the authentication state of the user changes inside the application.

You can set an event handler for this listener. This handler will receive the `user` object. If the `user` object is `null`, it means the user is signed-out, otherwise, they are signed-in.

You can access the current authenticated user's details using `auth().currentUser` from anywhere in the application. The user object will contain the `displayName`, `email`, and `photoURL` which were copied from Google to Firebase.

To learn more about the user object, refer to this [documentation](https://rnfirebase.io/reference/auth/user).

Let's create a state to track whether the user is authenticated or not. We should set the default value to `false`.

```JSX
const [authenticated, setAutheticated] = useState(false);
```

Let's set the `authenticated` state to `true` if the `user` object is not `null` in the `onAuthStateChanged` handler.

```JSX
auth().onAuthStateChanged((user) => {
  if(user) {
    setAutheticated(true);
  }
})
```

If the user is authenticated, we should display the `Authenticated` screen component.

```JSX
if (authenticated) {
  return <Authenticated />;
}

return <Authentication onGoogleButtonPress={onGoogleButtonPress} />;
```

I'm using `auth().currentUser` in the `Authenticated` screen to display the email address, name, and the user's profile picture.

```JSX
const user = auth().currentUser;
return (
  <View style={styles.screen}>
    <Text style={styles.title}>You're Logged In</Text>
    <Image source={{ uri: user?.photoURL }} style={styles.image} />
    <Text style={styles.text}>{user?.displayName}</Text>
    <Text style={styles.text}>{user?.email}</Text>
    <View style={{ marginTop: 30 }}>
      <Button title="Signout" onPress={() => auth().signOut()} />
    </View>
  </View>
);
```

![auth_screen.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1617216461125/wZi1-TDOw.gif)

### Sign out

We should use the `signOut` method in the auth module to sign a user out from the application.

Let's import the `auth` module in *Authenticated.js*.

```JSX
import auth from '@react-native-firebase/auth';
```

Let's call the `signOut` method when the user presses the signout button.

```JSX
<Button title="Signout" onPress={() => auth().signOut()} />
```

Now, when the user presses the button, the auth module will sign out the user from the application. This will trigger the `onAuthStateChanged` listener. The handler will receive `null` instead of the `user` object.

Thus, we should set the authenticated state to `false` if we receive `null`.

```JSX
auth().onAuthStateChanged((user) => {
  if(user) {
    setAuthenticated(true);
  } else {
    setAuthenticated(false);
  }
})
```

![signout.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1617216483909/kBaPp-hyr.gif)

### Let's recap
1. We set up our development environment and created a React Native app.

2. We cloned the starter code.

3. We created a Firebase project and enabled Google authentication in our project.

4. We installed the required packages and we added the dependencies to the `build.gradle` files.

5. We configured the Google SDK and signed in the user when the user presses the Google sign-in button.

6. We created a state to track the authentication state of the user and used the `onAuthStateChanged` handler to update the state.

7. We displayed the `Authenticated` screen when the user has been authenticated.

8. We used the `auth` module to sign the user out from the application from the `Authenticated` screen.

Congratulations, :partying_face: You did it.

Thanks for reading!

Happy coding.