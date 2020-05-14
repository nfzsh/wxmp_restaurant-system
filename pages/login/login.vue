<template>
	<view>
		<!-- #ifdef MP-WEIXIN -->
		<view v-if="isCanUse">
			<view>
				<view class='header'>
					<image src='../../static/wx_login.png'></image>
				</view>
				<view class='content'>
					<view>申请获取以下权限</view>
					<text>获得你的公开信息(昵称，头像、地区等)</text>
				</view>

				<button class='bottom' type='primary' open-type="getUserInfo" lang="zh_CN" @getuserinfo="wxGetUserInfo">
					授权登录
				</button>
			</view>
		</view>
		<!-- #endif -->
	</view>
</template>

<style>
	.header {
		margin: 90rpx 0 90rpx 50rpx;
		border-bottom: 1px solid #ccc;
		text-align: center;
		width: 650rpx;
		height: 300rpx;
		line-height: 450rpx;
	}

	.header image {
		width: 200rpx;
		height: 200rpx;
	}

	.content {
		margin-left: 50rpx;
		margin-bottom: 90rpx;
	}

	.content text {
		display: block;
		color: #9d9d9d;
		margin-top: 40rpx;
	}

	.bottom {
		border-radius: 80rpx;
		margin: 70rpx 50rpx;
		font-size: 35rpx;
	}
</style>

<script>
	export default {
		data() {
			return {
				nickName: null,
				avatarUrl: null,
				gender: null,
				province: null,
				city: null,
				country: null,
				isCanUse: uni.getStorageSync('isCanUse') || true //默认为true
			};
		},
		methods: {
			//第一授权获取用户信息===》按钮触发
			wxGetUserInfo() {
				let _this = this;
				uni.getUserInfo({
					provider: 'weixin',
					success: function(infoRes) {
						_this.nickName = infoRes.userInfo.nickName; //昵称
						_this.avatarUrl = infoRes.userInfo.avatarUrl; //头像
						_this.gender = infoRes.userInfo.gender;
						_this.province = infoRes.userInfo.province;
						_this.city = infoRes.userInfo.city;
						_this.country = infoRes.userInfo.country;
						try {
							uni.setStorageSync('isCanUse', false); //记录是否第一次授权  false:表示不是第一次授权
							_this.updateUserInfo();
						} catch (e) {}
					},
					fail(res) {}
				});
			},

			//登录
			login() {
				let _this = this;
				uni.showLoading({
					title: '登录中...'
				});

				// 1.wx获取登录用户code
				uni.login({
					provider: 'weixin',
					success: function(loginRes) {
						let code = loginRes.code;
						if (!_this.isCanUse) {
							//非第一次授权获取用户信息
							uni.getUserInfo({
								provider: 'weixin',
								success: function(infoRes) {
									//获取用户信息后向调用信息更新方法
									_this.nickName = infoRes.userInfo.nickName; //昵称
									_this.avatarUrl = infoRes.userInfo.avatarUrl; //头像
									_this.gender = infoRes.userInfo.gender;
									_this.province = infoRes.userInfo.province;
									_this.city = infoRes.userInfo.city;
									_this.country = infoRes.userInfo.country;
									_this.updateUserInfo(); //调用更新信息方法
								}
							});
						}

						//2.将用户登录code传递到后台置换用户SessionKey、OpenId等信息
						uni.request({
							url: 'http://localhost:8080/api/wechat/login',
							data: {
								code: code,
							},
							method: 'POST',
							header: {
								'content-type': 'application/x-www-form-urlencoded'
							},
							success: (res) => {
								//openId、或SessionKdy存储//隐藏loading
								uni.setStorageSync('token',res.header['token'])
								uni.hideLoading();
							}
						});
					},
				});
			},
			//向后台更新信息
			updateUserInfo() {
				let _this = this;
				uni.request({
					url: 'http://localhost:8080/api/wechat/getUserInfo', //服务器端地址
					data: {
						name: _this.nickName,
						avatarUrl: _this.avatarUrl,
						gender: _this.gender,
						province: _this.province,
						city: _this.city,
						country: _this.country
					},
					method: 'POST',
					header: {
						'content-type': 'application/json',
						'token': uni.getStorageSync('token')
					},
					success: (res) => {
						if (res.statusCode == 200) {
							uni.reLaunch({ //信息更新成功后跳转到小程序首页
								url: '/pages/index/index'
							});
						}
					}
				});
			}
		},
		onLoad() { //默认加载
			this.login();
		}
	}
</script>
