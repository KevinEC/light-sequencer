<template>
<div class="circle-light-root">
  <div class="circle-light" :style="colorStyleComputed">
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
        }
    },
    data() {
        return {
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
                boxShadow: `0 0 10px ${this.colorComputed}`,
                transition: `background-color ${this.transitionDurationComputed}s ${this.ease},
                             box-shadow ${this.transitionDurationComputed}s ${this.ease}`
            }
        },
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
        //border: 1px solid rgb(119, 119, 119);
        border-radius: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
        color: transparent;
    }
}

.show-help-text{
    .circle-light{
        color: black;
    }
}
</style>