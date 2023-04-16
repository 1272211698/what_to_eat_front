<template>
	<view class="content">
		<view>
			<button @click="showDrawer1" class="btn">登录</button>
			<uni-drawer maskClick ref="showRight1" mode="right" :mask-click="false">
				<scroll-view style="height: 100%;" scroll-y="true">
					<view class="info">
						<text class="info-text" size="mini">登录</text>
					</view>
					<view>
						<uni-easyinput class="uni-mt-5" trim="all" v-model="userName"
							placeholder="请输入用户名"></uni-easyinput>
						<uni-easyinput class="uni-mt-5" type="password" trim="all" v-model="password"
							placeholder="请输入密码"></uni-easyinput>
					</view>
					<button @click="submit(0)" class="btn" >登录</button>
				</scroll-view>
			</uni-drawer>

			<button @click="showDrawer2" class="btn" >注册</button>
			<uni-drawer maskClick ref="showRight2" mode="right" :mask-click="false">
				<scroll-view style="height: 100%;" scroll-y="true">
					<view class="info">
						<text class="info-text">注册</text>
					</view>
					<view>
						<uni-easyinput class="uni-mt-5" trim="all" v-model="nickName"
							placeholder="请输入昵称"></uni-easyinput>
						<uni-easyinput class="uni-mt-5" trim="all" v-model="userName"
							placeholder="请输入用户名"></uni-easyinput>
						<uni-easyinput class="uni-mt-5" type="password" trim="all" v-model="password"
							placeholder="请输入密码"></uni-easyinput>
					</view>
					<button @click="submit(1)" class="btn">注册</button>
				</scroll-view>
			</uni-drawer>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				nickName: "",
				userName: "",
				password: ""
			}
		},
		methods: {
			showDrawer1() {
				this.$refs.showRight1.open();
			},
			showDrawer2() {
				this.$refs.showRight2.open();
			},
			submit (flag) {
				if (flag == 1) {
					uni.request({
						url: 'http://localhost:8080/user/addUser?userName=' + this.userName + '&password=' +
							this.password + '&nickName=' + this.nickName,
						method: 'POST',
						//仅为示例，并非真实接口地址。
						success: (res) => {
							if (res.data.code == "0000") {
								console.log("注册成功，正在自动登录");
								this.nickname = "";
								uni.request({
									url: 'http://localhost:8080/user/loginForWX',
									data: {
										userName: this.userName,
										password: this.password
									}, //仅为示例，并非真实接口地址。
									success: (res) => {
										if (res.data.code == "0000") {
											this.userName = "";
											this.password = "";
											uni.setStorageSync("userId", res.data.userId);
											uni.switchTab({
												url: '../user/user'
											});
										} else {
											console.log("登录失败！")
										}
									}
								});
							} else {
								console.log("注册失败！")
							}
						}
					});
				} else {
					uni.request({
						url: 'http://localhost:8080/user/loginForWX',
						data: {
							userName: this.userName,
							password: this.password
						}, //仅为示例，并非真实接口地址。
						success: (res) => {
							if (res.data.code == "0000") {
								console.log("登录成功！");
								this.userName = "";
								this.password = "";
								uni.setStorageSync("userId", res.data.userId);
								uni.switchTab({
									url: '../user/user'
								});
							} else {
								console.log("登录失败！")
							}
						}
					});
				}
			}
		}
	}
</script>

<style>
	.uni-form-item .title {
		padding: 20rpx 0;
	}

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
		width: 100px;
		height: 40px;
		line-height: 18px;
		position: relative;
		top: 15px;
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

	.uni-mt-5 {
		margin-top: 5px;
	}
</style>