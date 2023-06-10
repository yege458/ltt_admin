<template>
	<div class="app-container">
		<div class="search">
			<el-form
				:model="queryParams"
				ref="queryRef"
				:inline="true"
				v-show="showSearch"
				label-width="68px"
			>
				<el-form-item label="操作人员" prop="username">
					<el-input
						v-model="queryParams.username"
						placeholder="请输入操作人员"
						clearable
						style="width: 240px"
					/>
				</el-form-item>

				<el-form-item label="操作时间" style="width: 308px">
					<el-date-picker
						v-model="dateRange"
						value-format="YYYY-MM-DD"
						type="daterange"
						range-separator="-"
						start-placeholder="开始日期"
						end-placeholder="结束日期"
					></el-date-picker>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" icon="Search" @click="handleQuery"
						>搜索</el-button
					>
					<el-button icon="Refresh" @click="resetQuery">重置</el-button>
				</el-form-item>
			</el-form>
		</div>

		<div class="content">
			<div class="tableContent">
				<el-table
					ref="operlogRef"
					v-loading="loading"
					:data="operlogList"
					@selection-change="handleSelectionChange"
					:default-sort="defaultSort"
					@sort-change="handleSortChange"
				>
					<el-table-column type="seq" width="55" align="center" title="序号" />
					<el-table-column
						label="操作人员"
						align="center"
						prop="username"
						:show-overflow-tooltip="true"
						sortable="custom"
						width="100"
					/>
					<el-table-column
						label="主机"
						align="center"
						prop="ip"
						width="130"
						:show-overflow-tooltip="true"
					/>
					<el-table-column
						label="请求参数"
						align="center"
						prop="params"
						width="600"
					/>
					<el-table-column
						label="操作内容"
						align="center"
						prop="operation"
						show-overflow-tooltip
					/>
					<el-table-column label="操作时间" align="center" prop="createDate" />
					<!-- <el-table-column
				label="操作"
				align="center"
				class-name="small-padding fixed-width"
			>
				<template #default="scope">
					<el-button
						type="text"
						icon="View"
						@click="handleView(scope.row, scope.index)"
						v-hasPermi="['monitor:operlog:query']"
						>详细</el-button
					>
				</template>
			</el-table-column> -->
				</el-table>
			</div>

			<pagination
				:total="total"
				v-model:page="queryParams.pageNum"
				v-model:limit="queryParams.pageSize"
				@pagination="getList"
			/>
		</div>

		<!-- 操作日志详细 -->
		<el-dialog title="操作日志详细" v-model="open" width="700px" append-to-body>
			<el-form :model="form" label-width="100px">
				<el-row>
					<el-col :span="12">
						<el-form-item label="操作模块："
							>{{ form.title }} / {{ typeFormat(form) }}</el-form-item
						>
						<el-form-item label="登录信息："
							>{{ form.operName }} / {{ form.operIp }} /
							{{ form.operLocation }}</el-form-item
						>
					</el-col>
					<el-col :span="12">
						<el-form-item label="请求地址：">{{ form.operUrl }}</el-form-item>
						<el-form-item label="请求方式：">{{
							form.requestMethod
						}}</el-form-item>
					</el-col>
					<el-col :span="24">
						<el-form-item label="操作方法：">{{ form.method }}</el-form-item>
					</el-col>
					<el-col :span="24">
						<el-form-item label="请求参数：">{{ form.operParam }}</el-form-item>
					</el-col>
					<el-col :span="24">
						<el-form-item label="返回参数：">{{
							form.jsonResult
						}}</el-form-item>
					</el-col>
					<el-col :span="12">
						<el-form-item label="操作状态：">
							<div v-if="form.status === 0">正常</div>
							<div v-else-if="form.status === 1">失败</div>
						</el-form-item>
					</el-col>
					<el-col :span="12">
						<el-form-item label="操作时间：">{{
							parseTime(form.operTime)
						}}</el-form-item>
					</el-col>
					<el-col :span="24">
						<el-form-item label="异常信息：" v-if="form.status === 1">{{
							form.errorMsg
						}}</el-form-item>
					</el-col>
				</el-row>
			</el-form>
			<template #footer>
				<div class="dialog-footer">
					<el-button @click="open = false">关 闭</el-button>
				</div>
			</template>
		</el-dialog>
	</div>
