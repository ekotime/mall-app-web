<template>
	<view class="content">
		<template>
			<t-table bordered stripe :table-layout="tableLayout" :pagination="pagination"
				:expanded-row-keys="expandedRowKeys" :expanded-row="expandedRow" :expandIcon="expandIcon" expandOnRowClick
				:max-height="fixedTopAndBottomRows ? 1000 : 800"
				row-key="ContractProductID" :data="openorderList" :columns="columns"
				@expand-change="rehandleExpandChange" @page-change="onPageChange" @filter-change="onFilterChange"
				@change="onChange" />
		</template>
	</view>
</template>

<script>
import uniLoadMore from '@/components/uni-load-more/uni-load-more.vue';
import empty from "@/components/empty";
import {
	formatDate
} from '@/utils/date';
import {
	fetchOpenorderList
} from '@/api/order.js';
import isNumber from 'lodash/isNumber';
export default {
	components: {
		uniLoadMore,
		empty
	},
	data() {
		return {
			tabCurrentIndex: 0,
			openorderParam: {
				key: ''
			},
			openorderList: [],
			openorderListIntial: [],
			tableLayout: 'fixed',
			// 是否冻结首尾两行
			fixedTopAndBottomRows: false,
			expandIcon: false,
			pagination: {
				current: 1,
				pageSize: 10,
				// defaultCurrent: 2,
				// defaultPageSize: 5,
				total: 0,
				showJumper: true,
			},
			filterValue: {},
			columns: [{
				colKey: 'PO Date',
				title: 'PO Date',
				width: '120',
				// sorter: true,
			},
			{
				colKey: 'PO NO.', title: 'PO NO.', width: '120',filter: {
					type: 'input',
					// 文本域搜索
					// component: Textarea,
					resetValue: '',
					// 按下 Enter 键时也触发确认搜索
					confirmEvents: ['onEnter'],
					props: {
						placeholder: '输入关键词过滤',
						onChange: this.oneEmailChange,
					},
					// 是否显示重置取消按钮，一般情况不需要显示
					showConfirmAndReset: true,
				},
			},
			{ colKey: 'Item NO.', title: 'Item NO.',width: '120',},
			{
				colKey: 'Product Name', title: 'Product Name', width: '200',// 浮层浅色背景，方向默认朝下出现
				ellipsis: {
					theme: 'light',
					placement: 'bottom',
				},
			},
			{
				colKey: 'Specific', title: 'Specific', width: '200',// 浮层浅色背景，方向默认朝下出现
				ellipsis: {
					theme: 'light',
					placement: 'bottom',
				},
			},
			{ colKey: 'ETD', title: 'ETD',width: '120',},
			// {colKey: 'Price',title: 'Price',sorter: true,},
			{ colKey: 'PO QTY', title: 'PO QTY',width: '60',ellipsisTitle: true,},
			{ colKey: 'Stock', title: 'Stock',width: '60',ellipsisTitle: true,},
			// {colKey: 'Open',title: 'Open',sorter: true,},
			{ colKey: 'Shipping', title: 'Shipping',width: '60',ellipsisTitle: true,},
			// { colKey: 'Shipping Value', title: 'Shipping Value',width: '100',ellipsisTitle: true,},
			// {colKey: 'INSP',title: 'INSP',sorter: true,},
			// {colKey: 'OnTheWay',title: 'OnTheWay',sorter: true,},
			// { colKey: 'Non-Delivery', title: 'Non-Delivery',width: '60',ellipsisTitle: true,},
			{ colKey: 'Estimate Date', title: 'Estimate Date',width: '120',ellipsisTitle: true,}
			],
			expandedRowKeys: [],
			expandedRow: (h, { row }) => (//<img src="https://werka.oss-eu-central-1.aliyuncs.com/web/0021/17-1222-124.png" style="width: 100%;" />
				<div class="more-detail">
					<p class="title">
						<b>创建时间:</b>
					</p>
					<p class="content">{row.StockProductID}</p>
					<br />
					<p class="title">
						<b>邮箱地址:</b>
					</p>
					<p class="content">{row.StockProductID}</p>
					<br />
					<p class="title">
						<b>签署方式:</b>
					</p>
					<p class="content">{row.StockProductID}</p>
				</div>
			),
		};
	},
	onLoad(options) {
		// #ifndef MP
		this.loadData()
		// #endif
	},
	filters: {
		formatStatus(status) {
			let statusTip = '';
			switch (+status) {
				case 0:
					statusTip = '等待付款';
					break;
				case 1:
					statusTip = '等待发货';
					break;
				case 2:
					statusTip = '等待收货';
					break;
				case 3:
					statusTip = '交易完成';
					break;
				case 4:
					statusTip = '交易关闭';
					break;
			}
			return statusTip;
		},
		formatProductAttr(jsonAttr) {
			let attrArr = JSON.parse(jsonAttr);
			let attrStr = '';
			for (let attr of attrArr) {
				attrStr += attr.key;
				attrStr += ":";
				attrStr += attr.value;
				attrStr += ";";
			}
			return attrStr
		},
		formatDateTime(time) {
			if (time == null || time === '') {
				return 'N/A';
			}
			let date = new Date(time);
			return formatDate(date, 'yyyy-MM-dd') //yyyy-MM-dd hh:mm:ss
		},
	},
	methods: {
		//获取订单列表
		loadData(type = 'refresh') {
			this.openorderParam.key = 'openorder:0021'
			fetchOpenorderList(this.openorderParam).then(response => {
				this.openorderList = JSON.parse(response.data)
				this.openorderListIntial = this.openorderList
				this.pagination.total = this.openorderList.length
			});
		},
		// filters 参数包含自定义过滤组件 日期选择器 的值
		onFilterChange(filters, ctx) {
			console.log('filter-change', filters, ctx);
			console.log(ctx.trigger)
			if (ctx.trigger === 'confirm') {
					this.filterValue = filters;
				// 模拟异步请求进行数据过滤
				this.request(this.filterValue);
			}else if(ctx.trigger === 'reset' || ctx.trigger === 'clear'){
				this.setFilters()
			}
		},
		request(filters) {
			const timer = setTimeout(() => {
				clearTimeout(timer);
				this.openorderList = this.openorderList.filter((item) => {
					let result = true;
					if (isNumber(filters.status)) {
						result = item.status === filters.status;
					}
					if (result && filters['PO NO.']) {
						result = item['PO NO.'].toLowerCase().includes(filters['PO NO.'].toLowerCase());
					}
					return result;
				});
				this.pagination.total = this.openorderList.length
			}, 100);
		},
		// 筛选、分页、排序等功能发生变化时，均会触发 change 事件
		onChange(info, context) {
			console.log('change', info, context, '筛选、分页、排序等功能发生变化均会触发');
		},
		setFilters() {
			this.filterValue = {};
			this.openorderList = this.openorderListIntial;
			this.pagination.total = this.openorderList.length
		},		// 分页变化时触发该事件
		onPageChange(pageInfo, newData) {
			if (!this.pagination.defaultCurrent) {
				// 受控用法所需，即使用 pagination.current 和 pagination.pageSize 时，必须保留恢复下面 2 行代码
				this.pagination.current = pageInfo.current;
				this.pagination.pageSize = pageInfo.pageSize;
			}
			console.log('page-change:', pageInfo, newData);
		},
		oneEmailChange(val, ctx) {
			console.log(val, ctx);
		},
		rehandleExpandChange(value, params) {
			this.expandedRowKeys = value;
			console.log('rehandleExpandChange', value);
			console.log('rehandleExpandChange', params);
		},
	},
}
</script>