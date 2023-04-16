<template>
	<view>
		<uni-popup ref="message" type="message">
			<uni-popup-message :type="msgType" :message="messageText" :duration="2000"></uni-popup-message>
		</uni-popup>
		<view class="example-body">
			<uni-fav :checked="checkList[0]" :content-text="contentText0" @click="favClick(0)" />
			<uni-fav :checked="checkList[1]" :content-text="contentText1" @click="favClick(1)" />
			<button @click="gotoTa()" size="mini" class="btn">他的主页</button>
		</view>

		<view style="width: 100%;">
			<uni-card :title="nickName" :extra="name" >
				<text class="uni-body">{{description}}</text>
				<view v-if="type == 0">
					<image style="width: 100%;" :src="link"></image>
				</view>
				<view v-else-if="type == 1">
					<xiaoVideoElement :src="link">
					</xiaoVideoElement>
				</view>
				<view slot="actions" class="card-actions">
					<view class="card-actions-item">
						<uni-icons type="star" size="18" color="#999"></uni-icons>
						<text class="card-actions-item-text">点赞量：{{likeNumber}}</text>
					</view>
					<view class="card-actions-item">
						<uni-icons type="heart" size="18" color="#999"></uni-icons>
						<text class="card-actions-item-text">收藏量：{{collectNumber}}</text>
					</view>
					<view class="card-actions-item">
						<uni-icons type="chatbubble" size="18" color="#999"></uni-icons>
						<text class="card-actions-item-text">评论量：{{commentNumber}}</text>
					</view>
				</view>
			</uni-card>
		</view>
		<form @submit="formSubmit" @reset="formReset">
			<view class="uni-form-item uni-column">
				<uni-easyinput type="textarea" autoHeight v-model="value" placeholder="请输入评论"></uni-easyinput>
			</view>
			<view class="uni-btn-v">
				<button form-type="submit" size="mini" class="btn">评论</button>
			</view>
		</form>
		<view v-for="(item, index) in comment">
			<view style="width: 100%;">
				<uni-card :title="item.nickName" :extra="item.description" :isFull="true" is-shadow="true">
					<text class="uni-body">{{ item.content }}</text>
				</uni-card>
			</view>
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
				id: "",
				description: "",
				type: "",
				link: "",
				name: "",
				nickName: "",
				likeNumber: "",
				collectNumber: "",
				commentNumber: "",
				taId: "",
				value: "",
				comment: [],
				checkList: [],
				contentText0: {
					contentDefault: '点赞',
					contentFav: '已点赞'
				},
				contentText1: {
					contentDefault: '收藏',
					contentFav: '已收藏'
				}
			}
		},
		onLoad: function(option) {
			this.id = option.id,
				this.name = option.name,
				this.description = option.description,
				this.type = option.type,
				this.link = option.link,
				this.nickName = option.nickName,
				this.likeNumber = option.likeNumber,
				this.collectNumber = option.collectNumber,
				this.commentNumber = option.commentNumber,
				this.taId = option.taId
			uni.request({
				url: 'http://localhost:8080/comment/findCommentsByPostId',
				data: {
					postId: this.id
				},
				header: {
					'userId': uni.getStorageSync("userId")
				},
				success: (res) => {
					if (res.data[0].status == "0") {
						this.comment = res.data[3].rows.item
						for (let i = 0; i < res.data[3].rows.item.length; i++) {
							this.comment[i].nickName = this.comment[i].nickName + "说："
						}
					} else {
						console.log("请登录");
						uni.navigateTo({
							url: '../user/login'
						});
					}
				}
			});
			uni.request({
				url: 'http://localhost:8080/post/isPost',
				data: {
					dishesId: this.id
				},
				header: {
					'userId': uni.getStorageSync("userId")
				},
				success: (res) => {
					console.log(res.data)
					if (res.data.code == "0000") {
						if (res.data.data.likeFlag == 0) {
							this.checkList[0] = false;
						} else {
							this.checkList[0] = true;
						}
						if (res.data.data.collectFlag == 0) {
							this.checkList[1] = false;
						} else {
							this.checkList[1] = true;
						}
						console.log(this.checkList[0]);
						console.log(this.checkList[1]);
						this.$forceUpdate()
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
				url: 'http://localhost:8080/comment/findCommentsByPostId',
				data: {
					postId: this.id
				},
				header: {
					'userId': uni.getStorageSync("userId")
				},
				success: (res) => {
					if (res.data[0].status == "0") {
						this.comment = res.data[3].rows.item
						for (let i = 0; i < res.data[3].rows.item.length; i++) {
							this.comment[i].nickName = this.comment[i].nickName + "说："
						}
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
			gotoTa() {
				uni.navigateTo({
					url: '../user/ta?taId=' + this.taId
				});
			},
			imageError: function(e) {
				console.error('image发生error事件，携带值为' + e.detail.errMsg)
			},
			videoErrorCallback: function(e) {
				uni.showModal({
					content: e.target.errMsg,
					showCancel: false
				})
			},
			formSubmit: function(e) {
				if (this.value != "") {
					uni.request({
						url: 'http://localhost:8080/comment/postComment?userId=' + uni.getStorageSync(
							"userId") + '&postId=' + this.id + '&content=' + this.value,
						method: 'POST',
						header: {
							'userId': uni.getStorageSync("userId")
						},
						//仅为示例，并非真实接口地址。
						success: (res) => {
							if (res.data.code == "0000") {
								console.log("评论成功");
								this.msgType = "success"
								this.messageText = "评论成功!"
								this.$refs.message.open()
								this.commentNumber = parseInt(this.commentNumber) + 1;
								this.value = "";
							} else if (res.data.code == "9999") {
								console.log("评论失败");
								this.msgType = "error"
								this.messageText = "评论失败!"
								this.$refs.message.open()
							} else if (res.data.code == "7777") {
								console.log("请登录");
								uni.navigateTo({
									url: '../user/login'
								});
							}
						}
					});
					uni.request({
						url: 'http://localhost:8080/comment/findCommentsByPostId',
						data: {
							postId: this.id
						},
						header: {
							'userId': uni.getStorageSync("userId")
						},
						success: (res) => {
							if (res.data[0].status == "0") {
								this.comment = res.data[3].rows.item
								for (let i = 0; i < res.data[3].rows.item.length; i++) {
									this.comment[i].nickName = this.comment[i].nickName + "说："
								}
							} else {
								console.log("请登录");
								uni.navigateTo({
									url: '../user/login'
								});
							}
						}
					});
				} else {
					console.log("请输入评论");
				}
			},
			favClick(index) {
				if (index == 0) {
					if (this.checkList[0] == false) {
						uni.request({
							url: 'http://localhost:8080/like/postLike?userId=' + uni.getStorageSync("userId") +
								'&postId=' + this.id,
							method: 'POST',
							header: {
								'userId': uni.getStorageSync("userId")
							},
							success: (res) => {
								if (res.data.code == "0000") {
									console.log("点赞成功");
									this.likeNumber = parseInt(this.likeNumber) + 1;
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
								'&postId=' + this.id,
							method: 'POST',
							header: {
								'userId': uni.getStorageSync("userId")
							},
							success: (res) => {
								if (res.data.code == "0000") {
									console.log("取消点赞");
									this.likeNumber = parseInt(this.likeNumber) - 1;
								} else if (res.data.code == "7777") {
									console.log("请登录");
									uni.navigateTo({
										url: '../user/login'
									});
								}
							}
						});
					}
				} else if (index == 1) {
					if (this.checkList[1] == false) {
						uni.request({
							url: 'http://localhost:8080/collect/postCollect?userId=' + uni.getStorageSync(
									"userId") +
								'&postId=' + this.id,
							method: 'POST',
							header: {
								'userId': uni.getStorageSync("userId")
							},
							success: (res) => {
								if (res.data.code == "0000") {
									console.log("收藏成功");
									this.collectNumber = parseInt(this.collectNumber) + 1;
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
								'&postId=' + this.id,
							method: 'POST',
							header: {
								'userId': uni.getStorageSync("userId")
							},
							success: (res) => {
								if (res.data.code == "0000") {
									console.log("取消收藏");
									this.collectNumber = parseInt(this.collectNumber) - 1;
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
				this.checkList[index] = !this.checkList[index];
				this.$forceUpdate()
			}
		},
		components: {
			xiaoVideoElement
		}

	}
</script>

<style>
	button:hover {
		background-color: #0062cc;
		box-shadow: 0px 5px 15px rgba(0, 0, 0, 0.2);
		transform: translateY(-2px);
	}

	.btn {
		background-color: #007bff;
		color: #fff;
		border: none;
		padding: 10px 20px;
		font-size: 16px;
		border-radius: 30px;
		cursor: pointer;
		box-shadow: 0px 2px 10px rgba(0, 0, 0, 0.1);
		transition: all 0.2s ease-in-out;
		width: 100px;
		height: 30px;
		line-height: 18px;
	}

	,
	.example-body {
		display: flex;
		padding: 10px 15px;
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
</style>