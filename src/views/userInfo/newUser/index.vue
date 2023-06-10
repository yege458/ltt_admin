import request from "@/utils/request";

<template>
	<div class="achievement h100">
		<div class="search">
			<el-form :inline="true" :model="searchData">
				<el-form-item label="名称" field="username">
					<el-input style="width: 150px" v-model="searchData.username" type="text" placeholder="名称" />
				</el-form-item>
				<el-form-item label="电话" field="phone">
					<el-input style="width: 150px" v-model="searchData.phone" type="text" placeholder="电话" />
				</el-form-item>

				<el-form-item label="拿证情况" field="phone">
					<el-select style="width: 200px" clearable v-model="searchData.ispass" class="m-2" filterable>
						<el-option label="未拿证" :value="0" />
						<el-option label="已拿证" :value="1" />
					</el-select>
				</el-form-item>

				<!-- <el-form-item label="驾校" field="torole">
					<el-select
						style="width: 200px"
						clearable
						v-model="searchData.torole"
						class="m-2"
						filterable
					>
						<el-option
							v-for="item in sysUser"
							:key="item.userId"
							:label="item.nickName"
							:value="item.userId"
						/>
					</el-select>
				</el-form-item> -->
				<el-form-item label="角色" field="role">
					<el-select style="width: 150px" clearable v-model="searchData.role" class="m-2" filterable>
						<el-option v-for="item in rolesData" :key="item.roleId" :label="item.roleName"
							:value="item.roleId" />
					</el-select>
				</el-form-item>

				<el-form-item title="">
					<vxe-button status="primary" content="搜索" @click="search"></vxe-button>
				</el-form-item>
				<el-form-item title=""> </el-form-item>
			</el-form>
		</div>
		<div class="tableContent">
			<vxe-table :show-overflow="true" align="center" :data="tableData" class="mytable-style" border
				:show-header-overflow="true">
				<vxe-column type="seq" title="序号" width="80"></vxe-column>
				<vxe-column field="accountnumber" title="账号"></vxe-column>
				<vxe-column field="username" title="姓名"></vxe-column>
				<vxe-column field="phone" title="电话"></vxe-column>
				<vxe-column field="nickName" title="角色名称"></vxe-column>
				<vxe-column field="ispass" title="拿证情况">
					<template #default="{ row }">
						<template v-if="row.ispass == 1">
							<vxe-button type="text" status="success" content="已拿证"></vxe-button>
						</template>
						<template v-if="row.ispass == 0">
							<vxe-button type="text" status="info" content="未拿证"></vxe-button>
						</template>
					</template>
				</vxe-column>
				<vxe-column field="remark" title="备注"></vxe-column>
				<vxe-column title="操作" width="250">
					<template #default="{ row }">
						<vxe-button status="primary" size="mini" content="修改" @click="editFn(row)"></vxe-button>

						<vxe-button status="primary" size="mini" content="授权" @click="addRol(row)"></vxe-button>

						<vxe-button status="danger" size="mini" content="删除" @click="delRow(row)"></vxe-button>
					</template>
				</vxe-column>
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

	<!-- 添加或修改用户配置对话框 -->
	<el-dialog title="用户授权" v-model="openYS" width="500px" append-to-body>
		<el-form :model="promissonData" :rules="rules" ref="userRef" label-width="80px">
			<el-row>
				<el-col :span="24">
					<el-form-item label="姓名" prop="username">
						<el-input style="width: 150px" v-model="promissonData.username" disabled />
					</el-form-item>
				</el-col>
			</el-row>

			<el-row>
				<el-col :span="24">
					<el-form-item label="驾校" prop="torole">
						<el-select v-model="promissonData.torole" class="m-2" style="width: 150px" filterable>
							<el-option v-for="item in sysUser" :key="item.userId" :label="item.nickName"
								:value="item.userId" />
						</el-select>
					</el-form-item>
				</el-col>
			</el-row>

			<el-row>
				<el-col :span="24">
					<el-form-item label="角色" prop="role">
						<el-select v-model="promissonData.role" class="m-2" style="width: 150px" filterable>
							<el-option v-for="item in rolesData" :key="item.roleId" :label="item.roleName"
								:value="item.roleId" />
						</el-select>
					</el-form-item>
				</el-col>
			</el-row>
		</el-form>
		<template #footer>
			<div class="dialog-footer">
				<el-button type="primary" @click="submitPromisson">确 定</el-button>
				<el-button @click="openYS = false">取 消</el-button>
			</div>
		</template>
	</el-dialog>

	<!-- 添加或修改用户配置对话框 -->
	<el-dialog title="用户修改" v-model="open" width="600px" append-to-body>
		<el-form :model="form" :rules="rules" ref="userRef" label-width="80px">
			<el-row>
				<el-col :span="12">
					<el-form-item label="账号" prop="accountnumber">
						<el-input v-model="form.accountnumber" placeholder="请输入账号" maxlength="30" />
					</el-form-item>
				</el-col>
			</el-row>
			<el-row>
				<el-col :span="12">
					<el-form-item label="手机号码" prop="phone">
						<el-input v-model="form.phone" placeholder="请输入手机号码" maxlength="11" />
					</el-form-item>
				</el-col>
				<el-col :span="12">
					<el-form-item label="姓名" prop="username">
						<el-input v-model="form.username" placeholder="姓名" maxlength="50" />
					</el-form-item>
				</el-col>
			</el-row>

			<el-row>
				<el-col :span="12">
					<el-form-item label="拿证状态" prop="ispass">
						<el-select v-model="searchData.ispass" class="m-2">
							<el-option label="未拿证" :value="0" />
							<el-option label="已拿证" :value="1" />
						</el-select>
					</el-form-item>
				</el-col>
			</el-row>

			<el-row>
				<el-col :span="24">
					<el-form-item label="备注">
						<el-input v-model="form.remark" type="textarea" placeholder="请输入内容"></el-input>
					</el-form-item>
				</el-col>
			</el-row>
		</el-form>
		<template #footer>
			<div class="dialog-footer">
				<el-button type="primary" @click="submitForm">确 定</el-button>
				<el-button @click="open = false">取 消</el-button>
			</div>
		</template>
	</el-dialog>
