<template>
  <h1>Music Sequencer</h1>
  <div class="music-grid">

    <!-- <div class="piano">
      <div id="C" class="key white"></div>
      <div id="Db" class="key black"></div>
      <div id="D" class="key white"></div>
      <div id="Eb" class="key black"></div>
      <div id="E" class="key white"></div>
      <div id="F" class="key white"></div>
      <div id="Gb" class="key black"></div>
      <div id="G" class="key white"></div>
      <div id="Ab" class="key black"></div>
      <div id="A" class="key white"></div>
      <div id="Bb" class="key black"></div>
      <div id="B" class="key white"></div>
      <div id="C" class="key white"></div>
    </div> -->

    <!-- PIANO -->
    <div class="piano">
      <div v-for="note in pianoKeys" :key="note"
           :id="note"
           :class="['key', { black: isBlackKey(note), white: !isBlackKey(note), highlighted: isNoteInScale(note) }]">
      </div>
    </div>

    <!-- Grid de celdas -->
    <div class="grid">
      <div v-for="(cell, index) in grid" :key="index" :class="['cell', { active: cell.active }]"
        :style="{ backgroundColor: cell.color }" @mousedown="activateCell(index)">
        <!-- Selector de notas para cada celda -->
        <select v-model="cell.note" @change="updateCellNote(index)" class="note-selector">
          <option v-for="note in availableNotes" :key="note" :value="note">{{ note }}</option>
        </select>
      </div>
    </div>


    <!-- --------------       CONTROLS  ----------------    -->
    <div class="controls">
      <ScaleSelector :selectedScale="selectedScale" :scales="scales" @update:selectedScale="updateSelectedScale" />
      <InstrumentSelector :selectedInstrument="selectedInstrument" :instruments="instruments"
        @update:selectedInstrument="updateSelectedInstrument" />

      <!-- Control de velocidad -->
      <div class="controls">
        <label>
          Secuenciador:
          <input type="checkbox" v-model="isSequencerActive" @change="toggleSequencer" />
        </label>


        <div class="speed-control">
          <label>Velocidad:</label>
          <input type="range" v-model="sequencerSpeed" min="100" max="1000" step="50" @input="updateSequencerSpeed" />
          <span>{{ sequencerSpeed }} ms</span>
        </div>
        <!-- Slider para el control de decay -->
        <div class="speed-control">
          <label>Decay</label>
          <input type="range" v-model="decayValue" min="0" max="2" step="0.05" @input="updateDecay" />
          <span>{{ decayValue }}s</span>
        </div>

      </div>

    </div>
  </div>
</template>

<script>
import ScaleSelector from './ScaleSelector.vue'; // Asegúrate de que la ruta sea correcta
import { SCALES } from '../constants/scales.js';
import { NOTES } from '../constants/notes.js';
import { INSTRUMENTS } from '@/constants/instruments';
import { Howl } from 'howler'; // Importa Howler.js
import InstrumentSelector from './InstrumentSelector.vue';

