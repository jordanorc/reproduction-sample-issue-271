# Sample to reproduce issue #271

For more information, see: https://github.com/apache/cordova-plugin-media/issues/271

## Reproducing

```
$ git clone https://github.com/jordanorc/reproduction-sample-issue-271
$ npm i
$ cordova platform add android
$ cordova run android
```

In logcat, the message will appear:

```
java.io.FileNotFoundException: /storage/emulated/0/tmprecording-1605557271628.3gp: open failed: EACCES (Permission denied)
```

## More information

### Android 10 related problem (API level 29)

The problem seems to be related to Android 10.

According to the [Android documentation](https://developer.android.com/training/data-storage/files/external-scoped).

> Apps targeting Android 10 (API level 29) and higher are given scoped access into an external storage device, or scoped storage, by default. Such apps can see only their app-specific directory

It only happen when using `cordova-android 9`, because it set the `defaultBuildToolsVersion` parameter to 29.

### Partial solution

Adding `<application android:requestLegacyExternalStorage="true" />` to `<edit-config>` in the config.xml file seems to solve the problem.

Looking the code, the problem is in [AudioPlayer.java](https://github.com/apache/cordova-plugin-media/blob/master/src/android/AudioPlayer.java#L111), in `generateTempFile()` function.

Keeping the temp file in app directory instead of external storage seems to solve the problem. 

```java
private String generateTempFile() {
    String tempFileName = "/data/data/" + handler.cordova.getActivity().getPackageName() + "/cache/tmprecording-" + System.currentTimeMillis() + ".3gp";
    return tempFileName;
}
```

