<template>
  <div class="flex items-center justify-center gap-2">
    <button class="px-4 py-2 bg-blue-600 text-white rounded">Preview</button>
    <!-- Botón para abrir el modal -->
    <button @click="openModal" class="px-4 py-2 bg-blue-600 text-white rounded">
      Advanced Customization
    </button>

    <!-- Modal -->
    <div
      v-if="isModalOpen"
      class="fixed inset-0 flex items-center justify-center bg-gray-800 bg-opacity-75"
    >
      <div
        class="bg-white rounded-lg shadow-lg p-6 w-full max-w-lg max-h-[80vh] overflow-y-auto"
      >
        <h2 class="text-xl font-semibold mb-4">Advanced Customization</h2>
        <form @submit.prevent="submitForm">
          <!-- Parámetros de la Ventana -->
          <div class="mb-4">
            <label
              for="window_height"
              class="block text-sm font-medium text-gray-700"
              >Window Height (mm)</label
            >
            <input
              v-model="formData.window_height"
              type="number"
              step="0.001"
              class="mt-1 block w-full rounded-md border-gray-300 shadow-sm"
            />
          </div>
          <div class="mb-4">
            <label
              for="window_width"
              class="block text-sm font-medium text-gray-700"
              >Window Width (mm)</label
            >
            <input
              v-model="formData.window_width"
              type="number"
              step="0.001"
              class="mt-1 block w-full rounded-md border-gray-300 shadow-sm"
            />
          </div>

          <!-- Selector de Layers -->
          <div class="mb-4">
            <label
              for="layer_selector"
              class="block text-sm font-medium text-gray-700"
              >Select Layer</label
            >
            <select
              v-model="selectedLayerIndex"
              class="mt-1 block w-full rounded-md border-gray-300 shadow-sm"
            >
              <option
                v-for="(layer, index) in formData.layers"
                :key="index"
                :value="index"
              >
                Layer {{ index + 1 }}
              </option>
            </select>
          </div>

          <!-- Sección de Edición de Layers Dinámicos -->
          <div v-if="selectedLayer" class="mb-4 border-t pt-4">
            <h3 class="text-lg font-semibold mb-2">
              Layer {{ selectedLayerIndex + 1 }}
            </h3>

            <div class="mb-4">
              <label class="block text-sm font-medium text-gray-700"
                >Thickness (mm)</label
              >
              <input
                v-model="thicknessValue"
                type="number"
                step="0.001"
                class="mt-1 block w-full rounded-md border-gray-300 shadow-sm"
              />
            </div>

            <div class="mb-4">
              <label class="block text-sm font-medium text-gray-700"
                >X0 (mm)</label
              >
              <input
                v-model="selectedLayer.x0"
                type="number"
                step="0.001"
                class="mt-1 block w-full rounded-md border-gray-300 shadow-sm"
              />
            </div>

            <!-- Physical Turns -->
            <div
              v-for="(turn, turnIndex) in selectedLayer.physical_turns"
              :key="turnIndex"
              class="mb-4"
            >
              <h4 class="text-sm font-medium text-gray-700">
                Physical Turn {{ turnIndex + 1 }}
              </h4>

              <!-- Condición para mostrar solo winding_number "void" -->
              <div v-if="turn.winding_number === 'void'">
                <label class="block text-sm font-medium text-gray-700"
                  >Winding Number</label
                >
                <p class="mt-1 italic text-gray-500">void</p>
              </div>

              <!-- Mostrar parámetros solo si winding_number no es "void" -->
              <div v-else>
                <label class="block text-sm font-medium text-gray-700"
                  >Width (mm)</label
                >
                <input
                  v-model="turn.width"
                  type="number"
                  step="0.001"
                  class="mt-1 block w-full rounded-md border-gray-300 shadow-sm"
                />
                <label class="block text-sm font-medium text-gray-700"
                  >Winding Number</label
                >
                <input
                  v-model="turn.winding_number"
                  type="text"
                  class="mt-1 block w-full rounded-md border-gray-300 shadow-sm"
                />
                <label class="block text-sm font-medium text-gray-700"
                  >Turn Number</label
                >
                <input
                  v-model="turn.turn_number"
                  type="number"
                  class="mt-1 block w-full rounded-md border-gray-300 shadow-sm"
                />
                <label class="block text-sm font-medium text-gray-700"
                  >Parallel Number</label
                >
                <input
                  v-model="turn.parallel_number"
                  type="number"
                  class="mt-1 block w-full rounded-md border-gray-300 shadow-sm"
                />
              </div>
            </div>
          </div>

          <!-- Botones de Acción -->
          <div class="flex justify-end">
            <button
              type="submit"
              class="px-4 py-2 bg-green-600 text-white rounded mr-2"
            >
              Save
            </button>
            <button
              @click="closeModal"
              type="button"
              class="px-4 py-2 bg-red-600 text-white rounded"
            >
              Cancel
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";

