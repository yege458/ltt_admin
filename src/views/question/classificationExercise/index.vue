import request from "@/utils/request";

<template>
  <div class="achievement h100">
    <div class="search">
      <el-form :inline="true" :model="searchData">
        <el-form-item label="题目类型">
          <el-select style="width: 150px" v-model="searchData.type" class="m-2" placeholder="题目类型" clearable>
            <el-option v-for="(key, val) in classTypeList" :key="val" :label="key" :value="val" />
          </el-select>
        </el-form-item>
        <el-form-item title="">
          <el-space :size="10">
            <div>
              <el-input v-model="searchData.lid" placeholder="输入题目/LID"></el-input>
            </div>
            <div>
              <vxe-button status="primary" content="查询" @click="search"></vxe-button>
            </div>
          </el-space>
        </el-form-item>

        <el-form-item title="">
          <el-space :size="10">
            <div>
              <el-upload class="upload-demo" accept=".xlsx, .xls" action="http://111.230.93.133:8083/lid/classLid/import"
                :headers="headers" :show-file-list="false" @change="uploadError" @progress="uploadProgress"
                :on-success="uploadSuccess">
                <el-button type="primary">导入题库</el-button>
              </el-upload>
            </div>
            <div>
              <el-button type="primary" @click="add">新增</el-button>
            </div>
            <!-- <div>
              <el-button type="primary" @click="exportData">导出</el-button>
            </div> -->
            <div>
              <el-button type="danger" @click="delDataByIds">批量删除</el-button>
            </div>
          </el-space>
        </el-form-item>
      </el-form>
    </div>
    <div class="tableContent">
      <vxe-table :show-overflow="true" align="center" ref="xTable" :data="tableData" class="mytable-style" border
        :show-header-overflow="true">
        <vxe-column type="checkbox" width="60"></vxe-column>
        <!-- <vxe-column field="id" title="id"></vxe-column> -->
        <vxe-column field="lid" title="LID"></vxe-column>
        <vxe-column field="city" title="城市"> </vxe-column>
        <!-- <vxe-column field="type" title="题目类型">
          <template #default="{ row }">
            {{ selectedTypeList[row.type] }}
          </template>
        </vxe-column> -->
        <vxe-column field="remark" title="备注"> </vxe-column>
        <vxe-column title="操作" width="200">
          <template #default="{ row }">
            <vxe-button status="primary" size="mini" content="修改" @click="editFn(row)"></vxe-button>
            <vxe-button status="danger" size="mini" content="删除" @click="delItem(row)"></vxe-button>
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
  <el-dialog :title="title" v-model="open" width="600px" append-to-body>
    <el-form :model="form" ref="userRef" label-width="80px">
      <!-- <el-row>
        <el-col :span="24">
          <el-form-item label="id主键" prop="id">
            <el-input v-model="form.id" />
          </el-form-item>
        </el-col>
      </el-row> -->
      <el-row>
        <el-col :span="12">
          <el-form-item label="lid主键" prop="lid">
            <el-input v-model="form.lid" placeholder="请输入lid主键" />
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="城市" prop="city">
            <el-input v-model="form.city" placeholder="请输入城市" />
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="12">
          <el-form-item label="题目类型" prop="type">
            <el-select v-model="form.type" placeholder="题目类型" clearable>
              <el-option v-for="(item, val) in classTypeList" :key="val" :label="item" :value="val" />
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="备注" prop="remark">
            <el-input v-model="form.remark" placeholder="请输入备注" />
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

<script setup>
import request from "@/utils/request";
import { onMounted, reactive, ref } from "vue";
import { VXETable } from "vxe-table";
import { ElMessage } from "element-plus";
import { decrypt2, encrypt2 } from "@/utils/jsencrypt";
import { saveAs } from "file-saver";
import { getToken } from "@/utils/auth";
const { proxy } = getCurrentInstance();

const open = ref(false);
const title = ref("");
const form = reactive({
  id: "",
  lid: "",
  city: "",
  remark: "",
  type: "",
});

const tableData = ref([]);
const searchData = ref({
  type: "", //题目类型 1选择题 2多选题 3判断题
  lid: "",
  city: "",
});

const headers = {
  Authorization: getToken(),
};

const subType = ref({
  1: "选择题",
  2: "多选题",
  3: "判断题",
});

