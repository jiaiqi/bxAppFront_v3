<template>
	
	<view class="authorization">
		<view class="container" v-if="isBindUser || client_env==='web' || client_env==='app'">
			<view class="right-top-sign"></view>
			<!-- 设置白色背景防止软键盘把下部绝对定位元素顶上来盖住输入框等 -->
			<view class="wrapper">
				<!-- <view class="left-top-sign">LOGIN</view> -->
				<view class="welcome">
					欢迎回来！<text  v-if="(client_env==='wxh5' || client_env==='wxmp') && !isShowUserLogin">请授权微信登录</text>
				</view>
				<view class="input-content"  v-if="client_env==='web' || client_env==='app' || isShowUserLogin">
					<view class="input-item">
						<text class="tit">帐号</text>
						<input 
							type="text" 
							v-model="user.user_no" 
							placeholder="请输入帐号"
							maxlength="20"
							data-key="mobile"
							@input="inputChange"
						/>
					</view>
					<view class="input-item">
						<text class="tit">密码</text>
						<input 
							type="mobile" 
							v-model="user.pwd" 
							placeholder="请输入密码"
							placeholder-class="input-empty"
							maxlength="20"
							password 
							data-key="password"
							@input="inputChange"
							@confirm="userLogined"
						/>
					</view>
				</view>
				<button  v-if="(client_env==='web' || client_env==='app' || client_env==='wxh5' ) && isShowUserLogin" class="confirm-btn bg-gradual-green text-green" @click="userLogined">{{isBindUser ? '提交绑定' : "登录"}}</button>
				<button  v-if="(client_env==='web' || client_env==='app' || client_env==='wxh5' ) && isShowUserLogin" class="confirm-btn bg-gradual-orange text-green" @click="toBack"> 暂不，继续使用</button>
				<button  v-if="(client_env==='wxh5' || client_env==='wxmp') && !isShowUserLogin" 
				class="confirm-btn bg-gray text-green" 
				lang="zh_CN" 
				type="primary" 
				open-type="getUserInfo" 
				@getuserinfo="saveWxUser" 
				:withCredentials="false"
				:disabled="disabled">立即授权</button>
				<button v-if="(client_env==='wxh5' || client_env==='wxmp') && !isShowUserLogin" 
				class="confirm-btn bg-grey text-black" 
				type="default" 
				@tap="navBack" 
				:disabled="false">暂不授权</button>
			</view>
			<!-- <view class="register-section">
				还没有账号?
				<text @click="toRegist">马上注册</text>
			</view> -->
		</view>
		
	</view>
</template>

