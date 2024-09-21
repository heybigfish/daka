<template>
	<view class="content">
		<view class="page-body">
			<mainHead />
			<!-- 当前位置：{{ address }} -->
			<!-- // 日期时间 -->
			<view class="sign-title-r">
				<view class="date">
					<view style="text-align: center;">{{ date }}<text style="margin: 0 1em;">{{ week }}</text>{{ Timer[0].time }}
						-
						{{ Timer[1].time }}</view>
				</view>
			</view>
			<!-- 打卡信息 -->
			<view class="uni-timeline">
				<!-- 上班打卡 -->
				<view class="uni-timeline-item uni-timeline-first-item">
					<view class="uni-timeline-item-divider" :style="{ background: (!isAm ? '#1AAD19' : '#1AAD19') }"></view>
					<view class="uni-timeline-item-content">
						<view>
							<view class="uni-timeline-item-content-t1">上班时间 &nbsp;{{ Timer[0].time }}</view>
							<view class="desc-pad mgsBox" v-if="isAm">
								<view class="time uni-timeline-item-content-t iswqbox">
									打卡时间 {{ (amSign.time).substring(10, 16) }}
									<view class="iswq">
										<uni-tag :text="amSign.mode" inverted type="success" size='small'></uni-tag>
									</view>
								</view>
								<view>
									{{ amSign.address }}
								</view>
							</view>
							<view v-else class="content-show">
								<view :class="['module', moduleColor]" @click="clickSign('am')">
									<view class="text">{{ moduleTitle }}</view>
									<view class="time">{{ time }}</view>
								</view>
								<view v-if="is === true">
									<view class="colorGreen" style="text-align: center;">
										已在考勤范围内
									</view>
								</view>
								<view v-else-if="is === false">
									<view class="colorRed" style="text-align: center;">
										不在考勤范围内 <text class="relocation" @click="relocation">重新定位</text>
									</view>
								</view>
								<view v-else-if="is === null">
									<view class="colorRed" style="text-align: center;">
										<text>请检查手机定位服务状态</text>
										<text class="relocation" @click="relocation">刷新</text>
									</view>
								</view>
							</view>
						</view>
					</view>
				</view>
				<view class="uni-timeline-item uni-timeline-last-item">
					<view class="uni-timeline-item-divider" :style="{ background: (isAm ? '#1AAD19' : '#bbb') }"></view>
					<view class="uni-timeline-item-content">
						<view>
							<view class="uni-timeline-item-content-t1">下班时间 &nbsp;{{ Timer[1].time }}</view>
							<view v-if="isAm" class="desc-pad mgsBox">
								<view class="desc-pad" v-if="isPm">
									<view class="time uni-timeline-item-content-t iswqbox">
										打卡时间:{{ (pmSign.time).substring(10, 16) }}
										<view class="iswq" v-if="pmSign.mode">
											<uni-tag :text="pmSign.mode" inverted type="success" size='small'></uni-tag>
										</view>
									</view>
									<view>{{ pmSign.address }}</view>
								</view>
								<view v-else class="content-show">
									<view :class="['module', modulePmColor]" @click="clickSign('pm')">
										<view class="text">{{ modulePmTitle }}</view>
										<view class="time">{{ time }}</view>
									</view>
									<view v-if="is === true">
										<view class="colorGreen" style="text-align: center;" v-if="is">
											已在考勤范围内
										</view>
									</view>
									<view v-else-if="is === false">
										<view class="colorRed" style="text-align: center;" v-if="!is">
											不再考勤范围内 <text class="relocation" @click="relocation">重新定位</text>
										</view>
									</view>
								</view>
							</view>
						</view>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
