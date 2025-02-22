---
title: "Javascript Injection"
sidebar_position: 1
date: 2021-04-07 13:00:00
---

# Injection

You can inject JavaScript code in different ways:
- [using the evaluateJavascript method](#using-the-evaluatejavascript-method);
- [using the callAsyncJavaScript method](#using-the-callasyncjavascript-method);
- [injecting a JavaScript file from URL](#injecting-a-javascript-file-from-url);
- [injecting JavaScript code from an asset file](#injecting-javascript-code-from-an-asset-file).

:::info
  These methods shouldn't be called in the `onWebViewCreated` or `onLoadStart` events, because, in these events, the WebView is not ready to handle it yet. Instead, you should call these methods, for example, inside the `onLoadStop` event or in any other events where you know the page is ready "enough".
:::

## Using the evaluateJavascript method

To evaluate JavaScript code, you can use the `InAppWebViewController.evaluateJavascript` method. It accepts a String as source code to be evaluated by the WebView and an optional [Content World](/docs/webview/javascript/content-worlds) and returns the result of the evaluation as a `dynamic`type.
The return type depends on the type you returned from the JavaScript side.

For example, using this code:
```dart
onLoadStop: (controller, url) async {
  var result = await controller.evaluateJavascript(source: "1 + 1");
  print(result.runtimeType); // int
  print(result); // 2
},
```
it will print `int` as the `result` variable runtime type with `2` as the value from the evaluation.

Instead, using this code:
```dart
onLoadStop: (controller, url) async {
  var result = await controller.evaluateJavascript(source: "new XMLSerializer().serializeToString(document);");
  print(result.runtimeType); // String
  print(result); // whole web page HTML String
},
```
it will print `String` as the `result` variable runtime type with the whole web page HTML String as the value from the evaluation.

The same for other JSON serializable JavaScript types such as `boolean` (`bool` in Dart) and `object literals` (`Map` in Dart).

## Using the callAsyncJavaScript method

Unlike the `evaluateJavascript` method, the `InAppWebViewController.callAsyncJavaScript` method evaluate and executes the specified source code string as an asynchronous JavaScript function.

The `functionBody` parameter is the JavaScript source code string to use as the function body. This method treats the string as an anonymous JavaScript function body and calls it with the named arguments in the arguments parameter.

The `arguments` parameter is a `Map` of the arguments to pass to the function call. Each key in the `Map` corresponds to the name of an argument in the `functionBody` string, and the value of that key is the value to use during the evaluation of the code.

Also, you can set an optional [Content World](/docs/webview/javascript/content-worlds).

Supported value types can be found in the official Flutter docs: [Platform channel data types support and codecs](https://flutter.dev/docs/development/platform-integration/platform-channels#codec), except for `Uint8List`, `Int32List`, `Int64List`, and `Float64List` that should be converted into a `List`. All items in a `List` or in a `Map` must also be one of the supported types.

The return type is not a simple `dynamic` type as the `evaluateJavascript` method. Instead, it returns an instance of `CallAsyncJavaScriptResult` where `value` is the success value and `error` is the failure value of the async JavaScript function.

Here is a simple example:
```dart
onLoadStop: (controller, url) async {
  final String functionBody = """
var p = new Promise(function (resolve, reject) {
   window.setTimeout(function() {
     if (x >= 0) {
       resolve(x);
     } else {
       reject(y);
     }
   }, 1000);
});
await p;
return p;
""";

  var result = await controller.callAsyncJavaScript(
    functionBody: functionBody,
    arguments: {'x': 49, 'y': 'my error message'});
  print(result?.value.runtimeType); // int
  print(result?.error.runtimeType); // Null
  print(result); // {value: 49, error: null}

  result = await controller.callAsyncJavaScript(
    functionBody: functionBody,
    arguments: {'x': -49, 'y': 'my error message'});
  print(result?.value.runtimeType); // Null
  print(result?.error.runtimeType); // String
  print(result); // {value: null, error: my error message}
},
```

## Injecting a JavaScript file from URL

You can add an external JavaScript file from a URL using the `InAppWebViewController.injectJavascriptFileFromUrl` method. It will append to the `document.body` a `<script>` HTML tag using the specified URL.

It accepts an `Uri` as the URL of the file and a `ScriptHtmlTagAttributes` parameter that represents the html `<script>` attributes to use when adding this external file to the web page document.

Use the `ScriptHtmlTagAttributes.onLoad` and `ScriptHtmlTagAttributes.onError` callbacks to listen when the script has been successfully loaded or if an error occurred while trying to load it.

:::caution
  `ScriptHtmlTagAttributes.onLoad` and `ScriptHtmlTagAttributes.onError` callbacks require `ScriptHtmlTagAttributes.id` to be set.
:::

Here is a simple example that injects the `jQuery` JavaScript library to the current web page:
```dart
onLoadStop: (controller, url) async {
  await controller.injectJavascriptFileFromUrl(
      urlFile: Uri.parse('https://code.jquery.com/jquery-3.3.1.min.js'),
      scriptHtmlTagAttributes: ScriptHtmlTagAttributes(id: 'jquery', onLoad: () {
        print("jQuery loaded and ready to be used!");
      }, onError: () {
        print("jQuery not available! Some error occurred.");
      }));
},
```

## Injecting JavaScript code from an asset file

You can inject a JavaScript file from the Flutter assets folder using the `InAppWebViewController.injectJavascriptFileFromAsset` method. It accepts a String representing the path of the asset file.

Unlike the `injectJavascriptFileFromUrl` method that creates a `<script>` HTML tag, this method instead evaluates directly the source code of that asset file.

:::info
  Check [Load files inside the assets folder](/docs/intro#load-files-inside-the-assets-folder) to understand how make your asset files visible to Flutter, otherwise they cannot be found.
:::

Let's assume we have a local file at `/assets/js/main.js` that contains the following JavaScript code:
```javascript
function add(a, b) {
  return a + b;
}
add(10, 20);
```

Here is a simple example that injects the JavaScript code of that asset file to the current web page:
```dart
onLoadStop: (controller, url) async {
  var result = await controller.injectJavascriptFileFromAsset(assetFilePath: "assets/js/main.js");
  print(result.runtimeType); // int
  print(result); // 30
},
```
