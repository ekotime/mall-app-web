<template>
	<view class="container" >
		<web-view :webview-styles="webviewStyles" src="http://192.168.20.203:9090/product/0115/10-1141-133.pdf"></web-view>
	</view>
	
</template>

<script>
export default {
	data() {
		return {
			openorderList: [],
			id: '',
			rowData: null,
			current: 0,
			currentShipment: 0,
			pic:'',
			visible: false,
			picList: [],
		};
	},
	async onLoad(options) {
		console.log('Received options:', options.id);
		
	},
	created() {

	},
	activated() {
		
	},
	methods: {
		
	}
};
</script>
