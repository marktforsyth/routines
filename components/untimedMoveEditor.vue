<template>
  <div class="untimed-move-editor-modal" @click="exitModal">
    <div class="untimed-move-editor">
      <h1 class="title" v-if="isEditing()">Edit Move</h1>
      <h1 class="title" v-else>Add Move</h1>

      <input
        class="label"
        type="text"
        placeholder="Label"
        @change="handleTypingLabel"
        :value="label"
      />

      <button class="submit-btn" @click="submit">Submit</button>
    </div>
  </div>
</template>

<script>
export default {
  props: ["close", "getRoutine", "setRoutine", "untimedMoveIndex"],
  data() {
    return {
      label: "",
    };
  },
  methods: {
    exitModal(event) {
      if (event.target.className === "untimed-move-editor-modal") {
        this.close();
      }
    },
    handleTypingLabel(event) {
      this.label = event.target.value;
    },
    isEditing() {
      return (
        this.getRoutine().untimed.length > 0 &&
        this.getRoutine().untimed.length > this.untimedMoveIndex
      );
    },
    submit() {
      if (this.label === "") {
        return;
      }

      let currentRoutine = this.getRoutine();
      let untimedMove = {
        label: this.label,
      };

      currentRoutine.untimed[this.untimedMoveIndex] = untimedMove;
      this.setRoutine(currentRoutine);
      this.close();
    },
  },
  mounted() {
    if (this.isEditing()) {
      this.label = this.getRoutine().untimed[this.untimedMoveIndex].label;
    }
  },
};
</script>

<style>
.untimed-move-editor-modal {
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

.untimed-move-editor-modal > .untimed-move-editor {
  border-radius: var(--round);
  padding: var(--large-gap);
  background-color: var(--element-bg);
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

@media (max-width: 500px) {
  .untimed-move-editor-modal > .untimed-move-editor {
    max-width: 80vw;
  }

  .untimed-move-editor-modal > .untimed-move-editor > .label {
    max-width: calc(100% - var(--large-gap));
  }
}

.untimed-move-editor-modal > .untimed-move-editor > .title {
  text-align: center;
  margin-bottom: var(--large-gap);
  margin-top: 0;
}

.untimed-move-editor-modal > .untimed-move-editor > .label {
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

.untimed-move-editor-modal > .untimed-move-editor > .submit-btn {
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
