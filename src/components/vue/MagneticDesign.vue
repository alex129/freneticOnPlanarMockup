<script setup lang="ts">
import { ref, computed } from 'vue';
import AdvandedCustomization from "./AdvancedCustomization.vue";
import Loader from "./Loader.vue";

const traceWidth = ref('');
const cuThickness = ref('');
const totalTurns = ref('');
const numParallels = ref('');
const numLayers = ref('');
const distanceToEdge = ref('');

const generalResults = ref({
  Ptol: '',
  windingTemperature: '',
  numPCBLayers: '',
  pcbThickness: '',
});

const isLoading = ref(false);
const isAnalyzing = ref(false);
const showWarning = ref(false);
const showDraw = ref(false);

const isPrimaryFilled = computed(() => {
  return traceWidth.value !== '' && cuThickness.value !== '' && totalTurns.value !== '' && numParallels.value !== '' && numLayers.value !== '' && distanceToEdge.value !== '';
});

const runSimulation = () => {
  console.log({
    traceWidth: traceWidth.value,
    cuThickness: cuThickness.value,
    totalTurns: totalTurns.value,
    numParallels: numParallels.value,
    numLayers: numLayers.value,
    distanceToEdge: distanceToEdge.value,
  });
};

const showLoader = () => {
  showDraw.value = true;
  isLoading.value = true;
  setTimeout(() => {
    isLoading.value = false;
  }, 2000);
};

const analize = () => {
  isAnalyzing.value = true;
  showLoader();
  showWarning.value = true;
  generalResults.value = {
    Ptol: '24',
    windingTemperature: "105",
    numPCBLayers: '3',
    pcbThickness: '1.6',
  };
  setTimeout(() => {
    isAnalyzing.value = false;
  }, 3000);
};
</script>

<template>
  <div class="min-h-screen bg-white">
    <header class="flex items-center justify-between p-4 border-b">
      <div class="text-lg font-semibold">Frenetic</div>
      <div class="flex items-center space-x-4">
        <button class="bg-blue-600 text-white px-4 py-2 rounded-full">
          NEW DESIGN
        </button>
        <div class="flex items-center space-x-2">
          <span>Lucas</span>
          <span
            class="w-8 h-8 bg-gray-200 rounded-full flex items-center justify-center"
            >LN</span
          >
        </div>
      </div>
    </header>

    <main class="p-4 space-y-4">
      <div class="flex space-x-4">
        <button class="text-sm font-semibold py-2 px-4 border rounded-full">
          Waveform
        </button>
        <button class="text-sm font-semibold py-2 px-4 border rounded-full">
          List of Designs
        </button>
        <button class="text-sm font-semibold py-2 px-4 border rounded-full">
          Core Optimizer™
        </button>
        <button class="text-sm font-semibold py-2 px-4 border rounded-full">
          Core
        </button>
        <button
          class="text-sm font-semibold py-2 px-4 bg-blue-600 text-white rounded-full"
        >
          Winding
        </button>
        <button class="text-sm font-semibold py-2 px-4 border rounded-full">
          Mechanical
        </button>
        <button class="text-sm font-semibold py-2 px-4 border rounded-full">
          Datasheet
        </button>
      </div>

      <div class="flex items-start gap-2">
        <select class="py-2 px-4 border rounded">
          <option>Planar</option>
        </select>

        <select class="py-2 px-4 border rounded">
          <option>Interleaved</option>
          <option>No Interleaved</option>
        </select>
      </div>

      <div class="grid grid-cols-3 gap-4">
        <div class="bg-blue-50 p-4 rounded col-span-1">
          <div class="flex space-x-4 mb-4">
            <button class="text-sm font-semibold py-2 px-4 bg-white border rounded">
              Primary
            </button>
            <button class="text-sm font-semibold py-2 px-4 bg-gray-200 border rounded">
              Secondary
            </button>
          </div>
          <div class="space-y-2">
            <input
              type="text"
              placeholder="Trace Width"
              class="w-full py-2 px-4 border rounded"
              v-model="traceWidth"
            />
            <input
              type="text"
              placeholder="Cu Thickness"
              class="w-full py-2 px-4 border rounded"
              v-model="cuThickness"
            />
            <input
              type="text"
              placeholder="Total N° Turns"
              class="w-full py-2 px-4 border rounded"
              v-model="totalTurns"
            />
            <input
              type="text"
              placeholder="N° Parallels"
              class="w-full py-2 px-4 border rounded"
              v-model="numParallels"
            />
            <input
              type="text"
              placeholder="N° Layers"
              class="w-full py-2 px-4 border rounded"
              v-model="numLayers"
            />
            <input
              type="text"
              placeholder="Total Distance to edge"
              class="w-full py-2 px-4 border rounded"
              v-model="distanceToEdge"
            />
          </div>
        </div>

        <div class="bg-gray-100 p-4 rounded col-span-2">
          <div class="bg-gray-200 h-[250px] rounded mb-4 flex items-center justify-center">
            <div v-if="isLoading" class="flex items-center justify-center h-full">
              <Loader />
            </div>
            <img v-else-if="showDraw" class="h-full" src="/planar_img.png" alt="Planar img" />
          </div>

          <div class="flex items-center justify-center gap-2">
            <button 
              class="px-4 py-2 rounded"
              :class="isPrimaryFilled ? 'bg-blue-600 text-white' : 'bg-gray-400 text-gray-200 cursor-not-allowed'" 
              @click="showLoader" 
              :disabled="!isPrimaryFilled"
            >
              Preview
            </button>
            <AdvandedCustomization @save="showLoader" :disabled="!showDraw"/>
          </div>
          <div class="flex items-center mt-4">
            <input type="checkbox" class="form-checkbox" />
            <span class="ml-2">Top & bottom layer only for connection</span>
          </div>
        </div>
      </div>

      <div class="grid grid-cols-2 gap-4 w-full">
        <div class="bg-white p-4 border rounded">
          <h3 class="font-semibold mb-2">Primary Results</h3>
          <p>Current density (A/mm²) = 6.5</p>
          <p>Rdc (mOhms) = 10.5</p>
          <p>Pprimary (W) = 13</p>
        </div>
        <div class="bg-white p-4 border rounded">
          <div v-if="isAnalyzing" class="flex items-center justify-center h-full">
            <Loader />
          </div>
          <div v-else>
            <h3 class="font-semibold mb-2">General Results</h3>
            <p>Ptotal (W) = {{ generalResults.Ptol }}</p>
            <p>Winding temperature (ºC) = {{ generalResults.windingTemperature }}</p>
            <p>N° PCB Layers = {{ generalResults.numPCBLayers }}</p>
            <p>PCB Thickness (mm) = {{ generalResults.pcbThickness }}</p>
          </div>
        </div>
      </div>

      <div v-if="showWarning" class="w-full bg-yellow-100 p-2 flex items-center text-sm"> <img src="warning_icon.png" alt="warning icon" class="w-6" >Only conduction losses are considered in this first versión of the planar Transformers. We are not including fringing field and proximity losses. Please be careful with the design and try to oversize the winding.</div>
      <div class="flex justify-end space-x-4 mt-4">
        <button class="py-2 px-6 bg-gray-200 rounded">SUGGEST PCB</button>
        <button class="py-2 px-6 rounded" :class="isPrimaryFilled ? 'bg-blue-600 text-white' : 'bg-gray-400 text-gray-200 cursor-not-allowed'"  :disabled="!isPrimaryFilled" @click="analize">RUN</button>
      </div>
    </main>
  </div>
</template>

<style scoped>
@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
.loader {
  border-width: 4px;
}
</style>
