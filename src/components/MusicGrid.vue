<template>
  <div class="music-grid">
    <h1>Music Sequencer</h1>
    <ScaleSelector
      :selectedScale="selectedScale"
      :scales="scales"
      @update:selectedScale="updateSelectedScale"
    />

    <!-- Control de velocidad -->
    <div class="controls">
      <label>
        Secuenciador:
        <input type="checkbox" v-model="isSequencerActive" @change="toggleSequencer" />
      </label>
      <div class="speed-control">
        <label>Velocidad:</label>
        <input
          type="range"
          v-model="sequencerSpeed"
          min="100"
          max="1000"
          step="50"
          @input="updateSequencerSpeed"
        />
        <span>{{ sequencerSpeed }} ms</span>
      </div>
    </div>

    <!-- Grid de celdas -->
    <div class="grid">
      <div
        v-for="(cell, index) in grid"
        :key="index"
        :class="['cell', { active: cell.active }]"
        :style="{ backgroundColor: cell.color }"
        @mousedown="activateCell(index)"
      >
        <!-- Selector de notas para cada celda -->
        <select v-model="cell.note" @change="updateCellNote(index)" class="note-selector">
          <option v-for="note in availableNotes" :key="note" :value="note">{{ note }}</option>
        </select>
      </div>
    </div>
  </div>
</template>

  
<script>
import ScaleSelector from './ScaleSelector.vue'; // Asegúrate de que la ruta sea correcta
import { SCALES } from '../constants/scales.js';
import { NOTES } from '../constants/notes.js';
export default {

  components: {
    ScaleSelector,
  },
  data() {
    return {
      selectedScale: 'C Major',
      scales: SCALES,
      notes: NOTES,
      grid: [],
      audioBuffers: {}, // Almacenará los sonidos precargados
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
      await this.preloadSounds(this.selectedScale);
    },
    async preloadSounds(scale) {
      this.audioBuffers = {};
      for (const note of this.notes[scale]) {
        const audio = new Audio(`https://github.com/gleitz/midi-js-soundfonts/blob/gh-pages/FatBoy/flute-mp3/${note}.mp3?raw=true`);
        await new Promise((resolve) => {
          audio.addEventListener('canplaythrough', resolve, { once: true });
          audio.load();
        });
        this.audioBuffers[note] = audio;
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
      if (this.audioBuffers[note]) {
        const audio = this.audioBuffers[note].cloneNode(true);
        audio.play().catch(error => {
          console.error('Error al reproducir el sonido:', error);
        });
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
        if (this.currentSequencerIndex < this.grid.length) {
          // Activa la celda y reproduce el sonido
          this.activateCell(this.currentSequencerIndex);
          this.currentSequencerIndex++;
        } else {
          this.currentSequencerIndex = 0; // Vuelve al inicio
        }
      }, this.sequencerSpeed); // Usa la velocidad configurada
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
  },
  beforeUnmount() {
    this.stopSequencer(); // Asegúrate de detener el secuenciador al desmontar el componente
  },
};
</script>
  
<style scoped>
.music-grid {
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
</style>