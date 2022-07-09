<template>
  <div class="page-container" @contextmenu="stopDefaultCtxMenu">
    <div class="content">
      <h1 class="title">Routine</h1>
      <p
        class="no-moves-yet"
        v-if="
          (!routine.moves || routine.moves.length === 0) &&
          (!routine.untimed || routine.untimed.length === 0)
        "
      >
        No moves have been created yet. Click the "+" button below to get
        started!
      </p>
      <div v-if="routine.moves && routine.moves.length > 0">
        <h2 class="section-title">Timed Moves</h2>
      </div>
      <div
        :class="'move' + (moveIndex === m ? ' active' : '')"
        v-for="(move, m) in routine.moves"
        v-show="!move.break || moveIndex === m"
        :key="m + 'index-routine'"
      >
        <Move
          :move="move"
          :m="m"
          :not-last="m !== routine.moves.length - 1"
          :move-index="moveIndex"
          :render-move="renderMove"
          :get-routine="getRoutine"
          :set-routine="setRoutine"
          :stop-and-reset="stopAndReset"
        />
      </div>

      <div v-if="routine.untimed && routine.untimed.length > 0">
        <h2 class="section-title">Untimed Moves</h2>
      </div>
      <div
        class="move"
        v-for="(untimedMove, u) in routine.untimed"
        :key="u + 'index-routine-untimed'"
      >
        <UntimedMove
          :label="untimedMove.label"
          :u="u"
          :not-last="u !== routine.untimed.length - 1"
          :get-routine="getRoutine"
          :set-routine="setRoutine"
        />
      </div>
    </div>

    <div class="ctrl-bar">
      <!-- <NuxtLink to="routine-selector">☰</NuxtLink> -->
      <button @click="openContextMenu">
        <img src="~assets/plus.svg" alt="Plus button" />
      </button>
      <button
        @click="playPause"
        :disabled="
          routine.moves &&
          moveIndex === routine.moves.length - 1 &&
          timeElapsed ===
            timestamp(moveIndex) + routine.moves[moveIndex].lenSeconds
        "
      >
        <div v-if="isPlaying">
          <img src="~assets/pause.svg" alt="Pause button" />
        </div>
        <div v-else><img src="~assets/play.svg" alt="Play button" /></div>
      </button>
      <button @click="restart">
        <img src="~assets/reload.svg" alt="Restart button" />
      </button>
    </div>

    <ContextMenu
      v-if="contextMenuOpen"
      :close="closeContextMenu"
      :actions="[
        {
          title: 'Timed',
          action: openMoveEditor,
        },
        {
          title: 'Untimed',
          action: openUntimedMoveEditor,
        },
      ]"
    />

    <MoveEditor
      v-if="moveEditorOpen"
      :close="closeMoveEditor"
      :get-routine="getRoutine"
      :set-routine="setRoutine"
      :move-index="routine.moves.length"
    />
    <UntimedMoveEditor
      v-if="untimedMoveEditorOpen"
      :close="closeUntimedMoveEditor"
      :get-routine="getRoutine"
      :set-routine="setRoutine"
      :untimed-move-index="routine.untimed.length"
    />

    <audio
      loop
      ref="whiteNoise"
      :src="require('~/assets/river-white-noise.mp3').default"
    ></audio>
  </div>
</template>

