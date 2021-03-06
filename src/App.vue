<template>
<div>
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
        :ease="light.ease"
        :pos="i+1"
        :track="light.track"
        @clicked="onLightClicked"
      />
      <div v-if="light.break" class="break"></div>
    </template>
  </div>
</div>
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
    },
    patternsComputed(){
      return this.patterns;
    }
  },
  mounted() {
    this.initAudioVariables();
    let clusterBl = [
      { pos: 14, color: '#723d46', transitionAttack: 2, holdColor: true, ease: this.cubicInOut },
      { pos: 7, color: '#e26d5c', transitionAttack: 2, holdColor: true, ease: this.cubicInOut },
    ];

    let clusterBlEnd = this.updateCluster(
      clusterBl, 
      ["color", "holdColor"], 
      ["transparent", false]
    );

    let clusterTr = [
      // { pos: 4, color: '#FFBF47', transitionAttack: 2, holdColor: true, ease: this.cubicInOut },
      { pos: 3, color: '#FFBF47', transitionAttack: 2, holdColor: true, ease: this.cubicInOut },
    ];

    let clusterTrEnd = this.updateCluster(
      clusterTr, 
      ["color", "holdColor"], 
      ["transparent", false]
    );

    let clusterSlow1 = [
      { pos: 7, color: '#e26d5c', transitionAttack: 3, holdColor: true, ease: this.slowEaseIn },
      { pos: 6, color: '#e26d5c', transitionAttack: 3, holdColor: true, ease: this.slowEaseIn },
      { pos: 13, color: '#723d46', transitionAttack: 3, holdColor: true, ease: this.slowEaseIn },
    ];

    let clusterSlow1End = this.updateCluster(
      clusterSlow1, 
      ["color", "holdColor"], 
      ["transparent", false]
    );

    let clusterWave = [
      { pos: 14, color: '#723d46', transitionAttack: 2, holdColor: true, ease: this.slowEaseIn },
      { pos: 11, color: '#FFBF47', transitionAttack: 2, holdColor: true, ease: this.slowEaseIn },
      { pos: 13, color: '#723d46', transitionAttack: 2, holdColor: true, ease: this.slowEaseIn },
      { pos: 7, color: '#723d46', transitionAttack: 2, holdColor: true, ease: this.slowEaseIn },
      { pos: 6, color: '#FFBF47', transitionAttack: 2, holdColor: true, ease: this.slowEaseIn },
      { pos: 2, color: '#723d46', transitionAttack: 2, holdColor: true, ease: this.slowEaseIn },
      { pos: 1, color: '#FFBF47', transitionAttack: 2, holdColor: true, ease: this.slowEaseIn },
    ];

    let clusterWaveEnd = this.updateCluster(
      clusterWave, 
      ["color", "holdColor"], 
      ["transparent", false]
    );
    
    this.offsetClusterToLoop(clusterBl, 1, 2);
    this.offsetClusterToLoop(clusterBlEnd, 7, 3);

    this.appendPatternToBeat(clusterTr, 5);
    this.appendPatternToBeat(clusterTrEnd, 11);

    this.offsetClusterToLoop(clusterSlow1, 20, 1);
    this.offsetClusterToLoop(clusterSlow1End, 26, 1);

    console.log("clusterWave:", clusterWave);
    this.offsetClusterToLoop(clusterWave, 24, 1);
    this.offsetClusterToLoop(clusterWaveEnd, 28, 1);
  },
  data(){
    return {
      audioContext: null,
      track: null,
      showHelpText: false,
      numAudioSources: 0,
      numActiveAudio: 1,
      beat: 0,
      loop: null,
      beatRange: 40, 
      bpm: 200,
      currentPattern: null,
      currentLights: null,
      cubicInOut: "cubic-bezier(0.4, 0, 0.2, 1)",
      slowEaseIn: "cubic-bezier(0.76, 0.07, 0.63, 0.82)",
      lights: [
        //15
        {
          break: true,
          track: 2,
        },
        {  },
        {
          track: 9,
          break: true,
        },
        { },
        { track: 7 },
        {
          break: true
        },
        { track: 4 },
        { track: 10 },
        { track: 3 },
        {
          break: true
        },
        { track: 8 },
        { track: 5 },
        {
          break: true
        },
        { track: 6 },
        { }
      ],
      patterns: []
    }
  },
  methods: {
    onLightClicked(track){
      console.log(this);
      this["gain"+track+"Active"] = !this["gain"+track+"Active"];

      if(this["gain"+track+"Active"]){
        ++this.numActiveAudio;
        this["gain"+track].gain.exponentialRampToValueAtTime(0.7, this.audioContext.currentTime + 2)
      } 
      else {
        --this.numActiveAudio;
        this["gain"+track].gain.exponentialRampToValueAtTime(0.1, this.audioContext.currentTime + 2)
      }
    },
    beatLoop(){
      this.beat = (this.beat + 1) % (this.beatRange+1);
      if(this.beat === 0) this.beat = 1;
      this.currentPattern = this.patterns.find(pattern => pattern.beat === this.beat);
    },
    toggleHelpText(){
      this.showHelpText = !this.showHelpText;
    },
    async initAudioVariables(){
      if(this.audioContext == null) this.audioContext = new AudioContext();

      await this.createAudioSource("/real_final/base.mp3");

      await this.createAudioSource("/real_final/add_2.mp3");
      await this.createAudioSource("/real_final/add_3.mp3");
      await this.createAudioSource("/real_final/add_1.mp3");
      await this.createAudioSource("/real_final/kicks.wav");
      await this.createAudioSource("/real_final/hats.wav");
      await this.createAudioSource("/real_final/add_4.mp3");
      await this.createAudioSource("/real_final/add_5.mp3");
      await this.createAudioSource("/real_final/add_6.mp3");
      await this.createAudioSource("/real_final/add_7.mp3");
    },
    start(){
      for(let i = 1; i <= this.numAudioSources; i++) {
        this["audioSource" + i].start(0);

        // init base track as on and the rest as muted
        if(i === 1) this["gain" + i].gain.setValueAtTime(1, 0);
        else this["gain" + i].gain.setValueAtTime(0.01, 0);
      }
      this.loop = window.setInterval(this.beatLoop, this.tempoComputed);
    },
    async createAudioSource(filePath){
      let num = ++this.numAudioSources;
      this["audioSource" + num] = this.audioContext.createBufferSource();
      this["gain" + num] = this.audioContext.createGain();
      
      let self = this;
      this["audioBuffer" + num] = await fetch(filePath)
        .then(res => res.arrayBuffer())
        .then(ArrayBuffer => self.audioContext.decodeAudioData(ArrayBuffer));
      
      this["audioSource" + num].buffer = this["audioBuffer" + num];

      this["audioSource" + num].connect(this["gain" + num]);
      this["gain" + num].connect(this.audioContext.destination);
      this["audioSource" + num].loop = true;
    },
    appendPatternToBeat(lights, beat){
      let pattern = this.patterns.find(pattern => pattern.beat === beat);
      if(pattern !== undefined)
        pattern.lights = [...pattern.lights, ...lights];
      else if(beat <= this.beatRange) {
        pattern = {beat: beat, lights: [...lights]};
        this.patterns.push(pattern);
      }
    },
    updateCluster(cluster, props, values){
      let result = cluster.map(light => { return {...light} });
      result.forEach(light => {
        props.forEach((prop, i) => {
          light[prop] = values[i];
        })
      });
      return result;
    },
    offsetClusterToLoop(cluster, beat, offset){
      let index;
      for(let i = 0; i < cluster.length; i++){
        if(i > 0) index = beat + i + offset;
        if(i === 0) index = beat
        if(index > this.beatLoop) return;
        this.appendPatternToBeat([cluster[i]], index);
      }
    }
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
  overflow: hidden;
  background-color: white;
  box-shadow: 0px 60px 30px rgba(0, 0, 0, 0.459),
              0 -10px 20px gray inset;
}
.break{
    width: 100%;
}
</style>