export default {
  components: {
    ScaleSelector,
    InstrumentSelector
  },
  data() {
    return {
      // sounds: {},      // Los sonidos cargados


      pianoKeys: ['C', 'Db', 'D', 'Eb', 'E', 'F', 'Gb', 'G', 'Ab', 'A', 'Bb', 'B', 'C'], // Notas del piano
      decayValue: 1,   // Valor por defecto del decay
      selectedScale: 'C Major',
      selectedInstrument: 'fiddle-mp3',
      scales: SCALES,
      notes: NOTES,
      instruments: INSTRUMENTS,
      grid: [],
      sounds: {}, // Almacenará los sonidos precargados con Howler.js
      isSequencerActive: false, // Estado del secuenciador
      sequencerInterval: null, // Intervalo del secuenciador
      currentSequencerIndex: 0, // Índice de la nota actual en la secuencia
      sequencerSpeed: 500, // Velocidad inicial del secuenciador (en ms)
      // C0 -> A7
      availableNotes: [
        'C0', 'Db0', 'D0', 'Eb0', 'E0', 'F0', 'Gb0', 'G0', 'Ab0', 'A0', 'Bb0', 'B0',
        'C1', 'Db1', 'D1', 'Eb1', 'E1', 'F1', 'Gb1', 'G1', 'Ab1', 'A1', 'Bb1', 'B1',
        'C2', 'Db2', 'D2', 'Eb2', 'E2', 'F2', 'Gb2', 'G2', 'Ab2', 'A2', 'Bb2', 'B2',
        'C3', 'Db3', 'D3', 'Eb3', 'E3', 'F3', 'Gb3', 'G3', 'Ab3', 'A3', 'Bb3', 'B3',
        'C4', 'Db4', 'D4', 'Eb4', 'E4', 'F4', 'Gb4', 'G4', 'Ab4', 'A4', 'Bb4', 'B4',
        'C5', 'Db5', 'D5', 'Eb5', 'E5', 'F5', 'Gb5', 'G5', 'Ab5', 'A5', 'Bb5', 'B5',
        'C6', 'Db6', 'D6', 'Eb6', 'E6', 'F6', 'Gb6', 'G6', 'Ab6', 'A6', 'Bb6', 'B6',
        'C7', 'Db7', 'D7', 'Eb7', 'E7', 'F7', 'Gb7', 'G7', 'Ab7', 'A7', 'Bb7', 'B7',
      ], // Notas disponibles
    };
  },
  mounted() {
    this.updateGrid();
  },
  methods: {

    async updateGrid() {
      this.grid = this.notes[this.selectedScale].map(note => ({
        note,
        active: false,
        color: '#fff',
      }));
      await this.preloadSounds(this.selectedScale, this.selectedInstrument);
    },


    async updatePiano() {
      this.grid = this.notes[this.selectedScale].map(note => ({
        note,
        active: false,
        color: '#fff',
      }));
      await this.preloadSounds(this.selectedScale, this.selectedInstrument);
    },


    async preloadSounds(scale) {
      this.sounds = {};
      for (const note of this.notes[scale]) {
        const sound = new Howl({
          src: [`https://github.com/gleitz/midi-js-soundfonts/blob/gh-pages/FatBoy/${this.selectedInstrument}/${note}.mp3?raw=true`],
          volume: 1,
          onload: () => console.log(`${note} loaded`),
          onloaderror: (id, error) => console.error(`Error loading ${note}: ${error}`)
        });
        this.sounds[note] = sound;
      }
    },

    activateCell(index) {
      const cell = this.grid[index];
      cell.active = true;
      cell.color = this.getRandomColor();
      this.playSound(cell.note);

      setTimeout(() => {
        cell.active = false;
        cell.color = '#fff';
      }, 500);
    },
    playSound(note) {
      if (this.sounds[note]) {
        this.sounds[note].play();
        this.applyDecay(note);  // Aplica el decay después de reproducir el sonido
      } else {
        console.error('Sonido no precargado:', note);
      }
    },
    toggleSequencer() {
      if (this.isSequencerActive) {
        this.startSequencer();
      } else {
        this.stopSequencer();
      }
    },
    startSequencer() {
      this.currentSequencerIndex = 0; // Reinicia el índice
      this.sequencerInterval = setInterval(() => {
        // Activa la celda y reproduce el sonido
        this.activateCell(this.currentSequencerIndex);
        this.currentSequencerIndex = (this.currentSequencerIndex + 1) % this.grid.length; // Vuelve al inicio sin pausa
      }, this.sequencerSpeed); // Usa la velocidad configuradaa
    },
    stopSequencer() {
      if (this.sequencerInterval) {
        clearInterval(this.sequencerInterval); // Detiene el intervalo
        this.sequencerInterval = null;
      }
    },
    updateSelectedScale(newScale) {
      console.log('Nuevo selectedScale recibido:', newScale);
      this.selectedScale = newScale; // Aquí actualizamos el valor de selectedScale
      this.updateGrid();
    },
    updateSelectedInstrument(newInstrument) {
      console.log('Nuevo selectedInstrument recibido:', newInstrument);
      this.selectedInstrument = newInstrument; // Aquí actualizamos el valor de selectedScale
      this.updateGrid();
    },
    updateSequencerSpeed() {
      if (this.isSequencerActive) {
        this.stopSequencer();
        this.startSequencer();
      }
    },
    updateCellNote(index) {
      const cell = this.grid[index];
      this.playSound(cell.note); // Reproduce la nueva nota seleccionada
    },
    getRandomColor() {
      const letters = '0123456789ABCDEF';
      let color = '#';
      for (let i = 0; i < 6; i++) {
        color += letters[Math.floor(Math.random() * 16)];
      }
      return color;
    },
    applyDecay(note) {
      const sound = this.sounds[note];
      if (sound) {
        // Puedes ajustar el fade-out de acuerdo con el valor del decay
        const fadeDuration = this.decayValue * 1000; // Convierte el decay de segundos a milisegundos
        sound.fade(1, 0, fadeDuration);  // Fade de 1 a 0 durante 'fadeDuration' ms
      }
    },
    updateDecay() {
      // Este método se puede usar si quieres aplicar el decay en tiempo real al sonido activo
      console.log(`Nuevo valor de decay: ${this.decayValue}`);
    },
    //Verifica si la nota está en la escala actual para pintarla de rojo.
    isNoteInScale(note) {
  const scaleNotes = this.notes[this.selectedScale].map(n => n.slice(0, -1));
  const inScale = scaleNotes.includes(note);
  console.log(`Note: ${note}, In Scale: ${inScale}`);
  return inScale;
},
    //Asigna las teclas negras
    isBlackKey(note) {
      return ['Db', 'Eb', 'Gb', 'Ab', 'Bb'].includes(note);
    },

  },
  beforeUnmount() {
    this.stopSequencer(); // Asegúrate de detener el secuenciador al desmontar el componente
  },
};
</script>

