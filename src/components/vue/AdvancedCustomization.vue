<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from "vue";

const isModalOpen = ref(false);
const emit = defineEmits(["save"]);

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

const editorContainer = ref(null);
let editorInstance;

const loadMonacoEditor = async () => {
  const monaco = await import("monaco-editor");
  return monaco;
};

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

const openModal = () => {
  isModalOpen.value = true;
  initializeEditor();
};

const closeModal = () => {
  isModalOpen.value = false;
};

const saveJson = () => {
  try {
    const jsonText = editorInstance.getValue();
    formData.value = JSON.parse(jsonText);
    emit('save');
    closeModal();
  } catch (e) {
    alert("Invalid JSON format. Please correct it and try again.");
  }
};

const downloadJson = () => {
  try {
    const jsonText = editorInstance.getValue();
    const blob = new Blob([jsonText], { type: "application/json" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = "customization.json";
    a.click();
    URL.revokeObjectURL(url);
  } catch (e) {
    alert("Error downloading JSON. Please try again.");
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

<template>
    <button @click="openModal" class="px-4 py-2 bg-blue-600 text-white rounded">
      Advanced Customization
    </button>

    <div
      v-if="isModalOpen"
      class="fixed inset-0 flex items-center justify-center bg-gray-800 bg-opacity-75"
    >
      <div
        class="bg-white rounded-lg shadow-lg p-6 w-full max-w-2xl max-h-full overflow-y-auto"
      >
        <h2 class="text-xl font-semibold mb-4">Advanced Customization</h2>

        <div ref="editorContainer" class="border rounded h-[600px]"></div>
        
        <div class="flex justify-end mt-4 gap-2">
          <button
            @click="saveJson"
            type="button"
            class="px-4 py-2 bg-green-600 text-white rounded"
          >
            Save
          </button>
          <button class="bg-gray-200 py-2 px-4 rounded" @click="downloadJson">Download</button>
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

</template>

<style scoped>
.modal-content {
  max-height: 80vh;
  overflow-y: auto;
}
</style>
