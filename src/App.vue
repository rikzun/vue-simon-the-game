<template>
  <div>
    <transition name="popup-opacity-l" @afterLeave="show">
      <div v-show="!start" class="popup">
        <h1>Simon the Game</h1>
        <label>Легкий<input type="radio" name="difficty" value="0" v-model.number="difficty"></label>
        <label>Средний<input type="radio" name="difficty" value="1" v-model.number="difficty"></label>
        <label>Сложный<input type="radio" name="difficty" value="2" v-model.number="difficty"></label>
        <button @click="start = true">Старт</button>
      </div>
    </transition>
    <transition name="popup-opacity-e">
      <div v-show="end" class="popup">
        <h1>Вы проиграли ;c</h1>
        <button @click="restart">Ещё раз</button>
      </div>
    </transition>
    <div class="container">
      <div class="game">
        <div class="cell" v-for="cell of cells" 
          @click="click"
          :class="{'cell--active': blacklight == cell}"
          :key="cell"
          :id="cell"
        ></div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { wait, randint } from './utils'

export default Vue.extend({
  name: 'App',
  components: {},
  data() {
    return {
      start: false,
      end: false,
      control: false,
      difficty: 0,
      index: 0,
      cells: ['green', 'red', 'yellow', 'blue'], 
      frequencies: [329.63, 261.63, 220, 164.81],
      array: [] as string[],
      blacklight: '',
      audioCtx: new window.AudioContext()
    }
  },
  methods: {
    async show() {
      const timeout = [1500, 1000, 400][this.difficty]
      await wait(1000)

      for (const oldCell of this.array) {
        this.choose(oldCell)
        await wait(timeout)
      }

      const newCell = this.cells[randint(0, 3)]
      this.choose(newCell)
      this.array.push(newCell)

      this.control = true
    },
    click(event: MouseEvent) {
      const cell = (event.target as HTMLTextAreaElement).id
      this.choose(cell)

      if (!this.control) return
      if (this.array[this.index] == cell) {
        this.index++

        if (this.array.length == this.index) {
          this.index = 0
          this.control = false
          this.show()
        }
      } else {
        this.end = true
      }
    },
    async choose(id: string) {
      if (this.audioCtx) {
        const gain = this.audioCtx.createGain()
        gain.connect(this.audioCtx.destination)
        gain.gain.value = .2

        const oscillator = this.audioCtx.createOscillator()
        oscillator.type = 'square'
        oscillator.frequency.value = this.frequencies[this.cells.indexOf(id)]
        oscillator.connect(gain)

        oscillator.start()
        oscillator.stop(this.audioCtx.currentTime + .2)
      }
      
      this.blacklight = id

      await wait(200)
      this.blacklight = ''
    },
    restart() {
      this.start = false
      this.end = false
      this.index = 0
      this.control = false
      this.array = []
    }
  }
});
</script>

<style scope>
@font-face {
  font-family: 'SF Pro Display';
  src: local('SF Pro Display Regular'), local('SFProDisplay-Regular'), url('assets/fonts/SFProDisplay-Regular.woff2') format('woff2'), url('assets/fonts/SFProDisplay-Regular.woff') format('woff'), url('assets/fonts/SFProDisplay-Regular.ttf') format('truetype');
  font-weight: normal;
  font-style: normal;
}

@font-face {
  font-family: 'SF Pro Display';
  src: local('SF Pro Display Light'), local('SFProDisplay-Light'), url('assets/fonts/SFProDisplay-Light.woff2') format('woff2'), url('assets/fonts/SFProDisplay-Light.woff') format('woff'), url('assets/fonts/SFProDisplay-Light.ttf') format('truetype');
  font-weight: 200;
  font-style: normal;
}

body {
  margin: 0;
  color: white;
  font-family: SF Pro Display, Regular !important;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  -khtml-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

.container {
  display: flex;
  position: absolute;
  height: 100%;
  width: 100%;
  justify-content: center;
  align-items: center;
  background-color: black;
}

.game {
  display: grid;
  height: 300px;
  width: 300px;
  border-radius: 20px;
  overflow: hidden;
  grid-template-columns: repeat(2, 1fr);
  grid-gap: 5px;
}

.cell {
  opacity: .6;
  transition: opacity 100ms linear;
}

.cell--active {
  opacity: 1;
  transition: opacity 0ms linear;
}

.popup {
  display: flex;
  flex-direction: column;
  position: fixed;
  z-index: 1;
  width: 100%;
  height: 100%;
  justify-content: center;
  align-items: center;
  backdrop-filter: blur(100px);
}

.popup-opacity-l-leave-active {
  transition: opacity 1000ms linear;
}

.popup-opacity-l-leave-to {
  opacity: 0;
}

.popup-opacity-e-enter {
  opacity: 0;
}

.popup-opacity-e-enter-active {
  transition: opacity 1000ms linear;
}

.popup-opacity-e-enter-to {
  opacity: 1;
}

button {
  padding: 0;
  border: none;
  font-size: 28px;
  margin-top: 20px;
  padding-inline: 15px;
  padding-block: 5px;
  background-color: rgba(255, 255, 255, .3);
  border: 2px solid rgba(255, 255, 255, .5);
  color: white;
  border-radius: 5px;
}

button:active {
  background-color: rgba(255, 255, 255, .4);
}

button:focus {
  outline: none;
}

h1 {
  font-weight: 200;
}

#green {
  background-color: #00FF00;
}
#red {
  background-color: #FF0000;
}
#yellow {
  background-color: #FFFF00;
}
#blue {
  background-color: #0000FF;
}
</style>