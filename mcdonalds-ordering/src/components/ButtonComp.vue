<template>
	<button @click="$emit('clicked')" class="p-2 border border-orange-400 rounded-md hover:bg-orange-200 cursor-pointer bg-white">
		<slot></slot>
	</button>
</template>

<script setup>

</script>

<style lang="scss" scoped></style>