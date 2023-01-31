<template>
	<view class="page">
		<image class="anthBg"
			src="https://jjh.iqweb.net/uploads/attach/2022/12/20221214/f0824a9892e0c80bafca25ed6f66fbc4.jpg"
			mode="widthFix"></image>
		<view class="authInfo" v-if="status<0 || status>1">
			<view class="title">
				认证信息
			</view>
			<view class="authMain">
				<view class="">
					<view>
						企业名称：{{authInfo.company}}
					</view>
					<view>
						法人姓名：{{authInfo.name}}
					</view>
					<view>
						认证状态：
						<text v-if="status==-1">已拒绝</text>
						<text v-if="status>1">已认证</text>
					</view>
					<view style="color: red;" v-if="status==-1">
						拒绝原因：{{authInfo.sh_msg || ''}}
					</view>
				</view>
			</view>
		</view>
		<view class="steps">
			<view class="steps_item">
				<view class="s_r">
					<view class="line" :style="{backgroundColor:'rgba(0,0,0,0)'}"></view>
					<view class="index" :style="{backgroundColor:backgroundColor,color:color}">
						1
					</view>
					<view class="line" :style="{backgroundColor:backgroundColor}"></view>
				</view>
				<view class="s_l">
					<view class="info_item">
						<text class="title">上传营业执照</text>
						<view class="content">
							<view>1.请上传清晰的<text>珠宝行业《营业执照》复印件照片，并加盖公章</text>，如示例图所示。</view>
							<view>2.无公章，可在纸条上手写<text>“本人保证提供的资料(身份证营业执照)均为真实有效且合法的资料，若本人提供虚假资料，本人愿承担与此相关的全部责任。正楷签名并按手印”</text>和营业执照一起拍照上传
								(不要遮营业执照重要信息)！</view>
							<view>3.图片要求:清晰无遮掩，平铺，要能看清法人、国徽等信息。</view>
						</view>
						<view class="uploadImg">
							<image @click="uploadImg(1)" :src="yyzzImg?yyzzImg:'../../static/images/yyzz.png'" mode="aspectFill"></image>
						</view>
					</view>
				</view>
			</view>
			<view class="steps_item">
				<view class="s_r">
					<view class="line" :style="{backgroundColor:backgroundColor}"></view>
					<view class="index" :style="{backgroundColor:backgroundColor,color:color}">
						2
					</view>
					<view class="line" :style="{backgroundColor:backgroundColor}"></view>
				</view>
				<view class="s_l">
					<view class="info_item">
						<text class="title">上传法人身份证</text>
						<view class="content">
							<view>1.请上传清晰的与<text>营业执照法人一致</text>的身份证正反面照片</view>
							<view>2.图片要求:清晰无遮掩，平铺，要能看清本人的头像和姓名等信息。</view>
						</view>
						<view class="uploadImg">
							<image @click="uploadImg(2)" :src="sfzzImg?sfzzImg:'../../static/images/sfzz.png'" mode="aspectFill"></image>
							<image @click="uploadImg(3)" :src="sfzfImg?sfzfImg:'../../static/images/sfzf.png'" mode="aspectFill"></image>
						</view>
					</view>
				</view>
			</view>
			<view class="steps_item">
				<view class="s_r">
					<view class="line" :style="{backgroundColor:backgroundColor}"></view>
					<view class="index" :style="{backgroundColor:backgroundColor,color:color}">
						3
					</view>
					<view class="line" :style="{backgroundColor:'rgba(0,0,0,0)'}"></view>
				</view>
				<view class="s_l">
					<view class="info_item">
						<text class="title">填写详细信息</text>
						<view class="form">
							<view class="form_item">
								<view>企业名称</view>
								<input type="text" :disabled="status>0" v-model="company" placeholder="请输入企业名称" />
							</view>
							<view class="form_item">
								<view>法人姓名</view>
								<input type="text" :disabled="status>0" v-model="name" placeholder="请输入法人姓名" />
							</view>
							<view class="form_item">
								<view>手机号</view>
								<input type="number" :disabled="status>0" v-model="phone" maxlength="11" placeholder="请输入手机号" />
							</view>
						</view>
					</view>
				</view>
			</view>
		</view>
		<view class="btn" @click="sub" v-if="status<1">
			{{textBtn}}
		</view>
		<view class="btn bgBtn" v-else>
			{{textBtn}}
		</view>
		<authorize :isShowAuth="isShowAuth" @authColse="authColse"></authorize>
	</view>
</template>

