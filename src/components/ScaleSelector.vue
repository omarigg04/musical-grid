<template>
    <div>
      <select v-model="localSelectedScale" @change="handleScaleChange">
        <option v-for="scale in scales" :key="scale" :value="scale">{{ scale }}</option>
      </select>
    </div>
  </template>
  
  <script>
  export default {
    props: {
      selectedScale: {
        type: String,
        required: true,
      },
      scales: {
        type: Array,
        required: true,
      },
    },
    data() {
      return {
        localSelectedScale: this.selectedScale, // Usamos una variable local para manejar el valor
      };
    },
    watch: {
      selectedScale(newVal) {
        this.localSelectedScale = newVal;
      }
    },
    methods: {
      handleScaleChange() {
        console.log('Emitiendo evento con el nuevo valor:', this.localSelectedScale);
        // Emitimos el evento para que el componente padre maneje la actualización
        this.$emit('update:selectedScale', this.localSelectedScale);
      },
    },
  };
  </script>
  