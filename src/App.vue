<template>
  <div class="overlay">
    <button class="help-btn" @click="toggleHelpText">show positions</button>
    <button class="help-btn" @click="start">start</button>
  </div>
  <div class="circle-light-wrapper" :class="classesComputed" v-if="currentLightsComputed">
    <template v-for="(light, i) in currentLightsComputed" :key="i">
      <circle-light
        :color="light.color"
        :transitionAttack="light.transitionAttack"
        :holdColor="light.holdColor"
        :pos="i+1"
      />
      <div v-if="light.break" class="break"></div>
    </template>
  </div>
  <audio id="song" controls src="/song.mp3"></audio>
</template>

<script>
import CircleLight from './components/CircleLight.vue'

export default {
  name: 'App',
  components: {
    CircleLight
  },
  computed: {
    tempoComputed(){
      return (60/this.bpm)*1000;
    },
    classesComputed(){
      if(this.showHelpText) return "show-help-text";
      return "";
    },
    lightsComputed() {
      return this.lights.map((light,i) => {
        let obj = {...light};
        obj.pos = i+1;
        return obj;
      });
    },
    currentLightsComputed(){
      if(this.currentPattern != undefined) {
        return this.lights.map((light, i) => {
          let newLight = {...light};
          let lightPattern = this.currentPattern.lights.find(light => light.pos === i+1);
          if(lightPattern != undefined) newLight = {...lightPattern, ...newLight};
          //console.log(newLight);
          return newLight;
        }, this)
      }
      else {
        return this.lights.map(light => {
          let newLight = {...light};
          newLight.color = 'transparent';
          return newLight;
        }, this)
      }
    }
  },
  mounted() {
    this.initAudioVariables();
    this.initEventListeners();
  },
  data(){
    return {
      audioContext: null,
      track: null,
      audioElm: null,
      audioSource: null,
      audioBuffer: null,
      showHelpText: false,
      beat: 0,
      loop: null,
      beatRange: 10, 
      bpm: 172,
      currentPattern: null,
      currentLights: null,
      lights: [
        //15
        {
          break: true
        },
        { },
        {
          break: true
        },
        { },
        { },
        {
          break: true
        },
        { },
        { },
        { },
        {
          break: true
        },
        { },
        { },
        {
          break: true
        },
        { },
        { }
      ]
      ,
      patterns: [
        {
          beat: 1,
          lights: [
            { pos: 9, color: '#C9CBA3', transitionAttack: 1 },
          ]
        },
        {
          beat: 2,
          lights: [
            { pos: 4, color: '#C9CBA3' },
          ]
        },
        {
          beat: 3,
          lights: [
            { pos: 5, color: '#C9CBA3' },
          ]
        },
        {
          beat: 4,
          lights: [
            { pos: 6, color: '#C9CBA3' },
          ]
        },
        {
          beat: 5,
          lights: [
            { pos: 7, color: '#C9CBA3' },
          ]
        },
        {
          beat: 7,
          lights: [
            { pos: 1, color: '#C9CBA3' },
          ]
        },
        {
          beat: 6,
          lights: [
            { pos: 9, color: '#C9CBA3', transitionAttack: 1 },
          ]
        },
      ]
    }
  },
  methods: {
    beatLoop(){
      this.beat = (this.beat + 1) % (this.beatRange+1);
      if(this.beat === 0) this.beat = 1;

      this.currentPattern = this.patterns.find(pattern => pattern.beat === this.beat);
      //console.log(this.currentLightsComputed);
    },
    toggleHelpText(){
      this.showHelpText = !this.showHelpText;
    },
    async initAudioVariables(){
       if(this.audioContext == null) this.audioContext = new AudioContext();
      // if(this.audioElm == null) this.audioElm = document.querySelector('#song');
      // if(this.track == null) this.track = this.audioContext.createMediaElementSource(this.audioElm);
      // this.track.connect(this.audioContext.destination);

      if(this.audioSource == null) 
        this.audioSource = this.audioContext.createBufferSource();
      
      let self = this;
      this.audioBuffer = await fetch("/song.mp3")
        .then(res => res.arrayBuffer())
        .then(ArrayBuffer => self.audioContext.decodeAudioData(ArrayBuffer));

      this.audioSource.buffer = this.audioBuffer;
      this.audioSource.connect(this.audioContext.destination);
      this.audioSource.loop = true;

    },
    initEventListeners(){
      //this.audioElm.addEventListener('ended', this.onEnded);
      //this.audioElm.addEventListener('timeupdate', this.onEnded);
    },
    start(){
      this.audioSource.start(0);
      this.loop = window.setInterval(this.beatLoop, this.tempoComputed);
    },
  }
}
</script>

<style lang="scss">
body{
  background-color: #1d1d1d;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  margin: 0;
  padding: 0;
  position: relative;
}
.overlay{
  position: absolute;
  width: 100%;
  top: 0;
  right: 0;
  display: flex;
  justify-content: flex-end;

  .help-btn{
    --btn-color: rgb(57, 99, 175);
    background-color: var(--btn-color);
    border: 3px solid var(--btn-color);
    font-weight: bold;
    border-radius: 2px;

    &:hover{
      cursor: pointer;
    }
  }
}
.circle-light-wrapper{
  height: 100vh;
  display: flex;
  flex-flow: row wrap;
  align-items: center;
  
  justify-content: center;
  border-radius: 70% 70% 50% 50%;
  background-color: rgb(221, 221, 221);
  box-shadow: 0px 60px 30px rgba(0, 0, 0, 0.459),
              0 -10px 20px gray inset;
}
.break{
    width: 100%;
}
</style>