import uniPopup from '@/components/uni-popup/uni-popup.vue'
import uniIcon from "@/components/uni-icon/uni-icon.vue"
import mainHead from "@/components/main/main-head.vue"
import { formateDate, pointInsideCircle, isSameDay, throttle } from "../../common/util.js"
import { setSignInfo, addSignInfo, getSignInfo, getInfo, key } from "./index.js"
export default {
	components: { uniIcon, uniPopup, mainHead },
	data() {
		return {
			name: "Navy_c",
			moduleColor: 'CBlue',
			modulePmColor: 'CBlue',
			activeClickBtn: true,  // 是否启动快捷打卡
			Timer: [{ time: "15:00", }, { time: "18:00" }],	//上下班时间
			isAm: false,								//上班是否打卡
			isPm: false,								//下班是否打卡
			amSign: { time: "", address: "", remarks: "", img: "" },			//上午打卡信息
			pmSign: { time: "", address: "", remarks: "", img: "" },			//下午打卡信息
			clickNum: 0,								//点击重新获取位置信息次数
			is: null,								//是否正常打卡（外勤打卡）
			isSign: false,							//是否打卡
			time: formateDate(new Date(), 'h:min:s'),	//当前时分秒
			date: formateDate(new Date(), 'Y-M-D'),
			week: "",						//当前星期几			
			latitude: "",							//当前经度
			longitude: "",							//当前维度
			address: "我的位置",
			allSign: [],								//所有打卡信息						
			signInfo: { mode: "", latitude: "", longitude: "", address: "", time: "", remarks: "" },	//打卡信息 （模式，经纬度，地址，时间）
			covers: [
				// 公司点信息
				{ id: 0, callout: { content: "化纤大脑", color: "red", display: "ALWAYS", }, latitude: 30.194146592881946, longitude: 120.26730929904514, iconPath: '../../../static/img/location.png' },
			],
			circles: [
				// 公司圆信息(latitude:30.18534,longitude:120.26457,);
				{ latitude: 30.194146592881946, longitude: 120.26730929904514, radius: 50, strokeWidth: 1, fillColor: "#7fff0099" },
			]
			// 30.194146592881946 120.26730929904514
		}
	},
	computed: {
		moduleTitle() {
			let setTime = this.Timer[0].time.replace(/:/g, '')
			let getTime = this.time.replace(/:/g, '').substring(0, 4)
			if (!this.is) {
				this.moduleColor = 'CGreen'
				return "外勤打卡"
			} else {
				if (getTime > setTime) {
					this.moduleColor = 'CYellow'
					return "迟到打卡"
				} else {
					this.moduleColor = 'CBlue'
					return "正常打卡"
				}
			}
		},
		modulePmTitle() {
			let setTime = this.Timer[1].time.replace(/:/g, '')
			let getTime = this.time.replace(/:/g, '').substring(0, 4)
			if (this.is) {
				if (getTime < setTime) {
					this.modulePmColor = 'CYellow'
					return "早退打卡"
				} else {
					this.modulePmColor = 'CBlue'
					return "正常打卡"
				}
			} else if (this.is === null) {
				this.modulePmColor = 'CAsh'
				return "定位失败"
			} else {
				this.modulePmColor = 'CGreen'
				return "外勤打卡"
			}

		}
	},
	// 初始化
	onLoad() {
		this.clickSign = throttle(this.clickSign, 1000)
		var sign = getSignInfo();
		if (sign != undefined) {
			var signA = sign.main.reverse();
			this.allSign = signA
			if (signA.length == 1) {
				if (isSameDay(signA[0].nowT)) {
					this.isSign = true;
					this.isAm = true;
					this.allSign = signA
					this.amSign = signA[0];
				}
			} else {
				// 理想状态 认为这是 上一天 的 下班信息
				if (signA[0]) {
					if (isSameDay(signA[0].nowT)) {
						this.isSign = true;
						this.isPm = true;
						this.allSign = signA;
						this.pmSign = signA[0];
					}
				}
				// 理想状态 认为这是 上一天 的 上班信息
				if (signA[1]) {
					if (isSameDay(signA[1].nowT)) {
						this.isSign = true;
						this.isAm = true;
						this.allSign = signA
						this.amSign = signA[1];
					}
				}
			}
		}
		this.getTime();
		this.getWeekDate();
		this.startLocationUpdate()
	},
	methods: {
		// 监听位置信息变化
		// 开始监听位置变化  
		startLocationUpdate() {
			console.log("js开始监听位置变化");
			uni.showLoading({ title: "获取位置信息...", mask: true });
			var that = this;
			let setAmTime = this.Timer[0].time.replace(/:/g, '')
			let setPmTime = this.Timer[1].time.replace(/:/g, '')
			let getTime = this.time.replace(/:/g, '').substring(0, 4)
			const func = throttle(that.autoSign, 2000)
			uni.startLocationUpdate({
				success: function () {
					console.log('开始监听位置变化');
					// 监听位置变化  
					uni.onLocationChange(function (res) {
						console.log('位置变化', res.latitude, res.longitude);
						that.latitude = res.latitude;
						that.longitude = res.longitude;
						that.covers[1] = { id: 1, latitude: res.latitude, longitude: res.longitude, iconPath: '../../static/location.png' }
						// 是否在正常打卡范围内
						var s = pointInsideCircle([that.latitude, that.longitude], [that.circles[0].latitude, that.circles[0].longitude], that.circles[0].radius);
						console.log("s~", s, setPmTime, getTime);
						that.is = s;
						that.signInfo.latitude = res.latitude;
						that.signInfo.longitude = res.longitude;
						that.signInfo.mode = s ? "正常打卡" : "外勤打卡";
						that.getAdd()
						var isTrue = that.is;
						if (that.activeClickBtn) {
							// 不在考勤范围内，不进行打卡操作
							if (!isTrue) {
								return;
							}
							// 时间小于等于上班时间，并且上班打卡未打卡，进行打卡操作
							if (setAmTime >= getTime && that.isAm === false) {
								func('am')
							} else if (setPmTime <= getTime && that.isPm === false && that.isAm === true) {
								// 时间大于等于下班时间，并且下班打卡未打卡，进行打卡操作
								func('pm')
							}
						}
					});
				},
				fail: function (err) {
					console.error('开始监听位置变化失败：', err);
				},
				complete: function () {
					console.log('开始监听位置变化完成');
					uni.hideLoading();
				}
			});
		},
		// 自动打卡
		autoSign(type) {
			console.log("自动打卡", type);

			this.clickSign(type)
		},
		getWeekDate() {
			var now = new Date();
			var day = now.getDay();
			var weeks = new Array("星期日", "星期一", "星期二", "星期三", "星期四", "星期五", "星期六");
			this.week = weeks[day];
			console.log(this.week);
		},
		// 获取实时地址
		getAdd() {
			if (this.isAm && this.isPm) {
				return;
			}
			if (this.is === true) {
				let address = this.covers[0].callout.content;
				this.address = address;
				this.signInfo.address = address;
				return;
			}
			var that = this;
			var url = `https://apis.map.qq.com/ws/geocoder/v1/?location=${that.latitude},${that.longitude}&key=${key}`;
			uni.request({
				url,
				success(res) {
					console.log("实时地址信息", res.data);

					var data = res.data;
					if (data.status != 0) {
						uni.showToast({ title: data.message, icon: "none" })
						return;
					}
					if (that.is === null) {
						that.address = "请检查位置信息！";
					}
					if (that.is === false) {
						let address = res.data.result.address + res.data.result.formatted_addresses.recommend
						that.address = address;
						that.signInfo.address = address;
					}
				},
				fail(err) {
					uni.showToast({ title: "请检查位置信息！", icon: "none" })
				}
			})
		},
		// 初始化数据  （公司的经纬度 公司名称 打卡范围 ）
		getData() {
			var that = this;
			var url = ``;
			uni.request({
				url,
				success(res) {
					console.log("打卡范围", res.data);
					let data = res.data;
					that.covers[0].callout.content = data.name;
					that.covers[0].latitude = that.circles[0].latitude = data.latitude;
					that.covers[0].longitude = that.circles[0].longitude = data.longitude;
					that.circles.radius = data.r;
				}
			})
		},
		// 重新定位
		relocation() {
			this.getLocation();
			// uni.navigateTo({url:"/pages/sign/sign",})
		},
		// 实时时间
		getTime() {
			var that = this;
			setInterval(function () {
				that.time = formateDate(new Date(), 'h:min:s')
			}, 1000)
		},
		// 获取当前位置
		getLocation() {
			var that = this;
			if (this.clickNum !== 0) {
				uni.showLoading({ title: "获取中...", mask: true })
			}
			if (this.clickNum >= 3) {
				uni.showToast({ title: "请稍后尝试！", icon: "none", mask: true });
				return;
			}
			this.clickNum++;
			uni.getLocation({
				type: 'gcj02', //返回可以用于uni.openLocation的经纬度
				success(res) {
					console.log("当前位置信息", res);

					uni.hideLoading();
					that.latitude = res.latitude;
					that.longitude = res.longitude;
					console.log(res.latitude, "---", res.longitude)
					that.covers[1] = { id: 1, latitude: res.latitude, longitude: res.longitude, iconPath: '../../static/location.png' }
					var s = pointInsideCircle([that.latitude, that.longitude], [that.circles[0].latitude, that.circles[0].longitude], that.circles[0].radius);


					that.is = s;
					that.signInfo.latitude = res.latitude;
					that.signInfo.longitude = res.longitude;
					that.signInfo.mode = s ? "正常打卡" : "外勤打卡";
					that.clickNum = 0;
					that.getAdd()
				},
				fail(err) {
					uni.hideLoading();
					that.clickNum = 0;
					that.address = "请检查位置信息"
					uni.showToast({ title: "请检查位置信息状态！", icon: "none", mask: true, duration: 3000 })
					console.log("err`~~", err)
				}
			});
		},
		// 点击打卡
		clickSign(type) {
			var that = this;
			var isTrue = this.is;
			if (isTrue === null) {
				uni.showToast({ title: "请检查位置信息状态！", icon: "none", mask: true, duration: 3000 })
				return;
			}
			uni.showLoading({ title: "打卡记录中...", mask: true });
			this.signInfo.time = formateDate(new Date(), 'Y-M-D h:min:s');
			var a = getSignInfo();
			if (a != undefined) {
				addSignInfo(getInfo(this.signInfo), a)
			} else {
				setSignInfo(getInfo(this.signInfo))
			}
			setTimeout(function () {
				uni.hideLoading();
				var sign = getSignInfo().main;
				that.allSign = sign.reverse();
				that.isSign = true;

				console.log(that.allSign, 999);
				if (that.isAm === false && type === 'am') {
					that.allSign[0].mode = that.moduleTitle
					that.isAm = true;
					that.amSign = that.allSign[0];
				}

				if (that.isPm === false && type === 'pm') {
					that.allSign[1].mode = that.modulePmTitle
					that.isPm = true;
					that.pmSign = that.allSign[1];
				}
				console.log("that.signInfo", that.signInfo, that.pmSign, that.modulePmTitle);

				uni.showToast({ title: "打卡成功！" })
			}, 200)
		},
	},
}
</script>

