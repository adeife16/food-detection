<template>
  <div class="p-4 max-w-4xl mx-auto">
    <h1 class="text-2xl font-bold text-center mb-8">Capture or Upload Image and Convert to Base64</h1>

    <!-- Camera Section -->
    <div class="mb-8">
      <div class="border-2 border-gray-300 rounded-lg overflow-hidden mb-4">
        <video ref="video" class="w-full h-auto" autoplay></video>
      </div>
      <div class="flex space-x-4">
        <button
          @click="captureImage"
          :disabled="cameraDisabled"
          class="bg-blue-500 text-white px-4 py-2 rounded shadow hover:bg-blue-600 transition duration-300 disabled:bg-gray-400"
        >
          Capture Image
        </button>
        <button
          @click="startCamera"
          :disabled="!cameraDisabled"
          class="bg-green-500 text-white px-4 py-2 rounded shadow hover:bg-green-600 transition duration-300 disabled:bg-gray-400"
        >
          Enable Camera
        </button>
      </div>
    </div>

    <!-- Upload Section -->
    <div class="mb-8">
      <h2 class="text-lg font-semibold mb-4">Upload an Image</h2>
      <div>
        <input
          type="file"
          accept="image/*"
          @change="handleFileUpload"
          class="block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded file:border-0 file:bg-gray-100 file:text-gray-700 hover:file:bg-gray-200"
        />
      </div>
      <div v-if="uploadedImagePreview" class="mt-4">
        <h3 class="font-medium mb-2">Preview</h3>
        <img :src="uploadedImagePreview" alt="Uploaded Image" class="w-64 h-64 object-cover rounded-lg border" />
      </div>
    </div>

    <!-- Base64 Output Section -->
    <div v-if="base64Image">
      <h2 class="text-lg font-semibold mb-4">Base64 Representation</h2>
      <img :src="base64Image" alt="Base64 Image" class="w-64 h-64 object-cover rounded-lg border mb-4" />
      <textarea
        v-model="base64Image"
        rows="10"
        class="w-full p-2 border border-gray-300 rounded-lg text-sm font-mono"
      ></textarea>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      base64Image: null, // Stores the Base64 representation of the image
      uploadedImagePreview: null, // Stores the preview of the uploaded image
      cameraStream: null, // Stores the MediaStream object
      cameraDisabled: false, // Flag to disable or enable the camera capture button
    };
  },
  methods: {
    async startCamera() {
      try {
        if (this.cameraStream) {
          this.stopCamera(); // Stop the current stream if already running
        }
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        this.cameraStream = stream; // Save the camera stream
        this.$refs.video.srcObject = stream;
        this.cameraDisabled = false; // Enable the capture button
      } catch (err) {
        console.error("Error accessing camera:", err);
      }
    },
    captureImage() {
      const video = this.$refs.video;
      const canvas = document.createElement("canvas");
      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;

      // Draw the video frame onto the canvas
      const ctx = canvas.getContext("2d");
      ctx.drawImage(video, 0, 0, canvas.width, canvas.height);

      // Convert the canvas image to base64
      this.base64Image = canvas.toDataURL("image/png");

      // Disable the camera
      this.stopCamera();
    },
    stopCamera() {
      if (this.cameraStream) {
        this.cameraStream.getTracks().forEach((track) => track.stop());
        this.cameraStream = null; // Clear the stream
        this.cameraDisabled = true; // Disable the capture button
      }
    },
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          this.uploadedImagePreview = e.target.result; // Display uploaded image preview
          this.base64Image = e.target.result; // Store Base64 string
        };
        reader.readAsDataURL(file); // Convert file to Base64
      }
    },
  },
  mounted() {
    this.startCamera();
  },
};
</script>

<style>
/* Tailwind CSS is already handled through a global setup or CDN, no additional scoped styles needed */
</style>
