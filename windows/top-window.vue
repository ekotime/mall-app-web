<template>
	<view class="top-window-header">
		<view class="left-header logo">
			<navigator class="logo" open-type="reLaunch" url="/pages/index/index-pc">
				<image src="../static/logo.png" mode="aspectFit" style="width: 30px;"></image>
				<text>Hello</text>
			</navigator>
		</view>
		<custom-tab-bar class="tab-bar-flex" direction="horizontal" :show-icon="false" :selected="current" @onTabItemTap="toSecondMenu" />
	</view>
</template>

<script>
	export default {
		data() {
			return {
				selected: {
					index: 0,
					category: 1,
					cart: 2,
					order:3,
					user: 4,
					product: 1,
					set: 4,
					order: 3,
				},
				current: 0,
				indexPage: [{
					tabBar: '/pages/index/index',
					index: '/pages/index/index-pc'
				}, {
					tabBar: '/pages/category/category',
					index: '/pages/category/category-pc?fid=1&sid=8'
				}, {
					tabBar: '/pages/cart/cart',
					index: '/pages/cart/cart-pc'
				}, {
					tabBar: '/pages/user/user',
					index: '/pages/user/user-pc'
				}, {
					tabBar: '/pages/openorder/openorder',
					index: '/pages/openorder/openorder-pc?state=-1'
				}]
			}
		},
		watch: {
			$route: {
				immediate: true,
				handler(newRoute) {
					const width = uni.getSystemInfoSync().screenWidth
					// console.log('screenWidth:'+width)
					if (width >= 768) {
						let path = newRoute.path
						let comp
						if (path === '/') {
							comp = 'index'
							path = '/pages/index/index'
						} else {
							comp = path.split('/')[2]
						}
						this.current = this.selected[comp]
						for (const item of this.indexPage) {
							if (path === item.tabBar) {
								uni.redirectTo({
									url: item.index
								})
							}
						}
					}
				}
			}
		},
		mounted() {},
		methods: {
			toSecondMenu(e) {
				const activeTabBar = '/' + e.pagePath
				for (const item of this.indexPage) {
					if (activeTabBar === item.tabBar) {
						uni.redirectTo({
							url: item.index
						})
					}
				}
			}
		}
	}
</script>

<style>
	.top-window-header {
		height: 60px;
		padding: 0 0px;
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		align-items: center;
		box-sizing: border-box;
		border-bottom: 1px solid #e1e1e1;
		background-color: #FFFFFF;
		color: #333;
	}

	.logo {
		display: flex;
		flex-direction: row;
		align-items: center;
		flex: 1;
	}

	.logo image {
		height: 30px;
		width: 30px;
	}

	.logo text {
		margin-left: 8px;
	}

/* 	.active {
		color: #4cd964;
		border-bottom: 2px solid;
	} */

	.tab-bar-flex {
		width: 360px;
	}



</style>