<script>
export default {
  data() {
    return {
      routine: {},
      moveIndex: 0,
      isPlaying: false,
      hasStarted: false,
      timeElapsed: 0,
      moveEditorOpen: false,
      untimedMoveEditorOpen: false,
      contextMenuOpen: false,
    };
  },
  methods: {
    stopDefaultCtxMenu(event) {
      event.preventDefault();
    },
    async playRoutine() {
      const waitUntil = async (action) => {
        if (action()) {
          return;
        }

        return new Promise((resolve, reject) => {
          const wait = setInterval(() => {
            try {
              if (action()) {
                clearInterval(wait);
                resolve();
              }
            } catch (err) {
              clearInterval(wait);
              reject(err);
            }
          }, 300);
        });
      };

      if (this.isPlaying) {
        await waitUntil(() => !window.speechSynthesis.speaking);
        if (this.hasStarted) {
          this.timeElapsed++;
        } else {
          window.speechSynthesis.speak(
            new SpeechSynthesisUtterance(
              "Starting routine. " + this.routine.moves[0].label + ". Start"
            )
          );
          this.hasStarted = true;
        }

        if (
          this.timeElapsed ===
          this.timestamp(this.moveIndex) +
            this.routine.moves[this.moveIndex].lenSeconds
        ) {
          if (this.moveIndex === this.routine.moves.length - 1) {
            window.speechSynthesis.speak(
              new SpeechSynthesisUtterance("Routine completed")
            );

            this.isPlaying = false;
            this.$refs.whiteNoise.pause();
          } else {
            if (!this.routine.moves[this.moveIndex].break) {
              window.speechSynthesis.speak(
                new SpeechSynthesisUtterance(
                  this.routine.moves[this.moveIndex + 2].label + ". 5"
                )
              );
            } else {
              window.speechSynthesis.speak(
                new SpeechSynthesisUtterance("Start")
              );
            }
          }
        }

        if (
          this.routine.moves[this.moveIndex].break &&
          this.routine.moves[this.moveIndex].lenSeconds -
            this.timeElapsed +
            this.timestamp(this.moveIndex) >
            0
        ) {
          window.speechSynthesis.speak(
            new SpeechSynthesisUtterance(
              this.routine.moves[this.moveIndex].lenSeconds -
                this.timeElapsed +
                this.timestamp(this.moveIndex)
            )
          );
        }

        setTimeout(this.playRoutine, 1000);
      }
    },
    playPause() {
      this.isPlaying = !this.isPlaying;

      if (!this.isPlaying) {
        this.$refs.whiteNoise.pause();
      } else {
        this.$refs.whiteNoise.play();
      }
    },
    restart() {
      this.timeElapsed = 0;
      this.moveIndex = 0;
      this.hasStarted = false;
    },
    stopAndReset() {
      this.isPlaying = false;
      this.restart();
    },
    renderMove(move, m) {
      if (this.timeElapsed < this.timestamp(m)) {
        return this.formatTime(move.lenSeconds);
      }
      if (
        this.timeElapsed >= this.timestamp(m) &&
        this.timeElapsed < this.timestamp(m) + move.lenSeconds
      ) {
        this.moveIndex = m;
        return this.formatTime(
          move.lenSeconds - this.timeElapsed + this.timestamp(m)
        );
      }

      return 0;
    },
    formatTime(seconds) {
      if (seconds < 60) {
        return seconds;
      }
      if (seconds < 3600) {
        return Math.floor(seconds / 60) + ":" + this.doubleDigit(seconds % 60);
      }
    },
    doubleDigit(num) {
      if (num < 10) {
        return "0" + num;
      }
      return num;
    },
    timestamp(m) {
      let output = 0;
      for (let i = 0; i < m; i++) {
        output += this.routine.moves[i].lenSeconds;
      }
      return output;
    },
    openMoveEditor() {
      this.moveEditorOpen = true;
      this.closeContextMenu();
      this.stopAndReset();
    },
    closeMoveEditor() {
      this.moveEditorOpen = false;
    },
    openUntimedMoveEditor() {
      this.untimedMoveEditorOpen = true;
      this.closeContextMenu();
    },
    closeUntimedMoveEditor() {
      this.untimedMoveEditorOpen = false;
    },
    openContextMenu() {
      this.contextMenuOpen = true;
    },
    closeContextMenu() {
      this.contextMenuOpen = false;
    },
    getRoutine() {
      return JSON.parse(localStorage.routine);
    },
    setRoutine(routine) {
      this.routine = routine;
      localStorage.routine = JSON.stringify(routine);
    },
  },
  watch: {
    isPlaying(val) {
      if (val) {
        this.playRoutine();
      }
    },
  },
  mounted() {
    if (!localStorage.routine) {
      localStorage.routine = JSON.stringify({ moves: [], untimed: [] });
    }

    this.routine = JSON.parse(localStorage.routine);
    this.$refs.whiteNoise.volume = 0.2;
  },
};
</script>

<style>
.page-container {
  background-color: var(--ctrl-bar-bg);
  color: var(--text);
}

.page-container > .content {
  background-color: var(--page-bg);
  border-bottom-left-radius: var(--round);
  border-bottom-right-radius: var(--round);
  width: calc(100vw - 2rem);
  height: calc(100vh - 6rem - 2 * var(--small-gap));
  overflow: scroll;
  letter-spacing: 0.2rem;
  padding: var(--small-gap);
}

@media (min-width: 500px) {
  .page-container > .content {
    padding-left: 10rem;
    padding-right: 10rem;
    width: calc(100vw - 20rem);
  }
}

@media (min-width: 990px) {
  .page-container > .content {
    padding-left: 30%;
    padding-right: 30%;
    width: calc(40vw);
  }
}

.page-container > .content > .title {
  margin-top: var(--small-gap);
  margin-bottom: calc(2 * var(--small-gap));
  text-align: center;
}

.page-container > .content > .no-moves-yet {
  line-height: 2;
  font-size: 1.3rem;
  text-align: center;
}

.page-container > .content div > .section-title {
  text-align: center;
  font-weight: normal;
}

.page-container > .content > .move {
  background-color: var(--element-bg);
  border-radius: var(--round);
  margin-top: var(--small-gap);
  border: 0.3rem solid transparent;
}

.page-container > .content > .active {
  border: 0.3rem solid var(--text);
}

.page-container > .ctrl-bar {
  background-color: var(--ctrl-bar-bg);
  height: 6rem;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: var(--small-gap);
}

.page-container > .ctrl-bar button {
  background-color: var(--page-bg);
  /* border: 0.1rem solid var(--accent); */
  /* color: var(--accent); */
  color: inherit;
  border: none;
  border-radius: var(--round);
  font-size: 2rem;
  width: 5rem;
  height: 5rem;
  display: flex;
  justify-content: center;
  align-items: center;
}

.page-container > .ctrl-bar button div {
  display: flex;
  justify-content: center;
  align-items: center;
}

.page-container > .ctrl-bar button img {
  width: 40%;
  height: 40%;
}

/* .page-container > .ctrl-bar button:nth-child(1) {
  padding-left: 0;
  padding-top: 0;
}

.page-container > .ctrl-bar button:nth-child(2) {
  padding-top: 0;
}

.page-container > .ctrl-bar button:nth-child(3) {
  padding-right: 0;
  padding-top: 0;
} */

/* .page-container > .ctrl-bar > .main-btn {
  font-size: 2rem;
  width: 5rem;
  height: 5rem;
  padding-right: 0;
  padding-top: 0;
} */
</style>
