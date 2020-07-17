---
title: PhoneNet 日志
top: false
cover: false
img: 'http://bucket.jiabi.tech/medias/featureimages/ssm.jpg'
toc: true
mathjax: true
date: 2020-04-09 13:33:46
password:
summary: PhoneNet Bugfix Log
tags:
    - PhoneNet
    - Log
categories:
    - J2EE课程
---

# PhoneNet 日志

> 由于实训日志的要求  
	这里记录了**一部分**有记录下来的修改日志，摘录了部分代码

## 已解决

- 修复了 login.jsp页面中不显示loginMsg的拼写错误
```html
<!--修改前-->
<div>${registerMsg}</div>
<!--修改后-->
<div>${loginMsg}</div>
```
- 增加了 login.jsp、register.jsp 页面的非空验证
- UserController 增加了生成 registerSuccess.jsp 激活链接
```html
<a class="btn btn-danger" href="https://${emailDomain}" target="_blank">
	现在激活
</a>
```
- 调整了 registerSuccess.jsp 页面结构与CSS样式
- 修改了 UserController 部分地址映射注解
- 增加了 UserController 中管理员登录的角色验证
```java
user.getRole()==0
```
- 添加了禁止删除默认管理员的 JavaScript
```javascript
if(id===1){
	alert("默认管理员不可删除！")
}
```
- 增加了 register.jsp 验证注册表单的 JavaScript
```html
<script type="text/javascript">
	$(function(){
		let passReg = /^[\w]{6,32}$/;
		let emailReg = /^([a-zA-Z0-9]+[_|\_|\.]?)*[a-zA-Z0-9]+@([a-zA-Z0-9]+[_|\_|\.]?)*[a-zA-Z0-9]+\.[a-zA-Z]{2,3}$/;
		//校验的是用户名是否存在
		$("#username").change(function(){
			//使用ajax 做username 的异步验证 checkUsername?username=xxxx
			$.get("usercheckname","name="+this.value,function(data){
				if(data.code==1){
					$("#usernameMsg").html("用户名已经存在").css("color","red");
					$("#registerBtn").attr("disabled",true);
				}else{
					$("#usernameMsg").html("用户名可用").css("color","green");
					$("#registerBtn").removeAttr("disabled");
				}
			})
		});
		//校验邮箱是否合法
		$("#email").change(function(){
			if(this.value.match(emailReg)){
				//使用ajax 做username 的异步验证 checkUsername?username=xxxx
				$.get("usercheckemail","email="+this.value,function(data){
					if(data.code==1){
						$("#emailMsg").html("邮箱已经存在").css("color","red");
						$("#registerBtn").attr("disabled",true);
					}else{
						$("#emailMsg").html("邮箱可用").css("color","green");
						$("#registerBtn").removeAttr("disabled");
					}
				})
			}else{
				$("#emailMsg").html("非法的邮箱格式").css("color","red");
				$("#registerBtn").attr("disabled",true);
			}
		});
		$("#password").change(function(){
			if($("#password").val().match(passReg)){
				$("#passMsg").html("密码可用").css("color","green");
				$("#registerBtn").removeAttr("disabled");
			}else{
				$("#passMsg").html("密码6-32位，支持字母数字下划线").css("color","red");
				$("#registerBtn").attr("disabled",true);
			}
		});
		$("#rePassword").change(function(){
			if($("#rePassword").val()!==$("#password").val()){
				$("#rePassMsg").html("两次密码不一致").css("color","red");
				$("#registerBtn").attr("disabled",true);
			}else{
				$("#rePassMsg").html("密码一致").css("color","green");
				$("#registerBtn").removeAttr("disabled");
			}
		});
	})
</script>
```
- 增加了 UserController 中激活成功后自动登录的代码
- 调整了 admin/admin.jsp 页面结构与CSS样式  
	使得后台管理页面显示的内容更多
