<template>
  <main>
    <div class="container">
      <div class="buttons-area area">
        <button class="btn item" @click="newNormalOrder">
          New Normal Order
        </button>
        <button class="btn item" @click="newVIPOrder">New VIP Order</button>
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
          class="item pending order"
        >
          {{
            order.type === vipType
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
            order.type === vipType
              ? `Processing VIP order id #${order.id}`
              : `Processing NORMAL order id #${order.id}`
          }}, {{ order.processingTime }}s
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
            order.type === vipType
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
      normalType: "NORMAL",
      vipType: "VIP",
    };
  },
  methods: {
    newNormalOrder() {
      const order = {
        id: this.generateOrderId(),
        type: this.normalType,
        botId: null,
        processingTime: 0,
        processingTimeId: null,
      };
      this.pendingOrders.push(order);
      this.processOrder();
    },
    newVIPOrder() {
      const order = {
        id: this.generateOrderId(),
        type: this.vipType,
        botId: null,
        processingTime: 0,
        processingTimeId: null,
      };

      // Insert the VIP order before the first normal order
      let insertIndex = this.pendingOrders.findIndex(
        (order) => order.type === this.normalType
      );

      // If there is no normal order, insert the VIP order at the end
      if (insertIndex === -1) {
        insertIndex = this.pendingOrders.length;
      }
      this.pendingOrders.splice(insertIndex, 0, order);
      this.processOrder();
    },
    generateOrderId() {
      // Generate order id by getting total number of orders and add 1, so it always increase
      return (
        this.pendingOrders.length +
        this.processingOrders.length +
        this.completeOrders.length +
        1
      );
    },
    processOrder() {
      if (this.pendingOrders.length) {
        // Find a bot that is not processing any order
        const bot = this.bots.find((bot) => !bot.processingOrderId);
        if (bot) {
          // Get the first order in pending orders
          const order = this.pendingOrders.shift();

          // Assign the order to the bot
          order.botId = bot.id;
          bot.processingOrderId = order.id;

          // Move the order from pending to processing
          this.processingOrders.push(order);

          // Start processing the order
          // Increase the processing time by 1 second using setInterval
          order.processingTimeId = setInterval(() => {
            order.processingTime++;

            // If the processing time is 10 seconds, stop the setInterval
            // and move the order from processing to complete
            if (order.processingTime >= 10) {
              clearInterval(order.processingTimeId);
              const orderIndex = this.processingOrders.findIndex(
                (processingOrder) => processingOrder.id === order.id
              );
              this.processingOrders.splice(orderIndex, 1);
              this.completeOrders.push(order);
              bot.processingOrderId = null;

              // Process the next order since we have a free bot
              this.processOrder();
            }
          }, 1000);
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
        const bot = this.bots.pop();

        // If the bot is processing an order, stop the processing
        if (bot.processingOrderId) {
          const orderIndex = this.processingOrders.findIndex(
            (order) => order.id === bot.processingOrderId
          );
          if (orderIndex !== -1) {
            const order = this.processingOrders[orderIndex];

            // Reset the processing time
            clearInterval(order.processingTimeId);
            order.processingTime = 0;
            order.botId = null;

            // Move the order from processing to pending
            this.pendingOrders.unshift(order);
            this.processingOrders.splice(orderIndex, 1);
            bot.processingOrderId = null;
          }
        }
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
