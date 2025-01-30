<template>
    <div>
      <select v-model="localSelectedInstrument" @change="handleInstrumentChange">
        <option v-for="instrument in instruments" :key="instrument" :value="instrument">{{ instrument }}</option>
      </select>
    </div>
  </template>
  
  <script>
  export default {
    props: {
      selectedInstrument: {
        type: String,
        required: true,
      },
      instruments: {
        type: Array,
        required: true,
      },
    },
    data() {
      return {
        localSelectedInstrument: this.selectedInstrument, // Usamos una variable local para manejar el valor
      };
    },
    watch: {
      selectedInstrument(newVal) {
        this.localSelectedInstrument = newVal;
      }
    },
    methods: {
      handleInstrumentChange() {
        console.log('Emitiendo evento con el nuevo valor:', this.localSelectedInstrument);
        // Emitimos el evento para que el componente padre maneje la actualización
        this.$emit('update:selectedInstrument', this.localSelectedInstrument);
      },
    },
  };
  </script>
  