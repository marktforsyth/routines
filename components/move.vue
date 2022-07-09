<template>
  <div>
    <div
      class="normal-move"
      v-show="!move.break"
      @contextmenu="openContextMenu"
    >
      <div class="label">{{ move.label }}</div>
      <div class="time">{{ renderMove(move, m) }}</div>
    </div>
    <div class="transition" v-show="move.break && moveIndex === m">
      Transition: {{ renderMove(move, m) }}
    </div>

    <ContextMenu
      v-if="contextMenuOpen"
      :close="closeContextMenu"
      :actions="contextActions()"
    />
    <MoveEditor
      v-if="editorOpen"
      :close="closeEditor"
      :get-routine="getRoutine"
      :set-routine="setRoutine"
      :move-index="m"
    />
  </div>
</template>

<script>
export default {
  props: [
    "move",
    "m",
    "notLast",
    "moveIndex",
    "renderMove",
    "getRoutine",
    "setRoutine",
    "stopAndReset",
  ],
  data() {
    return {
      contextMenuOpen: false,
      editorOpen: false,
    };
  },
  methods: {
    openContextMenu() {
      this.contextMenuOpen = true;
      this.stopAndReset();
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

      if (this.m > 0) {
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
      let origMove = currentRoutine.moves[this.m];

      currentRoutine.moves[this.m] = currentRoutine.moves[this.m - 2];
      currentRoutine.moves[this.m - 2] = origMove;
      this.setRoutine(currentRoutine);
      this.closeContextMenu();
    },
    moveDown() {
      let currentRoutine = this.getRoutine();
      let origMove = currentRoutine.moves[this.m];

      currentRoutine.moves[this.m] = currentRoutine.moves[this.m + 2];
      currentRoutine.moves[this.m + 2] = origMove;
      this.setRoutine(currentRoutine);
      this.closeContextMenu();
    },
    deleteMove() {
      let currentRoutine = this.getRoutine();
      if (this.notLast) {
        currentRoutine.moves.splice(this.m, 2);
      } else {
        currentRoutine.moves.splice(this.m - 1, 2);
      }
      this.setRoutine(currentRoutine);
      this.closeContextMenu();
    },
  },
};
</script>

<style>
.transition {
  text-align: center;
  font-size: 1rem;
  font-weight: bold;
  padding: var(--small-gap);
}

.normal-move {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: var(--large-gap);
}

.normal-move > .label {
  font-size: 1.5rem;
  line-height: 1.5;
  /* word-break: break-all; */
}

.normal-move > .time {
  font-size: 2rem;
  background-color: var(--page-bg);
  border-radius: var(--round);
  padding: var(--small-gap);
  margin-left: var(--small-gap);
  width: 6.5rem;
  min-width: 6.5rem;
  text-align: center;
}
</style>
