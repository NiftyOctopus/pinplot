<template>
  <canvas ref="mapCanvas" @click="onCanvasClick"></canvas>
</template>


<script setup lang="ts">
import { ref, onMounted, computed, watch } from 'vue';

const mapCanvas = ref(null);
const emit = defineEmits(['pin-location-selected', 'pin-selected']);

const props = defineProps(['pins']);

const renderPins = () => {
  if (!props.pins || !Array.isArray(props.pins)) {
    console.log("No pins to render")
    return;
  }
  console.log("Rendering pins:", props.pins);

  const ctx = mapCanvas.value.getContext("2d");
  //console.log("Canvas context:", ctx);

  // Clear the entire canvas
  ctx.clearRect(0, 0, mapCanvas.value.width, mapCanvas.value.height);

  for (const pin of props.pins) {
    // Draw a circle for each pin
    //console.log('Drawing pin', pin);
    ctx.beginPath();
    ctx.arc(pin.x, pin.y, 5, 0, 2 * Math.PI, false);
    
    ctx.fillStyle = 'blue'; // you can customize the pin color here
    ctx.fill();
    ctx.closePath();

    // Optionally: Draw pin label beside it (or implement hover/click to show label)
    ctx.font = "12px Arial";
    ctx.fillText(pin.id + ':' + pin.label, pin.x + 10, pin.y);
  }
};

const computedPins = computed(() => props.pins);
watch(computedPins, () => {
  renderPins();
});

onMounted(() => {
  console.log("Received Pins:", props.pins);

  const canvas  = mapCanvas.value;
  canvas.width  = window.innerWidth;
  canvas.height = window.innerHeight;

  // Initial rendering of pins
  console.log('Initial rendering of pins');
  renderPins();
});


const isPinClicked = (x:number, y:number) => {
  // Tolerance range in pixels around the pin center to detect click
  const tol = 5;

  for (const pin of props.pins) {
    const deltaX = Math.abs(pin.x - x);
    const deltaY = Math.abs(pin.y - y);

    if (deltaX < tol && deltaY < tol) {
      return pin; // Return the clicked pin data
    }
  }
  return null;
};


const onCanvasClick = (event) => {
  const x = event.clientX;
  const y = event.clientY;

  // Check if user clicked on existing pin
  const clickedPin = isPinClicked(x, y);

  if (clickedPin) {
    emit('pin-selected', clickedPin); // Selected existing pin
  
  } else {
    emit('pin-location-selected', { x, y });
  }

  renderPins();
};

</script>

  