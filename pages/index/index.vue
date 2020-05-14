<template>
	<view>
		<uni-card v-if="statuec" title="无需排队" thumbnail="http://img3.imgtn.bdimg.com/it/u=3472891741,1825282638&fm=26&gp=0.jpg"
		 note="友情提示s:避免高峰出行哦">
			无需排队，快来享用美食吧
		</uni-card>
		<uni-card v-if="statue==true&&index!=-1" title="您的号码为:" thumbnail="http://img3.imgtn.bdimg.com/it/u=3472891741,1825282638&fm=26&gp=0.jpg"
		 v-bind:note=note>
			<text style="color: red;font-size: 80rpx;text-align: center;">{{num}}</text>
			<br />
			<text style="font-size: 40rpx;">您前方还有{{index}}人</text>
			<image src="../../static/bg.jpg"></image>
		</uni-card>
		<view v-if="statue==true&&index==-1">
			<text style="font-size: 80rpx;">部分位置正在排队:</text>
			<view class="uni-title uni-common-mt uni-common-pl">请选择预约餐桌类型：</view>
			<view class="uni-list">
				<radio-group @change="radioChange">
					<label class="uni-list-cell uni-list-cell-pd" v-for="(item, index) in table" :key="item.num">
						<view>
							<radio v-if="item.count==0" :value="item.num" :checked="index === current" />
						</view>
						<view v-if="item.count==0">{{item.num}}人桌</view>
					</label>
				</radio-group>
			</view>
			<button class='bottom' type='primary' lang="zh_CN" @click="push(tn)">预约</button>
		</view>
	</view>
</template>

<style>
	.bg {
		height: 100%;
		background: url("https://cdn.nlark.com/yuque/0/2019/png/280373/1552908032971-assets/web-upload/e795fee4-c6cd-4951-b7d8-134aa9cd7c6b.png");
		background-size: cover;
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
				table: [],
				tablen: [],
				statue: false,
				index: null, //序号
				num: null, //编号
				tn: null, //桌子人数
				note: null,
				statuec: false,
				current: 0
			}
		},
		onLoad: function() {
			let _this = this;
			//总共数量
			uni.request({
				url: 'http://localhost:8080/api/wechat/getTableNumAll',
				method: 'GET',
				header: {
					'token': uni.getStorageSync('token')
				},
				success: (res) => {
					if (res.statusCode == 200) {
						_this.tablen = res.data.tableNumAll
					}
				}
			});
			//剩余数量
			uni.request({
				url: 'http://localhost:8080/api/wechat/getTableNum',
				method: 'GET',
				header: {
					'token': uni.getStorageSync('token')
				},
				success: (res) => {
					if (res.statusCode == 200) {
						_this.table = res.data.tableNum
						if (_this.table.length == _this.tablen.length) {
							_this.statue = false
							_this.statuec = false
						} else {
							_this.statue = true;
							_this.tn = _this.table[0].num
							uni.request({
								url: 'http://localhost:8080/api/wechat/selectIndex',
								method: 'GET',
								header: {
									'token': uni.getStorageSync('token')
								},
								success: (res) => {
									if (res.data.m == -1) {
										_this.index = -1
									} else {
										_this.index = res.data.m, //前面人数
											_this.num = res.data.num //编号
										if (res.data.m < 2)
											_this.note = '预计还有5分钟'
										else if (res.data.m < 5)
											_this.note = '预计还有10分钟'
										else if (res.data.m < 10)
											_this.note = '预计还有30分钟'
										else if (res.data.m < 20)
											_this.note = '预计还有1小时'
										else
											_this.note = '预计还有1小时以上'
									}
								}
							})
						}
					}
				}
			});

		},
		methods: {
			radioChange: function(evt) {
				let _this = this
				_this.tn = evt.detail.value
				for (let i = 0; i < _this.table.length; i++) {
					if (_this.table[i].num === evt.target.num) {
						_this.current = i;
						break;
					}
				}
			},
			push(tn) {
				let _this = this
				uni.request({
					url: `http://localhost:8080/api/wechat/add/${tn}`,
					method: 'GET',
					header: {
						'token': uni.getStorageSync('token')
					},
					success:(res) => {
						_this.num = res.data.num
						uni.request({
							url: 'http://localhost:8080/api/wechat/selectIndex',

							method: 'GET',

							header: {

								'token': uni.getStorageSync('token')
							},
							success:(res) => {
								_this.index=res.data.m
								if (res.data.m < 2)
									_this.note = '预计还有5分钟'
								else if (res.data.m < 5)
									_this.note = '预计还有10分钟'
								else if (res.data.m < 10)
									_this.note = '预计还有30分钟'
								else if (res.data.m < 20)
									_this.note = '预计还有1小时'
								else
									_this.note = '预计还有1小时以上'
							}
						})
					}
				})
			}
		}
	}
</script>
