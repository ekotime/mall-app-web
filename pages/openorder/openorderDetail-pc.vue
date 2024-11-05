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
							<div class="tdesign-demo-image-viewer__base">
							<t-image-viewer v-model="visible" :images="[pic]" :closeOnEscKeydown="false">
								<template #trigger="{ open }">
								<div class="tdesign-demo-image-viewer__ui-image" @click="open">
									<img alt="test" :src="pic" class="tdesign-demo-image-viewer__ui-image--img" />
									<div class="tdesign-demo-image-viewer__ui-image--hover">
									<span><browse-icon size="1.4em" /> 预览</span>
									</div>
								</div>
								</template>
							</t-image-viewer>
							</div>
							<!-- <t-image :src="pic" fit="contain" style="margin: auto;" /> -->
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
								v-for="item in picList"
								:key="item"
								:src="item"
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
import { BrowseIcon } from 'tdesign-icons-vue';
export default {
	components: {BrowseIcon,},
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
		console.log('Route query:', this.$route.query.id);
		this.id = this.$route.query.id;
		this.openorderList = JSON.parse(localStorage.getItem('openorderList'));
		this.rowData = this.openorderList.find(item => item.ContractProductID === this.id);
		this.pic = 'http://192.168.20.203:9090/product/'+this.rowData['CustNo']+'/'+this.rowData['DC Item NO.']+'.jpg';
		this.picList = this.rowData['检验图片'] ? this.rowData['检验图片'].split(',') : [];
		this.calCurrent();
	},
	activated() {
		this.id = this.$route.query.id;
		this.rowData = this.openorderList.find(item => item.ContractProductID === this.id);
		this.pic = 'http://192.168.20.203:9090/product/'+this.rowData['CustNo']+'/'+this.rowData['DC Item NO.']+'.jpg';
		this.picList = this.rowData['检验图片'] ? this.rowData['检验图片'].split(',') : [];
		console.log('pic:', this.pic);
		console.log('picList:', this.picList);
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

<style scoped>
.tdesign-demo-image-viewer__ui-image {
  width: 200px;
  height: 200px;
  display: inline-flex;
  position: relative;
  justify-content: center;
  align-items: center;
  border-radius: var(--td-radius-small);
  overflow: hidden;
}

.tdesign-demo-image-viewer__ui-image--hover {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  position: absolute;
  left: 0;
  top: 0;
  opacity: 0;
  background-color: rgba(0, 0, 0, 0.6);
  color: var(--td-text-color-anti);
  line-height: 22px;
  transition: 0.2s;
}

.tdesign-demo-image-viewer__ui-image:hover .tdesign-demo-image-viewer__ui-image--hover {
  opacity: 1;
  cursor: pointer;
}

.tdesign-demo-image-viewer__ui-image--img {
  width: auto;
  height: auto;
  max-width: 100%;
  max-height: 100%;
  cursor: pointer;
  position: absolute;
}

.tdesign-demo-image-viewer__ui-image--footer {
  padding: 0 16px;
  height: 56px;
  width: 100%;
  line-height: 56px;
  font-size: 16px;
  position: absolute;
  bottom: 0;
  color: var(--td-text-color-anti);
  background-image: linear-gradient(0deg, rgba(0, 0, 0, 0.4) 0%, rgba(0, 0, 0, 0) 100%);
  display: flex;
  box-sizing: border-box;
}

.tdesign-demo-image-viewer__ui-image--title {
  flex: 1;
}

.tdesign-demo-popup__reference {
  margin-left: 16px;
}

.tdesign-demo-image-viewer__ui-image--icons .tdesign-demo-icon {
  cursor: pointer;
}

.tdesign-demo-image-viewer__base {
  width: 230px;
  height: 230px;
  margin: 10px;
  border: 4px solid var(--td-bg-color-secondarycontainer);
  border-radius: var(--td-radius-medium);
}
</style>
