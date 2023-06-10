<template>
	<div class="selectModel" style="width: 100%">
		<el-select v-model="value" :disabled="disabled" placeholder="请选择企业" @change="change">
			<el-option :label="item['enterName']" :value="item['id']" v-for="item in tableData"></el-option>
		</el-select>
	</div>
</template>

<script lang="ts">
import { defineComponent, toRefs, reactive, watch, onMounted } from "vue";
import { getComponyList } from "@/utils/dict";
export default defineComponent({
	props: {
		defaultVal: {
			type: String,
		},
		disabled: {
			type: Boolean,
			default: false,
		},
	},
	emits: ["update:value", "updateCmptValue"],
	setup(props, ctx) {
		const state = reactive({
			value: "",
			tableData: [],
		});

		const handleUpdateValue = (value: string | any) => {
			ctx.emit("update:value", value);
			ctx.emit("updateCmptValue", value);
		};

		watch(
			() => props.defaultVal,
			(val: any) => {
				state.value = val;
			},
			{ immediate: true }
		);

		onMounted(() => {
			state.value = props.defaultVal as any;
			getTableData();
		});

		const change = (val: any) => {
			ctx.emit("update:value", val);
		};

		const getTableData = (keyWord?: any) => {
			getComponyList().then(({ data }) => {
				state.tableData = data.records;
			});
		};
		return {
			...toRefs(state),
			handleUpdateValue,
			getTableData,
			change,
		};
	},
});
</script>
<style lang="scss" scoped>
.selectModel {
	width: 100%;
	display: inline-block;
	position: relative;
}

.n-select {
	width: 100%;
}
</style>
