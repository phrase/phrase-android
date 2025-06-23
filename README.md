# Phrase Over the Air SDK for Android

Publish your translations faster and simpler than ever before. Stop waiting for the next deployment and start publishing all your translations in real-time directly in Phrase.

For more details on how OTA works head over to the Phrase Help Center: https://support.phrase.com/hc/en-us/articles/5804059067804-Over-the-Air-Strings

## Instructions

With the SDK, the app regularly checks for updated translations and downloads them in the background. Translations are fetched when `updateTranslations` is called, which should usually happen in `onCreate`.


### Requirements
- The SDK requires at least appcompat version 1.2.0. If using an older version of appcompat, consider using SDK version [2.1.3](https://github.com/phrase/phrase-android/releases/tag/2.1.3)
- The library depends on AndroidX to support backward compatible UI elements such as the toolbar.

### Include the SDK
Add a new repository to the root `build.gradle`:

```
allprojects {
    repositories {
        ...
        maven { url "https://maven.download.phrase.com" }
    }
}
```

Add the library as a dependency:

```
dependencies {
    implementation "com.phrase.android:ota-sdk:3.10.2"
    ...
}
```

### Jetpack Compose support
To enable Jetpack Compose support for OTA translations, follow these steps:
1. Add the library `implementation "com.phrase.android:ota-sdk-compose:3.10.2"` to the root build.gradle.
2. Wrap the Jetpack Compose code in `Phrase { ... }`.

### Configuration
Initialize the SDK in the application class and add the distribution ID and environment secret. Classes inheriting from Application should overwrite `attachBaseContext` to enable translations outside of the activity context:

```java
public class MainApplication extends Application {
  @Override
  public void onCreate() {
      super.onCreate();
      Phrase.setup(this, "DISTRIBUTION_ID", "ENVIRONMENT_TOKEN");
      Phrase.updateTranslations();
  }

  @Override
  protected void attachBaseContext(Context newBase) {
    super.attachBaseContext(Phrase.wrapApplicationContext(newBase));
  }
}
```

Iinject the SDK in each activity, e.g. by creating a base activity which all other activities inherit from:

```java
public class BaseActivity extends AppCompatActivity {
  @NonNull
  @Override
  public AppCompatDelegate getDelegate() {
    return Phrase.getDelegate(this, super.getDelegate());
  }
}
```

Translations can be used as usual in layouts:
```xml
<TextView android:text="@string/translation_key" />
```
And inside code:
```java
TextView text = (TextView) findViewById(R.id.text_id);
text.setText(R.string.translation_key);
```

Some libraries do not support automatically unwrapping the context and expect a specific class. In this case context wrapping in Jetpack Compose components can be disabled with:

```
Phrase(contextWrapping = false) {
    Text( text = phraseString(R.string.test) )
}
```

### Configurations for log levels:

Java
```java
PhraseLog.setLogLevel(Severity.Debug);
```

Kotlin
```kotlin
PhraseLog.logLevel = Severity.Verbose
```

Other supported logging values: `None`, `Error`, `Warning`, `Info`, `Debug`, `Verbose`

### Custom app version
The SDK uses the app version by default to return a release which matches the release constraints for the min and max version. The app version must use semantic versioning otherwise no translation update is returned. In case the app does not use semantic versioning it is possible to manually override the used app version.

Example:
`Phrase.setAppVersion("3.2.4");`
The version must be set before calling `updateTranslations()`.

### Set timeout
The default timeout for translation downloads is set to 10s. The default can be changed with:
```
// Timeout in milliseconds
Phrase.setTimeout(10000);
```

### Update callback
If the handling of successful translation updates is required, attach a callback handler:

```java
Phrase.updateTranslations(new TranslationsSyncCallback() {
    @Override
    public void onSuccess(boolean translationsChanged) {
    }

    @Override
    public void onFailure() {
    }
});
```

Translation updates can also be triggered manually. Newly fetched translations are displayed upon the next application launch.

To make the latest translations immediately available, use the method `Phrase.applyPendingUpdate()`. This can be combined with listening for translation updates:

```java
Phrase.updateTranslations(new TranslationsSyncCallback() {
    @Override
    public void onSuccess(boolean translationsChanged) {
      if(translationsChanged) {
         Phrase.applyPendingUpdate()
        // Custom logic to refresh UI
      }
    }

    @Override
    public void onFailure() {
    }
});
```

The UI does not display translations automatically and must be recreated.

### Configure US data center
Phrase US data center is also supported. The US data center can be configured by calling:

```
Phrase.setHost("https://ota.us.phrase.com/")
```

### Fallback
In case it is not possible to reach Phrase due to a missing network connection of the client or a service interruption, the SDK uses the bundled translations from the resource file. The regular updating of the bundled translations in the app is recommended. The SDK also caches translations locally on the device. If such a cache exists, it is used until the next translation update.

The SDK uses the most recent release for the translations. In case the versionName for the app is set, the most recent release that satisfies the version restrictions will be used.

### Add a new language
Creating the new language in Phrase and create a new release. The SDK fetches the language when this is the device language of a user. Regularly adding a new strings.xml for new languages files when releasing a new app version is recommended or users will only see the fallback translations determined by Android at the first start of the app.

### Auditing
The SDK is closed source and can not be viewed or modified. If it is an organization requirement, audits can be provided. Contact us for more details if required.

### Custom View Support
Custom views can be translated using styled attributes. Since TypedArray does not allow overwriting the resources, slight changes in the custom view are required:

### Kotlin example

Before:

```kotlin
context.obtainStyledAttributes(attrs, R.styleable.CustomView, defStyleAttr, 0).use {
    text = it.getText(R.styleable.CustomView_android_text)
}
```
After:

```kotlin
context.obtainStyledAttributes(attrs, R.styleable.CustomView, defStyleAttr, 0).use {
    text = it.getTextWithPhrase(R.styleable.CustomView_android_text)
}
```

### Java example

Before:

```java
final TypedArray ta = context.obtainStyledAttributes(attrs, R.styleable.CustomView, defStyleAttr, 0);
try {
    setText(ta.getText(R.styleable.CustomView_android_text));
} finally {
    ta.recycle();
}
```
After:

```java
final TypedArray ta = context.obtainStyledAttributes(attrs, R.styleable.CustomView, defStyleAttr, 0);
try {
    setText(PhraseTypedArray.getTextWithPhrase(ta, R.styleable.CustomView_android_text));
} finally {
    ta.recycle();
}
```

Example [app](https://github.com/phrase/android-sdk-example)

### Troubleshooting
**If translations are not being updated**
- Ensure distribution id and environment secret are correct.
- Ensure a release was created on for the current app version.
- Reload the ViewController to make changes appear immediately.

If the wrong version of a translation is being used, ensure a release with the latest translations and the current app version is available and the `versionName` for the app set and are using the `<major>.<minor>.<point>`. format.


## Known limitations

- Inflated menus and preferences are not supported yet
- Some libraries that do not support automatic context unwrapping can cause issues. For example `Hilt` requires to disable automatic context wrapping in Compose components https://github.com/phrase/phrase-android/releases/tag/3.5.0
