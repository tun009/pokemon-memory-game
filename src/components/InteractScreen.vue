<template>
  <div class="screen">
    <div class="countdown-timer">
      <span :class="{ warning: remainingTime <= 10 }"
        >Thời gian: {{ remainingTime }}s</span
      >
    </div>
    <div
      class="screen__inner"
      :style="{
        width: `${
          ((((920 - 16 * 4) / Math.sqrt(cardsContext.length) - 16) * 3) / 4 +
            16) *
          Math.sqrt(cardsContext.length)
        }px`,
      }"
    >
      <card-memmory
        v-for="(card, index) in cardsContext"
        :key="index"
        :ref="`card-${index}`"
        :cardsContext="cardsContext"
        :imgBackFaceUrl="`images/${card}.png`"
        :card="{ index, value: card }"
        :rules="rules"
        @onFlip="checkRule($event)"
      />
    </div>
  </div>
</template>

<script>
import Card from "./Card.vue";
export default {
  props: {
    cardsContext: {
      type: Array,
      default: function () {
        return [];
      },
    },
    countdownTime: {
      type: Number,
      default: 60,
    },
  },
  components: {
    CardMemmory: Card,
  },
  data() {
    return {
      rules: [],
      remainingTime: 0,
      timerInterval: null,
    };
  },
  mounted() {
    this.startCountdown();
  },
  beforeUnmount() {
    this.clearCountdown();
  },
  methods: {
    startCountdown() {
      this.remainingTime = this.countdownTime;
      this.clearCountdown(); // Đảm bảo không có interval nào đang chạy

      this.timerInterval = setInterval(() => {
        if (this.remainingTime > 0) {
          this.remainingTime -= 1;
        } else {
          this.clearCountdown();
          this.$emit("onTimeout");
        }
      }, 1000);
    },

    clearCountdown() {
      if (this.timerInterval) {
        clearInterval(this.timerInterval);
        this.timerInterval = null;
      }
    },

    checkRule(card) {
      if (this.rules.length === 2) return false;
      this.rules.push(card);
      if (
        this.rules.length === 2 &&
        this.rules[0].value === this.rules[1].value
      ) {
        console.log("Right...");
        this.$refs[`card-${this.rules[1].index}`].onEnabledDisabledMode();
        this.rules = [];

        const disabledElements = document.querySelectorAll(
          ".screen .card.disabled"
        );
        if (
          disabledElements &&
          disabledElements.length === this.cardsContext.length - 2
        )
          setTimeout(() => {
            this.clearCountdown();
            this.$emit("onFinish");
          }, 920);
      } else if (
        this.rules.length === 2 &&
        this.rules[0].value !== this.rules[1].value
      ) {
        console.log("wrong!");
        setTimeout(() => {
          this.$refs[`card-${this.rules[0].index}`].onFlipBackCard();
          this.$refs[`card-${this.rules[1].index}`].onFlipBackCard();
          this.rules = [];
        }, 800);
      } else return false;
    },
  },
};
</script>

<style scoped>
.screen {
  width: 100%;
  height: 100vh;
  position: absolute;
  top: 0;
  left: 0;
  z-index: 2;
  background-color: var(--dark);
  color: var(--light);
}

.screen__inner {
  width: calc(424px);
  display: flex;
  flex-wrap: wrap;
  margin: 2rem auto;
}

.countdown-timer {
  position: fixed;
  top: 20px;
  right: 20px;
  font-size: 24px;
  background-color: rgba(0, 0, 0, 0.7);
  padding: 10px 15px;
  border-radius: 5px;
  z-index: 10;
}

.countdown-timer .warning {
  color: #ff5252;
  font-weight: bold;
  animation: pulse 1s infinite;
}

@keyframes pulse {
  0% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
  100% {
    opacity: 1;
  }
}
</style>