</template>

<script lang="ts" setup>
import request from "@/utils/request";
import { onMounted, reactive, ref } from "vue";
import { ElMessage } from "element-plus";
import { VXETable } from "vxe-table";
const tableData = ref([]);
const searchData = ref({
	username: "",
	phone: "",
	ispass: "",
	torole: "",
	role: "",
});
const form = ref({
	accountnumber: "",
	phone: "",
	username: "",
	password: "",
	remark: "",
	ispass: 0,
});
const promissonData = ref({
	id: "",
	role: "",
	torole: "",
	username: "",
});
const open = ref(false);
const openYS = ref(false);
const page = reactive({
	currentPage: 1,
	pageSize: 20,
	totalResult: 0,
});

const rules = ref({
	userName: [
		{ required: true, message: "账号不能为空", trigger: "blur" },
		{
			min: 2,
			max: 20,
			message: "账号长度必须介于 2 和 20 之间",
			trigger: "blur",
		},
	],
	phonenumber: [{ required: true, message: "手机号不能为空", trigger: "blur" }],
});

onMounted(() => {
	getData();
	listRole();
	getSysUser();
});

//提交修改
const submitForm = () => {
	request({
		url: "/cuser/update",
		data: form.value,
		method: "post",
	}).then(({ data }) => {
		ElMessage({
			message: "修改成功！",
			type: "success",
		});
		open.value = false;
		getData();
	});
};

const search = () => {
	page.currentPage = 1;
	getData();
};

const getData = () => {
	let params = {
		pageNum: page.currentPage,
		pageSize: page.pageSize,
		...searchData.value,
	};

	request({
		url: "/cuser/list",
		data: params,
		method: "post",
	}).then(({ data }) => {
		tableData.value = data.records;
		page.totalResult = data.total;
	});
};

//删除
const delRow = async (row: any) => {
	const type = await VXETable.modal.confirm("您确定要删除吗？");
	if (type == "confirm") {
		request({
			url: "/cuser/delete",
			data: row,
			method: "post",
		}).then(({ data }) => {
			ElMessage({
				message: "修改成功！",
				type: "success",
			});
		});
		getData();
	}
};

//添加授权
const submitPromisson = () => {
	request({
		url: "/cuser/empower",
		method: "post",
		data: {
			id: promissonData.value.id,
			role: promissonData.value.role,
			torole: promissonData.value.torole,
		},
	}).then((res) => {
		ElMessage({
			message: "提交成功！",
			type: "success",
		});
		Object.keys(promissonData.value).forEach((key) => {
			promissonData.value[key] = "";
		});
		openYS.value = false;
		getData();
	});
};

const editFn = (row: any) => {
	form.value = { ...row };
	open.value = true;
};

const pageChange = (data) => {
	page.pageSize = data.pageSize;
	page.currentPage = data.currentPage;
	getData();
};

const addRol = (row) => {
	openYS.value = true;
	promissonData.value.id = row.id;
	promissonData.value.username = row.username;
};
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
//获取角色
const rolesData: any = ref([]);
const listRole = () => {
	return request({
		url: "/system/role/list",
		method: "get",
		params: {
			pageNum: 1,
			pageSize: 50,
		},
	}).then(({ rows }) => {
		rolesData.value = rows;
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
