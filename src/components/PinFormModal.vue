<template>
  <div class="modal fixed inset-0 flex items-center justify-center z-50">
    <div class="bg-white p-6 rounded shadow-lg w-96">
      <h2 class="text-xl mb-4">Add a New Pin</h2>

      <label for="coordinateX" class="block mb-2">X:</label>
      <input v-model.number="editableX" id="coordinateX" type="number" 
             class="border rounded w-full p-2 mb-4">

      <label for="coordinateY" class="block mb-2">Y:</label>
      <input v-model.number="editableY" id="coordinateY" type="number" 
             class="border rounded w-full p-2 mb-4">

      <label for="pinLabel" class="block mb-2">Pin Label:</label>
      <input v-model="pinLabel" ref="pinLabelInput" id="pinLabel" type="text" 
             class="border rounded w-full p-2 mb-4" @keyup.enter="savePin">

      <label for="pinTags" class="block mb-2">Tags (comma separated):</label>
      <input v-model="pinTags" id="pinTags" type="text" 
             class="border rounded w-full p-2 mb-4" @keyup.enter="savePin">

      <button @click="savePin" class="bg-blue-500 text-white rounded p-2 mr-2">Save</button>
      <button @click="closeModal" class="bg-gray-300 rounded p-2">Close</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch, nextTick, onMounted, onBeforeUnmount, defineProps, defineEmits } from 'vue';

const props = defineProps({
  x: Number,
  y: Number,
  pin: Object  // Pin object (optional)
});
const emit = defineEmits(['add-new-pin', 'update-pin', 'close-modal']);

const editableX = ref(props.x);
const editableY = ref(props.y);
const pinLabel = ref('');
const pinTags = ref('');

const pinLabelInput = ref(null);

// Populate fields based on provided pin
watch(() => props.pin, (newPin) => {
  if (newPin) {
    editableX.value = newPin.x;
    editableY.value = newPin.y;
    pinLabel.value = newPin.label;
    pinTags.value = newPin.tags?.join(', ');
  }
}, { immediate: true });

const savePin = () => {
  const newPin = {
    x: editableX.value,
    y: editableY.value,
    label: pinLabel.value,
    tags: pinTags.value.split(',').map(tag => tag.trim()),
    // Add the id of the pin if it's an edit, otherwise undefined
    id: props.pin ? props.pin.id : undefined
  };

  const event = props.pin?.id ? 'update-pin' : 'add-new-pin';
  emit(event, newPin);
  closeModal();
};

const closeModal = () => {
  editableX.value = props.x;
  editableY.value = props.y;
  pinLabel.value = '';
  pinTags.value = '';
  emit('close-modal');
};

// Automatically focus the pin name input when the modal opens
nextTick(() => {
  pinLabelInput.value && pinLabelInput.value.focus();
});

const handleEscapeKeyPress = (event: KeyboardEvent) => {
  if (event.key === 'Escape') {
    closeModal();
  }
};

onMounted(() => {
  document.addEventListener('keydown', handleEscapeKeyPress);
});

onBeforeUnmount(() => {
  document.removeEventListener('keydown', handleEscapeKeyPress);
});
</script>

<style scoped>
.modal {
  background-color: rgba(0, 0, 0, 0.7); /* Darker semi-transparent background */
}
</style>
