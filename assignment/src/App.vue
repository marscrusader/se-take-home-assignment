<template>
  <main>
    <div class="container">
      <div class="buttons-area area">
        <button class="btn item" @click="createNormalOrder">
          New Normal Order
        </button>
        <button class="btn item" @click="createVIPOrder">New VIP Order</button>
        <button class="btn item" @click="addBot">+ Bot</button>
        <button class="btn item" @click="removeBot">- Bot</button>
      </div>
      <div class="bots-area area">
        <p>Bots</p>
        <div v-for="bot in bots" :key="bot.id" class="item bot">
          {{
            `Bot id #${bot.id}, ${
              bot.processingOrderId
                ? `process order id #${bot.processingOrderId}`
                : "idle"
            }`
          }}
        </div>
      </div>
      <div class="pending-area area">
        <p>Pending Area</p>
        <div
          v-for="order in pendingOrders"
          :key="order.id"
          :class="{
            item: true,
            pending: true,
            order: true,
          }"
        >
          {{
            order.type === "VIP"
              ? `VIP order id #${order.id}`
              : `NORMAL order id #${order.id}`
          }}
        </div>
      </div>
      <div class="processing-area area">
        <p>Processing Area</p>
        <div
          v-for="order in processingOrders"
          :key="order.id"
          class="order processing item"
        >
          {{
            order.type === "VIP"
              ? `Processing VIP order id #${order.id}`
              : `Processing NORMAL order id #${order.id}`
          }}
        </div>
      </div>
      <div class="complete-area area">
        <p>Complete Area</p>
        <div
          v-for="order in completeOrders"
          :key="order.id"
          class="order completed item"
        >
          {{
            order.type === "VIP"
              ? `VIP order id #${order.id}`
              : `NORMAL order id #${order.id}`
          }}
        </div>
      </div>
    </div>
  </main>
</template>

<script>
export default {
  data() {
    return {
      pendingOrders: [],
      processingOrders: [],
      completeOrders: [],
      bots: [],
      currentBotId: 1,
    };
  },
  methods: {
    createNormalOrder() {
      const order = { id: this.generateOrderId(), type: "Normal" };
      this.pendingOrders.push(order);
      this.processOrder();
    },
    createVIPOrder() {
      const order = { id: this.generateOrderId(), type: "VIP" };
      let insertIndex = this.pendingOrders.findIndex(
        (order) => order.type === "Normal"
      );
      if (insertIndex === -1) {
        insertIndex = this.pendingOrders.length;
      }
      this.pendingOrders.splice(insertIndex, 0, order);
      this.processOrder();
    },
    generateOrderId() {
      return (
        this.pendingOrders.length +
        this.processingOrders.length +
        this.completeOrders.length +
        1
      );
    },
    processOrder() {
      for (const bot of this.bots) {
        if (!bot.processingOrderId && this.pendingOrders.length > 0) {
          const order = this.pendingOrders[0];
          order.botId = bot.id;
          bot.processingOrderId = order.id;
          this.processingOrders.push(order);
          this.pendingOrders.shift();
          setTimeout(() => {
            if (
              this.processingOrders.find(
                (processingOrder) => processingOrder.id === order.id
              )
            ) {
              this.processingOrders = this.processingOrders.filter(
                (processingOrder) => processingOrder.id !== order.id
              );
              this.completeOrders.push(order);
              bot.processingOrderId = null;
            }

            const botProcessingOrder = this.bots.find(
              (bot) => bot.processingOrderId === order.id
            );
            if (botProcessingOrder) {
              botProcessingOrder.processingOrderId = null;
            }

            this.processOrder();
          }, 10000);
        }
      }
    },
    addBot() {
      const bot = {
        id: this.currentBotId++,
        processingOrderId: null,
      };
      this.bots.push(bot);
      this.processOrder();
    },
    removeBot() {
      if (this.bots.length > 0) {
        const bot = this.bots[this.bots.length - 1];
        if (bot.processingOrderId) {
          const order = this.processingOrders.find(
            (order) => order.id === bot.processingOrderId
          );
          if (order) {
            this.pendingOrders.unshift(order);
            this.processingOrders = this.processingOrders.filter(
              (order) => order.id !== bot.processingOrderId
            );
            order.botId = null;
            bot.processingOrderId = null;
          }
        }
        this.bots.pop();
      }
    },
  },
};
</script>

<style scoped>
.container {
  width: 100%;
  height: 100vh;
  display: flex;
  justify-content: space-evenly;
}

.area {
  width: 25%;
  padding-left: 10px;
  display: flex;
  flex-direction: column;
}

.item {
  margin-bottom: 60px;
}

.buttons-area {
  display: flex;
  flex-direction: column;
}

.pending-area {
  display: flex;
  flex-direction: column;
}

.completed-area {
  display: flex;
  flex-direction: column;
}

.pending {
  background-color: yellow;
}

.bot {
  background-color: lightgrey;
}

.order {
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.order.processing {
  background-color: orange;
}

.order.completed {
  background-color: green;
}
</style>
