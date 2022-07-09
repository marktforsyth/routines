<template>
  <div class="context-menu-modal" @click="handleClick">
    <div class="context-menu">
      <div
        class="context-action"
        v-for="(action, a) in actions"
        :key="a + 'context-menu-item' + action.title"
      >
        <button @click="action.action">{{ action.title }}</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: ["close", "actions"],
  methods: {
    handleClick(event) {
      if (event.target.className === "context-menu-modal") {
        this.close();
      }
    },
  },
};
</script>

<style>
.context-menu-modal {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  background-color: rgba(0.1, 0.1, 0.1, 0.4);
  display: flex;
  justify-content: center;
  align-items: center;
}

.context-menu-modal > .context-menu {
  background-color: var(--element-bg);
  border-radius: var(--round);
  padding: var(--large-gap);
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  width: 60%;
  max-width: 25rem;
}

.context-menu-modal > .context-menu > .context-action button {
  font-size: 1.5rem;
  width: 100%;
  text-align: center;
  background-color: var(--page-bg);
  color: var(--text);
  border: none;
  border-radius: var(--round);
  padding: var(--small-gap);
}
</style>
