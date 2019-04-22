<template>
  <v-container class="player" grid-list-md text-xs-center>
    <v-layout row align-center justify-center>
      <v-flex xs2 class="player__controls">
        <v-icon @click="playPrevious()" large>skip_previous</v-icon>
        <v-icon v-if="playing && !paused" @click="pause()" large>pause</v-icon>
        <v-icon v-else @click="play()" large>play_arrow</v-icon>
        <v-icon @click="playNext()" large>skip_next</v-icon>
      </v-flex>
      <v-flex xs8>
        <v-slider v-model="currentProgress.percents" @change="changeProgress"></v-slider>
      </v-flex>
      <v-flex xs2>
        <v-slider v-model="volume" @change="changeVolume"></v-slider>
      </v-flex>
    </v-layout>
    <v-layout row align-center justify-center>
      <v-flex xs12>
        <v-card>
          <v-card-title>
            <v-text-field
                v-model="search"
                append-icon="search"
                label="Поиск"
                single-line
                hide-details
            ></v-text-field>
            <v-spacer></v-spacer>
          </v-card-title>
          <v-data-table
              :headers="headers"
              :items="tracks"
              :search="search"
              hide-headers
              hide-actions
          >
            <template slot="items" slot-scope="props">
              <tr @click="setTrack(props.item)">
                <td class="d-flex align-center">
                  <template v-if="currentTrack && props.item.file === currentTrack.file">
                    <span>{{ currentProgress.seconds }} sec</span>
                    <v-icon>arrow_right_alt</v-icon>
                  </template>
                </td>
                <td>{{ props.item.artist }}</td>
                <td class="text-xs-left">{{ props.item.track }}</td>
                <td class="text-xs-right">{{ props.item.duration }}</td>
                <td><v-spacer></v-spacer></td>
              </tr>
            </template>
            <v-alert slot="no-results" :value="true" color="error" icon="warning">
              Не найдено
            </v-alert>
          </v-data-table>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
  export default {
    name: 'MyPlayer',
    data () {
      return {
        headers: [
          { text: 'Artist', value: 'artist', sortable: false },
          { text: 'Track', value: 'track', sortable: false },
          { text: 'File', value: 'file', sortable: false}
        ],
        tracks: [
          { artist: 'Breath', track: 'Jazz-sax-2', file: '/tracks/1.mp3', duration: '00:05' },
          { artist: 'Super', track: 'Compressed-jazzkit', file: '/tracks/2.mp3', duration: '00:05'},
          { artist: 'Rhodes', track: 'Rhodes-walkdown', file: '/tracks/3.mp3', duration: '00:04'},
          { artist: 'Tylersrevenge', track: 'Idiosyncrasy', file: '/tracks/4.mp3', duration: '02:38'}
        ],
        search: '',
        audio: null,
        currentTrack: null,
        currentProgress: {
          percents: 0,
          seconds: 0,
        },
        volume: 100,
        paused: false,
        playing: false,
        events: {
          ended: () => {
            this.playNext()
          },
          timeupdate: () => {
            this.currentProgress.percents = Math.floor(this.audio.currentTime / this.audio.duration * 100)
            this.currentProgress.seconds = Math.floor(this.audio.currentTime)
          }
        }
      }
    },
    methods: {
      play () {
        if (!this.currentTrack) {
          this.currentTrack = this.tracks[0]
        }
        if (this.audio !== null) {
          if (this.paused) {
            this.paused = false
            this.playing = true
            this.audio.play()
            return
          }
          this.audio.pause()
          this.audio.src = this.currentTrack.file
          this.audio.load()
        } else {
          this.audio = new Audio(this.currentTrack.file)
          this.audio.addEventListener('ended', this.events.ended)
          this.audio.addEventListener('timeupdate', this.events.timeupdate)
        }
        this.audio.playbackRate = 1
        this.audio.volume = this.volume / 100
        this.playing = true
        this.audio.play()
      },
      changeVolume (value) {
        if (this.audio) this.audio.volume = value / 100
        this.volume = value
      },
      setTrack (item) {
        if (item.file === this.currentTrack.file) {
          if (this.paused) {
            this.play()
          } else {
            this.pause()
          }
          return
        }
        this.currentTrack = item
        this.paused = false
        this.play()
      },
      pause () {
        this.audio.pause()
        this.paused = true
      },
      playNext () {
        const index = this.currentIndex
        let nextIndex = index + 1
        if (nextIndex >= this.tracks.length) {
          nextIndex = 0
        }
        this.currentTrack = this.tracks[nextIndex]
        this.play()
      },
      playPrevious () {
        const index = this.currentIndex
        let previousIndex = index - 1
        if (previousIndex < 0) {
          previousIndex = this.tracks.length - 1
        }
        this.currentTrack = this.tracks[previousIndex]
        this.play()
      },
      changeProgress (value) {
        this.audio.currentTime =  value * (this.audio.duration / 100)
      }
    },
    computed: {
      currentIndex () {
        return this.tracks.findIndex(item => this.currentTrack.file === item.file)
      }
    }
  }
</script>

<style lang="scss" scoped>
  .player {
    max-width: 800px;
    &__controls {
      min-width: 120px;
    }
  }
</style>
