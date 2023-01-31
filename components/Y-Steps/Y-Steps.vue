<template>
	<!-- 获取一个数组，结构为[{date:'时间',info:'内容内容'}] -->
	<!-- @click事件返回点击标签元素的索引值 第一项为0 -->
	<view class="bg">
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
						<view class="content" :style="{WebkitLineClamp:lineNum!=0?lineNum:''}">
							<view>1.请上传清晰的<text>珠宝行业《营业执照》复印件照片，并加盖公章</text>，如示例图所示。</view>
							<view>2.无公章，可在纸条上手写<text>“本人保证提供的资料(身份证营业执照)均为真实有效且合法的资料，若本人提供虚假资料，本人愿承担与此相关的全部责任。正楷签名并按手印”</text>和营业执照一起拍照上传
								(不要遮营业执照重要信息)！</view>
							<view>3.图片要求:清晰无遮掩，平铺，要能看清法人、国徽等信息。</view>
						</view>
						<view class="uploadImg">
							<image src="../../static/images/yyzz.png" mode="widthFix"></image>
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
						<view class="content" :style="{WebkitLineClamp:lineNum!=0?lineNum:''}">
							<view>1.请上传清晰的与<text>营业执照法人一致</text>的身份证正反面照片</view>
							<view>2.图片要求:清晰无遮掩，平铺，要能看清本人的头像和姓名等信息。</view>
						</view>
						<view class="uploadImg">
							<image src="../../static/images/sfzz.png" mode="widthFix"></image>
							<image src="../../static/images/sfzf.png" mode="widthFix"></image>
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
								<input type="text" v-model="company" placeholder="请输入企业名称" />
							</view>
							<view class="form_item">
								<view>法人姓名</view>
								<input type="text" v-model="name" placeholder="请输入法人姓名" />
							</view>
							<view class="form_item">
								<view>手机号</view>
								<input type="number" v-model="phone" maxlength="11" placeholder="请输入手机号" />
							</view>
						</view>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		name: 'YSteps',
		props: {
			infoList: {
				type: Array,
				default: []
			},
			color: {
				type: String,
				default: '#000'
			},
			backgroundColor: {
				type: String,
				default: '#F1DA8E'
			},
			lineNum: {
				type: Number,
				default: 0
			}
		},
		data() {
			return {};
		},
		onLoad(e) {
			//获取列表
		},
		methods: {
			// topage(e) {
			// 	this.$emit('click', e);
			// }
		}
	};
</script>

<style lang="scss" scoped>
	page {
		background-color: #F9F9F7;
	}

	.bg {
		// margin: 20upx 0;
		background-color: #F9F9F7;
		display: flex;
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
							width: 260rpx;
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
								background-color: rgba(0, 0, 0, .1);
								padding: 8rpx 10rpx;
							}
						}
					}
				}

				// .info_item:active {
				// 	background-color: #f4f4f4;
				// }
			}
		}
	}

</style>
