<template>
	<view id="pageIndex">
		<headerSerch :dataConfig="lprice" :scrollTop='scrollTop' :bgColor="bgColor" :bgIndex='bgIndex' @click.native="bindEdit('headerSerch','default')"></headerSerch>
		<swiperBg :dataConfig="swiperBg" @click.native="bindEdit('swiperBg','default')" @swiperIndex='swiperIndex'></swiperBg>
		<view class="tabs">
			<view class="tabs_t">
				<view class="tabs_t_item" :class="{active:activeJTab==1}" @click="changeJTab(1)">
					金饰
				</view>
				<view class="tabs_t_item" :class="{active:activeJTab==2}" @click="changeJTab(2)">
					款式
				</view>
			</view>
			<view class="tabs_m" v-if="activeJTab==1">
				<view class="tabs_m_item" v-for="item in jinList" @click="gotoList(item.value,item.title,'pl',1)">
					<image :src="item.img" mode="widthFix"></image>
				</view>
			</view>
			<view class="tabs_m" v-else>
				<view class="tabs_m_menu" v-for="item in category" @click="gotoList(item.id,item.cate_name,'cid',1)">
					<image :src="item.pic" mode="widthFix"></image>
					<text>{{item.cate_name}}</text>
				</view>
			</view>
		</view>
		<view class="tabs">
			<view class="tabs_t">
				<view class="tabs_t_item" :class="{active:activeYTab==1}" @click="changeYTab(1)">
					银饰
				</view>
				<view class="tabs_t_item" :class="{active:activeYTab==2}" @click="changeYTab(2)">
					款式
				</view>
			</view>
			<view class="tabs_m" v-if="activeYTab==1">
				<view class="tabs_m_item" v-for="item in yinList" @click="gotoList(item.value,item.title,'pl',2)">
					<image :src="item.img" mode="widthFix"></image>
				</view>
			</view>
			<view class="tabs_m" v-else>
				<view class="tabs_m_menu" v-for="item in category" @click="gotoList(item.id,item.cate_name,'cid',2)">
					<image :src="item.pic" mode="widthFix"></image>
					<text>{{item.cate_name}}</text>
				</view>
			</view>
		</view>
		<view class="authImg" @click="gotoAuth()" v-if="authType<=1">
			<image src="https://jjh.iqweb.net/uploads/attach/2022/12/20221214/e48a9f9768cc3217a6bfaceb3029913e.jpg" mode="widthFix"></image>
		</view>
		<view class="ads">
			<image :src="item.img" mode="widthFix" v-for="(item,index) in ads"	 @click="gotoList(item.id,item.comment,'zc')"></image>
		</view>

		<view class="special">
			<image :src="item.img" mode="widthFix" v-for="(item,index) in zchang" @click="gotoList(item.id,item.title,'zc')" ></image>
		</view>
		<view class="new">
			<view class="title">
				平台<text>上新</text>
			</view>
			<view class="newList">
				<view class="newItem" v-for="item in newList" @click="goDetail(item)">
					<image :src="item.image" mode="aspectFill"></image>
				</view>
			</view>
		</view>
		<view class="navs">
			<view class="navItem" v-for="(item,index) in goodTitle" @click="changeNav(index,item)">
				<view>{{item.title}}</view>
				<view :class="{active:activeNav == index}">{{item.ltitle}}</view>
			</view>
		</view>
		<recommend :goodList="goodList" :authType='authType' :zcInfo='zcInfo' @click.native="bindEdit('goodList','aa')"></recommend>
		<view v-if="site_config && !isIframe" class="site-config" @click="goICP">{{ site_config }}</view>
		<view class="uni-p-b-98" v-if="!isIframe"></view>
		<couponWindow style="position:relative;z-index:10000;" :window="isCouponShow" @onColse="couponClose"
			:couponImage="couponObj.image" :couponList="couponObj.list"></couponWindow>

	</view>
</template>

