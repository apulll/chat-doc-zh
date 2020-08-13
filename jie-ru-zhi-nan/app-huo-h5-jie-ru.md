# APP or H5

## 1. H5

If you have your own designed web page icons/styles/buttons, you can also embed links into your web page as independent pages, allowing customers to click directly to jump to a dialogue page.

**STEPS: System management -&gt; Bots management -&gt; Mobile web access**

![](../.gitbook/assets/image%20%287%29.png)

You can embed the link directly in your own web page according to your needs, then you can click the button and directly jump to the customer service page.

\[Example\]

![](../.gitbook/assets/screencapture-chatbot-myqcloud-chatbot-h5-2019-11-15-14_08_56.png)

## 2. H5 page embedding in iOS <a id="h5&#x9875;&#x9762;&#x5D4C;&#x5165;ios-app&#x65B9;&#x6848;"></a>

### 2.1. Configuration <a id="&#x5DE5;&#x7A0B;&#x914D;&#x7F6E;"></a>

Add permission of camera and HTTP usage in Info.plist document.

```text
    <key>NSCameraUsageDescription</key>
    <string>为了可以上传图片和视频</string>
    <key>NSAppTransportSecurity</key>
    <dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
    </dict>
```

### 2.2. UI component <a id="ui&#x7EC4;&#x4EF6;"></a>

Recommend to use WKWebView \(iOS 8.0 +\) and create it with encoding. See our reference:

```text
var webView: WKWebView!
    override func viewDidLoad() {
        super.viewDidLoad()
       // Do any additional setup after loading the view, typically from a nib.
       let url = "http://chatbot.myqcloud.com/chatbot/h5/#/?productId=dtcm&uuid="
       let request = URLRequest(url: URL(string: url)!)
       webView.load(request)
    }
override func loadView() {
    let webConfiguration = WKWebViewConfiguration()
    webView = WKWebView(frame: .zero, configuration: webConfiguration)
    view = webView
}
```

Please use UIWebView if you want to be compatible with lower versions of the operating system.

### 2.3. Complement <a id="&#x8865;&#x5145;"></a>

If the Xcode console outputs the following log:

```text
[discovery] errors encountered while discovering extensions: Error Domain=PlugInKit Code=13 "query cancelled" UserInfo={NSLocalizedDescription=query cancelled}
```

Handling：

* Ignore it and it won't affect using the components. \(recommended\)
* Modify configuration
  1. Open Xcode menu: Product &gt; Scheme &gt; Edit Scheme
  2.  Add `key-value： OS_ACTIVITY_MODE = disable`in Environment Variables. 

## 3. H5 page embedding in Android <a id="h5&#x9875;&#x9762;&#x5D4C;&#x5165;android-app&#x65B9;&#x6848;"></a>

### 3.1. Web-view opens the gallery and acquires pictures from android files <a id="webview&#x6253;&#x5F00;&#x56FE;&#x5E93;&#x5E76;&#x83B7;&#x53D6;android&#x6587;&#x4EF6;&#x56FE;&#x7247;"></a>

【EXAMPLE】

1.web-view initialization

```text
private static final int FILE_SELECT_CODE = 0;
private ValueCallback<Uri> mUploadMessage;//回调图片选择，4.4以下
private ValueCallback<Uri[]> mUploadCallbackAboveL;//回调图片选择，5.0以上

    //允许JavaScript执行
    webView.getSettings().setJavaScriptEnabled(true);
    webView.getSettings().setLoadsImagesAutomatically(true);
    webView.setVerticalScrollBarEnabled(false);

    //运行webview通过URI获取安卓文件
    webView.getSettings().setAllowFileAccess(true);
    webView.getSettings().setAllowFileAccessFromFileURLs(true);
    webView.getSettings().setAllowUniversalAccessFromFileURLs(true);
    webView.setWebChromeClient(new MyWebChromeClient());//设置可以打开图片管理器
    webView.loadUrl("xxxxxxxx");
```

2.Inherit WebChromeClient and process it differently according to different versions of Android.

```text
private class MyWebChromeClient extends WebChromeClient {

    // For Android 3.0+
    public void openFileChooser(ValueCallback<Uri> uploadMsg) {
        mUploadMessage = uploadMsg;
        Intent i = new Intent(Intent.ACTION_GET_CONTENT);
        i.addCategory(Intent.CATEGORY_OPENABLE);
        i.setType("image/*");
    startActivityForResult(Intent.createChooser(i, "File Chooser"), FILE_SELECT_CODE);
    }

    // For Android 3.0+
    public void openFileChooser(ValueCallback uploadMsg, String acceptType) {
        mUploadMessage = uploadMsg;
        Intent i = new Intent(Intent.ACTION_GET_CONTENT);
        i.addCategory(Intent.CATEGORY_OPENABLE);
        i.setType("*/*");
        startActivityForResult(Intent.createChooser(i, "File Browser"), FILE_SELECT_CODE);
    }

    // For Android 4.1
    public void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType, String capture) {
        mUploadMessage = uploadMsg;
        Intent i = new Intent(Intent.ACTION_GET_CONTENT);
        i.addCategory(Intent.CATEGORY_OPENABLE);
        i.setType("image/*");
        startActivityForResult(Intent.createChooser(i, "File Chooser"), FILE_SELECT_CODE);
    }

    // For Android 5.0+
    public boolean onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback, WebChromeClient.FileChooserParams fileChooserParams) {
        mUploadCallbackAboveL = filePathCallback;
        Intent i = new Intent(Intent.ACTION_GET_CONTENT);
        i.addCategory(Intent.CATEGORY_OPENABLE);
        i.setType("*/*");
        startActivityForResult(
            Intent.createChooser(i, "File Browser"),
            FILE_SELECT_CODE);
        return true;
    }
}
```

3.Process the returned picture uri in the onActivityResult callback of the page.

```text
@Override
public void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (resultCode != Activity.RESULT_OK) {
           return;
    }

    switch (requestCode) {
        case FILE_SELECT_CODE: {
            if (Build.VERSION.SDK_INT >= 21) {//5.0以上版本处理
                Uri uri = data.getData();
                Uri[] uris = new Uri[]{uri};

               /* ClipData clipData = data.getClipData();  //选择多张
                if (clipData != null) {
                    for (int i = 0; i < clipData.getItemCount(); i++) {
                        ClipData.Item item = clipData.getItemAt(i);
                        Uri uri = item.getUri();
                        uris[i]=uri;
                    }
                }*/
                mUploadCallbackAboveL.onReceiveValue(uris);//回调给js
            } else {//4.4以下处理
                Uri uri = data.getData();
                mUploadMessage.onReceiveValue(uri);
            }
        }
        break;
    }
}
```

### 3.2. Notes <a id="&#x6CE8;&#x610F;&#x4E8B;&#x9879;"></a>

In Android5.0 and above systems, when the link loaded by WebView starts with Https, but the contents of the link, such as the picture, are Http links, then the picture will not be loaded and the picture will be cracked.

Solution：Set the loading mode to mixed mode \(MIXED\_CONTENT\_COMPATIBILITY\_MODE = 2\) before web-view loads the page \( at initialization time\), for example:

```text
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
webView.getSettings().setMixedContentMode(WebSettings.MIXED_CONTENT_COMPATIBILITY_MODE);
｝
webView.getSettings().setBlockNetworkImage(false)；
```



