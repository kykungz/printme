<template>
  <div ref="dropzone" class="dropzone">
    <div class="info">
      <span v-if="!name">Drop files here or click to upload</span>
      <span else>{{ name }}</span>
    </div>
    <input
      @dragenter="e => handleDragEnter(e)"
      @dragleave="e => handleDragLeave(e)"
      @change="e => handleDrop(e)"
      class="input"
      type="file"
    >
  </div>
</template>

<script>
import firebase from '@/fire'

export default {
  name: 'Dropzone',
  data () {
    return {
      name: ''
    }
  },
  methods: {
    handleDrop (e) {
      this.$refs.dropzone.classList.remove('dragover')
      const file = e.target.files[0]
      this.name = file.name
      const task = firebase.storage().ref(file.name).put(file)
      task.on(
        'state_changed',
        progress => {
          // TODO: Handle progress
          const percent = (progress.bytesTransferred / progress.totalBytes) * 100
        },
        err => {
          // TODO: Handle errors
        },
        () => {
          // TODO: Handle completed
        }
      )
    },
    handleDragEnter (e) {
      this.$refs.dropzone.classList.add('dragover')
    },
    handleDragLeave (e) {
      this.$refs.dropzone.classList.remove('dragover')
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.dropzone {
  position: relative;
  margin: auto;
  width: 80%;
  height: 300px;
  background-color: white;
  border-style: dashed;
  border-color: orange;
  border-width: medium;
  border-radius: 4px;
  transition: all 400ms;
}

.dropzone:hover, .dropzone.dragover {
  background-color: lightgray;
  transform: scale(1.03);
}

.info {
  position: absolute;
  left: 0;
  top: 40%;
  width: 100%;
  font-size: 1.5em;
  text-align: center;
}

.input {
  cursor: pointer;
  width: 100%;
  height: 100%;
  opacity: 0;
}

@keyframes pulse {
    0%, 100% { background-color: white; }
    50% { background-color: lightgray; }
}
</style>
