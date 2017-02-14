# web JS接入指南

# 一、SDK接入设置

## 1. 向云合同注册你的应用程序id

请到开发者应用等级页面进行等级，登记并选择移动应用进行设置后，将获得AppID，可立即用于开发。但应用登记完成后还需要提交审核，只有审核通过的应用才能正式发布使用。

## 2. 引用云合同 SDK JS文件

在你的页面引入yht.js，同时，如果你的页面编码不是UTF-8，请添加charset="utf-8"属性。

```html    
<script src="http://sdk.yunhetong.com/sdk/api/yht.js" type="text/javascript" charset="utf-8"></script>
```

手机端引用:
```html
<script src="http://sdk.yunhetong.com/sdk/api/m/yht.js" type="text/javascript" charset="utf-8"></script>
```


# 二、云合同初始化和Token管理

SDK js对签名和合同的操作请求需要带上Token并对Token进行校验，对token超时、为空，返回码为400时，系统回调请求 Token 服务

Token 的初始化

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
	var contractId='';
	var backUrl='';
	var backParaObj={code:'success'};
	var backPara=JSON.stringify(backParaObj);

	YHT.queryContract(
		function successFun(url){
			window.open(url);
		}, 
		function failFun(data){
			alert(data.code + " || " + data.msg);
		},
		contractId,
		backUrl,
		backPara
	);
```

| 名称         |描述                                                     |   
| ------------ |:-------------------------------------------------------:
| successFun   | 请求成功回调函数</br> 类型:function </br> url : 跳转地址  |
| failFun      | 请求失败回调函数</br> 类型:function </br> data : 错误信息 |
| contractId   | 合同ID号 </br> 类型:string                              |
| backUrl      | 回调地址  </br> 类型:string                              |
| backPara     | 回调参数 </br> 类型:string                               |

回调参数传递后，可以通过消息回调获取。


# 五、云合同签名管理

云合同签名管理是指第三方App对用户的签名信息进行管理的功能，方法调用后跳转到云合同SDK签名管理页面。

```javascript
	var backUrl='';
	var backParaObj={code:'success'};
	var backPara=JSON.stringify(backParaObj);

	YHT.querySign(
		function successFun(url){
			window.open(url);
		},
		function failFun(data){
			alert(data.code + " || " + data.msg);
		},
		backUrl,
		backPara
	);
```		

| 名称         |描述                                                     |   
| ------------ |:-------------------------------------------------------:
| successFun   | 请求成功回调函数</br> 类型:function </br> url : 跳转地址  |
| failFun      | 请求失败回调函数</br> 类型:function </br> data : 错误信息 |
| backUrl      | 回调地址  </br> 类型:string                              |
| backPara     | 回调参数 </br> 类型:string                               |

回调参数传递后，可以通过消息回调获取。