<template lang="pug">
.piano
  .key-container
    .key(v-for='note in notes' v-bind:class='getKeyClassByNote(note)' @click='toggleSelectedNote(note)')
      .label(v-if='!note.alt') {{ note.pc == 'C' ? note.name : note.pc }}
</template>

<script>
import { Note } from '@tonaljs/tonal';

export default {
  name: 'PianoKeys',
  created() {
    const startPitch = Note.midi('C2')
    const numberOfKeys = 49

    this.notes = [...Array(numberOfKeys).keys()]
      .map(i => startPitch + i)
      .map(Note.fromMidi)
      .map(Note.get)
      .reverse()
  },
  data() {
    return {
      chordNotes: new Set(),
      tonicNotes: new Set(),
      selectedNotes: new Set(),
      selectedChromas: new Set(),
    }
  },

  methods: {
    toggleSelectedNote(note) {
      const isSelected = this.selectedNotes.has(note.midi)
      this.updateSelectedNotes([note], !isSelected)
    },

    updateSelectedNotes(notes, isSelected = true) {
      notes.forEach(note => {
        if (isSelected) {
          this.selectedNotes.add(note.midi)
        } else {
          this.selectedNotes.delete(note.midi)
        }
      })
      this.selectedChromas.clear()
      this.selectedNotes.forEach(midi => this.selectedChromas.add(Note.get(Note.fromMidi(midi)).chroma))
      console.log(this.selectedNotes)
      this.$emit('updateSelectedNotes', [...this.selectedNotes].map(Note.fromMidi).map(Note.get))
    },

    clearSelectedNotes() {
      this.selectedNotes.clear()
      this.selectedChromas.clear()
      this.$emit('updateSelectedNotes', [...this.selectedNotes])
    },

    getKeyClassByNote(note) {
      let classes = {
        white: !note.alt,
        black: note.alt,
        chord: this.chordNotes.has(note.midi),
        tonic: this.tonicNotes.has(note.midi),
        selected: this.selectedNotes.has(note.midi),
        presented: this.selectedChromas.has(note.chroma)
      }
      classes[`key-${note.name}`] = true
      classes[`key-${note.pc}`] = true
      return classes
    },

    highlightChord(chord) {
      this.chordNotes.clear()
      this.tonicNotes.clear()

      if (chord) {
        let chordChromaSet = new Set(chord.notes.map(Note.chroma))
        let tonicChroma = Note.chroma(chord.tonic)

        this.notes.forEach(note => {
          if (chordChromaSet.has(note.chroma)) {
            this.chordNotes.add(note.midi)
          }
          if (tonicChroma == note.chroma) {
            this.tonicNotes.add(note.midi)
          }
        })
      }
    },

  }
}
</script>

<style lang="less" scoped>
.key-container {
  padding-top: 0.3em;
  padding-bottom: 0.3em;
}

.key {
  border: 1px solid black;
  display: block;

  margin: 0px;
  padding: 0px;

  user-select: none;
  cursor: pointer;

  border-left: 1px solid gray;
  border-right: 1px solid gray;
  border-top: 0px;
  border-bottom: 0px;

  .label {
    font-size: 10px;
    padding: 0px;
    margin: 0px;
    border: 0px;
  }

  &:hover {
    border-right: 2em solid rgb(204, 203, 203);
  }

  &.white {
    >.label {
      color: gray;
    }
    height: 0.8em;
    border-top: 1px solid gray;
  }

  &.black {
    >.label {
      color: white;
    }
    background-color: gray;
    height: 0.5em;
  }

  &.chord {
    border-left: 1em solid rgb(255, 217, 0);

    &.presented {
      border-left: 0.5em solid rgb(255, 217, 0);
    }
  }

  &.tonic {
    border-left: 1em solid rgb(71, 194, 0);

    &.presented {
      border-left: 0.5em solid rgb(71, 194, 0);
    }
  }

  &.selected {
    >.label {
      font-weight: bolder;
    }

    border-right: 8em solid rgb(255, 0, 0);

    &.chord {
      border-right: 8em solid rgb(255, 217, 0);
    }

    &.tonic {
      border-right: 8em solid rgb(71, 194, 0);
    }
  }

  &.key-C4>.label {
    font-weight: bold;
  }

  &.key-C2 {
    border-bottom: 1px solid gray;
  }

}
</style>
