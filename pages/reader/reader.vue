<template>
	<view class="">
		<view class="btns">
			<input type="text" :value="channel" placeholder="输入频道" @input="inputChannel" />
			<button type="default" class="btn" @click="connect" :disabled="connectControl">屏幕共享启动通话</button>
			<button type="default" class="btn" @click="leaveConnect" :disabled="leaveControl">结束共享停止通话</button>
		</view>
		<web-view :src="dcsRouter" id="reader">
		</web-view>
	</view>
</template>

<script>
	import ArRTC from 'ar-rtc-sdk';
	export default {
		data() {
			return {
				connectControl: false,
				leaveControl: true,
				dcsRouter: '',
				rtcClient: '',
				channel: '',
				appid: 'dc4d2e3e3e5a32ca133d6afa540c4978',
				token: '',
				client: '',
				client2: '',
				options: {
					appid: null,
					channel: null,
					uid: "web" + Math.random().toString(16).substr(2).toLowerCase(),
					token: null
				},
				localTracks: {
					videoTrack: null,
					audioTrack: null
				},
				localTracks2: {
					videoTrack: null,
					audioTrack: null
				},
				remoteUsers: {}
			}
		},
		methods: {
			async connect() {
				if (this.channel == '') {
					uni.showToast({
						title: "请输入频道",
						icon: 'none'
					})
				} else {
					this.connectControl = true;
					this.options.appid = this.appid;
					this.options.token = this.token;
					this.options.channel = this.channel;
					await this.join();
					this.leaveControl = false;
				}
			},
			async leaveConnect() {
				this.leaveControl = true;
				this.connectControl = false;
				for (let trackName in this.localTracks) {
					let track = this.localTracks[trackName];
					if (track) {
						track.stop();
						track.close();
						this.localTracks[trackName] = undefined;
					}
				}
				console.log(this.localTracks)
				await this.client.leave();
			},
			inputChannel(event) {
				this.channel = event.target.value
			},
			async join() {
				console.log(this.channel)
				let client = this.client = ArRTC.createClient({
					mode: "rtc",
					codec: "h264"
				});
				client.on("user-published", async (user, mediaType) => {
					// 订阅远端用户的音视频轨道
					await client.subscribe(user, mediaType);
					if (mediaType === 'video') {
						// 播放远端视频到指定窗口
						user.videoTrack.play("reader");
					} else if (mediaType === 'audio') {
						// 播放远端音频
						user.audioTrack.play();
					}
				});
				await client.join(this.options.appid, this.options.channel, this.options.token || null, this.options.uid)
					.then((
						uid) => {
						console.log('加入频道成功');
					})

				this.localTracks.videoTrack = await ArRTC.createScreenVideoTrack();
				
				this.localTracks.audioTrack = await ArRTC.createMicrophoneAudioTrack();
				// client2.publish(Object.values(this.localTracks2));
				client.publish(Object.values(this.localTracks));

			}
		},
		onLoad(data) {
			this.dcsRouter = data.dcsRouter
		}
	}
</script>

<style lang="scss">
	.btns {
		display: flex;

		input {
			z-index: 1;
			width: 33%;
			height: 100rpx;
			border: 1px solid black;
			background-color: #ccc;
		}

		.btn {
			z-index: 1;
			flex: 1;
		}
	}
</style>
