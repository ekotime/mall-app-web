<template>
	<view class="container" >
		<t-space direction="vertical" style="width: 100%;">
			<div>
				<t-layout>
					<t-header>
						<t-breadcrumb :maxItemWidth="'150'">
							<t-breadcrumbItem to="/" :replace="true">首页</t-breadcrumbItem>
							<t-breadcrumbItem to="/pages/openorder/openorder-pc" :replace="true">Open Order</t-breadcrumbItem>
							<t-breadcrumbItem>Open Order Detail</t-breadcrumbItem>
						</t-breadcrumb>
					</t-header>
					<t-layout>
						<t-aside style="display: flex; align-items: center;">
							<t-image src="https://werka.oss-eu-central-1.aliyuncs.com/web/0021/17-1222-124.png" fit="contain" style="margin: auto;" />
						</t-aside>
						<t-content>
							<!-- 显示产品信息 -->
							<t-form layout="vertical">
								<t-form-item label="PO Date" name="personalProfile">
									<t-input v-model="rowData['PO Date']" readonly />
								</t-form-item>
								<t-form-item label="PO NO." name="personalProfile">
									<t-input v-model="rowData['PO NO.']" readonly />
								</t-form-item>
								<t-form-item label="Item NO." name="name" initialData="TDesign">
									<t-input v-model="rowData['Item NO.']" readonly />
								</t-form-item>
								<t-form-item label="Product Name" name="personalProfile">
									<t-textarea v-model="rowData['Product Name']" readonly />
								</t-form-item>
								<t-form-item label="Specific" name="personalProfile">
									<t-textarea v-model="rowData['Specific']" readonly />
								</t-form-item>
							</t-form>
						</t-content>
					</t-layout>
					<t-footer>
						<t-divider>Shipment</t-divider>
						<t-steps theme="dot" :current="currentShipment" readonly>
							<t-step-item title="open" :content="String(rowData['Open'])"></t-step-item>
							<t-step-item title="Shipping" :content="String(rowData['Shipping'])"></t-step-item>
						</t-steps>
						<t-divider>Supply</t-divider>
						<t-steps theme="dot" :current="current" readonly>
							<t-step-item :title="'Non-Delivery ' + String(rowData['Estimate Date'])" :content="String(rowData['Non-Delivery'])"></t-step-item>
							<t-step-item title="OnTheWay" :content="String(rowData['OnTheWay'])"></t-step-item>
							<t-step-item title="INSP" :content="String(rowData['INSP'])"></t-step-item>
							<t-step-item title="Stock" :content="String(rowData['Stock'])"></t-step-item>
						</t-steps>
						<t-divider>QC Picture</t-divider>
						<t-space direction="vertical" >
							<t-space :breakLine="true" :style="{ height: '600px', 'overflow-y': 'scroll' }">
							<t-image
								v-for="item in list"
								:key="item"
								src="https://werka.oss-eu-central-1.aliyuncs.com/web/0021/17-1222-124.png"
								:fit="'contain'"
								:style="{ width: '360px', height: '120px' }"
								:lazy="true"
							/>
							</t-space>
						</t-space>
					</t-footer>
				</t-layout>
			</div>
		</t-space>

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
			list: Array.from({ length: 24 }).map((_, index) => index),
		};
	},
	async onLoad(options) {
		console.log('Received options:', options.id);
		
	},
	created() {
		console.log('Route query:', this.$route.query.id);
		this.id = this.$route.query.id;
		this.openorderList = JSON.parse(localStorage.getItem('openorderList'));
		this.rowData = this.openorderList.find(item => item.ContractProductID === this.id);
		this.calCurrent();
	},
	activated() {
		this.id = this.$route.query.id;
		this.rowData = this.openorderList.find(item => item.ContractProductID === this.id);
		this.calCurrent();
	},
	methods: {
		goBack() {
			uni.navigateBack();
		},
		calCurrent() {
			// this.current = this.rowData['Non-Delivery'] ? 0 : 0;
			this.current = this.rowData['OnTheWay'] ? 1 : this.current;
			this.current = this.rowData['INSP'] ? 2 : this.current;
			this.current = this.rowData['Stock'] ? 3 : this.current;
			this.currentShipment = this.rowData['Shipping'] ? 1 : 0;
		}
	}
};
</script>

<style lang='scss'>

</style>
