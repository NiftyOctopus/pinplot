<template>
  <div class="app-container">
    <MainCanvas :pins="pins" 
                @pin-location-selected="openPinFormModal" 
                @pin-selected="editExistingPin"/>
    <PinFormModal v-if="isModalOpen" 
                  :x="selectedX" 
                  :y="selectedY" 
                  :pin="selectedPin" 
                  @add-new-pin="addNewPin" 
                  @update-pin="updatePin"
                  @close-modal="isModalOpen = false" />
  </div>
</template>


<script setup lang="ts">
import MainCanvas from './components/MainCanvas.vue';
import PinFormModal from './components/PinFormModal.vue';

import { ref, onMounted } from 'vue';

const isModalOpen = ref(false);
const selectedX = ref<number | null>(null);
const selectedY = ref<number | null>(null);
const selectedPin = ref<any | null>(null);  // Reference for an optional selected pin

const pins = ref<any[]>([]);
const pinsLoaded = ref(false);

onMounted(() => {
  const storedPins = localStorage.getItem('pins');
  if (storedPins) {
    pins.value = JSON.parse(storedPins);
  }
  pinsLoaded.value = true; // Indicate that pins are loaded
});


const openPinFormModal = (location: { x: number, y: number }) => {
  selectedX.value = location.x;
  selectedY.value = location.y;
  isModalOpen.value = true;
};

const editExistingPin = (pin: any) => {
  console.log('Editing pin:', pin);
  selectedX.value = pin.x;
  selectedY.value = pin.y;
  selectedPin.value = pin;  // Set the selectedPin ref to the clicked pin data
  isModalOpen.value = true;
};

const addNewPin = (pinData: any) => {
  // Check if it's an edit operation
  if (pinData.id) {
    const index = pins.value.findIndex(p => p.id === pinData.id);
    if (index !== -1) {
      pins.value[index] = pinData;  // Update the pin data in the array
    }
  } else {
    // New pin
    // Generate id
    pinData.id = Math.random().toString(36).substring(2, 9);
    pins.value = [...pins.value, pinData];
  }

  // Save pins array to local storage
  localStorage.setItem('pins', JSON.stringify(pins.value));

  // Close the modal and reset values
  isModalOpen.value = false;
  selectedX.value = null;
  selectedY.value = null;
};


const updatePin = (updatedPin: any) => {
  const index = pins.value.findIndex(pin => pin.id === selectedPin.value.id);
  console.log(index)
  const newPins = [...pins.value];
  console.log(updatedPin)

  if (index !== -1) {
    newPins[index] = updatedPin;
    pins.value = newPins;
    localStorage.setItem('pins', JSON.stringify(pins.value));
  }

  isModalOpen.value = false;
  selectedPin.value = null;
  selectedX.value = null;
  selectedY.value = null;
};
</script>