// Estado del modal
const isModalOpen = ref(false);

// Datos del formulario basados en el YAML convertidos a milímetros
const formData = ref({
  window_height: 2.0e-3 * 1000,
  window_width: 9.0e-3 * 1000,
  n_gaps: 2,
  gap_size: 0.1e-3 * 1000,
  distance_leg_to_pcb: 0.25e-3 * 1000,
  layers_x0_default: 0.5e-3 * 1000,
  vertical_shift: null,
  layers_thickness_default: 0.21e-3 * 1000,
  distance_between_layers_default: 0.26e-3 * 1000,
  layers: [
    {
      thickness: null,
      x0: null,
      physical_turns: [
        {
          width: 3.67e-3 * 1000,
          winding_number: 1,
          turn_number: 1,
          parallel_number: 1,
        },
        { width: 0.25e-3 * 1000, winding_number: "void" },
        {
          width: 3.67e-3 * 1000,
          winding_number: 1,
          turn_number: 2,
          parallel_number: 1,
        },
      ],
    },
    {
      thickness: 0.15e-3 * 1000,
      physical_turns: [
        {
          width: 2.92e-3 * 1000,
          winding_number: 1,
          turn_number: 1,
          parallel_number: 2,
        },
        { width: 0.25e-3 * 1000, winding_number: "void" },
        {
          width: 3.42e-3 * 1000,
          winding_number: 1,
          turn_number: 2,
          parallel_number: 2,
        },
      ],
    },
    {
      x0: 0.7e-3 * 1000,
      physical_turns: [
        {
          width: 3.67e-3 * 1000,
          winding_number: 2,
          turn_number: 1,
          parallel_number: 1,
        },
        { width: 0.25e-3 * 1000, winding_number: "void" },
        {
          width: 3.47e-3 * 1000,
          winding_number: 2,
          turn_number: 2,
          parallel_number: 1,
        },
      ],
    },
  ],
});

// Índice de la capa seleccionada
const selectedLayerIndex = ref<number>(0);

// Obtener la capa seleccionada a través de un cálculo reactivo
const selectedLayer = computed(
  () => formData.value.layers[selectedLayerIndex.value]
);

// Cálculo para obtener el valor de thickness
const thicknessValue = computed({
  get: () =>
    selectedLayer.value.thickness ?? formData.value.layers_thickness_default,
  set: (value) => {
    if (selectedLayer.value) {
      selectedLayer.value.thickness = value;
    }
  },
});

// Función para abrir el modal
const openModal = () => {
  isModalOpen.value = true;
};

// Función para cerrar el modal
const closeModal = () => {
  isModalOpen.value = false;
};

// Función para manejar el envío del formulario
const submitForm = () => {
  console.log("Form data submitted:", formData.value);
  closeModal();
};
</script>

<style scoped>
/* Estilos personalizados */
.modal-content {
  max-height: 80vh; /* Limita la altura máxima del modal al 80% de la altura de la ventana */
  overflow-y: auto; /* Habilita el desplazamiento vertical */
}
</style>