<style scoped>
.music-grid {
  display: flex;
  justify-content: space-between;
  /* Espacio entre los elementos */
  align-items: center;
  /* Alineación vertical al centro */
  text-align: center;
  background-color: #f0f0f0;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

h1 {
  font-family: 'Arial', sans-serif;
  color: #333;
}

.scale-selector {
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #ccc;
  margin-bottom: 20px;
  font-size: 16px;
}

.controls {
  margin-top: 20px;
  margin-bottom: 20px;
}

.speed-control {
  margin-top: 10px;
}

.speed-control label {
  font-size: 16px;
  margin-right: 10px;
}

.speed-control input[type="range"] {
  width: 200px;
}

.grid {
  justify-content: center;
  display: grid;
  grid-template-columns: repeat(4, 100px);
  grid-gap: 15px;
  margin-top: 20px;
}

.cell {
  width: 100px;
  height: 100px;
  border: 1px solid #ccc;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.2s ease;
  font-family: 'Arial', sans-serif;
  font-size: 18px;
  color: #333;
  position: relative;
}

.cell:hover {
  transform: scale(1.05);
}

.cell.active {
  background-color: #ffcc00;
}

.note-selector {
  position: absolute;
  bottom: 5px;
  font-size: 14px;
  padding: 5px;
  border-radius: 5px;
  border: 1px solid #ccc;
  background-color: white;
}

.piano {
  width: 420px;
  height: 150px;
  display: flex;
  position: relative;

  padding: 5px;
}

.key {
  border: 1px solid black;
  box-sizing: border-box;
}

.key.white {
  width: 40px;
  height: 140px;
  background-color: white;
  position: relative;
}

.key.black {
  width: 25px;
  height: 90px;
  background-color: black;
  position: absolute;
  margin-left: -12.5px;
  z-index: 2;
}

/* Corrección en la posición de las teclas negras */
.key.black:nth-of-type(2) {
  left: 55px;
}

.key.black:nth-of-type(4) {
  left: 95px;
}

.key.black:nth-of-type(7) {
  left: 175px;
}

.key.black:nth-of-type(9) {
  left: 215px;
}

.key.black:nth-of-type(11) {
  left: 255px;
}

.highlighted {
  background-color: rgb(190, 104, 104) !important;
}
</style>