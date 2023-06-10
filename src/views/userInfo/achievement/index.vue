import request from "@/utils/request";

<template>
	<div class="achievement h100">
		<div class="search">
			<el-form :inline="true" :model="searchData">
				<el-form-item label="名称" field="username">
					<el-input style="width: 150px" v-model="searchData.username" type="text" placeholder="名称" />
				</el-form-item>
				<el-form-item label="账户" field="accountnumber">
					<el-input style="width: 150px" v-model="searchData.accountnumber" type="text" placeholder="账户" />
				</el-form-item>
				<el-form-item label="电话" field="phone">
					<el-input style="width: 150px" v-model="searchData.phone" type="text" placeholder="电话" />
				</el-form-item>
				<el-form-item label="驾校" field="torole">
					<el-select style="width: 150px" clearable v-model="searchData.torole" class="m-2" filterable>
						<el-option v-for="item in sysUser" :key="item.userId" :label="item.nickName" :value="item.userId" />
					</el-select>
				</el-form-item>
				<el-form-item title="">
					<vxe-button status="primary" content="搜索" @click="search"></vxe-button>
				</el-form-item>
			</el-form>
		</div>

		<div class="tableContent">
			<vxe-table :show-overflow="true" align="center" border :data="tableData" :show-header-overflow="true">
				<vxe-column type="seq" title="序号" width="80"></vxe-column>
				<vxe-column field="username" title="姓名"></vxe-column>
				<vxe-column field="phone" title="电话"></vxe-column>
				<vxe-column field="dcount" title="考试次数"></vxe-column>
				<vxe-column field="dright" title="正确数"></vxe-column>
				<vxe-column field="wrong" title="错题数"></vxe-column>
				<vxe-column field="result" title="考试结果">
					<template #default="{ row }">
						<n-button quaternary type="success" v-if="row.result == 1">
							合格
						</n-button>
						<n-button quaternary type="warning" v-if="row.result == 2">
							不合格
						</n-button>
						<n-button quaternary v-if="row.result == 3"> 未完成 </n-button>
					</template>
				</vxe-column>
				<vxe-column field="rightcount" title="合格次数"></vxe-column>

				<vxe-column field="score" title="分数"></vxe-column>
				<vxe-column field="accuracy" title="准确率">
					<template #default="{ row }"> {{ row.accuracy }}% </template>
				</vxe-column>
				<vxe-column field="dtime" title="考试时间"></vxe-column>
			</vxe-table>
		</div>

		<div class="bottom">
			<vxe-pager v-model:current-page="page.currentPage" v-model:page-size="page.pageSize" :total="page.totalResult"
				:layouts="[
					'PrevJump',
					'PrevPage',
					'Number',
					'NextPage',
					'NextJump',
					'Sizes',
					'FullJump',
					'Total',
				]" @page-change="pageChange">
			</vxe-pager>
		</div>
	</div>
</template>

<script lang="ts" setup>
import request from "@/utils/request";
import { onMounted, reactive, ref } from "vue";
import moment from "moment";

const tableData = ref([]);
const page = reactive({
	currentPage: 1,
	pageSize: 20,
	totalResult: 0,
});

const searchData = ref({
	username: "",
	accountnumber: "",
	phone: "",
	torole: "",
});

onMounted(() => {
	getData();
	getSysUser();
});

//获取系统用户
const sysUser: any = ref([]);
const getSysUser = () => {
	request({
		url: "/system/user/list",
		method: "get",
		params: {
			pageNum: 1,
			pageSize: 50,
		},
	}).then(({ rows }) => {
		sysUser.value = rows;
	});
};

const pageChange = (data) => {
	page.pageSize = data.pageSize;
	page.currentPage = data.currentPage;
	getData();
};

const search = () => {
	page.currentPage = 1;
	getData();
};

const getData = () => {
	request({
		url: "questioninfo/list",
		data: {
			...searchData.value,
			empower: 0, //是否授权
			pageSize: page.pageSize,
			pageNum: page.currentPage,
		},
		method: "post",
	}).then(({ data }) => {
		data.records.map((item) => {
			item.dtime = moment(item.dtime).format("YYYY-MM-DD HH:mm:ss");
		});
		tableData.value = data.records;
		page.totalResult = data.total;
	});
};
</script>

<style scoped less>
.achievement {
	padding: 10px;
	box-sizing: border-box;
	display: flex;
	flex-direction: column;
	justify-content: space-between;
}

.tableContent {
	flex: 1;
}
</style>