const classTypeList = ref({
  1: "扣分罚款题",
  2: "仪表、指示灯题",
  3: "特殊天气题",
  4: "速度题",
  5: "标志题",
  6: "交警手势题",
  7: "交通信号灯题",
  8: "高速公路题",
  9: "安全常识题",
  10: "道路标线题",
  11: "灯光使用题",
  12: "通行原则题",
  13: "文明驾驶题",
  14: "证件相关题",
  15: "紧急救援题",
  16: "事故处理题",
  17: "易混指示灯题",
  18: "责任判定题",
  19: "道路相关题",
  20: "紧急避险题",
  21: "故障类题",
  22: "扣留相关题",
  23: "操作装置题",
  24: "处罚时间题",
  25: "记分周期题",
  26: "登记相关题",
  27: "车辆功能英文缩写",
});

const page = reactive({
  currentPage: 1,
  pageSize: 20,
  totalResult: 0,
});

onMounted(() => {
  getData();
});

const search = () => {
  page.currentPage = 1;
  getData();
};

const getData = () => {
  let params = {
    lid: searchData.value.keyword,
    city: searchData.value.city,
    type: searchData.value.type,
    pageNum: page.currentPage,
    pageSize: page.pageSize,
  };
  request({
    url: "lid/classLid/List",
    data: params,
    method: "post",
  }).then(({ data }) => {
    // data.records.map((item) => {
    //   item.result = decrypt2(item.rightreslut);
    //   item.rightreslut = decrypt2(item.rightreslut);
    //   return item;
    // });
    console.log(data)
    tableData.value = data.records;
    page.totalResult = data.total;
  });
};

const delItem = async (item) => {
  const type = await VXETable.modal.confirm("您确定要删除吗？");
  const params = {
    ids: [item.id],
  };
  if (type == "confirm") {
    delList(params);
  }
};

const submitForm = () => {
  if (title.value === "新增定向练习") {
    request({
      url: "lid/classLid/add",
      method: "post",
      data: form.value,
    }).then(({ data }) => {
      ElMessage({
        message: "新增成功！",
        type: "success",
      });
      open.value = false;
      getData();
    });
  } else {
    request({
      url: "lid/classLid/update",
      method: "post",
      data: form.value,
    }).then(({ data }) => {
      ElMessage({
        message: "修改成功！",
        type: "success",
      });
      open.value = false;
      getData();
    });
  }
};

const pageChange = (data) => {
  page.pageSize = data.pageSize;
  page.currentPage = data.currentPage;
  getData();
};
const editFn = (row) => {
  form.value = { ...row };
  title.value = "修改定向练习";
  open.value = true;
};

// 新增
const add = () => {
  title.value = "新增定向练习";
  open.value = true;
};

//导出
const exportData = () => {
  proxy.$modal.loading("正在导出数据！");

  request({
    url: "/export",
    method: "get",
    headers: {
      "Content-Type": "application/x-www-form-urlencoded",
    },
    responseType: "blob",
  }).then((res) => {
    proxy.$modal.closeLoading();
    const blob = new Blob([res]);
    saveAs(blob, "题库.xlsx");
  });
};

//批量删除
const xTable = ref();
const delDataByIds = () => {
  const $table = xTable.value;
  const selectRecords = $table.getCheckboxRecords();

  if (!selectRecords.length) {
    ElMessage({
      message: "请选择数据！",
      type: "error",
    });
    return;
  }
  const idsStr = [...selectRecords].map((item) => item.id);
  const params = {
    ids: idsStr,
  };
  delList(params);
};

const delList = (params) => {
  request({
    url: "lid/classLid/del",
    method: "post",
    data: params,
  }).then(({ data }) => {
    VXETable.modal.message({
      content: "删除成功！",
      status: "success",
    });
    getData();
  });
};

//上传之前
const uploadProgress = () => {
  proxy.$modal.loading("正在上传数据！");
};

//上传成功
const uploadSuccess = () => {
  proxy.$modal.closeLoading();
  getData();
};
//导入
const uploadError = (err) => {
  if (err.response && err.response.code == 500) {
    ElMessage.error(err.response.msg);
  }
};

//模板下载
const downFile = () => {
  request({
    url: "/file/download?fileName=题库更新模板.xlsx",
    method: "get",
    headers: {
      ContentType: "application/x-www-form-urlencoded",
      responseType: "blob",
    },
  }).then((res) => {
    const blob = new Blob([res.data]);
    saveAs(blob, decodeURI("题库模板.xlsx"));
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
