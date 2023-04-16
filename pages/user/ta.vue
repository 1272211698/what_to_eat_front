<template>
	<view class="example-body">
		<uni-card :title="nickName" :extra="userName"
			thumbnail="https://mp-2a38b404-c00b-4fe8-ab5b-3dfc261f7584.cdn.bspapp.com/cloudstorage/12064e82-0f0f-49e9-bf3b-0f2d3e824271.jpg"
			:isFull="true" is-shadow="true">
		</uni-card>

		<view>
			<uni-card title="菜品" extra="喜欢和收藏的菜品"
				thumbnail="https://mp-2a38b404-c00b-4fe8-ab5b-3dfc261f7584.cdn.bspapp.com/cloudstorage/12064e82-0f0f-49e9-bf3b-0f2d3e824271.jpg"
				:isFull="true" is-shadow="true">
				<view>
					<button @click="showDrawer0" class="btn">他的喜爱</button>
					<uni-drawer maskClick ref="showRight0" mode="right" :mask-click="false">
						<scroll-view style="height: 100%;" scroll-y="true">
							<view class="info">
								<text class="info-text">菜品-他的喜爱</text>
							</view>
							<view v-for="item in dishesLikeArr" :key="item">
								<uni-card :title="item.name" :sub-title="item.price" :thumbnail="item.link"
									@click="detail1(item.id)" :isFull="true" is-shadow="true"></uni-card>
							</view>
						</scroll-view>
					</uni-drawer>
				</view>
				<view>
					<button @click="showDrawer1" class="btn">他的收藏</button>
					<uni-drawer maskClick ref="showRight1" mode="right" :mask-click="false">
						<scroll-view style="height: 100%;" scroll-y="true">
							<view class="info">
								<text class="info-text">菜品-他的收藏</text>
							</view>
							<view v-for="item in dishesCollectArr" :key="item"><uni-card :title="item.name"
									@click="detail1(item.id)" :sub-title="item.price" :thumbnail="item.link"
									:isFull="true" is-shadow="true"></uni-card></view>
						</scroll-view>
					</uni-drawer>
				</view>
			</uni-card>
		</view>
		<view>
			<uni-card title="发布信息" extra="点赞和收藏的发布信息"
				thumbnail="https://mp-2a38b404-c00b-4fe8-ab5b-3dfc261f7584.cdn.bspapp.com/cloudstorage/12064e82-0f0f-49e9-bf3b-0f2d3e824271.jpg"
				:isFull="true" is-shadow="true">
				<view>
					<button @click="showDrawer2" class="btn">他的点赞</button>
					<uni-drawer maskClick ref="showRight2" mode="right" :mask-click="false">
						<scroll-view style="height: 100%;" scroll-y="true">
							<view class="info">
								<text class="info-text">发布信息-他的点赞</text>
							</view>
							<view v-for="item in postLikeArr" :key="item"><uni-card :title="item.nickName"
									:sub-title="item.name"
									@click="detail2(item.id,item.link,item.type,item.description,item.nickName,item.name,item.likeNumber,item.collectNumber,item.commentNumber)"
									:thumbnail="item.link" :isFull="true" is-shadow="true">
									<text>{{item.description}}</text></uni-card></view>
						</scroll-view>
					</uni-drawer>
				</view>
				<view>
					<button @click="showDrawer3" class="btn">他的收藏</button>
					<uni-drawer maskClick ref="showRight3" mode="right" :mask-click="false">
						<scroll-view style="height: 100%;" scroll-y="true">
							<view class="info">
								<text class="info-text">发布信息-他的收藏</text>
							</view>
							<view v-for="item in postCollectArr" :key="item"><uni-card :title="item.nickName"
									:sub-title="item.name"
									@click="detail2(item.id,item.link,item.type,item.description,item.nickName,item.name,item.likeNumber,item.collectNumber,item.commentNumber)"
									:thumbnail="item.link" :isFull="true" is-shadow="true">
									<text>{{item.description}}</text>
								</uni-card></view>
						</scroll-view>
					</uni-drawer>
				</view>
				<view>
					<button @click="showDrawer4" class="btn">他的发布</button>
					<uni-drawer maskClick ref="showRight4" mode="right" :mask-click="false">
						<scroll-view style="height: 100%;" scroll-y="true">
							<view class="info">
								<text class="info-text">发布信息-他的发布</text>
							</view>
							<view v-for="item in postArr" :key="item"><uni-card :title="item.nickName"
									:sub-title="item.name" :thumbnail="item.link" :isFull="true"
									@click="detail2(item.id,item.link,item.type,item.description,item.nickName,item.name,item.likeNumber,item.collectNumber,item.commentNumber)"
									is-shadow="true">
									<text>{{item.description}}</text>
								</uni-card></view>
						</scroll-view>
					</uni-drawer>
				</view>
			</uni-card>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				nickName: "",
				userName: "",
				taId: "",
				showLeft: false,
				dishesLikeArr: [],
				dishesCollectArr: [],
				postLikeArr: [],
				postCollectArr: [],
				postArr: []
			}
		},
		onShow: function(e) {
			const value = uni.getStorageSync("userId");
			if (!value) {
				uni.navigateTo({
					url: '../user/login'
				});
			}
			uni.request({
				url: 'http://localhost:8080/user/selectUserById', //仅为示例，并非真实接口地址。
				header: {
					'userId': uni.getStorageSync("taId"),
				},
				success: (res) => {
					if (res.data.code == "7777") {
						uni.navigateTo({
							url: '../user/login'
						});
					}
					this.nickName = res.data[3].rows.item[0].nickName;
					this.userName = res.data[3].rows.item[0].userName;
				}
			});
			uni.request({
				url: 'http://localhost:8080/dishes/selectLikeByUserId', //仅为示例，并非真实接口地址。
				header: {
					'userId': uni.getStorageSync("taId"),
				},
				success: (res) => {
					if (res.data[0].status == 0) {
						this.dishesLikeArr = res.data[3].rows.item
						for (let i = 0; i < res.data[3].rows.item.length; i++) {
							this.dishesLikeArr[i].price = this.dishesLikeArr[i].price + "元"
						}
					} else {
						uni.navigateTo({
							url: '../user/login'
						});
					}
				}
			});
			uni.request({
				url: 'http://localhost:8080/dishes/selectCollectByUserId', //仅为示例，并非真实接口地址。
				header: {
					'userId': uni.getStorageSync("taId"),
				},
				success: (res) => {
					if (res.data[0].status == 0) {
						this.dishesCollectArr = res.data[3].rows.item
						for (let i = 0; i < res.data[3].rows.item.length; i++) {
							this.dishesCollectArr[i].price = this.dishesCollectArr[i].price + "元"
						}
					} else {
						uni.navigateTo({
							url: '../user/login'
						});
					}
				}
			});
			uni.request({
				url: 'http://localhost:8080/post/selectLikeByUserId', //仅为示例，并非真实接口地址。
				header: {
					'userId': uni.getStorageSync("taId"),
				},
				success: (res) => {
					if (res.data[0].status == 0) {
						this.postLikeArr = res.data[3].rows.item
					} else {
						uni.navigateTo({
							url: '../user/login'
						});
					}
				}
			});
			uni.request({
				url: 'http://localhost:8080/post/selectCollectByUserId', //仅为示例，并非真实接口地址。
				header: {
					'userId': uni.getStorageSync("taId"),
				},
				success: (res) => {
					if (res.data[0].status == 0) {
						this.postCollectArr = res.data[3].rows.item
					} else {
						uni.navigateTo({
							url: '../user/login'
						});
					}
				}
			});
			uni.request({
				url: 'http://localhost:8080/post/selectByUserId', //仅为示例，并非真实接口地址。
				header: {
					'userId': uni.getStorageSync("taId"),
				},
				success: (res) => {
					if (res.data[0].status == 0) {
						this.postArr = res.data[3].rows.item
					} else {
						uni.navigateTo({
							url: '../user/login'
						});
					}
				}
			});
		},
		onLoad: function(option) {
			const value = uni.getStorageSync("userId");
			if (!value) {
				uni.navigateTo({
					url: '../user/login'
				});
			}
			if (option.taId != value) {
				this.taId = option.taId
				uni.setStorageSync("taId", option.taId)
				uni.request({
					url: 'http://localhost:8080/user/selectUserById', //仅为示例，并非真实接口地址。
					header: {
						'userId': uni.getStorageSync("taId"),
					},
					success: (res) => {
						if (res.data.code == "7777") {
							uni.navigateTo({
								url: '../user/login'
							});
						}
						this.nickName = res.data[3].rows.item[0].nickName;
						this.userName = res.data[3].rows.item[0].userName;
					}
				});
				uni.request({
					url: 'http://localhost:8080/dishes/selectLikeByUserId', //仅为示例，并非真实接口地址。
					header: {
						'userId': uni.getStorageSync("taId"),
					},
					success: (res) => {
						if (res.data[0].status == 0) {
							this.dishesLikeArr = res.data[3].rows.item
							for (let i = 0; i < res.data[3].rows.item.length; i++) {
								this.dishesLikeArr[i].price = this.dishesLikeArr[i].price + "元"
							}
						} else {
							uni.navigateTo({
								url: '../user/login'
							});
						}
					}
				});
				uni.request({
					url: 'http://localhost:8080/dishes/selectCollectByUserId', //仅为示例，并非真实接口地址。
					header: {
						'userId': uni.getStorageSync("taId"),
					},
					success: (res) => {
						if (res.data[0].status == 0) {
							this.dishesCollectArr = res.data[3].rows.item
							for (let i = 0; i < res.data[3].rows.item.length; i++) {
								this.dishesCollectArr[i].price = this.dishesCollectArr[i].price + "元"
							}
						} else {
							uni.navigateTo({
								url: '../user/login'
							});
						}
					}
				});
				uni.request({
					url: 'http://localhost:8080/post/selectLikeByUserId', //仅为示例，并非真实接口地址。
					header: {
						'userId': uni.getStorageSync("taId"),
					},
					success: (res) => {
						if (res.data[0].status == 0) {
							this.postLikeArr = res.data[3].rows.item
						} else {
							uni.navigateTo({
								url: '../user/login'
							});
						}
					}
				});
				uni.request({
					url: 'http://localhost:8080/post/selectCollectByUserId', //仅为示例，并非真实接口地址。
					header: {
						'userId': uni.getStorageSync("taId"),
					},
					success: (res) => {
						if (res.data[0].status == 0) {
							this.postCollectArr = res.data[3].rows.item
						} else {
							uni.navigateTo({
								url: '../user/login'
							});
						}
					}
				});
				uni.request({
					url: 'http://localhost:8080/post/selectByUserId', //仅为示例，并非真实接口地址。
					header: {
						'userId': uni.getStorageSync("taId"),
					},
					success: (res) => {
						if (res.data[0].status == 0) {
							this.postArr = res.data[3].rows.item
						} else {
							uni.navigateTo({
								url: '../user/login'
							});
						}
					}
				});
			}else{
				uni.switchTab({
					url: '../user/user'
				});
			}
		},
		methods: {
			showDrawer0() {
				this.$refs.showRight0.open();
			},
			showDrawer1() {
				this.$refs.showRight1.open();
			},
			showDrawer2() {
				this.$refs.showRight2.open();
			},
			showDrawer3() {
				this.$refs.showRight3.open();
			},
			showDrawer4() {
				this.$refs.showRight4.open();
			},

			loginout() {
				uni.removeStorageSync("userId");
				console.log("登出成功！");
				uni.navigateTo({
					url: '../user/login'
				});
			},
			detail1(dishesId) {
				uni.navigateTo({
					url: '../detail/dishesDetail?id=' + dishesId
				});
			},
			detail2(id, link, type, description, nickName, name, likeNumber, collectNumber, commentNumber) {
				var url = '../detail/postDetail?id=' + id + '&link=' + link + '&type=' + type + '&description=' +
					description + '&nickName=' + nickName + '&name=' + name + '&likeNumber=' + likeNumber +
					'&collectNumber=' + collectNumber + '&commentNumber=' + commentNumber
				console.log(url);
				uni.navigateTo({
					url: '../detail/postDetail?id=' + id + '&link=' + link + '&type=' + type + '&description=' +
						description + '&nickName=' + nickName + '&name=' + name + '&likeNumber=' + likeNumber +
						'&collectNumber=' + collectNumber + '&commentNumber=' + commentNumber
				});
			}
		}
	}
</script>

<style>
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
		width: 160px;
		height: 40px;
		line-height: 18px;
	}

	,
	.info {
		padding: 15px;
		color: #666;
	}

	,
	.info-text {
		font-size: 14px;
		color: #666;
	}
</style>