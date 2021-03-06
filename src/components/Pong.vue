<template>
  <div class="pong">
    <h1 class="headline">
      What's keeping
      <br />us apart?
    </h1>
    <div class="net"></div>
    <div class="subheadline">
      <span v-if="step === 2">Jira</span>
      <span v-if="step === 3">Protected Sprints</span>
      <span v-if="step === 4">Overloaded Designers / Developers</span>
    </div>
    <div
      class="pad pad--left"
      :style="{top: `${padLeftHeight}px`, transform: `translate(${padLeftX}px, ${padLeftY}px) rotate(-90deg)`}"
    >Developer</div>
    <div
      class="pad pad--right"
      :style="{transform: `translate(${padRightX}px, ${padRightY}px) rotate(90deg)`}"
    >Designer</div>
    <div class="ball" :style="{transform: `translate(${ballX}px, ${ballY}px)`}"></div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      inertia: 8,
      inertiaX: undefined,
      inertiaY: undefined,
      canvasWidth: undefined,
      canvasHeight: undefined,
      paddingPercent: 2,
      padding: undefined,
      padWidth: undefined,
      padLeftX: undefined,
      padLeftY: undefined,
      padLeftHeight: undefined,
      padRightX: undefined,
      padRightY: undefined,
      padRightHeight: undefined,
      ballDiameter: undefined,
      ballX: undefined,
      ballY: undefined,
      rafID: undefined,
      lastTime: 0
    };
  },
  props: ["step"],
  mounted: function() {
    this.init();
    this.gameLoop();
    window.addEventListener("resize", this.init);
  },
  beforeDestroy: function() {
    window.removeEventListener("resize", this.init);
    window.cancelAnimationFrame(this.rafID);
  },
  computed: {},
  directives: {},
  methods: {
    init() {
      const slideDimensions = document
        .getElementsByClassName("slideshow")[0]
        .getBoundingClientRect();

      const padLeftDimensions = document
        .getElementsByClassName("pad--left")[0]
        .getBoundingClientRect();

      const ballDimensions = document
        .getElementsByClassName("ball")[0]
        .getBoundingClientRect();

      const padRightDimensions = document
        .getElementsByClassName("pad--right")[0]
        .getBoundingClientRect();

      this.canvasWidth = slideDimensions.width;
      this.canvasHeight = slideDimensions.height;

      this.padding = this.canvasWidth * (this.paddingPercent / 100);

      this.padWidth = padLeftDimensions.height;

      this.padLeftX = this.padding;
      this.padLeftY = this.padding;
      this.padLeftHeight = padLeftDimensions.width;

      this.padRightX = this.canvasWidth - this.padding;
      this.padRightY = this.padding;
      this.padRightHeight = padRightDimensions.width;

      this.ballDiameter = ballDimensions.width;

      if (this.ballX === undefined) {
        this.ballX = slideDimensions.width / 2 - this.ballDiameter / 2;
      }

      if (this.ballY === undefined) {
        this.ballY = slideDimensions.height / 2 - this.ballDiameter / 2;
      }

      if (this.inertiaX === undefined) {
        this.inertiaX = this.inertia;
      }

      if (this.inertiaY === undefined) {
        this.inertiaY = this.inertia;
      }
    },
    gameLoop(tFrame) {
      if (this.inertia >= 0.0001) {
        this.rafID = window.requestAnimationFrame(this.gameLoop);
      }

      const timeDifference = tFrame - this.lastTime;

      // Stop animation when ball is in middel of field and step is > 1
      if (
        this.step > 1 &&
        this.ballX >= this.canvasWidth * 0.33 &&
        this.ballX <= this.canvasWidth * 0.66 &&
        this.ballY >= this.canvasHeight * 0.01 &&
        this.ballY <= this.canvasHeight * 0.99
      ) {
        const slowdown = 0.95;
        this.inertia *= slowdown;
        this.inertiaX *= slowdown;
        this.inertiaY *= slowdown;
      }

      this.updateBall(timeDifference);
      this.updatePads(timeDifference);

      this.lastTime = tFrame;
    },
    updateBall(timeDifference) {
      if (timeDifference >= 16) {
        this.ballX += this.inertiaX * (timeDifference / 16);
        this.ballY += this.inertiaY * (timeDifference / 16);
      }

      // Right edge
      if (
        this.ballX + this.ballDiameter >=
          this.canvasWidth - this.padWidth - this.padding &&
        this.inertiaX > 0
      ) {
        this.inertiaX *= -1;
      }

      // Left edge
      if (this.ballX <= this.padding + this.padWidth && this.inertiaX < 0) {
        this.inertiaX *= -1;
      }

      // Bottom edge
      if (
        this.ballY + this.ballDiameter >= this.canvasHeight &&
        this.inertiaY > 0
      ) {
        this.inertiaY *= -1;
      }

      // Top Edge
      if (this.ballY <= 0 && this.inertiaY < 0) {
        this.inertiaY *= -1;
      }
    },
    updatePads() {
      const leftDistance = Math.abs(this.ballX - this.padLeftX);
      const rightDistance = Math.abs(this.ballX - this.padRightX);

      const headedLeft = this.inertiaX < 0;

      this.padLeftY = this.clampedPadPosition(
        this.padLeftHeight,
        this.padLeftY,
        leftDistance,
        headedLeft
      );

      this.padRightY = this.clampedPadPosition(
        this.padRightHeight,
        this.padRightY,
        rightDistance,
        !headedLeft
      );
    },
    clampedPadPosition(paddleHeight, currentPos, distance, headedToMe) {
      let jitter = (Math.random() - 0.5) * (this.canvasHeight / 20);
      let throttlingFactor = 0;

      if (headedToMe && distance <= this.canvasWidth * 0.3) {
        throttlingFactor = 0.08;
      } else if (headedToMe && distance <= this.canvasWidth * 0.4) {
        throttlingFactor = 0.008;
      } else {
        throttlingFactor = 0.0006;
        jitter = (Math.random() - 0.5) * (this.canvasHeight / 4);
      }

      if (this.inertia <= 1) {
        jitter = 0;
      }

      const endPosition = this.ballY - paddleHeight / 2 - jitter;
      const position =
        currentPos + (endPosition - currentPos) * throttlingFactor;
      const bottomClamped = Math.max(position, 0);

      return Math.min(bottomClamped, this.canvasHeight - paddleHeight);
    }
  }
};
</script>

<style lang="scss" scoped>
@import "../variables.scss";

$horizontal-padding: $space * 8;
$net-padding: $space * 8;

.headline,
.subheadline {
  position: absolute;
  top: $horizontal-padding;
  font-size: $h2size;
  line-height: $line-height;
  max-width: 35%;
}

.headline {
  left: 25%;
  transform: translateX(-50%);
}

.subheadline {
  left: 50%;
  padding-left: $net-padding;
  color: $blue;
}

.net {
  position: absolute;
  left: 50%;
  top: 0;
  height: 100%;
  transform: translateX(-50%);
  width: $space;
  background: repeating-linear-gradient(
    180deg,
    $blue,
    $blue ($space * 3),
    $black ($space * 3),
    $black ($space * 6.1)
  );
}

.pad {
  padding: ($space) ($space * 4) ($space * 2);
  transform-origin: 0% 0%;
}

.ball,
.pad {
  left: 0;
  top: 0;
  position: absolute;
  background-color: $white;
  color: $black;
  font-size: $h3size;
  will-change: transition;
}

.ball {
  width: $space * 4;
  height: $space * 4;
}
</style>
