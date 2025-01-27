<template>
    <div class="music-grid">
      <select v-model="selectedScale" @change="updateGrid">
        <option v-for="scale in scales" :key="scale" :value="scale">{{ scale }}</option>
      </select>
      <div class="grid">
        <div
          v-for="(cell, index) in grid"
          :key="index"
          :class="['cell', { active: cell.active }]"
          :style="{ backgroundColor: cell.color }"
          @mousedown="activateCell(index)"
        >
          {{ cell.note }}
        </div>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        selectedScale: 'C Major',
        scales: ['C Major', 'G Major', 'D Major'],
        grid: [],
        notes: {
          'C Major': ['C4', 'D4', 'E4', 'F4', 'G4', 'A4', 'B4', 'C5'],
          'G Major': ['G4', 'A4', 'B4', 'C5', 'D5', 'E5', 'F#5', 'G5'],
          'D Major': ['D4', 'E4', 'F#4', 'G4', 'A4', 'B4', 'C#5', 'D5'],
        },
        audioBuffers: {}, // Almacenará los sonidos precargados
      };
    },
    mounted() {
      this.updateGrid();
    },
    methods: {
      async updateGrid() {
        // Actualiza el grid con las notas de la escala seleccionada
        this.grid = this.notes[this.selectedScale].map(note => ({
          note,
          active: false,
          color: '#fff',
        }));
  
        // Precarga los sonidos de la escala seleccionada
        await this.preloadSounds(this.selectedScale);
      },
      async preloadSounds(scale) {
        // Limpia los buffers anteriores
        this.audioBuffers = {};
  
        // Precarga cada sonido de la escala
        for (const note of this.notes[scale]) {
          const audio = new Audio(`https://github.com/gleitz/midi-js-soundfonts/blob/gh-pages/FatBoy/acoustic_guitar_nylon-mp3/${note}.mp3?raw=true`);
          await new Promise((resolve) => {
            audio.addEventListener('canplaythrough', resolve, { once: true });
            audio.load(); // Forza la carga del archivo de audio
          });
          this.audioBuffers[note] = audio; // Almacena el audio en memoria
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
          const audio = this.audioBuffers[note].cloneNode(true); // Clona el audio para permitir múltiples reproducciones
          audio.play().catch(error => {
            console.error('Error al reproducir el sonido:', error);
          });
        } else {
          console.error('Sonido no precargado:', note);
        }
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
  };
  </script>
  
  <style scoped>
  .music-grid {
    text-align: center;
  }
  
  .grid {
    display: grid;
    grid-template-columns: repeat(4, 100px);
    grid-gap: 10px;
    margin-top: 20px;
  }
  
  .cell {
    width: 100px;
    height: 100px;
    border: 1px solid #000;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    transition: background-color 0.5s ease;
  }
  
  .cell.active {
    background-color: #ffcc00;
  }
  </style>