<template>
	<view>
		<uni-popup ref="message" type="message">
			<uni-popup-message :type="msgType" :message="messageText" :duration="2000"></uni-popup-message>
		</uni-popup>
		<uni-popup ref="inputDialog" type="dialog">
			<uni-file-picker v-model="imageValue" fileMediatype="all" mode="grid" @select="select" limit="1"
				sourceType></uni-file-picker>
			<uni-popup-dialog ref="inputClose" mode="input" title="发布内容"  placeholder="请输入内容"
				@confirm="dialogInputConfirm"></uni-popup-dialog>
		</uni-popup>
		<view class="example-body">
			<uni-fav :checked="checkList[0]" :content-text="contentText0" @click="favClick(0)" />
			<uni-fav :checked="checkList[1]" :content-text="contentText1" @click="favClick(1)" />
			<uni-fav :checked="checkList[2]" :content-text="contentText2" @click="favClick(2)" />
			<button size="mini" @click="goPost()" class="btn">发布信息</button>
		</view>
		<view style="width: 100%;">
			<uni-card :title="name" :sub-title="price" :extra="description" >
				<image style="width: 100%;" :src="link"></image>
				<view slot="actions" class="card-actions">
					<view class="card-actions-item">
						<uni-icons type="hand-up" size="18" color="#999"></uni-icons>
						<text class="card-actions-item-text">好评:{{goodNumber}}</text>
					</view>
					<view class="card-actions-item">
						<uni-icons type="hand-down" size="18" color="#999"></uni-icons>
						<text class="card-actions-item-text">差评:{{badNumber}}</text>
					</view>
					<view class="card-actions-item">
						<uni-icons type="star" size="18" color="#999"></uni-icons>
						<text class="card-actions-item-text">收藏:{{collectNumber}}</text>
					</view>
					<view class="card-actions-item">
						<uni-icons type="chatbubble" size="18" color="#999"></uni-icons>
						<text class="card-actions-item-text">评论:{{commentNumber}}</text>
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
	export default {
		data() {
			return {
				imageValue: [],
				msgType: '',
				messageText: '',
				id: "",
				name: "",
				description: "",
				goodNumber: "",
				badNumber: "",
				price: "",
				collectNumber: "",
				commentNumber: "",
				link: "",
				comment: [],
				checkList: [],
				value: "",
				contentText0: {
					contentDefault: '好评',
					contentFav: '已好评'
				},
				contentText1: {
					contentDefault: '差评',
					contentFav: '已差评'
				},
				contentText2: {
					contentDefault: '收藏',
					contentFav: '已收藏'
				}
			}
		},
		onLoad: function(option) {
			const item = JSON.parse(decodeURIComponent(option.id));
			this.id = item;
			uni.request({
				url: 'http://localhost:8080/dishes/selectDishesById',
				data: {
					id: this.id
				},
				header: {
					'userId': uni.getStorageSync("userId")
				},
				success: (res) => {
					console.log(res.data);
					if (res.data[0].status == "0") {
						console.log(res.data[3].rows.item[0]);
						this.name = res.data[3].rows.item[0].name,
							this.description = res.data[3].rows.item[0].description,
							this.goodNumber = res.data[3].rows.item[0].goodNumber,
							this.badNumber = res.data[3].rows.item[0].badNumber,
							this.price = res.data[3].rows.item[0].price + "元",
							this.collectNumber = res.data[3].rows.item[0].collectNumber,
							this.commentNumber = res.data[3].rows.item[0].commentNumber,
							this.link = res.data[3].rows.item[0].link
					} else {
						console.log("请登录");
						uni.navigateTo({
							url: '../user/login'
						});
					}
				}
			});
			uni.request({
				url: 'http://localhost:8080/comment/findCommentsByDishesId',
				data: {
					dishesId: this.id
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
				url: 'http://localhost:8080/dishes/isDishes',
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
						if (res.data.data.hateFlag == 0) {
							this.checkList[1] = false;
						} else {
							this.checkList[1] = true;
						}
						if (res.data.data.collectFlag == 0) {
							this.checkList[2] = false;
						} else {
							this.checkList[2] = true;
						}
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
				url: 'http://localhost:8080/dishes/selectDishesById',
				data: {
					id: this.id
				},
				header: {
					'userId': uni.getStorageSync("userId")
				},
				success: (res) => {
					console.log(res.data);
					if (res.data[0].status == "0") {
						console.log(res.data[3].rows.item[0]);
						this.name = res.data[3].rows.item[0].name,
							this.description = res.data[3].rows.item[0].description,
							this.goodNumber = res.data[3].rows.item[0].goodNumber,
							this.badNumber = res.data[3].rows.item[0].badNumber,
							this.price = res.data[3].rows.item[0].price + "元",
							this.collectNumber = res.data[3].rows.item[0].collectNumber,
							this.commentNumber = res.data[3].rows.item[0].commentNumber,
							this.link = res.data[3].rows.item[0].link
					} else {
						console.log("请登录");
						uni.navigateTo({
							url: '../user/login'
						});
					}
				}
			});
			uni.request({
				url: 'http://localhost:8080/comment/findCommentsByDishesId',
				data: {
					dishesId: this.id
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
			formSubmit: function(e) {
				if (this.value != "") {
					uni.request({
						url: 'http://localhost:8080/comment/dishesComment?userId=' + uni.getStorageSync(
							"userId") + '&dishesId=' + this.id + '&content=' + this.value,
						method: 'POST',
						header: {
							'userId': uni.getStorageSync("userId")
						},
						//仅为示例，并非真实接口地址。
						success: (res) => {
							if (res.data.code == "0000") {
								this.msgType = "success"
								this.messageText = "评论成功!"
								this.$refs.message.open()
								this.commentNumber = parseInt(this.commentNumber) + 1;
								this.value = "";
							} else if (res.data.code == "9999") {
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
						url: 'http://localhost:8080/comment/findCommentsByDishesId',
						data: {
							dishesId: this.id
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
			goPost() {
				this.$refs.inputDialog.open()
				// uni.navigateTo({
				// 	url: '../post/publish?id=' + this.id + '&name=' + this.name
				// });
			},
			dialogInputConfirm(val) {
				if (val != "") {
					uni.request({
						url: 'http://localhost:8080/post/addPost?userId=' + uni.getStorageSync(
								"userId") + '&dishesId=' + this.id + '&description=' + val +
							'&link=' + uni.getStorageSync(
								"link") + '&type=' + uni.getStorageSync(
								"type"),
						method: 'POST',
						header: {
							'userId': uni.getStorageSync("userId")
						},
						//仅为示例，并非真实接口地址。
						success: (res) => {
							uni.removeStorageSync('link');
							uni.removeStorageSync('type');
							if (res.data.code == "0000") {
								this.msgType = "success"
								this.messageText = "发布成功!"
								this.$refs.message.open()
							} else if (res.data.code == "9999") {
								this.msgType = "error"
								this.messageText = "发布失败!"
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
					console.log("请输入发布内容");
				}
			},
			select(e) {
				console.log('选择文件：', e)
				const tempFilePaths = e.tempFilePaths;
				const tempFiles = e.tempFiles;
				const imgUrl = tempFilePaths[0];
				const name = tempFiles[0].name;
				uniCloud.uploadFile({
					filePath: imgUrl,
					cloudPath: Date.now() + "-" + name,
					onUploadProgress: function(progressEvent) {
						console.log(progressEvent);
						var percentCompleted = Math.round(
							(progressEvent.loaded * 100) / progressEvent.total
						);
					},
					success: res => {
						console.log(res)
						uni.setStorageSync('link', res.fileID);
						if (e.tempFiles[0].extname == "jpg" || e.tempFiles[0].extname == "png" || e.tempFiles[
								0].extname == "jpeg" || e.tempFiles[0].extname == "gif" || e.tempFiles[0]
							.extname == "webp") {
							uni.setStorageSync('type', 0);
						} else if (e.tempFiles[0].extname == "mp4" || e.tempFiles[0].extname == "avi" || e
							.tempFiles[0].extname == "mpeg" || e.tempFiles[0].extname == "m4v" || e.tempFiles[
								0].extname == "mov") {
							uni.setStorageSync('type', 1);
						} else{
							console.log("上传的不是视频或图片")
						}
					},
					fail() {},
					complete() {}
				});
			},
			favClick(index) {
				if (index == 0) {
					if (this.checkList[0] == false) {
						uni.request({
							url: 'http://localhost:8080/like/dishesLike?userId=' + uni.getStorageSync("userId") +
								'&dishesId=' + this.id,
							method: 'POST',
							header: {
								'userId': uni.getStorageSync("userId")
							},
							success: (res) => {
								if (res.data.code == "0000") {
									console.log("好评成功");
									this.goodNumber = parseInt(this.goodNumber) + 1;
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
							url: 'http://localhost:8080/like/dishesLikeNo?userId=' + uni.getStorageSync("userId") +
								'&dishesId=' + this.id,
							method: 'POST',
							header: {
								'userId': uni.getStorageSync("userId")
							},
							success: (res) => {
								if (res.data.code == "0000") {
									console.log("取消好评");
									this.goodNumber = parseInt(this.goodNumber) - 1;
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
							url: 'http://localhost:8080/hate/dishesBad?userId=' + uni.getStorageSync("userId") +
								'&dishesId=' + this.id,
							method: 'POST',
							header: {
								'userId': uni.getStorageSync("userId")
							},
							success: (res) => {
								if (res.data.code == "0000") {
									console.log("差评成功");
									this.badNumber = parseInt(this.badNumber) + 1;
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
							url: 'http://localhost:8080/hate/dishesBadNo?userId=' + uni.getStorageSync("userId") +
								'&dishesId=' + this.id,
							method: 'POST',
							header: {
								'userId': uni.getStorageSync("userId")
							},
							success: (res) => {
								if (res.data.code == "0000") {
									console.log("取消差评");
									this.badNumber = parseInt(this.badNumber) - 1;
								} else if (res.data.code == "7777") {
									console.log("请登录");
									uni.navigateTo({
										url: '../user/login'
									});
								}
							}
						});
					}
				} else if (index == 2) {
					if (this.checkList[2] == false) {
						uni.request({
							url: 'http://localhost:8080/collect/dishesCollect?userId=' + uni.getStorageSync(
									"userId") +
								'&dishesId=' + this.id,
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
							url: 'http://localhost:8080/collect/dishesCollectNo?userId=' + uni.getStorageSync(
									"userId") +
								'&dishesId=' + this.id,
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
		}
	}
</script>

<style>
	.example-body {
		display: flex;
		padding: 10px 15px;
	}

	,
	button:hover {
		background-color: #0062cc;
		box-shadow: 0px 5px 15px rgba(0, 0, 0, 0.2);
		transform: translateY(-2px);
	}

	,
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
		width: 90px;
		height: 30px;
		line-height: 18px;
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