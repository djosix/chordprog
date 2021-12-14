<template lang="pug">
.container
  .chord-row(v-if='samplesAreLoaded')
    ChordBlock(@playChordNotes='playChordNotes')
    ChordBlock(@playChordNotes='playChordNotes')
    ChordBlock(@playChordNotes='playChordNotes')
    ChordBlock(@playChordNotes='playChordNotes')
    ChordBlock(@playChordNotes='playChordNotes')
    ChordBlock(@playChordNotes='playChordNotes')
    ChordBlock(@playChordNotes='playChordNotes')
    ChordBlock(@playChordNotes='playChordNotes')
  div(v-else) loading samples...
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import ChordBlock from "./components/ChordBlock.vue";

import * as Tonal from "@tonaljs/tonal";
import * as Tone from "tone";

// debugger;
window.Tonal = Tonal;
window.Tone = Tone;

import { Piano } from "@tonejs/piano";

export default {
  name: "App",
  created() {
    this.piano = new Piano({
      velocities: 5,
    });
    this.piano.toDestination();

    this.piano.load().then(() => {
      this.samplesAreLoaded = true
      console.log("loaded!");
    });
  },
  data() {
    return {
      samplesAreLoaded: false
    }
  },
  methods: {
    playChordNotes(notes) {
      if (notes.length) {
        notes.forEach(note => {
          this.piano.keyDown({ note: note.name, velocity: 0.2 });
          this.piano.keyUp({ note: note.name, time: '+2' })
        })
      }
    }
  },
  components: {
    ChordBlock,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  /* margin-top: 60px; */
}

.chord-row {
  border: 1px solid black;
  padding: 1em;
  overflow: scroll;
  white-space: nowrap;
}

.chord-block {
  display: inline-block;
}
</style>
