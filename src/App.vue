<script setup>
// https://www.studytonight.com/post/javascript-speech-recognition-example-speech-to-text

import { ref } from 'vue'
import { useAVLine } from 'vue-audio-visual'
import axios from 'axios'

const player = ref(null)
const canvas = ref(null)
let mySource = ref(null)
let action = ref('')
let output = ref('')

useAVLine(player, canvas, { src: mySource, canvHeight: 300, canvWidth: 1000, barColor: 'lime' })

const runSpeechRecognition = () => {
    var SpeechRecognition = SpeechRecognition || webkitSpeechRecognition;
    var recognition = new SpeechRecognition();

    recognition.onstart = () => { action.value = "listening, please ask your question..." };
    
    recognition.onspeechend = () => {
        action.value = "stopped listening...";
        recognition.stop();
    }
  
    recognition.onresult = async (event) => {
      var transcript = event.results[0][0].transcript;
        output.value = transcript

        try {
          let res = await axios.post('http://localhost:4001/api/text-to-audio-file', {
            text: event.results[0][0].transcript
          })

          if (res.data) {
            mySource.value = '/voice/' + res.data + '.mp3'
            setTimeout(() => { player.value.play() }, 500)
          }
        } catch (err) {
          console.log(err)
        }
    };
    recognition.start();
}

</script>

<template>

  <div class="btn-section">
    <button type="button" @click="runSpeechRecognition()">Ask question</button>
  </div>
  <div class="display-section">
      <div class="action" v-if="action">{{ action }}</div>
      <div class="output" v-if="output"><b>Question</b>: {{ output }}</div>
  </div>

  <div>
    <audio id="player" ref="player" :src="mySource" type="audio/mpeg" controls hidden></audio>
    <canvas ref="canvas" />
  </div>
</template>

<style>
  body {
    background-color: rgb(23, 23, 23);
  }

  canvas {
      padding: 0;
      margin: auto;
      display: block;
      width: 800px;
      position: absolute;
      top: 0;
      bottom: 0;
      left: 0;
      right: 0;
  }
  .btn-section {
    display: flex;
    justify-content: center;
    margin-top: 30px;
  }
  button {
    padding: 8px 13px;
    border-radius: 5px;
    background-color: rgb(81, 148, 81);
    color: white;
    font-weight: 700;
    font-size: 18px;
    border: none;
    cursor: pointer;
  }
  .display-section {
    width: 100%;
    text-align: center;
    color: white;
  }
  .action {
    margin-top: 10px;
    margin-bottom: 10px;
  }
  .output {
    max-width: 500px;
    padding: 20px;
    border-radius: 10px;
    border: 1px dotted white;
    display: inline-block;
  }
</style>
