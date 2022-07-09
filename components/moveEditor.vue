<template>
  <div class="move-editor-modal" @click="exitModal">
    <div class="move-editor">
      <h1 class="title" v-if="isEditing()">Edit Move</h1>
      <h1 class="title" v-else>Add Move</h1>

      <input
        class="label"
        type="text"
        placeholder="Label"
        @change="handleTypingLabel"
        :value="label"
      />

      <div class="time-setters-container">
        <div class="time-setter">
          <button @click="increase(0)">
            <img src="~assets/up.svg" alt="Up button" />
          </button>
          <div>{{ doubleDigit(minutes) }}</div>
          <button @click="decrease(0)">
            <img src="~assets/down.svg" alt="Down button" />
          </button>
        </div>
        <div class="time-setter">
          <button @click="increase(1)">
            <img src="~assets/up.svg" alt="Up button" />
          </button>
          <div>{{ doubleDigit(seconds) }}</div>
          <button @click="decrease(1)">
            <img src="~assets/down.svg" alt="Down button" />
          </button>
        </div>
      </div>

      <button class="submit-btn" @click="submit">Submit</button>
    </div>
  </div>
</template>

<script>
export default {
  props: ["close", "getRoutine", "setRoutine", "moveIndex"],
  data() {
    return {
      label: "",
      minutes: 0,
      seconds: 0,
    };
  },
  methods: {
    exitModal(event) {
      if (event.target.className === "move-editor-modal") {
        this.close();
      }
    },
    handleTypingLabel(event) {
      this.label = event.target.value;
    },
    increase(unit) {
      if (unit === 0) {
        if (this.minutes < 99) {
          this.minutes++;
        } else {
          this.hours = 0;
        }
      } else if (unit == 1) {
        if (this.seconds < 59) {
          this.seconds++;
        } else {
          this.seconds = 0;
        }
      }
    },
    decrease(unit) {
      if (unit === 0) {
        if (this.minutes > 0) {
          this.minutes--;
        } else {
          this.minutes = 99;
        }
      } else if (unit == 1) {
        if (this.seconds > 0) {
          this.seconds--;
        } else {
          this.seconds = 59;
        }
      }
    },
    doubleDigit(num) {
      if (num < 10) {
        return "0" + num;
      }
      return num;
    },
    isEditing() {
      return (
        this.getRoutine().moves.length > 0 &&
        this.getRoutine().moves.length > this.moveIndex
      );
    },
    submit() {
      if ((this.minutes === 0 && this.seconds === 0) || this.label === "") {
        return;
      }

      let currentRoutine = this.getRoutine();
      let move = {
        label: this.label,
        lenSeconds: this.minutes * 60 + this.seconds,
        break: false,
      };

      let index = this.moveIndex;
      if (!this.isEditing()) {
        if (currentRoutine.moves.length > 0) {
          currentRoutine.moves.push({ break: true, lenSeconds: 5 });
          index++;
        }
      }

      currentRoutine.moves[index] = move;

      this.setRoutine(currentRoutine);
      this.close();
    },
  },
  mounted() {
    if (this.isEditing()) {
      this.label = this.getRoutine().moves[this.moveIndex].label;

      let seconds = this.getRoutine().moves[this.moveIndex].lenSeconds;
      this.minutes = Math.floor(seconds / 60);
      this.seconds = seconds % 60;
    }
  },
};
</script>

<style>
.move-editor-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: rgba(0.3, 0.3, 0.3, 0.4);
  display: flex;
  justify-content: center;
  align-items: center;
}

.move-editor-modal > .move-editor {
  border-radius: var(--round);
  padding: var(--large-gap);
  background-color: var(--element-bg);
}

@media (max-width: 500px) {
  .move-editor-modal > .move-editor {
    max-width: 80vw;
  }

  .move-editor-modal > .move-editor > .label {
    max-width: calc(100% - var(--large-gap));
  }
}

.move-editor-modal > .move-editor > .title {
  text-align: center;
  margin-bottom: var(--large-gap);
  margin-top: 0;
}

.move-editor-modal > .move-editor > .label {
  border-radius: var(--round);
  border: none;
  outline: none;
  font-size: 1.5rem;
  font-family: inherit;
  padding: var(--small-gap);
  background-color: var(--page-bg);
  color: var(--text);
  margin-bottom: var(--large-gap);
  text-align: center;
}
/* 
.move-editor-modal > .move-editor > .untimed-checkbox-container {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: var(--large-gap);
  font-size: 2rem;
  margin-bottom: var(--large-gap);
}

.move-editor-modal
  > .move-editor
  > .untimed-checkbox-container
  > .untimed-checkbox {
  border-radius: var(--round);
  background-color: var(--page-bg);
  width: 3rem;
  height: 3rem;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}

.move-editor-modal
  > .move-editor
  > .untimed-checkbox-container
  > .untimed-checkbox
  div {
  border-radius: var(--round);
  background-color: var(--text);
  width: 1rem;
  height: 1rem;
} */

.move-editor-modal > .move-editor > .time-setters-container {
  display: flex;
  justify-content: space-evenly;
  align-items: center;
  margin-bottom: calc(2 * var(--large-gap));
}

.move-editor-modal > .move-editor > .time-setters-container > .time-setter {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
}

.move-editor-modal
  > .move-editor
  > .time-setters-container
  > .time-setter
  button {
  background-color: var(--page-bg);
  border: none;
  color: var(--text);
  font-family: inherit;
  padding: var(--small-gap);
  border-radius: var(--round);
}

.move-editor-modal
  > .move-editor
  > .time-setters-container
  > .time-setter
  button
  img {
  width: 60%;
  height: 60%;
}

.move-editor-modal > .move-editor > .time-setters-container > .time-setter div {
  font-size: 4rem;
  padding: var(--large-gap) var(--small-gap);
  width: 5rem;
  text-align: center;
}

.move-editor-modal > .move-editor > .submit-btn {
  border: none;
  background-color: var(--page-bg);
  border-radius: var(--round);
  padding: var(--small-gap);
  font-size: 1.5rem;
  color: var(--text);
  font-family: inherit;
  text-align: center;
  width: 100%;
}
</style>
