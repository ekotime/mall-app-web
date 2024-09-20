<template>
	<view class="left-window-style">
		<view class="second-menu">
			<!-- <keep-alive> -->
			<component v-bind:is="active" :hasLeftWin="hasLeftWin" :leftWinActive="leftWinActive"></component>
			<!-- </keep-alive> -->
		</view>
	</view>
</template>

<script>
	// import indexPage from '@/pages/tabBar/index/index.nvue'
	import categoryPage from '@/pages/tabBar/category/category.nvue'
	import { mapMutations,mapState } from 'vuex'
	let isPcObserver, isPhoneObserver
	export default {
		data() {
			return {
				nav: [
					'index',
					'category',
					'product'
				],
				isPC: false
			}
		},
		components: {
			// indexPage,
			categoryPage
		},
		computed: {
			...mapState({
				active: state => state.active,
				hasLeftWin: state => !state.noMatchLeftWindow,
				leftWinActive: state => state.leftWinActive.split('/')[3],
			})
		},
		mounted() {
			isPcObserver = uni.createMediaQueryObserver(this)
			isPhoneObserver = uni.createMediaQueryObserver(this)

			isPcObserver.observe({
				minWidth: 768
			}, matched => {
				this.isPC = matched
				const pageUrl = this.$route.path
				console.log('pageUrl:'+pageUrl)
				if (!pageUrl) return
				const pageName = this.$route.path.split('/')[3]
				console.log('pageName:'+pageName)
				if (pageUrl === '/' || this.nav.includes(pageName)) {
					const tabbarUrl = pageName ? (pageName === 'index' ? '/' : `/pages/${pageName}/${pageName}`) : '/'
					console.log('tabbarUrl:'+tabbarUrl)
					if (pageUrl === '/' || pageUrl === tabbarUrl) {
						uni.switchTab({
							url: pageUrl,
						})
						console.log('switchTab:'+pageUrl)
					}
					
				} else {
					uni.reLaunch({
						url: pageUrl
					})
					console.log('reLaunch:'+pageUrl)
				}
			})

			isPhoneObserver.observe({
				maxWidth: 767
			}, matched => {
				this.isPC = !matched
				if (matched) {
					const pageUrl = this.$route.path
					console.log('isphone:')
					console.log('pageUrl:'+pageUrl)
					const tabbarName = this.$route.path.split('/')[2]
					console.log('tabbarName:'+tabbarName)
					const tabbarUrl = tabbarName && (tabbarName === 'index' ? '/' : `/pages/${tabbarName}/${tabbarName}`)
					console.log('tabbarUrl:'+tabbarUrl)
					uni.switchTab({
						url: tabbarUrl,
						success(e) {
							uni.navigateTo({
								url: tabbarUrl
							})
							console.log('navigateTo pageUrl:'+pageUrl)
						}
					})
					console.log('switchTab tabbarUrl:'+tabbarUrl)
				}

			})
		},
		unmounted() {
			isPcObserver.disconnect()
			isPhoneObserver.disconnect()
		},
		watch: {
			isPC: {
				immediate: true,
				handler(newMatches) {
					this.setMatchLeftWindow(newMatches)
					// this.handlerRoute(this.$route)
				}
			},
			// #ifndef VUE3
			$route: {
				immediate: true,
				handler(newRoute) {
					this.handlerRoute(newRoute)
				}
			},
			// #endif
			// #ifdef VUE3
			$route(newRoute) {
				this.handlerRoute(newRoute)
			}
			// #endif
		},

		methods: {
			...mapMutations(['setMatchLeftWindow', 'setActive', 'setLeftWinActive']),

			handlerRoute(newRoute) {
				if (this.isPC) {
					if (newRoute.path === '/') {
						// uni.redirectTo({
						// 	url: '/pages/component/view/view'
						// })
					} else if (!newRoute.matched.length) {
						uni.redirectTo({
							url: '/pages/error/404'
						})
					} else {
						this.setLeftWinActive(newRoute.path)
						//console.log('leftWinActive:'+newRoute.path)
						let active = newRoute.path.split('/')[2]
						if (this.nav.includes(active)) {
							if (active === 'index') {
								active = 'categoryPage'
							}
							if (active === 'category') {
								active = 'categoryPage'
							}
							if (active === 'product') {
								active = 'categoryPage'
							}
							//console.log('active:'+active)
							this.setActive(active)
						} else {
							this.setActive('')
						}
					}
				}
			},

			switchTab() {

			}
		}
	}
</script>

<style>
	@import '../common/uni-nvue.css';

	.left-window-style {
		min-height: calc(100vh - var(--top-window-height));
		background-color: #f8f8f8;
	}

	.second-menu {
		width: 350px;
		background-color: #F8F8F8;
	}

	.icon-image {
		width: 30px;
		height: 30px;
	}
</style>
