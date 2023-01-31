<template>
	<view v-if="isShow" class="headers" :class="{showHead:scrollTop>0}" :style="{background:bgColor[bgIndex],height:(sysHeight+43+(scrollTop==0?160:0))+'px'}">
		<view class="mp-header" :class="{bgFFF:scrollTop>0}">
			<view class="sys-head" :style="{height:sysHeight + 'px'}"></view>
			<view class="serch-box" style="height: 43px;">
				<view class="serch-wrapper acea-row row-middle">
					<view class="logo">
						<!-- <image :src="logoConfig" mode="heightFix"></image> -->
						<swiper :autoplay="true" :interval="3000" :duration="1000" :vertical="true">
							<swiper-item v-for="(item,index) in dataConfig" :key="index">
								<view class="swiper-item">
									<view class="title" :class="{bgRed:scrollTop>0}">
										{{item.title}}
									</view>
									<view class="price" :class="{colorRed:scrollTop>0}">
										￥{{item.price}}
									</view>
								</view>
							</swiper-item>
						</swiper>
					</view>
					<navigator url="/pages/goods_list/index" class="input acea-row row-middle fillet" hover-class="none"><text class="iconfont icon-sousuo"></text>
						搜索商品</navigator>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	let app = getApp();
	let statusBarHeight = uni.getSystemInfoSync().statusBarHeight;
	import {
		goPage
	} from '@/libs/order.js'
	export default {
		name: 'headerSerch',
		props: {
			scrollTop:{
				type:Number,
				default:0
			},
			bgColor:{
				type:Array,
				default:()=>[]
			},
			bgIndex:{
				type:Number,
				default:0
			},
			dataConfig: {
				type: Object,
				default: () => {}
			}
		},
		data() {
			return {
				sysHeight: statusBarHeight,
				isShow: true,
			};
		},
		watch: {
			
		},
		mounted() {
			let that = this;
			// #ifdef MP
			this.$nextTick(function() {
				// 获取小程序头部高度
				let info = uni.createSelectorQuery().in(this).select(".mp-header");
				info.boundingClientRect(function(data) {
					that.marTop = data.height
				}).exec()
			})
			// #endif
		},
		methods: {
			goPage() {
				goPage().then(res => {
					uni.navigateTo({
						url: '/pages/goods_search/index'
					})
				})
			}
		}
	}
</script>

<style lang="scss">
	.headers{
		position: fixed;
		left: 0;
		top: 0;
		width: 100%;
		z-index: -1;
	}
	.showHead{
		z-index: 999;
	}
	.bgFFF{
		background: #fff;
	}
	.mp-header {
		position: relative;
		left: 0;
		top: 0;
		width: 100%;
		z-index: 999;
		.serch-wrapper {
			height: 100%;
			padding: 0 210rpx 0 20rpx;

			.logo {
				flex: 1.3;
				height: 70rpx;
				margin-right: 20rpx;
				swiper{
					height: 100%;
					.swiper-item{
						.title{
							font-size: 20rpx;
							background-color: #fff;
							padding: 2rpx 0;
							text-align: center;
							white-space: nowrap;
							border-radius: 10rpx;
							color: #C0A24F;
						}
						.bgRed{
							background-color: red;
							color: #fff;
						}
						.price{
							font-size: 28rpx;
							text-align: center;
							font-weight: 700;
							color: #fff;
						}
						.colorRed{
							color: red;
						}
					}
				}
				image {
					width: 100%;
					height: 100%;
				}
			}

			.input {
				height: 70rpx;
				padding: 0 0 0 30rpx;
				background: rgba(247, 247, 247, 1);
				border: 1px solid rgba(241, 241, 241, 1);
				color: #999;
				font-size: 28rpx;
				flex: 3;

				.iconfont {
					margin-right: 20rpx;
					color: #555555;
				}

				// 没有logo，直接搜索框
				&.on {
					width: 70%;
				}

				// 设置圆角
				&.fillet {
					border-radius: 80rpx;
				}

				// 文本框文字居中
				&.row-center {
					padding: 0;
				}
			}
		}
	}
</style>
