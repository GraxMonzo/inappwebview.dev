---
sidebar_position: 14
date: 2022-10-12 12:00:00
---

# Migration Guide from 5.x.x

## Requirements

Version `6.x.x` now requires minimum Flutter version to be `3.0.0`,
Android `minSdkVersion` to be `19` (`android/app/build.gradle`),
and the minimum iOS version to be `9.0` (`ios/Podfile`) with XCode version `>= 14`.

## `Uri` type to `WebUri` type

The usage of `Uri` type has been replaced with the new `WebUri` type.

`WebUri` is a class that implements the `Uri` interface to maintain also the raw string value used by `Uri.parse`.

This class is used because some strings coming from the native platform side
are not parsed correctly or could lose letter case information,
so `rawValue` can be used as a fallback value.

Basic examples:
```dart
// InAppWebView example
InAppWebView(
  initialUrlRequest:
    URLRequest(url: WebUri('https://flutter.dev'))
)

// example of letter case difference
final uri = WebUri('scheme://customDomain');
print(uri.rawValue); // scheme://customDomain
print(uri.isValidUri); // true
print(uri.uriValue.toString()); // scheme://customdomain
print(uri.toString()); // scheme://customdomain

uri.forceToStringRawValue = true;
print(uri.toString()); // scheme://customDomain

// example of a not valid URI
// Uncaught Error: FormatException: Invalid port (at character 14)
final invalidUri = WebUri('intent://not:valid_uri');
print(invalidUri.rawValue); // intent://not:valid_uri
print(invalidUri.isValidUri); // false
print(invalidUri.uriValue.toString()); // ''
print(invalidUri.toString()); // intent://not:valid_uri
```

Check [WebUri](/docs/web-uri) for more details.

## Deprecated Option classes

Option classes, such as `InAppWebViewGroupOptions`, `InAppBrowserClassOptions` or
`ChromeSafariBrowserClassOptions`, have been marked as `deprecated` in favor of the new ones
without using the `android` and `ios` properties for platform-specific options.

Now, options/settings are all in the same class without platform distinction.
To know if a specific option/setting is available for a specific platform, check the code docs directly.

For example, the old:
```dart
InAppWebViewGroupOptions(
  crossPlatform: InAppWebViewOptions(
      mediaPlaybackRequiresUserGesture: false
  ),
  android: AndroidInAppWebViewOptions(
    useHybridComposition: true,
  ),
  ios: IOSInAppWebViewOptions(
    allowsInlineMediaPlayback: true,
  )
);
```

would now be:
```dart
InAppWebViewSettings(
  mediaPlaybackRequiresUserGesture: false,
  useHybridComposition: true,
  allowsInlineMediaPlayback: true,
);
```

## Deprecated classes/properties/methods

All the classes/properties/methods that starts with the platform name prefix,
such as `AndroidInAppWebViewController`, `AndroidServiceWorkerController`, `IOSCookieManager`, `androidOnPermissionRequest`, `iosOnNavigationResponse`, etc...
have been marked as `deprecated` in favor of the corresponding class/property/method without the platform name prefix.

For example `AndroidInAppWebViewController.setWebContentsDebuggingEnabled(true);`
would now be `InAppWebViewController.setWebContentsDebuggingEnabled(true);`,
`androidOnPermissionRequest` event would now be `onPermissionRequest` event, etc...

Check the code docs to know which class/property/method you should use now.

## Changelog

Check the [CHANGELOG](https://github.com/pichillilorenzo/flutter_inappwebview/blob/master/CHANGELOG.md) file to know all the BREAKING CHANGES.
