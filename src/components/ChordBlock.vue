<template lang="pug">
.chord-block
  h2.chord-title(@click='playChordNotes') {{ title || '-' }}
  .chord-comment {{ comment || name || '-' }}
  hr
  table
    tr
      th.field-label chord
      td.field-input
        input(v-model='symbol' :list='datalistId' :placeholder='chordOptions' onclick="this.select()" onchange="this.blur()")
        datalist(:id='datalistId')
          option(v-for='option in chordOptions' :value='option' @click='alert') {{ option }}
    tr
      th.field-label comment
      td.field-input
        input(v-model='comment')
  PianoKeys(@updateSelectedNotes='onUpdateSelectedNotes' ref='pianoKeys')
  .buttons
    button(type="button" @click='clearSelectedNotes') clear
    .space
    button(type="button" @click='fillChordNotes') fill
</template>

<script>
import { v4 as uuid } from 'uuid'
import PianoKeys from './PianoKeys.vue'

import { Note, Chord } from '@tonaljs/tonal'
Chord

export default {
  name: 'ChordBlock',

  beforeCreate() {
    this.datalistId = uuid()
  },

  data() {
    return {
      title: '-',
      name: '-',
      comment: '',
      symbol: '',
      chord: null,
      notes: new Set(),
      chordOptions: []
    }
  },

  methods: {
    onUpdateSelectedNotes(notes) {
      this.notes = notes
      this.chordOptions = Chord.detect(this.notes)

      this.highlightChord()
      this.playChordNotes()
    },

    highlightChord() {
      let chord = null;
      if (this.chord) {
        chord = this.chord
      } else if (this.notes.length) {
        chord = {
          notes: [...new Set(this.notes.map(note => note.pc))],
          tonic: Note.get(Note.fromMidi(Math.min(...this.notes.map(Note.midi)))).pc
        }
      }
      this.$refs.pianoKeys.highlightChord(chord)
    },

    clearSelectedNotes() {
      this.$refs.pianoKeys.clearSelectedNotes()
    },

    playChordNotes() {
      this.$emit('playChordNotes', this.notes)
    },

    fillChordNotes() {
      if (this.chord) {
        this.$refs.pianoKeys.clearSelectedNotes()
        let notes = [this.chord.tonic + '3', ...this.chord.notes.map(note => note + '4')]
        this.$refs.pianoKeys.updateSelectedNotes(notes.map(Note.get), true)
      }
    }

  },
  watch: {
    symbol(value) {
      if (value == '') {
        this.chord = null
        this.title = ''
        this.name = ''
      } else {
        let temp = Chord.get(this.symbol)
        this.chord = temp.empty ? null : temp;
        this.title = this.chord ? this.chord.symbol : this.symbol || '?'
        this.name = this.chord ? this.chord.name : '(unknown chord)'
      }
      this.highlightChord()
    }
  },
  components: {
    PianoKeys
  }
}
</script>

<style scoped>
.chord-block {
  display: inline-block;
  border: 2px solid black;
  border-radius: 0.4em;
  padding: 0.5em;
  margin-left: 0.2em;
  margin-right: 0.2em;
}

.chord-title {
  user-select: none;
  cursor: pointer;
}

.chord-title:hover {
  color: red;
}

.field-label {
  text-align: right;
}
.space {
  display: inline-block;
  width: 0.2em;
}
</style>
