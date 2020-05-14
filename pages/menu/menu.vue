<template>
	<view>
		<uni-popup ref="popup" type="message">
			<uni-popup-message type="success" message="结算成功" :duration="2000"></uni-popup-message>
		</uni-popup>
		<button v-if="isopen" class='bottom' type='primary' lang="zh_CN" @click="push">点击扫码</button>
		<view v-else>
			<swiper class="screen-swiper round-dot" :indicator-dots="true" :circular="true" :autoplay="true" interval="5000"
			 duration="500">
				<swiper-item v-for="(item,index) in 4" :key="index">
					<image :src="'/static/'+index+'.jpg'" mode="aspectFill"></image>
				</swiper-item>
			</swiper>
			<view class="VerticalBox">
				<scroll-view class="VerticalNav nav" scroll-y scroll-with-animation :scroll-top="verticalNavTop" style="height:calc(100vh - 375upx)">
					<view class="cu-item" :class="index==tabCur?'text-green cur':''" v-for="(item,index) in list" :key="index" @tap="TabSelect"
					 :data-id="index">
						{{item.name}}
					</view>
				</scroll-view>
				<scroll-view class="VerticalMain" scroll-y scroll-with-animation style="height:calc(100vh - 375upx)"
				 :scroll-into-view="'main-'+mainCur" @scroll="VerticalMain">
					<view class="padding-top padding-lr" v-for="(item,index) in list" :key="index" :id="'main-'+index">
						<view class="cu-bar solid-bottom bg-white">
							<view class="action">
								<text class="cuIcon-title text-green"></text> {{item.name}}</view>
						</view>
						<view class="cu-list menu-avatar">

							<view class="cu-item" v-for="(food,i) in menu" v-if="food.type==index" :key="i" style="height: 250upx">
								<view class="cu-avatar radius lg" :style="'background-image:url(http://localhost:8080/api/wechat/show/'+food.pic+')'">
								</view>
								<view class="content">
									<view class="text-pink"><text class="text-cut">{{food.name}}</text></view>
									<view class="text-gray text-sm flex"> <text class="text-cut">{{food.message}}</text>
									</view>
									<view></view><text class="text-cut" style="color: red;">{{food.price}}元</text>
									<view></view>
									<view>
										<uni-number-box :key="i" value="0" @change="add($event,food)"></uni-number-box>
									</view>
								</view>
							</view>

						</view>
					</view>
					<view style="float: right; display: inline;width: 100%;" class="button_div content">
						<button style="text-align: right;" class="cu-btn bg-green round lg shadow" @click="buy">结算</button>
					</view>
				</scroll-view>
			</view>
		</view>
	</view>
</template>

<script>
	import uniNumberBox from "@/components/uni-number-box/uni-number-box.vue"
	export default {
		components: {
			uniNumberBox
		},
		data() {
			return {
				isopen: true,
				tn: null,
				list: [{
						name: "主食",
						id: '0'
					},
					{
						name: '菜品',
						id: '1'
					},
					{
						name: '特色菜',
						id: '2'
					},
					{
						name: '酒水',
						id: '3'
					},
					{
						name: '饮料',
						id: '4'
					}
				],
				menu: [],
				tabCur: 0,
				mainCur: 0,
				verticalNavTop: 0,
				load: true,
				lists: []
			}
		},
		onLoad() {
			uni.showLoading({
				title: '加载中...',
				mask: true
			});
			let _this = this;
			uni.request({
				url: 'http://localhost:8080/api/wechat/getAllMenu',
				method: 'GET',
				header: {
					'token': uni.getStorageSync('token')
				},
				success: (res) => {
					if (res.statusCode == 200)
						_this.menu = res.data.menus
					for (let i = 0; i < _this.menu.length; i++) {
						_this.menu['value'] = 0
					}
				}
			})
		},
		onReady() {
			uni.hideLoading()
		},
		methods: {
			TabSelect(e) {
				this.tabCur = e.currentTarget.dataset.id;
				this.mainCur = e.currentTarget.dataset.id;
				this.verticalNavTop = (e.currentTarget.dataset.id - 1) * 50
			},
			VerticalMain(e) {
				// #ifdef MP-ALIPAY
				return false //支付宝小程序暂时不支持双向联动 
				// #endif
				let that = this;
				let tabHeight = 0;
				if (this.load) {
					for (let i = 0; i < this.list.length; i++) {
						let view = uni.createSelectorQuery().select("#main-" + this.list[i].id);
						view.fields({
							size: true
						}, data => {
							this.list[i].top = tabHeight;
							tabHeight = tabHeight + data.height;
							this.list[i].bottom = tabHeight;
						}).exec();
					}
					this.load = false
				}
				let scrollTop = e.detail.scrollTop + 10;
				for (let i = 0; i < this.list.length; i++) {
					if (scrollTop > this.list[i].top && scrollTop < this.list[i].bottom) {
						this.verticalNavTop = (this.list[i].id - 1) * 50
						this.tabCur = this.list[i].id
						return false
					}
				}
			},
			add(value, food) {
				let _this = this;
				for (let i = 0; i < _this.menu.length; i++) {
					if (_this.menu[i].id == food.id) {
						_this.menu[i]["value"] = value;
					}
				}
			},
			push() {
				let _this = this
				uni.scanCode({
					success: (result) => {
						_this.isopen = false
						_this.tn = result.result
					}
				})
			},
			buy() {
				let _this = this
				for (let i = 0; i < _this.menu.length; i++) {
					if (_this.menu[i].value != null) {
						uni.request({
							url: 'http://localhost:8080/api/wechat/addList',
							data: {
								tables: {
									id: _this.tn
								},
								menu: {
									id: _this.menu[i].id
								},
								num: _this.menu[i].value,
								price: _this.menu[i].price
							},
							method: 'POST',
							header: {
								'token': uni.getStorageSync('token')
							},

							success: (res) => {
								uni-uni.showToast({
									title: '提交成功！',
									icon:"success"
								});
								if (res.statusCode == 200) {
									for(let i=0;i<_this.menu.length;i++){
										_this.menu[i].value=0
									}
								}
							}
						})
					}

				}
			},
			open() {
				this.$refs.popup.open()
			}
		}
	}
</script>

<style>
	.button_div {
		position: fixed;
		bottom: var(--window-bottom);
	}

	.bottom {
		border-radius: 80rpx;
		margin: 70rpx 50rpx;
		font-size: 35rpx;
	}

	.fixed {
		position: fixed;
		z-index: 99;
	}

	.VerticalNav.nav {
		width: 200upx;
		white-space: initial;
	}

	.VerticalNav.nav .cu-item {
		width: 100%;
		text-align: center;
		background-color: #fff;
		margin: 0;
		border: none;
		height: 50px;
		position: relative;
	}

	.VerticalNav.nav .cu-item.cur {
		background-color: #f1f1f1;
	}

	.VerticalNav.nav .cu-item.cur::after {
		content: "";
		width: 8upx;
		height: 30upx;
		border-radius: 10upx 0 0 10upx;
		position: absolute;
		background-color: currentColor;
		top: 0;
		right: 0upx;
		bottom: 0;
		margin: auto;
	}

	.VerticalBox {
		display: flex;
	}

	.VerticalMain {
		background-color: #f1f1f1;
		flex: 1;
	}
</style>