<script>
	import couponWindow from '@/components/couponWindow/index';
	import headerSerch from './components/headerSerch';
	import swiperBg from './components/swiperBg';
	import recommend from './components/recommend';
	import { getUserInfo } from '@/api/user.js';
	import {
		getShare,
		follow
	} from '@/api/public.js';
	// #ifdef MP
	import {
		SUBSCRIBE_MESSAGE,
		TIPS_KEY
	} from '@/config/cache';
	// #endif
	import {
		getTemlIds,
		siteConfig
	} from '@/api/api.js';
	import {
		mapGetters
	} from 'vuex';
	import {
		getIndexData,
		getCouponV2,
		getCouponNewUser
	} from '@/api/api.js';
	import {
		toLogin
	} from '@/libs/login.js';
	let app = getApp();
	let statusBarHeight = uni.getSystemInfoSync().statusBarHeight;
	export default {
		computed: mapGetters(['isLogin', 'uid']),
		components: {
			couponWindow,
			headerSerch,
			swiperBg,
			recommend
		},
		data() {
			return {
				scrollTop:0,
				activeNav:0,
				activeJTab:1,
				activeYTab:1,
				followHid: true,
				followUrl: '',
				followCode: false,
				navH: statusBarHeight,
				subscribe: false,
				iShidden: false,
				goodType: 3,
				loading: false,
				loadend: false,
				loadTitle: '下拉加载更多', //提示语
				page: 1,
				limit: this.$config.LIMIT,
				numConfig: 0,
				couponObj: {},
				isCouponShow: false,
				shareInfo: {},
				site_config: '',
				isIframe: app.globalData.isIframe,
				headerSerch: {}, //头部搜索
				swiperBg: {}, //轮播
				goodList:[],
				goodTitle:[],
				isBorader: '',
				authType:0,
				bgIndex:0,
				bgColor:[],// linear-gradient 渐变色需要四个颜色属性
				category:[],
				jinList:[],
				yinList:[],
				ads:[],
				zchang:[],
				newList:[],
				zcInfo:{},
				lprice:[]
			};
		},
		onPageScroll(e) {
			this.scrollTop = e.scrollTop
		},
		onLoad(options) {
			let that = this;
			// #ifdef H5
			if (app.globalData.isIframe) {
				setTimeout(() => {
					let active;
					document.getElementById('pageIndex').children.forEach(dom => {
						dom.addEventListener('click', (e) => {
							e.stopPropagation();
							e.preventDefault();
							if (dom === active) return;
							dom.classList.add('borderShow');
							active && active.classList.remove('borderShow');
							active = dom;
						})
					})
				});
			}
			if (app.globalData.isIframe) {
				uni.hideTabBar();
			}
			this.getFollow();
			if (that.$wechat.isWeixin()) {
				that.$wechat.location().then(res => {
					uni.setStorageSync('user_latitude', res.latitude);
					uni.setStorageSync('user_longitude', res.longitude);
				})
			} else {
				// #endif	
				uni.getLocation({
					type: 'wgs84',
					success: function(res) {
						try {
							uni.setStorageSync('user_latitude', res.latitude);
							uni.setStorageSync('user_longitude', res.longitude);
						} catch {}
					}
				});

				// #ifdef H5	
			}
			// #endif
			this.getIndexData();
			this.setOpenShare();
			// #ifdef MP
			if (this.$Cache.get(TIPS_KEY)) this.iShidden = true;
			this.getTemlIds();
			// #endif
		},
		// #ifdef MP
		//发送给朋友
		onShareAppMessage: function() {
			// 此处的distSource为分享者的部分信息，需要传递给其他人
			let that = this;
			return {
				title: this.shareInfo.title,
				path: '/pages/index/index',
				imageUrl: this.shareInfo.img
			};
		},
		//分享到朋友圈
		onShareTimeline: function() {
			return {
				title: this.shareInfo.title,
				imageUrl: this.shareInfo.img
			};
		},
		// #endif
		onShow() {
			siteConfig().then(res => {
				this.site_config = res.data.record_No
			}).catch(err => {
				return this.$util.Tips({
					title: err.msg
				});
			});
			/*
			 * 获取用户信息
			 */
			getUserInfo().then(res => {
				this.authType = res.data.auth_type
				uni.setStorageSync('authType',this.authType)
			});
			if (!app.globalData.isIframe) {
				uni.showTabBar();
				if (this.isLogin) {
					this.getCoupon();
				}
			}
		},
		methods: {
			goDetail(item){
				uni.navigateTo({
					url:`/pages/goods_details/index?id=${item.id}`
				})
			},
			swiperIndex(e){
				this.bgIndex = e
			},
			gotoAuth(){
				uni.navigateTo({
					url:'/pages/auth/auth'
				})
			},
			gotoList(id,title,type,b){
				let listUrl = '/pages/goods_list/index?title='+title
				let url = ''
				if(type=='pl'){
					url = listUrl+'&pl='+id+'&b='+b
				}else if(type=='cid'){
					url = listUrl+'&cid='+id+'&b='+b
				}else if(type=='zc'){
					url = listUrl+'&zc='+id
				}
				uni.navigateTo({
					url
				})
			},
			changeNav(type,item){
				this.activeNav = type
				this.goodList = item.sons
				this.zcInfo = item
			},
			changeJTab(type){
				this.activeJTab = type
			},
			changeYTab(type){
				this.activeYTab = type
			},
			bindEdit(name, dataName) {
				if (app.globalData.isIframe) {
					window.parent.postMessage({
							name: name,
							dataName: dataName,
							params: {}
						},
						'*'
					);
					return;
				}
			},
			getFollow() {
				follow().then(res => {
					this.followUrl = res.data.path;
				}).catch((err) => {
					return this.$util.Tips({
						title: err.msg
					});
				});
			},
			// 优惠券弹窗
			getCoupon() {
				const tagDate = uni.getStorageSync('tagDate') || '',
					nowDate = new Date().toLocaleDateString();
				if (tagDate === nowDate) {
					this.getNewCoupon();
				} else {
					getCouponV2().then(res => {
						const {
							data
						} = res;
						if (data.list.length) {
							this.isCouponShow = true;
							this.couponObj = data;
							uni.setStorageSync('tagDate', new Date().toLocaleDateString());
						} else {
							this.getNewCoupon();
						}
					});
				}
			},
			// 新用户优惠券
			getNewCoupon() {
				const oldUser = uni.getStorageSync('oldUser') || 0;
				if (!oldUser) {
					getCouponNewUser().then(res => {
						const {
							data
						} = res;
						if (data.show) {
							if (data.list.length) {
								this.isCouponShow = true;
								this.couponObj = data;
								uni.setStorageSync('oldUser', 1);
							}
						} else {
							uni.setStorageSync('oldUser', 1);
						}
					});
				}
			},
			// 优惠券弹窗关闭
			couponClose() {
				this.isCouponShow = false;
				if (!uni.getStorageSync('oldUser')) {
					this.getNewCoupon();
				}
			},
			// 授权关闭
			// authColse: function(e) {
			// 	this.isShowAuth = e;
			// },
			// #ifdef MP
			getTemlIds() {
				let messageTmplIds = wx.getStorageSync(SUBSCRIBE_MESSAGE);
				if (!messageTmplIds) {
					getTemlIds().then(res => {
						if (res.data) wx.setStorageSync(SUBSCRIBE_MESSAGE, JSON.stringify(res.data));
					});
				}
			},
			// #endif
			goICP() {
				// #ifdef H5
				window.open('http://beian.miit.gov.cn/');
				// #endif
				// #ifdef MP
				uni.navigateTo({
					url: `/pages/annex/web_view/index?url=https://beian.miit.gov.cn/`
				});
				// #endif
			},
			onLoadFun() {},
			getIndexData() {
				getIndexData().then(res => {
					this.subscribe = res.data.subscribe;
					this.swiperBg = res.data.banner
					this.bgColor = []
					this.swiperBg.forEach(item=>{
						this.bgColor.push("linear-gradient("+item.color+",#f1f1f1)")
					})
					this.category = res.data.category
					res.data.class.forEach(item=>{
						if(item.type==1){
							this.jinList.push(item)
						}else{
							this.yinList.push(item)
						}
					})
					this.ads = res.data.ads
					this.zchang = res.data.zchang
					this.goodTitle = res.data.list
					this.zcInfo = res.data.list[0]
					this.goodList = res.data.list[0].sons
					this.newList = res.data.is_new
					this.lprice = res.data.lprice
					// #ifdef H5
					localStorage.setItem("itemName", res.data.site_name);
					// #endif
					uni.setNavigationBarTitle({
						title: res.data.site_name
					});
				});
			},
			// 微信分享；
			setOpenShare: function() {
				let that = this;
				getShare().then(res => {
					let data = res.data.data;
					this.shareInfo = data;
					// #ifdef H5
					let url = location.href;
					if (this.$store.state.app.uid) {
						url = url.indexOf('?') === -1 ? url + '?spread=' + this.$store.state.app.uid : url +
							'&spread=' + this.$store.state.app.uid;
					}
					if (that.$wechat.isWeixin()) {
						let configAppMessage = {
							desc: data.synopsis,
							title: data.title,
							link: url,
							imgUrl: data.img
						};
						that.$wechat.wechatEvevt(['updateAppMessageShareData', 'updateTimelineShareData'],
							configAppMessage);
					}
					// #endif
				});
			}
		},
		onReachBottom: function() {
			// this.getGroomList();
		}
	};
