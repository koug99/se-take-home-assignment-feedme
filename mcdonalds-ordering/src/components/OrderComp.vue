<template>
	<div class="p-1 bg-white border border-red-400 rounded-md flex flex-col gap-1 h-fit" :class="{ 'grow-anim': props.order.status == 'cooking'}">
    <span class="font-semibold text-lg">Order #{{ props.order.ID }} <span v-if="props.order.type == 'vip'" class="font-semibold text-orange-400">VIP</span></span>
    <span v-if="props.order.status != 'completed'" class="capitalize">Status: {{ props.order.status }}</span>
    <span v-else class="capitalize">Prepared By: Bot #{{ props.order.botID }}</span>
  </div>
</template>

<script setup>
const props = defineProps({
  orderID: Number,
  orderType: String,
  orderStatus: String,
  order: Object,
})
</script>

<style lang="css" scoped>
@keyframes grow {
  0% {
    transform: scale(1)
  }
  50% {
    transform: scale(1.05)
  }
  100% {
    transform: scale(1)
  }
}

.grow-anim {
  animation: grow 1.5s infinite;
}
</style>
