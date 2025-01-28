<template>
  <div class="flex flex-col items-center justify-center min-h-screen bg-gray-100">
    <!-- Camera Feed Wrapper -->
    <div class="relative w-[300px] h-[300px] bg-white shadow-lg rounded-lg overflow-hidden">
      <canvas
        ref="canvasElement"
        id="jeeliz_canvas"
        class="absolute inset-0 w-full h-full"
      ></canvas>
      <!-- UCL Club Logo -->
      <img
        v-if="currentLogo"
        :src="currentLogo"
        alt="Club Logo"
        class="absolute w-16 h-16"
        :style="logoStyle"
      />
    </div>

    <!-- Change Logo Button -->
    <button
      @click="applyRandomLogo"
      class="mt-6 px-6 py-3 bg-blue-600 text-white text-lg font-medium rounded-lg shadow-md hover:bg-blue-700"
    >
      Change Logo
    </button>

    <!-- Start Camera Button -->
    <button
      @click="requestCameraPermission"
      class="mt-6 px-6 py-3 bg-green-600 text-white text-lg font-medium rounded-lg shadow-md hover:bg-green-700"
    >
      Start Camera
    </button>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

// List of UCL club logos
const logos = [
  "/logos/barcelona.png",
  "/logos/arsenal.png",
  "/logos/chelsea.png",
  "/logos/psg.png",
  // Add more logos here
];

// State for the current logo
const currentLogo = ref("");
const logoStyle = ref({ top: '0px', left: '0px' });

// Jeeliz settings
let filterHandle = null;
let canvasElement = null;

// Apply a random logo
const applyRandomLogo = () => {
  const randomIndex = Math.floor(Math.random() * logos.length);
  currentLogo.value = logos[randomIndex];
};

// Function to request camera permissions
const requestCameraPermission = async () => {
  try {
    // Requesting camera access
    const stream = await navigator.mediaDevices.getUserMedia({ video: true });
    console.log("Camera access granted");
    startFaceTracking(stream);
  } catch (err) {
    console.error("Error accessing camera: ", err);
    alert("Camera access is required for this feature.");
  }
};

// Initialize Jeeliz Face Filter
const initJeeliz = async () => {
  await new Promise((resolve) => {
    JeelizFaceFilter.init({
      canvasId: 'jeeliz_canvas',
      NNCpath: 'https://cdn.jsdelivr.net/npm/jeeliz-face-filter@0.9.0/',
      callbackReady: () => {
        console.log('Jeeliz Face Filter ready!');
        filterHandle = JeelizFaceFilter;
        resolve();
      }
    });
  });
};

// Start face tracking and overlay logo on face
const startFaceTracking = (stream) => {
  JeelizFaceFilter.start({
    canvasId: 'jeeliz_canvas',
    videoStream: stream,  // Pass the camera stream to Jeeliz
    faceTrackingCallback: (result) => {
      // If a face is detected, apply the logo position
      if (result && result.face) {
        const face = result.face;
        const facePosition = face.mesh[0]; // You can adjust this based on the mesh points

        // Calculate the logo position based on face position
        logoStyle.value = {
          top: `${facePosition[1]}px`,
          left: `${facePosition[0]}px`,
        };
      }
    }
  });
};

// Start face filter on mount (optional, you can remove this if you don't need it)
onMounted(() => {
  applyRandomLogo(); // Optionally initialize the random logo here
});
</script>

<style>
/* No custom styles needed as Tailwind is used */
</style>
