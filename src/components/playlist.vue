<template>
  <div class="playlist">
    <div class="row">
      <div class="left col-xs-12 col-lg-3 col-xl-4 col-sticky">
      <album-block :album-image="images[0].url"></album-block>
        <h2>{{playListName}}</h2>
        <div class="owner">
          <span>By</span>
          <a href="">{{owner.display_name}}</a>
        </div>
        <p class="song-count">{{tracks.length}} songs</p>
        <button class="btn btn-green" v-on:click="greenButtonClicked">{{playState}}</button>
      </div>
      <div class="right col-xs-12 col-lg-9 col-xl-8" >
        <track-display v-for="(track,index) in tracks" :track-title="track.track.name" :artist="track.track.artists" :album="track.track.album.name" :trackNum="index" :duration="track.track.duration_ms" v-on:togglePlay="trackButtonClicked"></track-display>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'playlist',
  data () {
    return {
      playListName: '',
      owner: '',
      tracks: [],
      images: '',
    }
  },
  methods: {
    getPlayList() {
      var playlistId = location.href.split('/').slice(-1)[0];
      console.log(playlistId);
      this.$http.get('https://api.spotify.com/v1/users/spotify/playlists/' + playlistId, {headers: {
        Authorization: 'Bearer ' + this.getLocalStorage('spotify-token')
      }})
      .then(function(res) {
        this.playListName = res.body.name;
        this.tracks = res.body.tracks.items;
        this.owner = res.body.owner;
        this.images = res.body.images;
        // console.log(this.images[0].url);
        console.log(this.tracks);
        console.log('res', res.body);

      }, function(error) {
        // console.log(error.bodyText);
        var errorContent = JSON.parse(error.bodyText);
        if (errorContent.error.message === 'The access token expired') {
          var authUrl = 'https://accounts.spotify.com/authorize?client_id=fb0e57d017fe4c1b817fd5f624ebdf4d&redirect_uri=http:%2F%2Flocalhost:8080%2Fcallback&scope=user-read-private%20user-read-email&response_type=token&state=123';
          this.setLocalStorage('previous-url', location.href);
          window.location.href = authUrl;
        }
      })
    },
    trackButtonClicked(trackNum) {
      console.log('state toggled!', this.tracks, trackNum);
      // this.isPlaying = this.isPlaying ? false : true;// don't change play state here, change it at the parent
      this.$emit('playTrack', this.tracks, trackNum);
      console.log('play!')
    },
    greenButtonClicked() {
      console.log('green button is clicked!');
      this.$emit('greenBtnClicked', this.tracks);
      console.log('green!')
    }
  },
  computed: {
    playState: function() {
      return this.$store.state.isPlaying ? "PAUSE" : "PLAY";
    }
  },
  created: function() {
    this.getPlayList();
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.btn-green {
  background-color: #1db954;
  padding: 11px 44px 12px;
  transition: all .2s;
}

.btn-green:hover {
  background-color: #1df269;
}

.song-count {
  text-transform: uppercase;
}

.row {
  margin: 0;
  padding: 2em;
}

.owner, .song-count {
  color: hsla(0,0%,100%,.6);
}

.owner a {
  color: white;
}

.left {
  text-align: center;
}
</style>
