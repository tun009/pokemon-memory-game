<template>
  <main-screen
    v-if="statusMatch === 'default'"
    @onStart="onHandleBeforeStart($event)"
  />
  <interact-screen
    v-if="statusMatch === 'match'"
    :cardsContext="settings.cardsContext"
    :countdownTime="countdownTime"
    @onFinish="onGetResult"
    @onTimeout="onTimeout"
  />
  <result-screen
    v-if="statusMatch === 'result'"
    :timer="timer"
    :isTimeout="isTimeout"
    @onStartAgain="statusMatch = 'default'"
  />
</template>

<script>
import MainScreen from "./components/MainScreen.vue";
import InteractScreen from "./components/InteractScreen.vue";
import ResultScreen from "./components/ResultScreen.vue";

import { shuffled } from "./utils/array";

export default {
  name: "App",
  components: {
    MainScreen,
    InteractScreen,
    ResultScreen,
  },
  data() {
    return {
      settings: {
        totalOfBlocks: 0,
        cardsContext: [],
        startedAt: null,
      },
      timer: 0,
      statusMatch: "default",
      countdownTime: 60, // Thời gian đếm ngược (giây)
      isTimeout: false, // Biến để kiểm tra xem đã hết thời gian chưa
    };
  },
  methods: {
    onHandleBeforeStart(configs) {
      this.settings.totalOfBlocks = configs.totalOfBlocks;

      const firstCards = Array.from(
        { length: this.settings.totalOfBlocks / 2 },
        (_, i) => i + 1
      );
      const secondCards = [...firstCards];
      const cards = [...firstCards, ...secondCards];

      this.settings.cardsContext = shuffled(shuffled(shuffled(cards)));
      this.settings.startedAt = new Date().getTime();
      this.isTimeout = false; // Reset trạng thái timeout

      // Cập nhật thời gian đếm ngược từ cấu hình
      if (configs.countdownTime) {
        this.countdownTime = configs.countdownTime;
      }

      this.statusMatch = "match";
    },

    onGetResult() {
      this.statusMatch = "result";
      this.timer = new Date().getTime() - this.settings.startedAt;
    },

    onTimeout() {
      this.statusMatch = "result";
      this.isTimeout = true;
    },
  },
};
</script>

<style lang="css" scoped>
.copyright {
  position: fixed;
  left: 50%;
  transform: translateX(-50%);
  bottom: 1.5rem;
  color: var(--light);
  z-index: 3;
  font-size: 1.5rem;
}

.copyright a {
  color: #f4dc26;
}
</style>
