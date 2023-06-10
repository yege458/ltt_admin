<template>
	<i-frame v-model:src="url"></i-frame>
</template>

<script setup>
import iFrame from "@/components/iFrame";

const url = ref("http://124.222.190.50:8080/swagger-ui/index.html");
</script>