- 增加了 GoodsServiceImpl 中自动生成 Top3 类别热销商品的代码
- 修改了 GoodsTypeService 中的 queryByLevel   
	以方便在 Controller 中调用
- 优化了 index.jsp 的嵌入 JavaScript 以及部分CSS样式
```html
		<script type="text/javascript">
		$(function() {
			$.get("getGoodsIndex",null,function(arr){
				for(var i=0,s="";i<3;i++,s=""){
					for(var j=0;j<arr[i].length;j++){
						var g=arr[i][j];
						s+="<span class='sindex'><a class='siximg' href='${pageContext.request.contextPath}/getGoodsById?id="+
								g.id+"'><img src='./fmwimages/"+g.picture+"' width='234px' height='234px' /></a><a class='na'>"+
								g.name+"</a><div style='width:220px;height:60px;overflow:hidden;'><p class='chip'>" +
								g.intro+"</p></div><p class='pri'>￥"+g.price+"元</p></span>";
					}
					$("#data"+(i+1)).html(s);
				}
			});
		});
		</script>
```
- 调整了 goodsDetail.jsp、goodsList.jsp 页面结构以及CSS样式  
- 增加了所有页面图片的超链接
- 增加了首页商品名称超链接
- 增加了对 addGoods.jsp、addGoodsType.jsp 的非空验证与异常捕获
- 修复了 DatePicker.js 中关于 endData 的BUG
- 向 ViewCart 添加了 picture 字段用于在购物车中显示商品缩略图
- 增加了 cart.jsp 页面显示商品缩略图的相关代码并在商品名字添加超链接
- 调整了 cart.jsp、orderDetail.jsp 页面结构以及CSS样式 
- 增加了 OrderController 删除订单的权限验证
- 解决了提交订单跳转至支付页后，刷新会造成订单重复提交的问题
- 增加了 orderService.queryOrderMoneyByOid(oid)   
	以便在直接下单时，支付页面显示
- 增加了 OrderController 中对支付页面处理的相关代码
- 调整了 pay.jsp 页面结构以及CSS样式
- 增加了 header.jsp 用户名链接跳转至 self_info.jsp
- 调整了 self_info.jsp 页面结构以及CSS样式
- 优化了 UserAddressServiceImpl 中 updateDefault 相关代码
- 增加了 OrderController 中处理 confirm.jsp 的相关代码
- 调整了 confirm.jsp 页面结构以及CSS样式
- 完善了 NoLoginFilter
- 新增了 AdminFilter   
	以完成对管理员相关操作的权限验证
- 解决了 admin/admin.jsp 注销后返回页面错误登录的BUG
- 解决了 GoodsTypeController 中 queryNameAndFlag 未进行非空验证的BUG  
- 解决了 admin/login.jsp 背景图片被过滤的问题
- 增加了触发器deleteOrder
	t_order 表删除记录自动删除 t_orderdetail 表对应的记录
```sql
CREATE TRIGGER `deleteOrder` AFTER DELETE ON `t_order` FOR EACH ROW
	 delete from t_orderdetail where oid = OLD.id;
```

- Tips：
	- 浏览器的缓存会影响调试
    - Chrome DevTools 可以禁用缓存
    - 正如ChromiumDev的推文所述，此设置仅在 Chrome DevTools 打开时才有效。
	- 另外应该把GoodsTypeDao.queryNameAndFlag(name,flag)改成动态sql查询

## 未解决

- loginMsg、registerMsg 等错误信息中文乱码
- EmailUtils发送激活邮件中文乱码
- 激活成功自动登录设置 session 跳转后处于未登录状态
```java
// 测试URL
// http://localhost:8086/PhoneNet_war_exploded/activate?e=amlhYmljaGl1QGdtYWlsLmNvbQ==&c=MjAyMDA0MDcwMTA4NDMwNTkzYzE=
```
- 购物车为空可以结算的BUG
- t_cart 表 money 字段没有自动更新 **触发器？**
- 部署在服务器上存取 goodsImages 路径错误
