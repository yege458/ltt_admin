<template>
	<router-view />
</template>

<script setup>
import useSettingsStore from "@/store/modules/settings";
import { handleThemeStyle } from "@/utils/theme";
import { getmark } from "./utils/watermark";
import { getUserProfile } from "@/api/system/user";

const { watermark } = getmark();
onMounted(() => {
	getUserProfile().then((response) => {
		watermark(response.data.userName);
	});

	nextTick(() => {
		// 初始化主题样式
		handleThemeStyle(useSettingsStore().theme);
	});
});
</script>
