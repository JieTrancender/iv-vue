<template>
	<view>
		<cu-custom bgColor="bg-gradual-blue"><block slot="content">个人中心</block></cu-custom>
		<view v-if="!hasLogin">
			<view class="padding flex flex-wrap justify-between aligin-center bg-white">
				<button class="cu-btn round" @click="logout">注销</button>
				<button class="cu-btn round" @click="login">登录</button>
			</view>
		</view>
		<view v-else class="cu-list menu-avatar comment solids-to">
			<view class="cu-item">
				<view class="cu-avatar round" style="background-image:url(userInfo.avatarUrl || '');"></view>
				<view class="content">
					<view class="text-grey">{{ userInfo.nickName }}</view>
					<view class="text-gray text-content text-df">
						{{ userInfo.city }} {{ userInfo.province }} {{ userInfo.country }}
					</view>
					<view class="margin-top-sm flex justify-between">
						<view class="text-gray text-df">2018年12月4日</view>
					</view>
				</view>
			</view>
		</view>
		
		<!-- <block v-if="TabCur==0">
			<view class="cu-bar bg-white  margin-top solid-bottom">
				<view class="action">
					<text class="cuIcon-title text-blue"></text>等分列
				</view>
				<view class="action"></view>
			</view> -->
			<!-- <view class="bg-white padding">
				<view class="grid col-1 margin-bottom">
					<label for="account">账号:</label>
					<input type="text" id="account" placeholder="please input account.">
				</view>
				<view class="grid col-1 margin-bottom">
					<label for="password">密码:</label>
					<input type="password" placeholder="please input password." id="password">
				</view>
			</view> -->
			
			<!-- <view class="cu-bar bg-white  margin-top solid-bottom">
				<view class="action">
					<text class="cuIcon-title text-blue"></text>等高
				</view>
				<view class="action"></view>
			</view>
			<view class="bg-white padding">
				<view class="grid col-4 grid-square">
					<view class="bg-img" v-for="(item,index) in avatar" :key="index" :style="[{ backgroundImage:'url(' + avatar[index] + ')' }]"></view>
				</view>
			</view> -->
			<view class="bg-gray margin"></view>
			<view>
				<view class="padding flex flex-wrap justify-between align-center bg-white">
					<button class="cu-btn" @click="clearIvNote" @tap="loadModal">默认</button>
					<button class="cu-btn round" @click="requestIvNote">圆角</button>
					<button class="cu-btn cuIcon">
						<text class="cuIcon-emojifill"></text>
					</button>
				</view>
			</view>
			<view class="bg-gray margin"></view>
			<view v-if="[notes.lenth > 0]" class="bg-white padding">
				<li v-for="item in notes" v-bind:key="item.id">
					<span>{{ item.id }}. {{ item.content }}  at {{ item.createdTime }}</span>
				</li>
			</view>
			
			<!-- <view class="bg-gray margin"></view>
			<view class="bg-gray margin"></view>
			<view v-if="isNotEmpty">
				<span>bottom...............</span>
			</view>
			<view class="bg-gray margin"></view>
			<view v-if="isNotEmpty">
				<span>bottom...............</span>
			</view> -->
			<view class="cu-load load-modal" v-if="loadModalBool">
				<!-- <view class="cuIcon-emojifill text-orange"></view> -->
				<image src="/static/logo.png" mode="aspectFit"></image>
				<view class="gray-text">加载中...</view>
			</view>
			<view class="load-progress" :class="loadProgressNum!=0?'show':'hide'" :style="[{top:CustomBar+'px'}]">
				<view class="load-progress-bar bg-green" :style="[{transform: 'translate3d(-' + (100-loadProgressNum) + '%, 0px, 0px)'}]"></view>
				<view class="load-progress-spinner text-green"></view>
			</view>
		<!-- </block> -->
	</view>
</template>

<script>
	export default {
		onLoad() {
			console.log("onLoad.....");
		},
		data() {
			return {
				CustomBar: this.CustomBar,
				TabCur: 0,
				avatar:['https://ossweb-img.qq.com/images/lol/web201310/skin/big10001.jpg','https://ossweb-img.qq.com/images/lol/web201310/skin/big81005.jpg','https://ossweb-img.qq.com/images/lol/web201310/skin/big25002.jpg','https://ossweb-img.qq.com/images/lol/web201310/skin/big99008.jpg'],
				tabNav: ['Flex布局', 'Grid布局', '辅助布局'],
				notes: [],
				loadProgressNum: 0,
				loadModalBool: false,
				userInfo: {},
				hasLogin: false,
				canIUse: wx.canIUse('button.open-type.getUserInfo'),
				testData: 1,
				nickName: '',
			};
		},
		onShow() {
			console.log("onShow......");
		},
		onHide() {
			console.log("onHide......");
		},
		methods: {
			tabSelect(e) {
				this.TabCur = e.currentTarget.dataset.id;
				this.scrollLeft = (e.currentTarget.dataset.id - 1) * 60
			},
			requestIvNote(e) {
				console.log(e);
				this.loadProgress();
				uni.request({
				        url: 'https://keyboard-hero.com/iv/notes'
				    })
				    .then(data => {//data为一个数组，数组第一项为错误信息，第二项为返回数据
				        var [error, res]  = data;
						this.notes = res.data;
						console.log("res:", res);
						
						this.loadProgressNum = 100;
				    })
			},
			clearIvNote(e) {
				console.log(e);
				this.loadModal();
				this.notes = [];
				this.loadModalBool = false;
			},
			loadProgress(e) {
				this.loadProgressNum = this.loadProgressNum + 10;
				if (this.loadProgressNum < 100) {
					setTimeout(() => {
						this.loadProgress();
					}, 100);
				} else {
					this.loadProgressNum = 0;
				}
				
				console.log("loadProgress", this.loadProgressNum);
			},
			loadModal(e) {
				this.loadModalBool = true;
				setTimeout(() => {
					this.loadModalBool = false;
				}, 100);
			},
			login() {
				console.log("login......");
				uni.login({
					provider: 'weixin'
				})
				.then(data => {
					let [err, res] = data;
					console.log(err, res);
					
					uni.request({
						url: 'https://keyboard-hero.com/iv/verify',
						data: {
							code: res.code
						},
						method: "POST"
					})
					.then(data => {
						let [err, res] = data;
						console.log(err, res);
						
						// 获取用户信息
						uni.getUserInfo({
							provider: 'weixin',
						})
						.then(data => {
							let [err, res] = data;
							console.log(err, res);
							
							this.userInfo = res.userInfo;
							this.hasLogin = true;
						})
					})
					
				})
			},
			logout() {
				this.hasLogin = false;
				this.userInfo = {};
			}
		},
		computed: {
			isNotEmpty() {
				return this.notes.length > 0;
				
				return true;
			}
		}
	}
</script>

<style>
	page {
		/* padding-top: 45px; */
	}
</style>