<script>
	import { mapGetters } from 'vuex';
	import authorize from '@/components/Authorize';
	let interval = ''
	import {
		getAuthInfo,
		updateAuth
	} from '@/api/user'
	export default {
		components: {
			// #ifdef MP
			authorize
			// #endif
		},
		computed: mapGetters(['isLogin']),
		onLoad(option) {
			this.getAuthInfo()
		},
		onShow() {
			if (this.isLogin === false) {
				this.isShowAuth = true
			} else {
				if(this.status==1){
					interval = setInterval(()=>{
						this.getAuthInfo(false)
					},5000)
				}
			}
		},
		onHide() {
			clearInterval(interval)
		},
		onUnload() {
			clearInterval(interval)
		},
		data() {
			return {
				color:'#000',
				backgroundColor:'#F1DA8E',
				textBtn: '提交认证',
				name: '',
				phone: '',
				company: '',
				tip: '',
				yyzzImg:'',
				sfzzImg:'',
				sfzfImg:'',
				status:0,
				authInfo:{},
				isShowAuth: false, //是否隐藏授权
			};
		},
		methods: {
			/**
			 * 上传文件
			 * 
			 */
			uploadImg: function(type) {
				if(this.status>0) return
				let that = this;
				that.$util.uploadImageOne('upload/image', function(res){
					if(type==1){
						that.yyzzImg = res.data.url
					}else if(type==2){
						that.sfzzImg = res.data.url
					}else if(type==3){
						that.sfzfImg = res.data.url
					}
				});
			},
			// 授权关闭
			authColse: function(e) {
				this.isShowAuth = e;
				this.getAuthInfo()
			},
			getAuthInfo(a = true) {
				let that = this
				if (a) {
					uni.showLoading({
						title: '加载中...'
					})
				}
				getAuthInfo().then(user => {
					// that.tip = user.data.tip
					uni.hideLoading()
					// if (user.data.status == null) {
					// 	return
					// }
					that.name = user.data.name
					that.phone = user.data.phone
					that.company = user.data.company
					that.sfzzImg = user.data.sfz_photo
					that.yyzzImg = user.data.yyzz_img
					that.sfzfImg = user.data.sfz_gh
					that.status = user.data.auth_type
					that.authInfo = user.data
					if (that.status == -1) {
						clearInterval(interval)
						that.textBtn = '重新认证'
						uni.showToast({
							title: user.data.fail_message,
							icon: 'none'
						})
					} else if (that.status == 1) {
						that.textBtn = '审核中'
					} else if (that.status > 1) {
						clearInterval(interval)
						that.textBtn = '已认证'
					} else {
						clearInterval(interval)
					}
				}).catch(error => {
					uni.hideLoading()
					uni.showToast({
						title: error,
						icon: 'none'
					})
				})
			},
			sub() {
				let that = this
				let regPhone = /^(13[0-9]|14[0-9]|15[0-9]|16[0-9]|17[0-9]|18[0-9]|19[0-9])\d{8}$/
				// let regSfz =  /^[1-9]\d{5}(19|20)\d{2}((0[1-9])|(1[0-2]))(([0-2][1-9])|10|20|30|31)\d{3}[0-9Xx]$/
				console.log(that.yyzzImg);
				if (!that.yyzzImg) {
					return uni.showToast({
						title: '请上传营业执照',
						icon: 'none'
					})
				}
				if (!that.sfzzImg) {
					return uni.showToast({
						title: '请上传身份证正面照',
						icon: 'none'
					})
				}
				if (!that.sfzfImg) {
					return uni.showToast({
						title: '请上传身份证反面照',
						icon: 'none'
					})
				}
				if (!that.company) {
					return uni.showToast({
						title: '请输入企业名称',
						icon: 'none'
					})
				}
				if (!that.name) {
					return uni.showToast({
						title: '请输入法人姓名',
						icon: 'none'
					})
				}
				if (!regPhone.test(that.phone)) {
					return uni.showToast({
						title: '请输入正确的手机号',
						icon: 'none'
					})
				}
				
				let data = {
					name: that.name,
					phone: that.phone,
					company: that.company,
					yyzz_img:that.yyzzImg,
					sfz_photo:that.sfzzImg,
					sfz_gh:that.sfzfImg
				}
				uni.showLoading({
					title: '加载中...'
				})
				updateAuth(data).then(user => {
					uni.showToast({
						title: user.msg,
						icon: 'none'
					})
					uni.hideLoading()
					setTimeout(() => {
						that.getAuthInfo()
					}, 500)
				}).catch(error => {
					uni.hideLoading()
					uni.showToast({
						title: error,
						icon: 'none'
					})
				})

			}
		},

	}
