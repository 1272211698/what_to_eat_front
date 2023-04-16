<template>
	<view class="content">
		<uni-popup ref="message" type="message">
			<uni-popup-message :type="msgType" :message="messageText" :duration="2000"></uni-popup-message>
		</uni-popup>
		<view v-for="(item, index) in postArr" style="width: 100%;">
			<uni-section title="双标题卡片" type="line">
				<uni-card :title="item.nickName" :extra="item.name">
					<text class="uni-body">{{item.description}}</text>
					<view v-if="item.type == 0">
						<image style="width: 100%;" :src="item.link"></image>
					</view>
					<view v-else-if="item.type == 1">
						<xiaoVideoElement :src="item.link">
						</xiaoVideoElement>
					</view>
					<view slot="actions" class="card-actions">
						<view class="card-actions-item" @click="favClick(1,item.id)">
							<uni-icons type="star" size="18" color="#999"></uni-icons>
							<text class="card-actions-item-text">收藏</text>
						</view>
						<view class="card-actions-item" @click="favClick(0,item.id)">
							<uni-icons type="heart" size="18" color="#999"></uni-icons>
							<text class="card-actions-item-text">点赞</text>
						</view>
						<view class="card-actions-item"
							@click="detail(item.id,item.link,item.type,item.description,item.nickName,item.name,item.likeNumber,item.collectNumber,item.commentNumber,item.userId)">
							<uni-icons type="more" size="18" color="#999"></uni-icons>
							<text class="card-actions-item-text">详情</text>
						</view>
					</view>
				</uni-card>
			</uni-section>
		</view>

	</view>
</template>