<script>
export default {
	// 账号授权页面
	name: 'AccountExec',
	data() {
		return {
			disabled: !this.isWeixinClient(),
			backUrl: '',
			user:{
				user_no:"",
				pwd:""
			},
			isShowUserLogin:true,
			client_env:uni.getStorageSync("client_env"),
			isThirdParty:uni.getStorageSync("isThirdParty"),// 是否需要第三方认证
			isBindUser:false,
		};
	},
	created() {
	},
	onShow() {
		// if(uni.getStorageSync("isLogin")){
		// 	console.log("不进行初始化授权",uni.getStorageSync("isLogin"),uni.getStorageSync("backUrl"))
		// 	if(uni.getStorageSync("backUrl")&& uni.getStorageSync("backUrl") !== '/'){
		// 		uni.redirectTo({
		// 			url:uni.getStorageSync("backUrl")
		// 		})
		// 	}else{
		// 		uni.redirectTo({
		// 			url:this.$api.homePath
		// 		})
		// 	}
		// }else{
		// 	console.log("进行初始化授权",uni.getStorageSync("isLogin"),uni.getStorageSync("backUrl"))
		// 	this.initLogin();
		// 	if(uni.getStorageSync("isThirdParty")){
		// 	}
		// }
		 if (navigator.userAgent.indexOf('iPhone') !== -1) {
		    	let linkUrl = window.location + '';
		    	uni.setStorageSync('linkUrl', linkUrl);
		    }
	},
	onLoad(option) {
		let self = this
		uni.showLoading({
			title:"请稍后"
		})
		
		if(uni.getStorageSync("isLogin")){
			console.log("不进行初始化授权",uni.getStorageSync("isLogin"))
			if(uni.getStorageSync("backUrl") && uni.getStorageSync("backUrl") !== '/'){
				uni.redirectTo({
					url:uni.getStorageSync("backUrl")
				})
			}else{
				uni.redirectTo({
					url:this.$api.homePath
				})
			}
		}else{
			console.log("进行初始化授权",uni.getStorageSync("isLogin"))
			this.initLogin();
			if(uni.getStorageSync("isThirdParty")){
			}
		}
		if (navigator.userAgent.indexOf('iPhone') !== -1) {
			let linkUrl = window.location + '';
			uni.setStorageSync('linkUrl', linkUrl);
		}
		
	},
	methods: {
		toBack(){
			if(uni.getStorageSync("isLogin")){
				if(uni.getStorageSync("backUrl")){
					uni.redirectTo({
						url:uni.getStorageSync("backUrl")
					})
				}else{
					uni.redirectTo({
						url:this.$api.homePath
					})
				}
			}else{
				this.initLogin()
			}
			
		},
		navBack() {
			uni.navigateBack({
				animationDuration: 200
			}).then().catch((e)=>{
				uni.redirectTo({
					url:"/pages/basics/home/home"
				})
			});
		},
		inputChange(e){
			// console.log(e)
		},
		initLogin() {
			let that = this;
			let code = null;
			// 公众号环境
			code = this.$route.query.code;
			console.log('code:', code);
			let isLogin = uni.getStorageSync('isLogin');
			console.log('是否绑定账号:', isLogin);
			let isWeixinClient = this.isWeixinClient();
			const client_env = uni.getStorageSync('client_env');
			// uni.showToast({
			// 	title:isWeixinClient + "是否登录： "+ isLogin
			// })
			console.log('是否微信环境', isWeixinClient);
			if(!isLogin){
				if (isWeixinClient) {
					//微信环境(小程序或者公众号)
					if (code || client_env === 'wxmp') {
						// 已经得到code,进行验证登录
						console.log('已获取到code,即将进行验证登录');
						this.saveWxUser();
					} else if (!code) {
						//公众号 未授权 -> 获取授权
						console.log('未发现code,尝试获取重定向链接');
						that.getAuthorized();
					}
				} else if (!isWeixinClient) {
					// 非微信环境(H5或APP)
					uni.showToast({
						title: '请在微信中访问此页面',
						icon: 'none',
						duration: 3000
					});
				}
			}else{
				let url = uni.getStorageSync('backUrl');
				
				console.log('that.backUrl', that.backUrl,"===",url);
				if (url) {
					url = that.getDecodeUrl(url)
					if(url && url.lastIndexOf("backUrl=") !== -1){
						url = url.substring(url.lastIndexOf("backUrl=")+8,url.length)
					}
					uni.reLaunch({
						url:url
					})
				} else {
					uni.reLaunch({
						url:that.$api.homePath
					})
				}
			}
			
		},
		async getAuthorized() {
			let self = this
			// 公众号环境获取回调地址(在回调地址中获取code)
			let url = self.getServiceUrl('wx', 'srvwx_public_page_authorization', 'operate');
			let req = [
				{
					data: [
						{
							app_no: self.$api.appNo.wxh5,
							redirect_uri: self.$api.frontEndAddress
						}
					],
					serviceName: 'srvwx_public_page_authorization'
				}
			];
			let burl = uni.getStorageSync("backUrl")
			// if(burl){
			// 	req[0].data[0].redirect_uri = self.$api.frontEndAddress + "?backUrl=" + encodeURIComponent( burl)
			// }
			this.$http.post(url, req).then((response)=>{
				console.log('授权 response:', response);
				// uni.showModal({
				// 	content: "请求完成"+ JSON.stringify(response.data),
				// 	showCancel: false
				// });
				if (response.data.response[0].response.authUrl) {
					console.log('redirctUrl:', response.data.response[0].response.authUrl);
					window.location.href = response.data.response[0].response.authUrl;
				} else {
					uni.showToast({
						title: response.data.resultMessage,
						icon: 'none'
					});
				}
			});
			
		},
		saveWxUser() {
			// 静默登录(验证登录)
			let that = this;
			const isWeixinClient = that.isWeixinClient();
			if (isWeixinClient) {
				const url = this.getServiceUrl('wx', 'srvwx_app_login_verify', 'operate');
				// #ifdef MP-WEIXIN
				wx.login({
					// 获取小程序code
					success(res) {
						if (res.code) {
							// that.checkAuthorization();
							//验证登录
							let url = that.$api.verifyLogin.url;
							let req = [
								{
									data: [
										{
											code: res.code,
											app_no: that.$api.appNo.wxmp
										}
									],
									serviceName: 'srvwx_app_login_verify'
								}
							];
							that.$http.post(url, req).then(response => {
								if (response.data.resultCode === 'SUCCESS') {
									uni.hideLoading()
									console.log('授权成功', response);
									let resData = response.data.response[0].response;
									let loginMsg = {
										bx_auth_ticket: resData.bx_auth_ticket,
										expire_time: resData.expire_time
									};
									let expire_timestamp = parseInt(new Date().getTime() / 1000) + loginMsg.expire_time; //过期时间的时间戳(秒)
									uni.setStorageSync('bx_auth_ticket', resData.bx_auth_ticket);
									uni.setStorageSync('expire_time', resData.expire_time); // 有效时间
									uni.setStorageSync('expire_timestamp', expire_timestamp); // 过期时间
									if(resData.login_user_info.user_no){
										uni.setStorageSync('login_user_info',resData.login_user_info)
										console.log('resData.login_user_info',resData.login_user_info)
									}
									if (resData.login_user_info.data) {
										uni.setStorageSync('visiter_user_info', resData.login_user_info.data[0]);
										// this.$store.commit('setOpenid', resData.login_user_info.data[0].openid);
									}
									uni.setStorageSync('isLogin',true)
									console.log('that.backUrl',that.backUrl)
									
									
									// 获取用户微信信息
									that.getUserInfo()
									uni.redirectTo({
										url: that.$api.homePath
									});

								} else if (response.data.resultCode === 'FAILURE') {
									uni.setStorageSync('isLogin', false);
									uni.showToast({
										title: response.data.resultMessage
									});
								}
							});
						} else {
							uni.setStorageSync('isLogin', false); // 登录状态
							console.log('登录失败！' + res.errMsg);
						}
					}
				});
				// #endif
				// #ifdef H5
				// 公众号环境
				let req = [
					{
						data: [
							{
								code: that.$route.query.code,
								app_no: that.$api.appNo.wxh5
							}
						],
						serviceName: 'srvwx_app_login_verify'
					}
				];
				
				that.$http.post(url, req).then(response => {
					if (response.data.resultCode === 'SUCCESS' && response.data.response[0].response) {
						console.log('授权成功', response);
						let resData = response.data.response[0].response;
						let loginMsg = {
							bx_auth_ticket: resData.bx_auth_ticket,
							expire_time: resData.expire_time
						};
						uni.setStorageSync('isLogin',true)
						console.log("登录成功",uni.getStorageSync("isLogin"),resData)
						let expire_timestamp = parseInt(new Date().getTime() / 1000) + loginMsg.expire_time; //过期时间的时间戳(秒)
						uni.setStorageSync('expire_time', resData.expire_time); // 有效时间
						// let expire_timestamp = parseInt(new Date().getTime() / 1000) + 10; //过期时间的时间戳(秒)
						// uni.setStorageSync('expire_time', 10); // 有效时间
						uni.setStorageSync('bx_auth_ticket', resData.bx_auth_ticket);
						uni.setStorageSync('expire_timestamp', expire_timestamp); // 过期时间
						
						if(resData.login_user_info && resData.login_user_info.user_no){
							uni.setStorageSync('login_user_info',resData.login_user_info)
							console.log('resData.login_user_info',resData.login_user_info)
						}
						if (resData.login_user_info && resData.login_user_info.data) {
							uni.setStorageSync('visiter_user_info', resData.login_user_info.data[0]);
							// this.$store.commit('setOpenid', resData.login_user_info.data[0].openid);
						}
						// 获取回调前记录的url 并再回调后 再次进入该url，没有该url时 进入 home
						let url = uni.getStorageSync('backUrl');
						
						console.log('that.backUrl', that.backUrl,"===",url);
						uni.hideToast()
						uni.hideLoading()
						if (url && url !== '/') {
							url = that.getDecodeUrl(url)
							// alert("2::" + url + uni.getStorageSync('bx_auth_ticket'))
							if(url && url.lastIndexOf("backUrl=") !== -1){
								url = url.substring(url.lastIndexOf("backUrl=")+8,url.length)
								// console.log("授权成功，准备返回用户界面url",url)
							}
							uni.reLaunch({
								url:url
							})
						} else {
							uni.reLaunch({
								url:that.$api.homePath
							})
						}
						// uni.showModal({
						//     title: '提示',
						//     content: '是否继续绑定已有百想系统帐号？',
						// 	cancelText: "不用了",  
						// 	confirmText: "是", 
						//     success: function (res) {
						//         if (res.confirm) {
						//             that.isBindUser = true
						//         } else if (res.cancel) {
						//             console.log('用户点击取消');
						// 			if (url) {
						// 				url = that.getDecodeUrl(url)
						// 				// alert("2::" + url + uni.getStorageSync('bx_auth_ticket'))
						// 				if(url && url.lastIndexOf("backUrl=") !== -1){
						// 					url = url.substring(url.lastIndexOf("backUrl=")+8,url.length)
						// 					console.log("授权成功，准备返回用户界面url",url)
						// 				}
						// 				uni.reLaunch({
						// 					url:url
						// 				})
						// 			} else {
						// 				uni.reLaunch({
						// 					url:that.$api.homePath
						// 				})
						// 			}
						//         }
						//     }
						// });
					} else {
						uni.setStorageSync('isLogin',false)
						// self.$store.commit('setAuthorState', false);
						console.log('授权失败');
						// uni.showToast({
						// 	title:'授权失败',
						// 	icon:'none'
						// })
					}
				});
				
				// #endif
			}
		},
		 getUserInfo: function(loginType, cb) {  
			 let self = this
			wx.login({
				success: function() {  
					wx.getUserInfo({  
						success: function(res) {  
							self.setWxUserInfo(res.userInfo)
							uni.setStorageSync('wxuserinfo',res.userInfo)
							console.log(res) 
						}, 
						fail: function() {  
							//3.授权友好提示
							wx.showModal({  
								title: '提示',  
								content: "您还未授权登陆，部分功能将不能使用，是否重新授权？",  
								showCancel: true,  
								cancelText: "不用了",  
								confirmText: "是",  
								success: function(res) {  
									//4.确认授权调用wx.openSetting  
									if (res.confirm) {  
										if (wx.openSetting) { //当前微信的版本 ，是否支持openSetting  
											wx.openSetting({  
												success: (res) => {  
													if (res.authSetting["scope.userInfo"]) { //如果用户重新同意了授权登录  
														wx.getUserInfo({  
															success: function(res) {  
																uni.setStorageSync('wxuserinfo',res.userInfo)
																console.log(res)  
																self.setWxUserInfo(res.userInfo)
															}  
														})  
													} else { //用户还是拒绝  
														console.log(res)  
													}  
												},  
												fail: function() { //调用失败，授权登录不成功  
													
												}  
											})  
										} else {  
											console.log(res)  
										}  
									} else {  
										console.log(res)  
									}  
								}  
							})  
						}  
					})  
				}  
			})  
		} ,
		judgeClientEnviroment() {
			let client_env = '';
			// #ifdef H5
			const isWeixin = this.isWeixinClient();
			console.log('isWeixin', isWeixin);
			client_env = isWeixin ? 'wxh5' : 'web';
			if(client_env === 'web'){
				this.isShowUserLogin = true
			}
			// #endif
			// #ifdef MP-WEIXIN
			client_env = 'wxmp';
			//#endif
			// #ifdef APP-PLUS
			client_env = 'app';
			this.isShowUserLogin = true
			// #endif
			let client_type = '';
			switch (uni.getSystemInfoSync().platform) {
				case 'android':
					console.log('运行Android上');
					client_type = 'android';
					break;
				case 'ios':
					console.log('运行iOS上');
					client_type = 'ios';
					break;
				default:
					client_type = 'web';
			}
			this.client_type = client_type;
			this.client_env = client_env;
			uni.setStorageSync('client_env', client_env);
			uni.setStorageSync('client_type', client_type);
			this.client_env = client_env
			return {
				client_env: client_env,
				client_type: client_type
			};
		},
		async userLogined(e){
			console.log('srvuser_login', e);
			let that = this
			let url = that.getServiceUrl('sso','srvuser_login','operate')
			// https://srvms.100xsys.cn/sso/operate/srvuser_login
			let req = [{
				"serviceName":"srvuser_login",
				"data":[that.user],
				}]
			if(that.isBindUser){
				url = that.$api.bindWxUser
				req = [{
				"serviceName":"srvwx_user_bind",
				"data":[that.user],
				}]
			}
				if(that.isInvalid(that.user.user_no)  &&  that.isInvalid(this.user.pwd)){
					let response = await that.$http.post(url, req)
						console.log('srvuser_login', response);
						if (response.data.state === 'SUCCESS' && response.data.response[0].response) {
							let res = response.data.response[0].response
							let expire_timestamp = parseInt(new Date().getTime() / 1000) + res.expire_time; //过期时间的时间戳(秒)
							uni.setStorageSync('bx_auth_ticket', res.bx_auth_ticket);
							uni.setStorageSync('expire_time', res.expire_time); // 有效时间
							uni.setStorageSync('expire_timestamp', expire_timestamp); // 过期时间
							if(res.login_user_info.user_no){
								uni.setStorageSync('login_user_info',res.login_user_info)
								console.log('res.login_user_info',res.login_user_info)
							}
							if (res.login_user_info.data) {
								uni.setStorageSync('visiter_user_info', resData.login_user_info.data[0]);
								// this.$store.commit('setOpenid', resData.login_user_info.data[0].openid);
							}
							uni.setStorageSync('isLogin',true)
							
							console.log('that.backUrl',that.backUrl)
							
							console.log("userLogined",response.data)
							let url = uni.getStorageSync('backUrl');
							if (url && url !== '/') {
								url = that.getDecodeUrl(url)
								// alert("2::" + url + uni.getStorageSync('bx_auth_ticket'))
								if(url && url.lastIndexOf("backUrl=") !== -1){
									url = url.substring(url.lastIndexOf("backUrl=")+8,url.length)
									// console.log("授权成功，准备返回用户界面url",url)
								}
								uni.reLaunch({
									url:url
								})
							} else {
								uni.reLaunch({
									url:that.$api.homePath
								})
							}
						}
				}else{
					this.user.pwd = ''
					uni.showToast({
					    title: '帐号或密码错误',
					    duration: 2000
					});
				}
			
		}
	}
};
</script>

