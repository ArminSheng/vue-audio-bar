<template>
  <div
    @click.prevent.stop="toggle"
    :class="{
      'voice-bg-right': right,
      'played': played
    }"
    class="voice-bg">
    <div
      :style="{
        backgroundImage: 'url(' + icons[speakerStatus] + ')'
      }"
      class="speaker"></div>
  </div>
</template>

<style lang="scss" scoped>
  .voice-bg {
    background-size: contain;
    height: 30px;
    width: 170px;
    background-repeat: no-repeat;
    position: relative;
    display: inline-block;
    background-image: url(../../assets/shequ/speaking_bg_left_n.png);
    &:active, &.played {
      background-image: url(../../assets/shequ/speaking_bg_left_s.png);
    }
  }
  .voice-bg-right {
    transform: rotateY(180deg);
    float: right;
  }

  .speaker {
    background-image: url(../../assets/shequ/icon_speaking_white_left_03.png);
    background-size: contain;
    position: absolute;
    right: 8px;
    top: 50%;
    transform: translateY(-50%);
    width: 20px;
    height: 20px;
    background-repeat: no-repeat;
  }
</style>

<script>
  import voice_3 from '../../assets/shequ/icon_speaking_white_left_03.png'
  import voice_2 from '../../assets/shequ/icon_speaking_white_left_02.png'
  import voice_1 from '../../assets/shequ/icon_speaking_white_left_01.png'

  // Create event bus
  import Vue from 'vue'
  const bus = new Vue()

  // Audio instance
  window.__BBC_VOICE_BAR__ = window.__BBC_VOICE_BAR__ ||
    new Audio()

  // Speaker icon
  const VOICE = [voice_1, voice_2, voice_3]

  // Intervaler
  let ST = ''

  // Audio 'onend' listeners
  const LISTENERS = []

  export default {
    name: 'audio-bar',
    props: {
      src: '',
      right: false
    },
    data () {
      return {
        playing: false,
        speakerStatus: 2,
        icons: VOICE,
        played: false,
        audio: window.__BBC_VOICE_BAR__
      }
    },

    watch: {
      src () {
        this.played = false
        this.stop()
      }
    },

    mounted () {
      const {audio, stop} = this
      LISTENERS.push(stop)

      if (!audio.onended) {
        audio.onended = () => {
          // Trigger
          LISTENERS.forEach(fn => fn())
        }
      }

      bus.$on('voicePlay', (voice) => {
        if (this.src !== voice) {
          this.playing && this.stop()
        }
      })
    },

    beforeDestroy () {
      this.destroy()
    },

    methods: {
      destroy () {
        this.stop()
      },

      stopSpeaker () {
        clearInterval(ST)
        this.speakerStatus = 2
      },

      play () {
        const {audio, src} = this
        audio.src = src
        this.played = true

        // Broadcast
        bus.$emit('voicePlay', src)

        // Shining speaker
        this.speakerEco()

        if (!audio.error) {
          audio.play()
          this.playing = true
        }
      },

      stop () {
        if (this.playing) {
          this.stopSpeaker()
          this.audio.pause()
          this.playing = false
        }
      },

      toggle () {
        const {stop, playing, play} = this
        playing ? stop() : play()
      },

      speakerEco () {
        ST = setInterval(() => {
          this.speakerStatus === 2
            ? this.speakerStatus = 0
            : this.speakerStatus++
        }, 500)
      }
    }
  }
</script>