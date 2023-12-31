<template>
	<div class="table-demo-container layout-padding">
		<div class="table-demo-padding layout-padding-view layout-padding-auto">
			<TableSearch v-if="state.tableData.search[0].options.length > 0" :search="state.tableData.search" @search="onSearch" />
			<Table
				ref="tableRef"
				v-bind="state.tableData"
				class="table-demo"
				@delRow="onTableDelRow"
				@pageChange="onTablePageChange"
				@sortHeader="onSortHeader"
				@importTable="onImportTableData"
			/>
		</div>
	</div>
</template>

<script setup lang="ts" name="basicsBasic">
import { defineAsyncComponent, reactive, ref, onMounted } from 'vue';
import { ElMessage } from 'element-plus';
import { getParentIdListApi, getBaseDaListApi, getBaseDownloadApi } from '/@/api/basics/basic.ts';
import { useI18n } from 'vue-i18n';
// 引入导出Excel表格依赖
import * as FileSaver from 'file-saver';
import * as XLSX from 'xlsx';
// 引入组件
const Table = defineAsyncComponent(() => import('/@/components/table/index.vue'));
const TableSearch = defineAsyncComponent(() => import('/@/components/search/search.vue'));
// 定义变量内容
const { t } = useI18n();
const tableRef = ref<RefType>();
const state = reactive<TableDemoState>({
	tableData: {
		// 列表数据（必传）
		data: [],
		// 表头内容（必传，注意格式）
		header: [
			{ key: 'dataname', colWidth: '', title: 'message.pages.name1', type: 'text', isCheck: true },
			{ key: 'datacode', colWidth: '', title: 'message.pages.code', type: 'text', isCheck: true },
			{ key: 'type', colWidth: '', title: 'message.pages.groupType', type: 'text', isCheck: true },
			{ key: 'runstatus', colWidth: '', title: 'message.pages.state', type: 'status', isCheck: true },
			{ key: 'creator', colWidth: '', title: 'message.pages.creator', type: 'text', isCheck: true },
			{ key: 'createtime', title: 'message.pages.creationTime', type: 'text', isCheck: true },
		],
		// 配置项（必传）
		config: {
			total: 0, // 列表总数
			loading: true, // loading 加载
			isBorder: false, // 是否显示表格边框
			isSerialNo: true, // 是否显示表格序号
			isSelection: true, // 是否显示表格多选
			isOperate: true, // 是否显示表格操作栏
			isEditBtn: false, //是否显示修改按钮
		},
		// 搜索表单，动态生成（传空数组时，将不显示搜索，注意格式）
		search: [
			{
				label: 'message.pages.parentNode',
				prop: 'parentid',
				placeholder: 'message.pages.placeParentNode',
				required: false,
				type: 'select',
				options: [],
			},
			{ label: 'message.pages.name1', prop: 'dataName', placeholder: 'message.pages.placeName1', required: false, type: 'input' },
			{ label: 'message.pages.code', prop: 'dataCode', placeholder: 'message.pages.placeCode', required: false, type: 'input' },
		],
		// 给后端的数据
		form: {
			parentid: '',
			dataName: '',
			dataCode: '',
		},
		// 搜索参数（不用传，用于分页、搜索时传给后台的值，`getTableData` 中使用）
		page: {
			pageNum: 1,
			pageSize: 10,
		},
		// 打印标题
		printName: '表格打印演示',
		// 弹窗表单
		dialogConfig: [{ label: '父节点', prop: 'workno', placeholder: '请输入父节点', required: true, type: 'select' }],
	},
});

// 初始化列表数据
const getTableData = async () => {
	state.tableData.config.loading = true;
	const form = state.tableData.form;
	let data = {
		parentid: form.parentid,
		dataName: form.dataName,
		dataCode: form.dataCode,
		page: state.tableData.page,
	};
	const res = await getBaseDaListApi(data);
	state.tableData.data = res.data;
	state.tableData.config.total = state.tableData.data.length;
	setTimeout(() => {
		state.tableData.config.loading = false;
	}, 500);
};
// 下拉框数据
const getSelect = async () => {
	const res = await getParentIdListApi();
	state.tableData.search[0].options = res.pars;
};
// 搜索点击时表单回调
const onSearch = (data: EmptyObjectType) => {
	state.tableData.form = Object.assign({}, state.tableData.form, { ...data });
	tableRef.value.pageReset();
};
// 删除当前项回调
const onTableDelRow = (row: EmptyObjectType) => {
	ElMessage.success(`${t('message.allButton.deleteBtn')}${row.dataname}${t('message.hint.success')}`);
	getTableData();
};
// 分页改变时回调
const onTablePageChange = (page: TableDemoPageType) => {
	state.tableData.page.pageNum = page.pageNum;
	state.tableData.page.pageSize = page.pageSize;
	getTableData();
};
// 拖动显示列排序回调
const onSortHeader = (data: TableHeaderType[]) => {
	state.tableData.header = data;
};
// 导出
const onImportTableData = async () => {
	const res = await getBaseDownloadApi();
	let blob = new Blob([res], {
		// 这里一定要和后端对应，不然可能出现乱码或者打不开文件
		type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
	});
	if (window.navigator.msSaveOrOpenBlob) {
		navigator.msSaveBlob(blob, fileName);
	} else {
		const link = document.createElement('a');
		link.href = window.URL.createObjectURL(blob);
		link.download = `${t('message.router.basicsBasic')} ${new Date().toLocaleString()}.xls`; // 在前端也可以设置文件名字
		link.click();
		//释放内存
		window.URL.revokeObjectURL(link.href);
	}
};
// 页面加载时
onMounted(() => {
	getSelect();
});
</script>

<style scoped lang="scss">
.table-demo-container {
	.table-demo-padding {
		padding: 15px;
		.table-demo {
			flex: 1;
			overflow: hidden;
		}
	}
}
</style>