<style lang="scss">
.authorization {
	width: 100%;
	height: 100%;
	background-color: #fff;
	.images {
		width: 100%;
		margin: 50upx 0;
	}
	.text {
		width: 90%;
		display: block;
		text-align: center;
		padding: 20upx 5%;
	}
	.button-box {
		margin-top: 50upx;
		height: 200upx;
		display: flex;
		flex-direction: column;
		justify-content: space-around;
		align-items: center;
		.buttons {
			width: 70%;
			height: 70upx;
			line-height: 70upx;
			border-radius: 0upx;
			font-size: 30upx;
		}
	}
}
</style>
<style lang='scss'>
	page{
		background: #fff;
	}
	.container{
		padding-top: 115px;
		position:relative;
		width: 100vw;
		height: 100vh;
		overflow: hidden;
		background: #fff;
	}
	.wrapper{
		position:relative;
		z-index: 90;
		background: #fff;
		padding-bottom: 40upx;
	}
	.back-btn{
		position:absolute;
		left: 40upx;
		z-index: 9999;
		/* padding-top: var(--status-bar-height); */
		top: 40upx;
		font-size: 40upx;
		/* color: $font-color-dark; */
	}
	.left-top-sign{
		font-size: 120upx;
		/* color: $page-color-base; */
		position:relative;
		left: -16upx;
	}
	.right-top-sign{
		position:absolute;
		top: 80upx;
		right: -30upx;
		z-index: 95;
		&:before, &:after{
			display:block;
			content:"";
			width: 400upx;
			height: 80upx;
			background: #b4f3e2;
		}
		&:before{
			transform: rotate(50deg);
			border-radius: 0 50px 0 0;
		}
		&:after{
			position: absolute;
			right: -198upx;
			top: 0;
			transform: rotate(-50deg);
			border-radius: 50px 0 0 0;
			/* background: pink; */
		}
	}
	.left-bottom-sign{
		position:absolute;
		left: -270upx;
		bottom: -320upx;
		border: 100upx solid #d0d1fd;
		border-radius: 50%;
		padding: 180upx;
	}
	.welcome{
		position:relative;
		left: 50upx;
		top: -90upx;
		font-size: 46upx;
		color: #555;
		text-shadow: 1px 0px 1px rgba(0,0,0,.3);
	}
	.input-content{
		padding: 0 60upx;
	}
	.input-item{
		display:flex;
		flex-direction: column;
		align-items:flex-start;
		justify-content: center;
		padding: 0 30upx;
		background:#f8f6fc;
		height: 120upx;
		
		border-radius: 4px;
		margin-bottom: 50upx;
		&:last-child{
			margin-bottom: 0;
		}
		.tit{
			height: 50upx;
			line-height: 56upx;
			/* font-size: $font-sm+2upx; */
			/* color: $font-color-base; */
		}
		input{
			height: 60upx;
			/* font-size: $font-base + 2upx; */
			/* color: $font-color-dark; */
			width: 100%;
			/* box-sizing: border-box;
			padding: 20upx; */
		}	
	}

	.confirm-btn{
		width: 630upx;
		height: 76upx;
		line-height: 76upx;
		border-radius: 50px;
		margin-top: 70upx;
		/* background: $uni-color-primary; */
		color: #fff;
		/* font-size: $font-lg; */
		&:after{
			border-radius: 100px;
		}
	}
	.forget-section{
		/* font-size: $font-sm+2upx; */
		/* color: $font-color-spec; */
		text-align: center;
		margin-top: 40upx;
	}
	.register-section{
		position:absolute;
		left: 0;
		bottom: 50upx;
		width: 100%;
		/* font-size: $font-sm+2upx; */
		/* color: $font-color-base; */
		text-align: center;
		text{
			/* color: $font-color-spec; */
			margin-left: 10upx;
		}
	}
</style>