<style>
.map {
	width: 100%;
	height: 260px;
}

.uni-list-cell-left {
	padding: 0 30upx;
}

.text-desc {
	display: flex;
	justify-content: space-between;
	padding: 10upx 20upx
}

.colorRed {
	color: red;
}

.colorGreen {
	color: #32CD32;
}

.colorYellow {
	color: yellow;
}

.colorBlue {
	color: #007aff;
}

.bgBlue {
	background-color: #007aff;
}

.bgGreen {
	background-color: #32CD32;
}

.bgAsh {
	background-color: #C8C7CC;
}

.uni-timeline {
	padding: 30upx 20upx;
}

.uni-timeline-item-content-t {
	font-weight: bold;
}

.desc-pad {
	padding: 0 0upx
}

.desc-pad .bz {
	color: rgb(0, 122, 255);
}

.desc-pad .bz .icon {
	color: rgb(0, 122, 255)
}

.uni-timeline-last-item .uni-timeline-item-divider {
	background: #bbb;
}

.CBlue {
	background-color: #007aff;
	box-shadow: 0 3px 3px #007aff;
}

.CGreen {
	background-color: #32CD32;
	box-shadow: 0 3px 3px #32CD32;
}

.CYellow {
	background-color: #FFA500;
	box-shadow: 0 3px 3px #FFA500;
}

.CAsh {
	background-color: #C8C7CC;
	box-shadow: 0 3px 3px #C8C7CC;
}

.module {
	overflow: hidden;
	margin: 20upx auto;
	width: 220upx;
	height: 220upx;
	border-radius: 50%;
	color: #fff;
	text-align: center;
}

.module .text {
	font-size: 20px;
	margin: 50upx auto 10upx;
}

.uni-timeline-item .uni-timeline-item-content {
	width: 100%;
	padding-right: 20upx;
}

.content-show {
	width: 100%
}



.mgsBox {
	padding: 20upx;
	margin-bottom: 30upx;
	border-radius: 8upx;
	background-color: #f1f1f1;
}

.iswqbox {
	display: flex;
	align-items: center;
}

.iswq {
	width: 180upx;
	margin-left: 10upx;
}

.relocation {
	color: #0000FF;
}
</style>
