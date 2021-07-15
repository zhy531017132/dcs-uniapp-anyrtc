<template>
	<view class="content" :style="'height:'+screenHeight+'px'">
		<view class="img">预览转换在线体验</view>
		<view class="img2">
			<button type="default" @tap="uploadFile">上传文件</button>
		</view>
		<view class="">使用说明：</view>
		<text>
			1.点击上传文件，目前支持word，excel，ppt。
			2.跳转到预览页面，填入频道，开始共享屏幕启动通讯，此操作需要在PC端。
			3.需要加入此频道的人需要返回主页面点直播栏，输入对应频道并观看分享实时通讯，此操作可在PC端也可在移动端。
			4.主演讲者退出则退出，其他加入对应频道退出的不影响。
			5.因为是带语音功能，所以需要设备持有麦克风。
		</text>
		<view class="progress-box">
			<progress :percent="percent" show-info v-if="isShow" stroke-width="6" />
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: 'Hello',
				uploadFilePath: '',
				percent: 0,
				isShow: false,
				screenHeight: ''
			}
		},
		onLoad() {
			this.screenHeight = uni.getSystemInfoSync().windowHeight - 1
		},
		methods: {
			uploadFile() {
				//
				uni.chooseFile({
					"count": 1,
					"success": (res) => {
						this.uploadFilePath = res.tempFilePaths[0];
					},
					"fail": (res) => {
						console.log(res, "文件路径返回失败")
					},
					complete: () => {
						var uploadTask = uni.uploadFile({
							url: "https://fcsapi.yozodocs.com/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX", // 此接口地址因为收费，故注释
							filePath: this.uploadFilePath,
							name: "file",
							formData: {
								"convertType ": 61
							},
							success: (data) => {
								this.isShow = false
								console.log(JSON.parse(data.data).data.viewUrl.replace('http',
									'https'));
								uni.navigateTo({
									url: '../reader/reader?dcsRouter=' + JSON.parse(data
										.data).data.viewUrl.replace('http', 'https')
								})
							},
						})
						uploadTask.onProgressUpdate((res) => {
							this.isShow = true;
							this.percent = res.progress
						})
					}
				})
			}
		}
	}
</script>

<style>
	.img {
		width: 100%;
		height: 350rpx;
		background-image: url(../../static/yozopng1.png);
		color: white;
		font-size: 50rpx;
		text-align: center;
		line-height: 350rpx;
	}

	.img2 {
		width: 100%;
		height: 600rpx;
		background-image: url(../../static/yozo2.png);

	}

	button {
		width: 340rpx;
		margin-top: 270rpx;
	}

	.content {
		background-color: #ccc;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
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

	.progress-box {
		width: 100%;
	}
</style>
