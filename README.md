# firebase-frashlytics
Google Firebase (Firebase Crashlytics)
## How To Install

### Add the lines below to `Packages/manifest.json`

for version `10.7.0`
```csharp
"com.google.firebase.crashlytics": "https://github.com/google-for-unity/firebase-crashlytics.git#10.7.0",
```

dependency `external-dependency-manager-1.2.175`, `firebase-app-core-10.7.0`
```csharp
"com.google.firebase.core": "https://github.com/google-for-unity/firebase-app-core.git#10.7.0",
"com.google.external-dependency-manager": "https://github.com/google-for-unity/external-dependency-manager-for-unity.git#1.2.175",
```

## Maybe you need
<details><summary>Template code here</summary>

```

 using Firebase;
  using Firebase.Crashlytics;
  using Firebase.Extensions;
  using System;
  using UnityEngine;

  // Handler for UI buttons on the scene.  Also performs some
  // necessary setup (initializing the firebase app, etc) on
  // startup.
  public class FirebaseCrashlyticManager : MonoBehaviour {
    public GUISkin fb_GUISkin;
    private Vector2 controlsScrollViewVector = Vector2.zero;
    private Vector2 scrollViewVector = Vector2.zero;
    bool UIEnabled = true;
    private string logText = "";
    const int kMaxLogSize = 16382;
    DependencyStatus dependencyStatus = DependencyStatus.UnavailableOther;
    protected bool firebaseInitialized = false;

    // When the app starts, check to make sure that we have
    // the required dependencies to use Firebase, and if not,
    // add them if possible.
    public virtual void Start() {
      FirebaseApp.CheckAndFixDependenciesAsync().ContinueWithOnMainThread(task => {
        dependencyStatus = task.Result;
        if (dependencyStatus == DependencyStatus.Available) {
          InitializeFirebase();
        } else {
          Debug.LogError(
            "Could not resolve all Firebase dependencies: " + dependencyStatus);
        }
      });
    }

    // Exit if escape (or back, on mobile) is pressed.
    public virtual void Update() {
      if (Input.GetKeyDown(KeyCode.Escape)) {
          Application.Quit();
      }
    }

    // Handle initialization of the necessary firebase modules:
    void InitializeFirebase() {
      var app = FirebaseApp.DefaultInstance;
      firebaseInitialized = true;
    }

    // End our session when the program exits.
    void OnDestroy() { }

    // Causes an error that will crash the app at the platform level (Android or iOS)
    public void ThrowUncaughtException() {
      DebugLog("Causing a platform crash.");
      throw new InvalidOperationException("Uncaught exception created from UI.");
    }

    // Log a caught exception.
    public void LogCaughtException() {
      DebugLog("Catching an logging an exception.");
      try
      {
          throw new InvalidOperationException("This exception should be caught");
      } catch (Exception ex) {
          Crashlytics.LogException(ex);
      }
    }

    // Write to the Crashlytics session log
    public void WriteCustomLog(String s) {
      DebugLog("Logging message to Crashlytics session: " + s);
      Crashlytics.Log(s);
    }

    // Add custom key / value pair to Crashlytics session
    public void SetCustomKey(String key, String value) {
      DebugLog("Setting Crashlytics Custom Key: <" + key + " / " + value + ">");
      Crashlytics.SetCustomKey(key, value);
    }

    // Set User Identifier for this Crashlytics session 
    public void SetUserID(String id)
    {
      DebugLog("Setting Crashlytics user identifier: " + id);
      Crashlytics.SetUserId(id);
    }

    // Output text to the debug log text field, as well as the console.
    public void DebugLog(string s) {
      print(s);
      logText += s + "\n";

      while (logText.Length > kMaxLogSize) {
        int index = logText.IndexOf("\n");
        logText = logText.Substring(index + 1);
      }

      scrollViewVector.y = int.MaxValue;
    }

    void DisableUI() {
      UIEnabled = false;
    }

    void EnableUI() {
      UIEnabled = true;
    }

    // Render the log output in a scroll view.
    void GUIDisplayLog() {
      scrollViewVector = GUILayout.BeginScrollView(scrollViewVector);
      GUILayout.Label(logText);
      GUILayout.EndScrollView();
    }

    // Render the buttons and other controls.
    void GUIDisplayControls() {
      if (UIEnabled) {
        controlsScrollViewVector =
            GUILayout.BeginScrollView(controlsScrollViewVector);
        GUILayout.BeginVertical();

        if (GUILayout.Button("Throw Exception")) {
          ThrowUncaughtException();
        }

        if (GUILayout.Button("Log Caught Exception")) {
          LogCaughtException();
        }

        if (GUILayout.Button("Log message")) {
          WriteCustomLog("This is a log message.");
        }

        if (GUILayout.Button("Set Custom Keys")) {
          SetCustomKey("MyKey", "TheValue");
        }

        if (GUILayout.Button("Set User ID")) {
          SetUserID("SomeUserId");
        }

        if (GUILayout.Button("Perform All Actions")) {
          DebugLog("All actions will be performed. To view issues:");
          DebugLog(" 1. Force close app");
          DebugLog(" 2. Relaunch app");
          DebugLog(" 3. Visit Firebase Crashlytics console.");
          DebugLog("    Add a time filter to more easily identify the errors if needed.\n");

          WriteCustomLog("This is a log message.");
          SetCustomKey("MyKey", "TheValue");
          SetUserID("SomeUserId");
          LogCaughtException();
          ThrowUncaughtException();
        }

        GUILayout.EndVertical();
        GUILayout.EndScrollView();
      }
    }


    // Render the GUI:
    void OnGUI() {
      GUI.skin = fb_GUISkin;
      if (dependencyStatus != DependencyStatus.Available) {
          GUILayout.Label("One or more Firebase dependencies are not present.");
          GUILayout.Label("Current dependency status: " + dependencyStatus.ToString());
          return;
      }
      Rect logArea, controlArea;

      if (Screen.width < Screen.height) {
          // Portrait mode
          controlArea = new Rect(0.0f, 50.0f, Screen.width, Screen.height * 0.5f);
          logArea = new Rect(0.0f, Screen.height * 0.5f, Screen.width, Screen.height * 0.5f);
      } else {
          // Landscape mode
          controlArea = new Rect(50.0f, 0.0f, Screen.width * 0.5f, Screen.height);
          logArea = new Rect(Screen.width * 0.5f, 0.0f, Screen.width * 0.5f, Screen.height);
      }

      GUILayout.BeginArea(logArea);
      GUIDisplayLog();
      GUILayout.EndArea();

      GUILayout.BeginArea(controlArea);
      GUIDisplayControls();
      GUILayout.EndArea();
    }
  }

```
