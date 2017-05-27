# web JS接入指南

# 一、SDK接入设置

## 1. 向云合同注册你的应用程序id

请到开发者应用等级页面进行登记，登记并选择移动应用进行设置后，将获得AppID，可立即用于开发。但应用登记完成后还需要提交审核，只有审核通过的应用才能正式发布使用。

## 2. 引用云合同 SDK JS文件

<b style="color:red">请勿下载yht.js到本地引用，当yht.js进行版本迭代后可能会产生不可预知的bug。</b>

在你的页面引入yht.js，同时，如果你的页面编码不是UTF-8，请添加charset="utf-8"属性。

```html    
<script src="http://sdk.yunhetong.com/sdk/api/yht.js" type="text/javascript" charset="utf-8"></script>
```

手机端引用:

```html
<script src="http://sdk.yunhetong.com/sdk/api/m/yht.js" type="text/javascript" charset="utf-8"></script>
```


# 二、云合同初始化和Token管理

SDK js对签名和合同的操作请求需要带上Token并对Token进行校验，对token超时、为空，返回码为400时，系统回调请求 Token 服务。

Token 的初始化：

```javascript		
	var tokenUnableListener = function(obj){ //当token不合法时，SDK会回调此方法
		$.ajax({
			type: 'POST',
			url: 'token',//第三方服务器获取token的URL，云合同SDK无法提供
			cache : false,
			dataType: 'json',
			data:{},
			success:function(data, textStatus, jqXHR){
				YHT.setToken(data.value.token);//重新设置token
				YHT.do(obj);//调用此方法，会继续执行上次未完成的操作
			},
			error:function(data){
				alert(data.code + " || " + data.msg);
			}
		});
	}	
	YHT.init("AppID", tokenUnableListener);//必须初始化YHT
```

# 三、获取访问Token

```javascript
	$.ajax({
		type: 'POST',
		url: 'token',//第三方服务器获取token的URL，云合同SDK无法提供
		cache : false,
		dataType: 'json',
		data:{},
		success:function(data, textStatus, jqXHR){
			YHT.setToken(data.value.token);//设置token
			var contracts  = data.value.contractList;//获取合同列表
		},
		error:function(data){
			alert(data.code + " || " + data.msg);
		}
	});
```

# 四、云合同合同管理

云合同合同管理模块是指第三方 App 对自己合同进行操作的功能，方法调用后跳转到云合同SDK合同详情页面。

```javascript
	$.ajax({
		type: 'POST',
		async:false,//请使用同步
		url: 'token_contract',//第三方服务器获取token的URL，云合同SDK无法提供
		cache: false,
		dataType: 'json',
		data: {},
		success: function (data, textStatus, jqXHR) {
			var contractId='';
			var backUrl='';
			var noticeParams='';
			YHT.queryContract(
				function successFun(url){
					window.open(url);
				}, 
				function failFun(data){
					alert(data.code + " || " + data.msg);
				},
				contractId,
				backUrl,
				noticeParams
			);
		},
		error: function (data) {

		}
	});
```

| 名称         |描述   			  |  类型     |参数             |
| ------------ | --------------  | --------- |:---------------:
| successFun   | 请求成功回调函数  | function | url : 跳转地址  |
| failFun      | 请求失败回调函数  | function | data : 错误信息 |
| contractId   | 合同ID号         | string   |                 |
| backUrl      | 回调地址         | string   |                 |
| noticeParams | 回调参数         | string   |                 |

回调参数传递后，可以通过消息回调获取。


# 五、iOS调用H5页面

第三方App调用H5页面方法，方法调用后跳转到云合同SDK合同查看页面。

``` objective-c
/**
 *  根据'contractID'、'token'直接展示拼接的 url 地址在’UIWebView‘展示即可，
 *
 **/  
- (void)refresh{
    NSMutableString *__urlStr = [NSMutableString stringWithString:@"https://sdk.yunhetong.com/sdk/contract/hView?];
    [__urlStr appendFormat:@"?contractId=%ld", [self.contractID longValue]];
    [__urlStr appendFormat:@"&token=%@", [YHTTokenManager sharedManager].token];

    self.contractURL = [NSURL URLWithString:__urlStr];
    [[NSURLCache sharedURLCache] removeAllCachedResponses];
    NSURLRequest *__request = [NSURLRequest requestWithURL:self.contractURL
                                               cachePolicy:NSURLRequestReloadIgnoringCacheData
                                           timeoutInterval: 20.0];
    [self loadRequest:__request];
}
```		

查看[sdk-iOS-H5 demo](https://github.com/lvxunDev/yunhetong-sdk-iOS-H5-)。



# 六、android调用H5页面

第三方开发者只需要调用Html5页面即可进行合同的签署、查看。

``` java

	//云合同Html5的路径
	String url = "https://sdk.yunhetong.com/sdk/contract/hView?contractId=获取的合同ID&token=获取的token" 
	//参数为第三方服务端获取。
	WebView mYhtWebView = new WebView(this);
    mYhtWebView.setScrollContainer(false);
    mYhtWebView.setScrollbarFadingEnabled(false);
    mYhtWebView.setScrollBarStyle(View.SCROLLBARS_OUTSIDE_OVERLAY);
    WebSettings settings = mYhtWebView.getSettings();
    settings.setDefaultTextEncodingName("UTF-8");
    settings.setJavaScriptEnabled(true);
    settings.setSupportZoom(true);
    settings.setDisplayZoomControls(false);
    settings.setLayoutAlgorithm(WebSettings.LayoutAlgorithm.NORMAL);
    settings.setUseWideViewPort(true);
    settings.setLoadWithOverviewMode(true);
    settings.setLoadsImagesAutomatically(true);
    mYhtWebView.setWebViewClient(new WebViewClient(){
        @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
        @Override
        public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
            view.loadUrl(String.valueOf(request.getUrl()));
            return true;
        }

        @Override
        public boolean shouldOverrideUrlLoading(WebView view, String url) {
            view.loadUrl(url);
            return true;
        }
    });
	//调用云合同Html5
	mWebView.loadUrl(url);
	//配置可酌情删减。
```		