</template>

<script setup name="Operlog">
import { list, delOperlog, cleanOperlog } from "@/api/monitor/operlog";
import request from "@/utils/request";
import moment from "moment";

const { proxy } = getCurrentInstance();
const { sys_oper_type, sys_common_status } = proxy.useDict(
	"sys_oper_type",
	"sys_common_status"
);

const operlogList = ref([]);
const open = ref(false);
const loading = ref(true);
const showSearch = ref(true);
const ids = ref([]);
const single = ref(true);
const multiple = ref(true);
const total = ref(0);
const title = ref("");
const dateRange = ref([]);
const defaultSort = ref({ prop: "operTime", order: "descending" });

const data = reactive({
	form: {},
	queryParams: {
		pageNum: 1,
		pageSize: 10,
		title: undefined,
		username: undefined,
		businessType: undefined,
		status: undefined,
	},
});

const { queryParams, form } = toRefs(data);

/** 查询登录日志 */
function getList() {
	loading.value = true;
	request
		.post("/log/list", {
			pageNum: queryParams.value.pageNum,
			pageSize: queryParams.value.pageSize,
			startTime: moment(dateRange.value[0]).format("YYYY-MM-DD HH:mm:ss"),
			endTime: moment(dateRange.value[1]).format("YYYY-MM-DD HH:mm:ss"),
			username: queryParams.value.username,
		})
		.then(({ data }) => {
			data.map((item) => {
				item.createDate = moment(item.createDate).format("YYYY-MM-DD HH:mm:ss");
			});
			operlogList.value = data;
			// total.value = data.total;
			loading.value = false;
		});
	//   list(proxy.addDateRange(queryParams.value, dateRange.value)).then(response => {
	//     operlogList.value = response.rows;
	//     total.value = response.total;
	//     loading.value = false;
	//   });
}
/** 操作日志类型字典翻译 */
function typeFormat(row, column) {
	return proxy.selectDictLabel(sys_oper_type.value, row.businessType);
}
/** 搜索按钮操作 */
function handleQuery() {
	queryParams.value.pageNum = 1;
	getList();
}
/** 重置按钮操作 */
function resetQuery() {
	dateRange.value = [];
	proxy.resetForm("queryRef");
	queryParams.value.pageNum = 1;
	proxy.$refs["operlogRef"].sort(
		defaultSort.value.prop,
		defaultSort.value.order
	);
}
/** 多选框选中数据 */
function handleSelectionChange(selection) {
	ids.value = selection.map((item) => item.operId);
	multiple.value = !selection.length;
}
/** 排序触发事件 */
function handleSortChange(column, prop, order) {
	queryParams.value.orderByColumn = column.prop;
	queryParams.value.isAsc = column.order;
	getList();
}
/** 详细按钮操作 */
function handleView(row) {
	open.value = true;
	form.value = row;
}
/** 删除按钮操作 */
function handleDelete(row) {
	const operIds = row.operId || ids.value;
	proxy.$modal
		.confirm('是否确认删除日志编号为"' + operIds + '"的数据项?')
		.then(function () {
			return delOperlog(operIds);
		})
		.then(() => {
			getList();
			proxy.$modal.msgSuccess("删除成功");
		})
		.catch(() => {});
}
/** 清空按钮操作 */
function handleClean() {
	proxy.$modal
		.confirm("是否确认清空所有操作日志数据项?")
		.then(function () {
			return cleanOperlog();
		})
		.then(() => {
			getList();
			proxy.$modal.msgSuccess("清空成功");
		})
		.catch(() => {});
}
/** 导出按钮操作 */
function handleExport() {
	proxy.download(
		"monitor/operlog/export",
		{
			...queryParams.value,
		},
		`config_${new Date().getTime()}.xlsx`
	);
}

getList();
</script>
<style lang="scss" scoped>
.app-container {
	display: flex;
	flex-direction: column;
	height: 100%;
	overflow: hidden;
	box-sizing: border-box;
}
.content {
	height: calc(100% - 30px);
	display: flex;
	flex-direction: column;
	.tableContent {
		height: calc(100% - 40px);
	}
	.pagination-container {
		padding: 0 !important;
		margin-top: 20px;
	}
}
</style>
