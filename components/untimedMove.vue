<template>
  <div class="untimed-move" @contextmenu="openContextMenu">
    <div class="label">{{ label }}</div>
    <div class="checkbox" @click="checked = !checked">
      <div v-show="checked"></div>
    </div>

    <ContextMenu
      v-if="contextMenuOpen"
      :close="closeContextMenu"
      :actions="contextActions()"
    />
    <UntimedMoveEditor
      v-if="editorOpen"
      :close="closeEditor"
      :get-routine="getRoutine"
      :set-routine="setRoutine"
      :untimed-move-index="u"
    />
  </div>
</template>

<script>
export default {
  props: ["label", "u", "notLast", "getRoutine", "setRoutine"],
  data() {
    return {
      checked: false,
      contextMenuOpen: false,
      editorOpen: false,
    };
  },
  methods: {
    openContextMenu() {
      this.contextMenuOpen = true;
    },
    closeContextMenu() {
      this.contextMenuOpen = false;
    },
    openEditor() {
      this.editorOpen = true;
      this.closeContextMenu();
    },
    closeEditor() {
      this.editorOpen = false;
    },
    contextActions() {
      let actions = [
        {
          title: "Edit",
          action: this.openEditor,
        },
        {
          title: "Delete",
          action: this.deleteMove,
        },
      ];

      if (this.u > 0) {
        actions.push({
          title: "Move Up",
          action: this.moveUp,
        });
      }

      if (this.notLast) {
        actions.push({
          title: "Move Down",
          action: this.moveDown,
        });
      }

      return actions;
    },
    moveUp() {
      let currentRoutine = this.getRoutine();
      let origUntimedMove = currentRoutine.untimed[this.u];

      currentRoutine.untimed[this.u] = currentRoutine.untimed[this.u - 1];
      currentRoutine.untimed[this.u - 1] = origUntimedMove;
      this.setRoutine(currentRoutine);
      this.closeContextMenu();
    },
    moveDown() {
      let currentRoutine = this.getRoutine();
      let origUntimedMove = currentRoutine.untimed[this.u];

      currentRoutine.untimed[this.u] = currentRoutine.untimed[this.u + 1];
      currentRoutine.untimed[this.u + 1] = origUntimedMove;
      this.setRoutine(currentRoutine);
      this.closeContextMenu();
    },
    deleteMove() {
      let currentRoutine = this.getRoutine();
      currentRoutine.untimed.splice(this.u, 1);
      this.setRoutine(currentRoutine);
      this.closeContextMenu();
    },
  },
};
</script>

<style>
.untimed-move {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: var(--large-gap);
}

.untimed-move > .label {
  font-size: 1.5rem;
  line-height: 1.5;
}

.untimed-move > .checkbox {
  width: 3rem;
  height: 3rem;
  border-radius: var(--round);
  background-color: var(--page-bg);
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
}

.untimed-move > .checkbox div {
  border-radius: var(--round);
  width: 1rem;
  height: 1rem;
  background-color: var(--text);
}
</style>
