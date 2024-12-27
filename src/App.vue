<template>
  <div id="app">
    <h1>Camera Access App</h1>
    <div>
      <video ref="video" autoplay playsinline></video>
    </div>
    <div>
      <button @click="camera('user')">Use Front Camera</button>
      <button @click="camera('environment')">Use Back Camera</button>
    </div>
    <div>
      <label for="zoom">Zoom: </label>
      <input
        id="zoom"
        type="range"
        min="1"
        max="3"
        step="0.1"
        v-model="zoomLevel"
        @input="adjustZoom"
      />
      <span>{{ zoomLevel }}x</span>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      localStream: null, // Stores the camera stream
      videoElement: null, // Reference to the video element
      zoomLevel: 1, // Default zoom level
      currentTrack: null, // Store the current video track for zoom control
    };
  },
  methods: {
    camera(face) {
      // Stop the current stream before starting a new one
      this.stop();
      this.gum(face);
    },
    stop() {
      // Stop all tracks in the current video stream
      if (this.localStream) {
        this.localStream.getTracks().forEach((track) => track.stop());
        this.localStream = null;
        this.currentTrack = null;
      }
    },
    gum(face) {
      // Access the user's camera based on the specified facing mode
      const constraints = {
        video: {
          facingMode: face === "user" ? face : { exact: face },
        },
      };

      navigator.mediaDevices
        .getUserMedia(constraints)
        .then((stream) => {
          this.localStream = stream; // Save the stream for future reference
          if (this.videoElement) {
            this.videoElement.srcObject = stream; // Attach the stream to the video element
          } else {
            console.error("Video element is not defined.");
          }

          // Get the video track for zoom control
          this.currentTrack = stream.getVideoTracks()[0];
        })
        .catch((error) => {
          console.error("Error accessing camera:", error);
          alert(
            `Could not access the camera. Please check your permissions or device. Error: ${error.message}`
          );
        });
    },
    adjustZoom() {
      if (this.currentTrack) {
        const capabilities = this.currentTrack.getCapabilities();
        if (capabilities.zoom) {
          const zoom = this.zoomLevel;
          this.currentTrack.applyConstraints({
            advanced: [{ zoom: zoom }],
          }).catch((error) => {
            console.error("Error applying zoom:", error);
          });
        }
      }
    },
  },
  mounted() {
    // Reference the video element after the component is mounted
    this.videoElement = this.$refs.video;

    // Start with the back-facing camera by default
    this.camera("environment");
  },
  beforeDestroy() {
    // Stop the stream when the component is destroyed
    this.stop();
  },
};
</script>

<style scoped>
#app {
  text-align: center;
  padding: 20px;
}

video {
  width: 100%;
  max-width: 600px;
  height: auto;
  border: 2px solid #ccc;
}

button {
  margin: 10px;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
}

button:hover {
  background-color: #f0f0f0;
}

input[type="range"] {
  width: 200px;
}

label {
  font-size: 16px;
}

span {
  font-size: 18px;
  font-weight: bold;
  margin-left: 10px;
}
</style>
