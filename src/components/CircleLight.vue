<template>
<div class="circle-light-root" :class="track !== undefined ? 'interactive' : '' ">
  <div class="circle-light" :style="colorStyleComputed" @click="handleClick">
      {{ this.pos }}
  </div>
</div>
</template>

<script>
export default {
    props: {
        color: String,
        pos: Number,
        holdColor: Boolean,
        transitionAttack: {
            type: Number,
            default: null
        },
        ease: {
            type: String,
            default: "ease-out"
        },
        track: Number
    },
    data() {
        return {
            activated: false,
            backgroundColor: this.color,
            hold: false,
            internalColor: null,
            transitionDuration: null,
            transitionDurationDefault: 0.1
        }
    },
    watch: {
        transitionAttack(newValue, oldValue){
            if(newValue === null && oldValue !== null) this.transitionDuration = this.oldValue;
            if(newValue !== null) this.transitionDuration = newValue;
        },
        holdColor(newValue){
            if(newValue) this.internalColor = this.color;
            if(newValue != undefined) this.hold = newValue;
        }
    },
    computed: {
        colorComputed(){
            if(this.activated && this.color && this.color != "transparent" ){
                return this.color;
            }
            if(this.activated) {
              return "rgba(0,128,0, 0.3)";  
            } 
            if(this.hold) return this.internalColor;
            if(this.color) return this.color
            return "transparent";
        },
        transitionDurationComputed(){
            if(this.transitionDuration === null && this.transitionAttack === null) return this.transitionDurationDefault;
            return this.transitionDuration;
        },
        colorStyleComputed(){
            return {
                backgroundColor: this.colorComputed,
                boxShadow: `0 0 20px 30px ${this.colorComputed}`,
                transition: `background-color ${this.transitionDurationComputed}s ${this.ease},
                             box-shadow ${this.transitionDurationComputed}s ${this.ease}`
            }
        },
        activatedComputed(){
            return this.activated;
        },

    },
    methods:{
        handleClick(){
            console.log("clicked in child");
            this.activated = !this.activated;
            this.$emit('clicked', this.track);
        }

    }
}
</script>

<style lang="scss">
.circle-light-root{
    --size: 13vw;
    --max-size: 14vh;
    flex: var(--size) 0 0;
    height: var(--size);
    max-width:var(--max-size);
    max-height:var(--max-size);
    margin: 0 3vw;

    .circle-light{
        height: 100%;
        width: 100%;
        border-radius: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
        color: transparent;
    }
}
:root{
    --interactive-color: rgba(0,128,0, 0.1);
    --interactive-color-intense: rgba(0,128,0, 0.3);
}

.circle-light-root.interactive{
    animation: MovingGlow 3s infinite;
    border-radius: 100%;
}

@keyframes MovingGlow {
    0%{
        box-shadow: 5px 5px 10px var(--interactive-color),
                inset 0 0 10px var(--interactive-color);
    }

    75%{
        box-shadow: 2px -1px 10px var(--interactive-color-intense),
                inset 0 0 10px var(--interactive-color-intense);
    }

    90%{
        box-shadow: -1px 5px 10px var(--interactive-color),
                inset 0 0 10px var(--interactive-color);
    }
}

.show-help-text{
    .circle-light{
        color: black;
    }
}
</style>