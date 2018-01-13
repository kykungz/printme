<template>
  <div ref="dropzone" class="dropzone">
    <div class="info">
      <span>Drop files here or click to upload</span>
    </div>
    <input
      class="select"
      type="file"
      @change="e => handleChange(e)"
      multiple
    >
    <div
      @dragover.prevent
      @dragenter="e => handleDragEnter(e)"
      @dragleave="e => handleDragLeave(e)"
      @drop="e => handleDrop(e)"
      class="input"
    ></div>
  </div>
</template>

<script>
import firebase from '@/fire'

export default {
  name: 'Dropzone',
  data () {
    return {
    }
  },
  methods: {
    handleChange (e) {
      this.$refs.dropzone.classList.remove('dragover')
      console.log(e)
      console.log(e.target.files.length)
      // for (var i = 0; i < e.target.files.length; i++) {
      //   const file = e.target.files[i]
      //   const task = firebase.storage().ref(file.name).put(file)
      //   task.on(
      //     'state_changed',
      //     progress => {
      //       // TODO: Handle progress
      //       const percent = (progress.bytesTransferred / progress.totalBytes) * 100
      //       console.log(file.name + ': ' + percent)
      //     },
      //     err => {
      //       // TODO: Handle errors
      //     },
      //     () => {
      //       firebase.database().ref('files/').push({
      //         name: file.name,
      //         src: task.snapshot.downloadURL
      //       })
      //     }
      //   )
      // }
    },
    handleDragEnter (e) {
      this.$refs.dropzone.classList.add('dragover')
    },
    handleDragLeave (e) {
      this.$refs.dropzone.classList.remove('dragover')
    },
    handleDrop (e) {
      this.$refs.dropzone.classList.remove('dragover')
      event.preventDefault()
      const length = e.dataTransfer.items.length
      for (let i = 0; i < length; i++) {
        const entry = e.dataTransfer.items[i].webkitGetAsEntry()
        if (entry) {
          this.traverseFileTree(entry)
        }
      }
    },
    traverseFileTree (entry) {
      if (entry.isFile) {
        entry.file(file => {
          this.upload(file, entry.fullPath)
        })
      } else if (entry.isDirectory) {
        const dirReader = entry.createReader()
        dirReader.readEntries(entries => {
          for (let i = 0; i < entries.length; i++) {
            this.traverseFileTree(entries[i])
          }
        })
      }
    },
    upload (file, path) {
      console.log(file)
      const task = firebase.storage().ref(path).put(file)
      task.on(
        'state_changed',
        progress => {
          // TODO: Handle progress
          const percent = (progress.bytesTransferred / progress.totalBytes) * 100
          console.log(file.name + ': ' + percent)
        },
        err => {
          // TODO: Handle errors
        },
        () => {
          firebase.database().ref('files/').push({
            name: file.name,
            src: task.snapshot.downloadURL
          })
        }
      )
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

.select {
  position: absolute;
  z-index: 2;
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
