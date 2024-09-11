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
        class="bg-white rounded-lg shadow-lg p-6 w-full max-w-2xl max-h-full overflow-y-auto"
      >
        <h2 class="text-xl font-semibold mb-4">Advanced Customization</h2>

        <!-- Editor de JSON usando Monaco Editor -->
        <div ref="editorContainer" class="border rounded h-[600px]"></div>
        
        <!-- Botones de Acción -->
        <div class="flex justify-end mt-4">
          <button
            @click="saveJson"
            type="button"
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
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from "vue";

// Estado del modal
const isModalOpen = ref(false);

// Datos del formulario en formato JSON
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

// Editor de Monaco
const editorContainer = ref(null);
let editorInstance;

// Función para cargar Monaco Editor dinámicamente
const loadMonacoEditor = async () => {
  const monaco = await import("monaco-editor");
  return monaco;
};

// Función para inicializar el editor
const initializeEditor = async () => {
  const monaco = await loadMonacoEditor();
  if (editorContainer.value) {
    editorInstance = monaco.editor.create(editorContainer.value, {
      value: JSON.stringify(formData.value, null, 2),
      language: "json",
      theme: "vs-dark",
      automaticLayout: true,
    });
  }
};

// Función para abrir el modal
const openModal = () => {
  isModalOpen.value = true;
  initializeEditor();
};

// Función para cerrar el modal
const closeModal = () => {
  isModalOpen.value = false;
};

// Guardar el JSON
const saveJson = () => {
  try {
    const jsonText = editorInstance.getValue();
    formData.value = JSON.parse(jsonText);
    console.log("Updated data:", formData.value);
    closeModal();
  } catch (e) {
    alert("Invalid JSON format. Please correct it and try again.");
  }
};

onMounted(() => {
  if (isModalOpen.value) {
    initializeEditor();
  }
});

onBeforeUnmount(() => {
  if (editorInstance) {
    editorInstance.dispose();
  }
});
</script>

<style scoped>
/* Estilos personalizados */
.modal-content {
  max-height: 80vh;
  overflow-y: auto;
}
</style>