<script>
	import xiaoVideoElement from '../../components/xiao-video-component/xiao-video-component.vue'
	export default {
		data() {
			return {
				msgType: '',
				messageText: '',
				postArr: [],
				page: 1,
				limit: 100
			}
		},
		onShow: function(e) { //nvue暂不支持滚动监听，可用bindingx代替
			uni.request({
				url: 'http://localhost:8080/post/selectPostAllVo',
				data: {
					page: this.page,
					limit: this.limit
				}, //仅为示例，并非真实接口地址。
				header: {
					'userId': uni.getStorageSync("userId")
				},
				success: (res) => {
					if (res.data.code == "0000") {
						this.postArr = res.data.data;
						console.log(res.data);
					} else {
						console.log("请登录");
						uni.navigateTo({
							url: '../user/login'
						});
					}
				}
			});
		},
		onLoad() {
			uni.request({
				url: 'http://localhost:8080/post/selectPostAllVo',
				data: {
					page: this.page,
					limit: this.limit
				}, //仅为示例，并非真实接口地址。
				header: {
					'userId': uni.getStorageSync("userId")
				},
				success: (res) => {
					if (res.data.code == "0000") {
						this.postArr = res.data.data;
						console.log(res.data);
					} else {
						console.log("请登录");
						uni.navigateTo({
							url: '../user/login'
						});
					}
				}
			});
		},
		onPullDownRefresh() {
			uni.request({
				url: 'http://localhost:8080/post/selectPostAllVo',
				data: {
					page: this.page,
					limit: this.limit
				}, //仅为示例，并非真实接口地址。
				header: {
					'userId': uni.getStorageSync("userId")
				},
				success: (res) => {
					if (res.data.code == "0000") {
						this.postArr = res.data.data;
						console.log(res.data);
					} else {
						console.log("请登录");
						uni.navigateTo({
							url: '../user/login'
						});
					}
				}
			});
			setTimeout(function() {
				uni.stopPullDownRefresh();
			}, 500);
		},
		methods: {
			favClick(flag, id) {
				uni.request({
					url: 'http://localhost:8080/post/isPost',
					data: {
						dishesId: id
					},
					header: {
						'userId': uni.getStorageSync("userId")
					},
					success: (res) => {
						if (res.data.code == "0000") {
							if (flag == 0) {
								if (res.data.data.likeFlag == 0) {
									uni.request({
										url: 'http://localhost:8080/like/postLike?userId=' + uni
											.getStorageSync("userId") +
											'&postId=' + id,
										method: 'POST',
										header: {
											'userId': uni.getStorageSync("userId")
										},
										success: (res) => {
											if (res.data.code == "0000") {
												this.msgType = "success"
												this.messageText = "点赞成功!"
												this.$refs.message.open()
											} else if (res.data.code == "7777") {
												console.log("请登录");
												uni.navigateTo({
													url: '../user/login'
												});
											}
										}
									});
								} else {
									uni.request({
										url: 'http://localhost:8080/like/postLikeNo?userId=' + uni.getStorageSync("userId") +
											'&postId=' + id,
										method: 'POST',
										header: {
											'userId': uni.getStorageSync("userId")
										},
										success: (res) => {
											if (res.data.code == "0000") {
												this.msgType = "warn"
												this.messageText = "取消点赞"
												this.$refs.message.open()
											} else if (res.data.code == "7777") {
												console.log("请登录");
												uni.navigateTo({
													url: '../user/login'
												});
											}
										}
									});
								}
							} else {
								if (res.data.data.collectFlag == 0) {
									uni.request({
										url: 'http://localhost:8080/collect/postCollect?userId=' + uni
											.getStorageSync(
												"userId") +
											'&postId=' + id,
										method: 'POST',
										header: {
											'userId': uni.getStorageSync("userId")
										},
										success: (res) => {
											if (res.data.code == "0000") {
												this.msgType = "success"
												this.messageText = "收藏成功!"
												this.$refs.message.open()
											} else if (res.data.code == "7777") {
												console.log("请登录");
												uni.navigateTo({
													url: '../user/login'
												});
											}
										}
									});
								} else {
									uni.request({
										url: 'http://localhost:8080/collect/postCollectNo?userId=' + uni.getStorageSync(
												"userId") +
											'&postId=' + id,
										method: 'POST',
										header: {
											'userId': uni.getStorageSync("userId")
										},
										success: (res) => {
											if (res.data.code == "0000") {
												this.msgType = "warn"
												this.messageText = "取消收藏"
												this.$refs.message.open()
											} else if (res.data.code == "7777") {
												console.log("请登录");
												uni.navigateTo({
													url: '../user/login'
												});
											}
										}
									});
								}
							}
						} else {
							console.log("请登录");
							uni.navigateTo({
								url: '../user/login'
							});
						}
					}
				});
			},
			videoErrorCallback: function(e) {
				uni.showModal({
					content: e.target.errMsg,
					showCancel: false
				})
			},
			imageError: function(e) {
				console.error('image发生error事件，携带值为' + e.detail.errMsg)
			},
			detail(id, link, type, description, nickName, name, likeNumber, collectNumber, commentNumber, userId) {
				uni.navigateTo({
					url: '../detail/postDetail?id=' + id + '&link=' + link + '&type=' + type + '&description=' +
						description + '&nickName=' + nickName + '&name=' + name + '&likeNumber=' + likeNumber +
						'&collectNumber=' + collectNumber + '&commentNumber=' + commentNumber + '&taId=' + userId
				});
			}
		},
		components: {
			xiaoVideoElement
		}
	}
</script>

<style>
	.content {
		overflow: hidden;
	}


	.custom-cover {
		flex: 1;
		flex-direction: row;
		position: relative;
	}

	.cover-content {
		position: absolute;
		bottom: 0;
		left: 0;
		right: 0;
		height: 40px;
		background-color: rgba($color: #000000, $alpha: 0.4);
		display: flex;
		flex-direction: row;
		align-items: center;
		padding-left: 15px;
		font-size: 14px;
		color: #fff;
	}

	.card-actions {
		display: flex;
		flex-direction: row;
		justify-content: space-around;
		align-items: center;
		height: 45px;
		border-top: 1px #eee solid;
	}

	.card-actions-item {
		display: flex;
		flex-direction: row;
		align-items: center;
	}

	.card-actions-item-text {
		font-size: 12px;
		color: #666;
		margin-left: 5px;
	}

	.cover-image {
		flex: 1;
		height: 150px;
	}

	.no-border {
		border-width: 0;
	}
</style>