<template>
	<view>
		<uni-card v-for="(item,index) in bill" :key="index" title="东林美食" mode="title" :is-shadow="true" thumbnail="https://5b0988e595225.cdn.sohucs.com/images/20180123/c82e7abcb4f84a3c8e3e24e6b84e8c26.png"
		 :extra="date(item.payTime)" :note="'共计'+item.price+'元'">
			<view v-for="(menu,i) in list" v-if="menu.bill.id==item.id" :key="i">
				<view class="flex"><text class="flex-sub">{{menu.menu.name}}</text>
					<text class="flex-sub">单价：{{menu.price}}</text>
					<text class="flex-sub">数量：{{menu.num}}</text></view>
			</view>
			<view><button style="text-align: right;" class="cu-btn bg-green round lg shadow" @click="showDiv(item.id)">点击评价</button></view>
		</uni-card>
		<view :hidden="userFeedbackHidden" class="popup_content">
			<view class="popup_title">写下您的反馈意见</view>
			<view class="popup_textarea_item">
				<textarea class="popup_textarea" placeholder='有什么想告诉我们的...' v-model="feedbackContent">
						</textarea>
				<view @click="submitFeedback()">
					<button class="popup_button">提交反馈</button>
				</view>
			</view>
		</view>
		<view class="popup_overlay" :hidden="userFeedbackHidden" @click="hideDiv()"></view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				bill: [],
				list: [],
				userFeedbackHidden: true, // 默认隐藏
				feedbackContent: '', // 用户反馈内容
				bid: null
			}
		},
		onLoad() {
			let _this = this
			uni.request({
				url: 'http://localhost:8080/api/wechat/selectBill',
				method: 'GET',
				header: {
					'token': uni.getStorageSync('token')
				},
				success: (res) => {
					if (res.statusCode == 200) {
						_this.bill = res.data.bill
						for (let i = 0; i < _this.bill.length; i++) {
							let id = _this.bill[i].id
							console.log(id)
							uni.request({
								url: "http://localhost:8080/api/wechat/selectLists/" + id,
								method: 'GET',
								header: {
									'token': uni.getStorageSync('token')
								},
								success: (res) => {
									for (let j = 0; j < res.data.list.length; j++) {
										_this.list.push(res.data.list[j])
									}
									console.log(_this.list)
								}
							})
						}
					}
				}
			})

		},
		methods: {
			date(payTime) {
				return "付款时间：" + payTime.replace("T", " ").substring(0, 19)
			},
			showDiv(id) { // 显示输入弹出框
				this.bid = id;
				this.userFeedbackHidden = false;
			},
			hideDiv() { // 隐藏输入弹出框
				this.userFeedbackHidden = true;
			},
			submitFeedback() { // 提交反馈
				var _this = this;
				this.userFeedbackHidden = true;
				// 提交反馈内容
				uni.request({
					url:'http://localhost:8080/api/wechat/note',
					data:{
						id: _this.bid,
						note: _this.feedbackContent
					},
					method:'POST',
					header: {
						'token': uni.getStorageSync('token')
					},
					success: (res) => {
						if(res.statusCode==200){
							uni-uni.showToast({
								title: '提交成功！',
								icon:"success"
							});
						}
					}
				})
			}

		}
	}
</script>

<style>
	.popup_overlay {

		position: fixed;
		top: 0%;
		left: 0%;
		width: 100%;
		height: 100%;
		background-color: black;
		z-index: 1001;
		-moz-opacity: 0.8;
		opacity: .80;
		filter: alpha(opacity=88);
	}

	.popup_content {
		position: fixed;
		top: 50%;
		left: 50%;
		width: 520upx;
		height: 550upx;
		margin-left: -270upx;
		margin-top: -270upx;
		border: 10px solid white;
		background-color: white;
		z-index: 1002;
		overflow: auto;
		border-radius: 20upx;
	}

	.popup_title {
		padding-top: 20upx;
		width: 480upx;
		text-align: center;
		font-size: 32upx;
	}

	.popup_textarea_item {
		padding-top: 5upx;
		height: 240upx;
		width: 440upx;
		background-color: #F1F1F1;
		margin-top: 30upx;
		margin-left: 20upx;
	}

	.popup_textarea {
		width: 410upx;
		font-size: 26upx;
		margin-left: 20upx;
	}

	.popup_button {
		color: white;
		background-color: #4399FC;
		border-radius: 20upx;
	}
</style>
