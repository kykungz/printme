<template>
  <div ref="dropzone" class="dropzone">
    <div v-if="Object.keys(tasks).length === 0" class="info">
      &ndash; Drop files here &ndash;
    </div>
    <div class="status">
      <div v-for="key in Object.keys(tasks)" class="task">
        <span class="task-path"> {{ tasks[key].path }} </span>
        <span v-if="tasks[key].percent < 100" class="task-percent">
          {{ tasks[key].percent }}%
        </span>
        <span v-else class="task-percent"> Completed </span>
        <progress class="progress" :value="tasks[key].percent" max="100">0%</progress>
      </div>
    </div>
    <div
      @dragover.prevent
      @dragenter="e => handleDragEnter(e)"
      @dragleave="e => handleDragLeave(e)"
      @drop="e => handleDrop(e)"
      class="dropbox"
    ></div>
  </div>
</template>

<script>
import firebase from '@/fire'

export default {
  name: 'Dropzone',
  data () {
    return {
      tasks: {}
    }
  },
  methods: {
    handleDragEnter (e) {
      this.$refs.dropzone.classList.add('dragover')
      console.log('enter')
    },
    handleDragLeave (e) {
      this.$refs.dropzone.classList.remove('dragover')
      console.log('leave')
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
      const taskNum = this.tasks.length
      this.$set(this.tasks, path, { path, percent: 0 })
      task.on(
        'state_changed',
        progress => {
          // TODO: Handle progress
          const percent = (progress.bytesTransferred / progress.totalBytes) * 100
          console.log(file.name + ': ' + percent)
          // this.tasks[path].percent = parseInt(percent)
          console.log('setting');
          this.$set(this.tasks, path, { path, percent: parseInt(percent) })
        },
        err => {
          // TODO: Handle errors
        },
        () => {
          firebase.database().ref('storage/').push({
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
  overflow: scroll;
  position: relative;
  margin: auto;
  width: 90%;
  height: 350px;
  background-color: whitesmoke;
  border-style: dashed;
  border-color: rgba(255, 184, 0, 0.8);
  border-width: medium;
  border-radius: 4px;
  transition: all 400ms;
}

.dropzone.dragover, .dropzone:hover {
  background-color: lightgray;
}

.dropzone.dragover {
  transform: scale(1.03);
}

.info {
  position: absolute;
  left: 0;
  top: 40%;
  width: 100%;
  font-size: 1.5em;
  text-align: center;
  color: black;
  opacity: 0.5;
}

.status {
  z-index: 20;
  position: absolute;
  width: 100%;
  display: block;
}

.dropbox {
  position: absolute;
  cursor: pointer;
  width: 100%;
  height: 100%;
}

.task {
  padding: 10px;
  border-bottom: solid thin gray;
  background-color: white;
}

.task-path {
  font-size: 18px;
}

.task-percent {
  float: right;
}

.progress {
  display: block;
  width: 100%;
}
</style>
