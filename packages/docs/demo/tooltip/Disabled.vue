<script lang="ts" setup>
	import { ref } from 'vue'

	const disabled = ref(false)
</script>

<template>
	<vr-tooltip
		:disabled="disabled"
		content="click to close tooltip function"
		placement="bottom">
		<vr-button @click="disabled = !disabled"
			>click to {{ disabled ? 'active' : 'close' }} tooltip function</vr-button
		>
	</vr-tooltip>
</template>
