<template>
  <main>
    <div class="container">
      <div class="buttons-area area">
        <button class="btn">New Normal Order</button>
        <button class="btn">New VIP Order</button>
        <button class="btn">+ Bot</button>
        <button class="btn">- Bot</button>
      </div>
      <div class="bots-area area">asfds</div>
      <div class="pending-area area">dddd</div>
      <div class="complete-area area">asfwewe</div>
    </div>
  </main>
</template>

<script>
export default {
  data() {
    return {
      pendingOrders: [],
      completeOrders: [],
      bots: 0,
    };
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
}

.buttons-area {
  display: flex;
  justify-content: space-evenly;
  flex-direction: column;
}
</style>