</script>

<style lang="less">
	.page {
		background-color: #fff;
		padding-bottom: 50rpx;
	}

	.anthBg {
		width: 100%;
	}

	.authInfo{
		margin: 0 30rpx 20rpx;
		border-radius: 20rpx;
		background-image: linear-gradient(90deg,#FAF3E3,#FDEED1);
		padding: 20rpx;
		.title{
			font-size: 34rpx;
			font-weight: 700;
			margin-bottom: 20rpx;
		}
		.authMain{
			padding: 20rpx;
			border-radius: 20rpx;
			background-color: #fff;
			view{
				margin-bottom: 20rpx;
			}
		}
	}
	.steps {
		display: flex;
		flex-direction: column;
	
		.steps_item {
			display: flex;
	
			.s_r {
				padding: 0 20rpx;
				display: flex;
				flex-direction: column;
				align-items: center;
	
				.line {
					flex: 1;
					width: 5rpx;
					background-color: #fff;
				}
	
				.index {
					margin: 20rpx 0;
					width: 40rpx;
					height: 40rpx;
					font-size: 12px;
					text-align: center;
					line-height: 40rpx;
					border-radius: 50rpx;
				}
			}
	
			.s_l {
				display: flex;
				flex-direction: column;
				flex: 1;
				padding: 20rpx 0;
	
				.info_item {
					background-color: #FFFFFF;
					margin-right: 30upx;
					border-radius: 10upx;
					display: flex;
					flex-direction: column;
					justify-content: center;
					padding: 30upx;
					box-shadow: 0 10rpx 30rpx #ddd;
	
					.title {
						font-size: 18px;
						font-weight: 500;
						color: rgba(51, 51, 51, 1);
						line-height: 25px;
						margin-bottom: 20rpx;
					}
	
					.content {
						font-size: 14px;
						font-weight: 400;
						color: rgba(102, 102, 102, 1);
						line-height: 20px;
						overflow: hidden;
						text-overflow: ellipsis;
						display: -webkit-box;
	
						display: flex;
						flex-direction: column;
						text{
							color: red;
						}
					}
					.uploadImg{
						margin-top: 20rpx;
						display: flex;
						justify-content: space-between;
						image{
							width: 280rpx;
							height: 176rpx;
							box-shadow: 0 0 4rpx 2rpx rgba(0, 0, 0, .09);
						}
					}
					.form{
						.form_item{
							display: flex;
							margin-bottom: 20rpx;
							align-items: center;
							view{
								margin-right: 20rpx;
								width: 120rpx;
							}
							input{
								flex: 1;
								border-radius: 4rpx;
								box-shadow: 0 0 4rpx 2rpx rgba(0, 0, 0, .09);
								padding: 8rpx 10rpx;
							}
						}
					}
				}
			}
		}
	}
	.hui {
		color: #666;
	}

	.tab {
		margin-top: 10rpx;

		.tabitem {
			background-color: rgba(0, 0, 0, .09);
			width: 40%;
			height: 80rpx;
			text-align: center;
			line-height: 80rpx;
			box-shadow: 0 0 6rpx 3rpx rgba(0, 0, 0, .09);

			&:first-child {
				border-radius: 20rpx 0 0 20rpx;
			}

			&:last-child {
				border-radius: 0 20rpx 20rpx 0;
			}
		}

		.active {
			background-color: #fff;
		}
	}

	.authList {
		padding: 20rpx 0;

		.authitem {
			background-color: #fff;
			padding: 40rpx 30rpx;
			border-bottom: 1rpx solid #eee;
			font-size: 28rpx;

			view {
				white-space: nowrap;
			}
			image{
				width: 150rpx;
				height: 150rpx;
			}
			input {
				flex: 1;
				margin-left: 50rpx;
				text-align: right;
			}
		}
	}

	.tip {
		padding: 40rpx 30rpx;
		font-size: 26rpx;
		color: #999;
	}

	.btn {
		width: 80%;
		padding: 30rpx 0;
		text-align: center;
		font-size: 34rpx;
		font-weight: 700;
		margin: 50rpx auto 0;
		background-color: #F1DA8E;
		border-radius: 50rpx;
	}
	.bgBtn{
		background-color: #d3bf7c;
	}
	.bgc {
		background-color: rgba(255, 0, 0, .5);
	}
</style>
