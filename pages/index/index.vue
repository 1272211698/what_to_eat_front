<template>
	<view class="content">
		<uni-popup ref="message" type="message">
			<uni-popup-message :type="msgType" :message="messageText" :duration="2000"></uni-popup-message>
		</uni-popup>
		<view class="uni-margin-wrap">
			<uni-swiper-dot :info="info" :current="current" field="content" mode="nav">
				<swiper class="swiper" @change="change" circular :indicator-dots="true" :autoplay="true"
					:interval="2000" :duration="500">
					<swiper-item>
						<image style="width: 100%;height: 100%;"
							src="https://mp-2a38b404-c00b-4fe8-ab5b-3dfc261f7584.cdn.bspapp.com/cloudstorage/d3a8ea7e-8556-4910-9834-67ee5089135c.png">
						</image>
					</swiper-item>
					<swiper-item>
						<image style="width: 100%;height: 100%;"
							src="https://mp-2a38b404-c00b-4fe8-ab5b-3dfc261f7584.cdn.bspapp.com/cloudstorage/12064e82-0f0f-49e9-bf3b-0f2d3e824271.jpg">
						</image>
					</swiper-item>
				</swiper>
			</uni-swiper-dot>
		</view>
		<view class="vi">
			<uni-search-bar :radius="100" clearButton="auto" @confirm="search" v-model="searchValue">
			</uni-search-bar>
			<button @click="showDrawer" class="btn" size="mini">分类</button>
			<uni-drawer maskClick ref="showRight" mode="right" :mask-click="false">
				<scroll-view style="height: 100%;" scroll-y="true">
					<view class="info">
						<text class="info-text">选择分类</text>
					</view>
					<view>
						<uni-data-checkbox :map="classMap" mode="button" multiple v-model="value"
							:localdata="classArr"></uni-data-checkbox>
					</view>

					<button @click="submit()" class="btn1">确定</button>
				</scroll-view>
			</uni-drawer>
		</view>

		<view v-for="(item, index) in dishesArr" style="width: 100%;">
			<uni-section title="双标题卡片" type="line">
				<uni-card :title="item.name" :extra="item.price" :thumbnail="item.link"
					>
					<text class="uni-body">{{item.description}}</text>
					<view slot="actions" class="card-actions">
						<view class="card-actions-item" @click="favClick(0,item.id)">
							<uni-icons type="hand-up" size="18" color="#999"></uni-icons>
							<text class="card-actions-item-text">好评</text>
						</view>
						<view class="card-actions-item" @click="favClick(1,item.id)">
							<uni-icons type="hand-down" size="18" color="#999"></uni-icons>
							<text class="card-actions-item-text">差评</text>
						</view>
						<view class="card-actions-item" @click="favClick(2,item.id)">
							<uni-icons type="star" size="18" color="#999"></uni-icons>
							<text class="card-actions-item-text">收藏</text>
						</view>
						<view class="card-actions-item" @click="detail(item.id)">
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
	export default {
		data() {
			return {
				msgType: '',
				messageText: '',
				classMap: {
					text: 'content',
					value: 'id'
				},
				value: [],
				classArr: [],
				dishesArr: [],
				page: 1,
				limit: 100,
				searchValue: "",
				info: [{
					content: '今天吃什么'
				}, {
					content: '推荐菜品'
				}],
				current: 0,
			}
		},
		onShow: function(e) {
			uni.request({
				url: 'http://localhost:8080/dishes/selectDishesAll',
				data: {
					page: this.page,
					limit: this.limit
				}, //仅为示例，并非真实接口地址。
				header: {
					'userId': uni.getStorageSync("userId")
				},
				success: (res) => {
					if (res.data.code == "0") {
						this.dishesArr = res.data.data;
						for (let i = 0; i < res.data.data.length; i++) {
							this.dishesArr[i].price = this.dishesArr[i].price + "元"
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
				url: 'http://localhost:8080/classification/selectClassificationAll',
				data: {
					page: this.page,
					limit: this.limit
				}, //仅为示例，并非真实接口地址。
				header: {
					'userId': uni.getStorageSync("userId")
				},
				success: (res) => {
					if (res.data.code == "0") {
						this.classArr = res.data.data;
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
				url: 'http://localhost:8080/dishes/selectDishesAll',
				data: {
					page: this.page,
					limit: this.limit
				}, //仅为示例，并非真实接口地址。
				header: {
					'userId': uni.getStorageSync("userId")
				},
				success: (res) => {
					if (res.data.code == "0") {
						this.dishesArr = res.data.data;
						for (let i = 0; i < res.data.data.length; i++) {
							this.dishesArr[i].price = this.dishesArr[i].price + "元"
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
				url: 'http://localhost:8080/classification/selectClassificationAll',
				data: {
					page: this.page,
					limit: this.limit
				}, //仅为示例，并非真实接口地址。
				header: {
					'userId': uni.getStorageSync("userId")
				},
				success: (res) => {
					if (res.data.code == "0") {
						this.classArr = res.data.data;
						console.log(this.classArr)
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
				url: 'http://localhost:8080/dishes/selectDishesAll',
				data: {
					page: this.page,
					limit: this.limit
				}, //仅为示例，并非真实接口地址。
				header: {
					'userId': uni.getStorageSync("userId")
				},
				success: (res) => {
					if (res.data.code == "0") {
						this.dishesArr = res.data.data;
						for (let i = 0; i < res.data.data.length; i++) {
							this.dishesArr[i].price = this.dishesArr[i].price + "元"
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
			favClick(flag, id) {
				uni.request({
					url: 'http://localhost:8080/dishes/isDishes',
					data: {
						dishesId: id
					},
					header: {
						'userId': uni.getStorageSync("userId")
					},
					success: (res) => {
						console.log(res.data)
						if (res.data.code == "0000") {
							if (flag == 0) {
								if (res.data.data.likeFlag == 0) {
									uni.request({
										url: 'http://localhost:8080/like/dishesLike?userId=' + uni
											.getStorageSync("userId") +
											'&dishesId=' + id,
										method: 'POST',
										header: {
											'userId': uni.getStorageSync("userId")
										},
										success: (res) => {
											if (res.data.code == "0000") {
												this.msgType = "success"
												this.messageText = "好评成功!"
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
										url: 'http://localhost:8080/like/dishesLikeNo?userId=' + uni
											.getStorageSync("userId") +
											'&dishesId=' + id,
										method: 'POST',
										header: {
											'userId': uni.getStorageSync("userId")
										},
										success: (res) => {
											if (res.data.code == "0000") {
												this.msgType = "warn"
												this.messageText = "取消好评"
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
							} else if (flag == 1) {
								if (res.data.data.hateFlag == 0) {
									uni.request({
										url: 'http://localhost:8080/hate/dishesBad?userId=' + uni
											.getStorageSync("userId") +
											'&dishesId=' + id,
										method: 'POST',
										header: {
											'userId': uni.getStorageSync("userId")
										},
										success: (res) => {
											if (res.data.code == "0000") {
												this.msgType = "success"
												this.messageText = "差评成功!"
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
										url: 'http://localhost:8080/hate/dishesBadNo?userId=' + uni
											.getStorageSync("userId") +
											'&dishesId=' + id,
										method: 'POST',
										header: {
											'userId': uni.getStorageSync("userId")
										},
										success: (res) => {
											if (res.data.code == "0000") {
												this.msgType = "warn"
												this.messageText = "取消差评"
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
										url: 'http://localhost:8080/collect/dishesCollect?userId=' +
											uni.getStorageSync(
												"userId") +
											'&dishesId=' + id,
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
										url: 'http://localhost:8080/collect/dishesCollectNo?userId=' +
											uni.getStorageSync(
												"userId") +
											'&dishesId=' + id,
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
			submit() {
				if (this.value.length == 0) {
					uni.request({
						url: 'http://localhost:8080/dishes/selectDishesAll',
						data: {
							page: this.page,
							limit: this.limit
						}, //仅为示例，并非真实接口地址。
						header: {
							'userId': uni.getStorageSync("userId")
						},
						success: (res) => {
							if (res.data.code == "0") {
								this.dishesArr = res.data.data;
								for (let i = 0; i < res.data.data.length; i++) {
									this.dishesArr[i].price = this.dishesArr[i].price + "元"
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
					uni.request({
						url: 'http://localhost:8080/dishes/selectDishesByClassification',
						data: {
							ids: this.value
						}, //仅为示例，并非真实接口地址。
						header: {
							'userId': uni.getStorageSync("userId")
						},
						success: (res) => {
							if (res.data[0].status == "0") {
								this.dishesArr = res.data[3].rows.item;
							} else {
								console.log("请登录");
								uni.navigateTo({
									url: '../user/login'
								});
							}
						}
					});
				}
			},
			showDrawer() {
				this.$refs.showRight.open();
			},
			change(e) {
				this.current = e.detail.current;
			},
			detail(dishesId) {
				uni.navigateTo({
					url: '../detail/dishesDetail?id=' + dishesId
				});
			},
			search(res) {
				uni.request({
					url: 'http://localhost:8080/dishes/selectDishesByName',
					data: {
						name: this.searchValue
					}, //仅为示例，并非真实接口地址。
					header: {
						'userId': uni.getStorageSync("userId")
					},
					success: (res) => {
						if (res.data[0].status == "0") {
							this.dishesArr = res.data[3].rows.item;
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
		}
	}
</script>

<style>
	button:hover {
		background-color: #0062cc;
		box-shadow: 0px 5px 15px rgba(0, 0, 0, 0.2);
		transform: translateY(-2px);
	}

	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
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
		width: 65px;
		height: 30px;
		line-height: 18px;
		position: relative;
		top: 15px;
	}


	.btn1 {
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
		height: 40px;
		line-height: 18px;
		position: relative;
		top: 15px;
	}


	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}

	.uni-margin-wrap {
		width: 690rpx;
		width: 100%;
	}

	.swiper {
		height: 300rpx;
	}

	.swiper-item {
		display: block;
		height: 300rpx;
		line-height: 300rpx;
		text-align: center;
	}

	.vi {
		display: flex;
		flex-flow: row nowrap;
		justify-content: space-between;
		position: relative;
		top:20rpx;
	}

	.info {
		padding: 15px;
		color: #666;
	}

	,
	.info-text {
		font-size: 14px;
		color: #666;
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