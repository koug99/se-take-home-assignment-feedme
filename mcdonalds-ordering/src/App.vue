<template>
  <h1 class="text-4xl">McDonald's Ordering System</h1>
  <div class="flex gap-2 m-4 flex-wrap">
    <div class="flex  flex-col gap-2">
      <ButtonComp @clicked="newOrder">
        New Normal Order
      </ButtonComp>
      <ButtonComp @clicked="newOrder('vip')">
        New VIP Order
      </ButtonComp>
      <ButtonComp @clicked="addBot">
        + Bot
      </ButtonComp>
      <ButtonComp @clicked="removeBot">
        - Bot
      </ButtonComp>
    </div>
    <div class="grid grid-cols-2 gap-2 flex-1 h-[90vh] overflow-auto">
      <div class="flex flex-col gap-1 h-[80%]">
        <h2 class="font-bold text-xl">Pending Orders</h2>
        <div class="bg-blue-100 border border-blue-400 flex-1 flex gap-2 flex-wrap p-2 overflow-auto">
          <OrderComp
            v-for="order of pendingOrders"
            :key="order.ID"
            :order="order"
          />
        </div>
      </div>
      <div class="flex flex-col gap-1 h-[80%]">
        <h2 class="font-bold text-xl">Completed Orders</h2>
        <div class="bg-green-100 border border-green-400 flex-1 flex gap-2 flex-wrap p-2 overflow-auto">
          <OrderComp
            v-for="order of completedOrders"
            :key="order.ID"
            :order="order"
          />
        </div>
      </div>
      <div class="flex flex-col gap-1 col-span-2">
        <h2 class="font-bold text-xl">Cooking Bots</h2>
        <div class="bg-orange-100 border border-orange-400 flex-1 flex gap-2 flex-wrap p-2">
          <div v-for="bot of bots" :key="bot.ID" class="p-1 bg-white border border-red-400 rounded-md flex flex-col gap-1 h-fit">
            <span class="font-semibold text-lg">Bot #{{ bot.ID }}</span>
            <span class="capitalize">Status: {{ bot.status }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import ButtonComp from './components/ButtonComp.vue'
import OrderComp from './components/OrderComp.vue'

// 10 SECONDS
const COOK_TIME = 1000 * 10

const pendingOrders = reactive([]);
const completedOrders = reactive([]);
const orderCounter = ref(0);
const botCounter = ref(0);

const bots = reactive([]);

const newOrder = (type = 'normal') => {
  orderCounter.value++;
  const orderObj = {
    ID: orderCounter.value,
    type,
    status: 'pending',
    botID: null,
    timeoutID: null,
    createdAt: new Date(),
  }
  if(type == 'vip') {
    // Inserting after first normal order will ensure it is behind other VIP orders
    const firstNormal = pendingOrders.findIndex((x) => x.type == 'normal');
    // Clear the processing order if there is one
    if(firstNormal > -1 && pendingOrders[firstNormal].status == 'cooking') {
      clearProcess(firstNormal)
    }
    // Insert the VIP order after the first normal order
    pendingOrders.splice(firstNormal < 1 ? 0 : firstNormal, 0, orderObj);
  } else {
    pendingOrders.push(orderObj);
  }
  processOrder();
}

const addBot = () => {
  botCounter.value++;
  bots.push({
    ID: botCounter.value,
    status: 'idle'
  })
  processOrder();
}
const removeBot = () => {
  if(!bots.length) return;
  const newestBot = bots.at(-1);
  const orderIdx = pendingOrders.findIndex((order) => order.botID == newestBot.ID && order.status == 'cooking' );
  clearProcess(orderIdx)
  bots.pop();
}

const processOrder = () => {
  const availableBot = bots.find((bot) => bot.status == 'idle');

  // Only process if a bot is available
  if (availableBot) {

    pendingOrders.some((order) => {
      if(order.status == 'pending') {
        // Set a queued task to complete the order and free the bot
        const intID = setTimeout(() => {
          // Complete the order
          completedOrders.push({
            ...order,
            status: 'completed'
          })
          const orderIdx = pendingOrders.findIndex((x) => x.ID == order.ID)
          // Free the bot
          const botIdx = bots.findIndex((x) => x.ID == availableBot.ID)
          bots[botIdx].status = 'idle'

          // Remove from pending orders
          if(orderIdx > -1) {
            pendingOrders.splice(orderIdx, 1)
          }
          // Get another order to process
          processOrder();
        }, COOK_TIME);
        order.timeoutID = intID;
        order.status = 'cooking';
        order.botID = availableBot.ID;
        availableBot.status = 'active';
        return true // Early terminate on first successful run
      }
    })
  }
}

const clearProcess = (orderIdx) => {
  if(orderIdx > -1) {
    clearTimeout(pendingOrders[orderIdx].timeoutID)
    // Reset bot status so that it can pick up other orders
    const botIdx = bots.findIndex((bot) => bot.ID == pendingOrders[orderIdx].botID)
    if(botIdx > -1) {
      bots[botIdx].status = 'idle';
    }
    // Clear processing information on the order
    pendingOrders[orderIdx].status = 'pending';
    pendingOrders[orderIdx].botID = null;
    pendingOrders[orderIdx].timeoutID = null;
  }
}

</script>

<style scoped></style>
