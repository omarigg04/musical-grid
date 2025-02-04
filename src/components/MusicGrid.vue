<template>
  <!-- Eliminamos el InstrumentSelector ya que no es necesario para el synth -->
  <h1>Music Sequencer</h1>
  <div class="music-grid">
    <div class="piano">
      <div v-for="note in pianoKeys" :key="note" :id="note"
        :class="['key', { black: isBlackKey(note), white: !isBlackKey(note), highlighted: isNoteInScale(note) }]">
      </div>
    </div>

    <div class="grid">
      <div v-for="(cell, index) in grid" :key="index" :class="['cell', { active: cell.active }]"
        :style="{ backgroundColor: cell.color }" @mousedown="activateCell(index)">
        <select v-model="cell.note" @change="updateCellNote(index)" class="note-selector">
          <option v-for="note in availableNotes" :key="note" :value="note">{{ note }}</option>
        </select>
      </div>
    </div>

    <div class="controls">
      <ScaleSelector :selectedScale="selectedScale" :scales="scales" @update:selectedScale="updateSelectedScale" />

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
import ScaleSelector from './ScaleSelector.vue';
import { SCALES } from '../constants/scales.js';
import { NOTES } from '../constants/notes.js';
import * as Tone from 'tone';

document.addEventListener("click", async () => {
  // Verifica si el AudioContext está suspendido y reanúdalo
  if (Tone.context.state === "suspended") {
    await Tone.start();
    console.log("Tone.js y AudioContext iniciados correctamente.");
  }

  // Ahora puedes crear nodos de audio sin problemas
  // const osc = new Tone.Oscillator(440, "sine").toDestination();
  // osc.start();
});

export default {
  components: {
    ScaleSelector
  },
  data() {
    return {
      pianoKeys: ['C', 'Db', 'D', 'Eb', 'E', 'F', 'Gb', 'G', 'Ab', 'A', 'Bb', 'B', 'C'],
      decayValue: 0.5,
      selectedScale: 'C Major',
      scales: SCALES,
      notes: NOTES,
      grid: [],
      isSequencerActive: false,
      sequencerInterval: null,
      currentSequencerIndex: 0,
      sequencerSpeed: 500,
      availableNotes: ['C4', 'Db4', 'D4', 'Eb4', 'E4', 'F4', 'Gb4', 'G4', 'Ab4', 'A4', 'Bb4', 'B4'],
      synth: null,  // Cambiamos sampler por synth
      synthSettings: {
        oscillator: {
          type: 'sine' // Tipo de onda inicial
        },
        envelope: {
          attack: 0.1,
          decay: 0.5,
          sustain: 0.3,
          release: 0.2
        }
      }
    };
  },
  mounted() {
    this.updateGrid();
    // this.initializeSynth();
    Tone.start().then(() => {
      console.log('Tone.js ha sido inicializado');
      this.synth = new Tone.Synth().toDestination();
    }).catch((error) => {
      console.error('Error al inicializar Tone.js:', error);
    });

  },
  methods: {

    // Inicializa el sampler con los sonidos

    initializeSynth() {
      if (this.synth) this.synth.dispose(); // Limpiar instancia previa
      this.synth = new Tone.Synth(this.synthSettings).toDestination();
      this.part = new Tone.Part((time, value) => {
        this.synth.triggerAttackRelease(value.note, "8n", time);
      }, []).start(0);
    },

    async updateGrid() {
      this.grid = this.notes[this.selectedScale].map(note => ({
        note,
        active: false,
        color: '#fff',
      }));
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
      //create a synth and connect it to the main output (your speakers)
      const synth = new Tone.Synth().toDestination();

      //play a middle 'C' for the duration of an 8th note
      synth.triggerAttackRelease(note, "8n");


    },

    //     playSound(note) {
    //       const synth = new Tone.Synth().toDestination();

    //     // use an array of objects as long as the object has a "time" attribute
    //     new Tone.Part(
    //     (time = 0.5) => {
    //     // the value is an object which contains both the note and the velocity
    //     synth.triggerAttackRelease(note, '8n', time);

    //     Tone.Draw.schedule(function () {
    //       //this callback is invoked from a requestAnimationFrame
    //       //and will be invoked close to AudioContext time
    //       document.getElementById('app').innerHTML += ' ' + note;
    //     }, time);
    //   },
    //   [
    //     { time: '0.5', note: note },

    //   ]
    // ).start(0);
    // Tone.Transport.start();
    //     },


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
      this.selectedScale = newScale;
      this.updateGrid();
      // this.initializeSampler();  // Recarga el sampler al cambiar la escala
    },

    updateSelectedInstrument(newInstrument) {
      this.selectedInstrument = newInstrument;
      this.initializeSampler();  // Recarga el sampler al cambiar el instrumento
    },

    updateSequencerSpeed() {
      if (this.isSequencerActive) {
        this.stopSequencer();
        this.startSequencer();
      }
    },

    updateCellNote(index) {
      const cell = this.grid[index];
      this.playSound(cell.note);
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
      // Tone.js aplica decay con una envolvente (envelope)
      const envelope = new Tone.AmplitudeEnvelope({
        attack: 0.1,
        decay: this.decayValue,
        sustain: 0,
        release: 0.2,
      }).toDestination();

      envelope.triggerAttackRelease(note, this.decayValue);
    },

    updateDecay() {
      // Actualizamos el decay del envelope del synth
      if (this.synth) {
        this.synth.envelope.decay = this.decayValue;
      }
    },

    isNoteInScale(note) {
      const scaleNotes = this.notes[this.selectedScale].map(n => n.slice(0, -1));
      const inScale = scaleNotes.includes(note);
      return inScale;
    },

    isBlackKey(note) {
      return ['Db', 'Eb', 'Gb', 'Ab', 'Bb'].includes(note);
    },
  },
  beforeUnmount() {
    this.stopSequencer();
    if (this.synth) {
      this.synth.dispose();
    }
  }
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