</script>

<style lang="scss">
	#pageIndex{
		height: 100vh;
	}
	.ads{
		margin: 20rpx;
		image{
			width: 100%;
			height: 240rpx;
			margin-bottom: 20rpx;
		}
	}
	.special{
		margin: 20rpx;
		image{
			width: 100%;
			height: 240rpx;
			margin-bottom: 20rpx;
		}
	}
	.authImg{
		margin: 20rpx;
		image{
			width: 100%;
			border-radius: 20rpx;
		}
	}
	.auth{
		margin: 20rpx;
		padding: 20rpx;
		border-radius: 20rpx;
		background-image: linear-gradient(135deg, #FDE6BA, #FCCC7F);
		display: flex;
		flex-direction: column;
		align-items: center;
		.title{
			text-align: center;
			font-size: 40rpx;
			margin-bottom: 10rpx;
			font-weight: 700;
		}
		.ctitle{
			text-align: center;
			padding: 4rpx 10rpx;
			background-color: #F9CB69;
			border-radius: 10rpx;
		}
		.auth_main{
			margin: 30rpx 0;
			background-color: rgba(255, 255, 255, 0.5);
			border-radius: 20rpx;
			padding: 30rpx 20rpx;
			display: flex;
			flex-wrap: wrap;
			
			.auth_item{
				width: 49%;
				display: flex;
				margin-right: 10rpx;
				&:nth-child(1),&:nth-child(2){
					margin-bottom: 30rpx;
				}
				&:nth-child(2n){
					margin-right: 0;
				}
				image{
					width: 80rpx;
					height: 80rpx;
					flex-shrink: 0;
					margin-right: 10rpx;
				}
				.text{
					view{
						font-size: 30rpx;
						font-weight: 600;
						&:last-child{
							font-size: 20rpx;
							color: #666;
						}
					}
				}
			}
		}
		.auth_btn{
			background-image: linear-gradient(135deg, #EC880E, #ED7F06);
			padding: 20rpx 30rpx;
			border-radius: 50rpx;
			color: #fff;
		}
	}
	.navs{
		margin: 20rpx;
		display: flex;
		overflow: auto;
		.navItem{
			flex-shrink: 0;
			padding: 2rpx 50rpx;
			border-right: 1px solid #999;
			&:last-child{
				border: none;
			}
			&:first-child{
				padding: 2rpx 50rpx 2rpx 0;
			}
			view{
				text-align: center;
				font-size: 36rpx;
				margin-bottom: 8rpx;
				&:last-child{
					font-size: 24rpx;
					color: #999;
					padding: 0 20rpx;
					border-radius: 40rpx;
				}
				
			}
			.active{
				background-color: red;
				color: #fff !important;
				
			}
		}
	}
	.new{
		margin: 20rpx;
		border-radius: 20rpx;
		padding: 20rpx;
		background-color: #fff;
		.title{
			font-size: 30rpx;
			font-weight: 700;
			margin-bottom: 20rpx;
			text{
				color: #FF7113;
			}
		}
		.newList{
			display: flex;
			overflow-x: auto;
			.newItem{
				flex-shrink: 0;
				height: 180rpx;
				width: 180rpx;
				margin-right: 20rpx;
				image{
					width: 100%;
					height: 100%;
					border-radius: 20rpx;
				}
			}
		}
	}
	.realTime{
		margin: 20rpx;
		border-radius: 20rpx;
		padding: 20rpx;
		background-image: linear-gradient(45deg, #fff,#fff, #FFF2E3);
		.title{
			margin-bottom: 20rpx;
			font-size: 30rpx;
			text{
				color: red;
				font-weight: 700;
				margin-left: 10rpx;
			}
		}
		.ritem_top{
			display: flex;
			align-items: center;
			justify-content: space-between;
			background-image: linear-gradient(45deg, #fff, #FDCBC6);
			padding: 20rpx;
			border-radius: 10rpx;
			.ritem_top_l{
				flex: 1;
				.rTtitle{
					font-size: 36rpx;
					font-weight: 700;
					color: #5F181E;
				}
				.rT_m{
					display: flex;
					justify-content: space-between;
					align-items: flex-end;
					.rT_m_l{
						font-size: 20rpx;
						text{
							margin: 0 6rpx;
						}
					}
					.rT_m_r{
						font-size: 24rpx;
						color: red;
						text{
							font-size: 36rpx;
						}
					}
				}
				
			}
			.ritem_top_r{
				width: 20%;
				image{
					width: 100%;
					height: 80rpx;
				}
			}
		}
		.ritem_cen{
			display: flex;
			margin: 8rpx 0;
			.cen_item{
				display: flex;
				background-color: #FEF5E9;
				border-radius: 10rpx;
				flex: 1;
				margin-right: 8rpx;
				&:last-child{
					margin-right: 0;
				}
				padding: 10rpx;
				.cen_item_l{
					display: flex;
					flex-direction: column;
					.ctitle{
						font-size: 24rpx;
						color: #999;
					}
					.cTitle{
						font-size: 28rpx;
					}
					.price{
						font-size: 20rpx;
						color: red;
						text{
							font-size: 28rpx;
						}
					}
				}
				image{
					width: 20%;
					height: 50rpx;
				}
			}
		}
		.ritem_bom{
			display: flex;
			.ritem_bom_l{
				display: flex;
				flex-direction: column;
				background-color: #FEF5E9;
				border-radius: 10rpx;
				width: 32.6%;
				margin-right: 8rpx;
				padding: 10rpx;
				flex-shrink: 0;
				.ctitle{
					font-size: 24rpx;
					color: #999;
				}
				.cTitle{
					font-size: 28rpx;
				}
				.price{
					font-size: 20rpx;
					color: red;
					text{
						font-size: 28rpx;
					}
				}
				image{
					width: 100%;
					height: 80rpx;
				}
			}
			.ritem_bom_r{
				display: flex;
				flex-wrap: wrap;
				width: 67%;
				.cen_item{
					display: flex;
					background-color: #FEF5E9;
					border-radius: 10rpx;
					width: 49%;
					margin-right: 8rpx;
					padding: 10rpx;
					margin-bottom: 8rpx;
					&:nth-child(2n){
						margin-right: 0;
					}
					.cen_item_l{
						display: flex;
						flex-direction: column;
						.ctitle{
							font-size: 24rpx;
							color: #999;
						}
						.cTitle{
							font-size: 28rpx;
						}
						.price{
							font-size: 20rpx;
							color: red;
							text{
								font-size: 28rpx;
							}
						}
					}
					image{
						width: 20%;
						height: 50rpx;
					}
				}
			}
		}
	}
	.tabs{
		background-color: #fff;
		margin: 20rpx;
		border-radius: 20rpx;
		overflow: hidden;
		.tabs_t{
			display: flex;
			justify-content: center;
			.tabs_t_item{
				padding: 20rpx 0;
				flex: 1;
				font-weight: 700;
				text-align: center;
				background-color: #F4EAD1;
			}
			.active{
				background-color: #fff;
				color: red;
			}
		}
		.tabs_m{
			border-radius: 10px;
			overflow: hidden;
			display: flex;
			flex-wrap: wrap;
			margin: 10rpx;
			.tabs_m_menu{
				padding: 20rpx 0;
				display: flex;
				flex-direction: column;
				align-items: center;
				width: 24.5%;
				height: 200rpx;
				margin-right: 4rpx;
				margin-bottom: 4rpx;
				&:nth-child(4n){
					margin-right: 0;
				}
				image{
					flex-shrink: 0;
					width: 80%;
					height: 100rpx;
				}
			}
			.tabs_m_item{
				display: flex;
				flex-direction: column;
				align-items: center;
				width: 24.5%;
				height: 200rpx;
				margin-right: 4rpx;
				margin-bottom: 4rpx;
				&:nth-child(4n){
					margin-right: 0;
				}
				.title{
					font-weight: 700;
				}
				.num{
					font-size: 26rpx;
					color: #B99F62;
				}
				image{
					width: 100%;
					height: 100%;
				}
			}
		}
		
	}
	page {
		// background: #fff;
	}

	.swiperCon {
		margin: 20rpx 0 !important;

		/* #ifdef MP */
		/deep/.swiperBg {
			margin: 20rpx 0 !important;
		}

		/* #endif */
		/deep/.swiper {
			swiper,
			.swiper-item,
			image {
				height: 190rpx !important;
			}
		}
	}

	.site-config {
		margin: 40rpx 0;
		font-size: 24rpx;
		text-align: center;
		color: #666;

		&.fixed {
			position: fixed;
			bottom: 69px;
			left: 0;
			width: 100%;
		}
	}

	/* #ifdef MP */
	.indexTip {
		position: fixed;
		right: 42rpx;
		z-index: 1000;

		.tip {
			width: 400rpx;
			border-radius: 6rpx;
			background-color: #fff;
			padding: 15rpx 22rpx;
			position: relative;

			&::before {
				content: "";
				width: 0;
				height: 0;
				border-left: 15rpx solid transparent;
				border-right: 15rpx solid transparent;
				border-bottom: 17rpx solid #fff;
				position: absolute;
				top: -14rpx;
				right: 95rpx;
			}
		}

		.text {
			font-size: 22rpx;
			color: #333;
			width: 320rpx;

			image {
				width: 30rpx;
				height: 16rpx;
				display: inline-block;
			}
		}

		.iconfont {
			color: #cdcdcd;
			font-size: 32rpx;
		}
	}

	/* #endif */
	/* #ifdef H5 */
	.follow {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		background-color: rgba(0, 0, 0, 0.36);
		height: 80rpx;
		font-size: 28rpx;
		color: #fff;
		padding: 0 30rpx;
		z-index: 100000;

		.iconfont {
			font-size: 30rpx;
			margin-left: 29rpx;
		}

		.bnt {
			width: 160rpx;
			height: 50rpx;
			background-color: #E93323;
			border-radius: 25rpx;
			font-size: 24rpx;
			text-align: center;
			line-height: 50rpx;
		}
	}

	.followCode {
		.mask {
			z-index: 10000;
		}

		.pictrue {
			width: 500rpx;
			height: 720rpx;
			border-radius: 12px;
			left: 50%;
			top: 50%;
			margin-left: -250rpx;
			margin-top: -360rpx;
			position: fixed;
			z-index: 10001;

			img {
				width: 100%;
				height: 100%;
			}
		}
	}

	/* #endif */
</style>
