

## 安卓原生调用JS

- 4.4版本之前

```
// mWebView = new WebView(this) // 当前webview对象
// 通过loadUrl方法进行调用 参数通过字符串的方式传递
mWebView.loadUrl('javascript: 方法名("args1, args2, ...")')


// UI线程中运行
runOnUiThread(new Runnable(){
	@Override
	public void run(){
		// 通过loadUrl方法进行调用，参数通过字符串的方式传递
		mWebView.loadUrl('javascript:方法名("args1, args2, ...")')
		
		// 安卓中原生弹框
		Toast.makeText(Activity名.this, '调用方法...', Toast.LENGTH_SHORT).show()
	}
})
```

- 4.4版本之后，包括4.4

```
// 通过异步的方式执行JS代码，并获取返回值
mWebView.evaluateJavascript('javascript: 方法名("args1, args2, ...")', new ValueCallback(){
	@override
	// 这个方法会在执行完毕之后触发，其中value就是js代码执行的返回值(如果有的话)
	public void onReceiveValue(String value){
		
	}
})
```

## JS调用安卓

- 安卓设置

```
// 获取webView的设置对象，方便后续修改
WebSettings webSetting = mWebView.getSettings()
// 设置安卓允许JS脚本 必须哟
webSettings.setJavascriptEnabled(true)
// 暴露一个叫做JSBridge的对象到webView的全局环境
mWebView.addJavascriptInterface(getJSBridge(), 'JSBridge')
```

```
// 安卓4.2版本以上，本地方法要加上注解@JavascriptInterface 否则无法使用
private Object getJSBridge(){
	// 实例化新对象
	Object insertObj = new Object(){
		@JavascriptInterface
			// 对象内部方法1
			public String fun1(){
				return 'fun1'
			}
			
		@JavascriptInterface
			public String fun2(){
				return 'fun2'
			}
	}
	
	// 返回实例化的对象
	return insertObj
}
```

- JS调用

```
window.JSBridge.fun1() // 返回 fun1
window.JSBridge.fnn2() // 返回 fun2
```

## ios原生调用JS

```
class ViewController: UIVieController, WKNavigationDelegate, WKScriptMessageHandler{
	
	// 加载完毕会触发，类似Vue的生命周期钩子
	func webView(_ webView: WKWebView, didFinish navigation: WKNavigation!){
		print('触发了...');
		
		// wkWebView 调用j代码，其中doSomething会被当作js解析
		webView.evaluateJavaScript('doSomething()');
	}
}
```

## JS调用ios原生

```
window.webkit.messageHandlers.方法名.postMessage(数据)

wkWebView.configuration.userContentController.add(self, name: 方法名)

func userContentCOntroller(_ userContentCOntroller: WKUserContentController, didReceivemessage: WKScriptMessage){
	// message.body 就是传递过来的数据
	print('传递过来的数据为:'. message.body)
}
```





