<template>
  <div>    
    <div class="player-container" >
      <video :src="videoSource" width="100%" height="500" controls ref="videoPlayer" ></video>
    </div>
    <div class="container">
      <h4 >
        {{videoInfo.title}}
      </h4>
      <strong>Category: </strong><span>{{videoInfo.category}}</span>
      <br>
      <strong>Anime Info: </strong><span>{{videoInfo.animeInfo}}</span>
      <hr>
      <episode-list :paginatedEpisodeList="paginatedEpisodeList" @selectedEpisode="gotoEpisode" :activeEpisode="episodeNumber" @changeEpisode="changeEpisode" ></episode-list>
      
    </div>
  </div>
</template>

<script>
  const cheerio = require("cheerio");
  import EpisodeList from './EpisodeList';
  export default {

    components: {
      EpisodeList
    },

    data: () => ({

      videoSource: '',
      videoInfo: {
        title: '',
        category: '',
        animeInfo: ''
      },
      episodeList: [],
      paginatedEpisodeList: [],
      start: 0,
      end: 20,
      episodeNumber: 0,
      totalEpisodes: 0,
      

    }),

    computed: {
    
    },

    methods: {

      getVideo() {

        this.$http.get( this.$url + this.$route.query.title )
        .then( (response) => {
          const html = response.data;

        // get additional information of the selected anime
          this.getSelectedVideoInfo( html );

          const $ = cheerio.load( html );

          let ajaxRequest = $('li[class="vidcdn"]').find('a').attr('data-video').replace('load.php','ajax.php') + '&refer=https://gogoanime.ai/';
          this.$http.get('https:' + ajaxRequest )
          .then(( res ) => {
            this.videoSource = res.data.source[0].file;
            this.$refs.videoPlayer.src = this.videoSource;
          });

        });
      },

      getSelectedVideoInfo( html ) {

        const $ = cheerio.load( html );
        this.videoInfo.title = $('div[class="title_name"]').find('h2').text();
        this.videoInfo.category = $('div[class="anime_video_body_cate"]').find('a').first().text();
        this.videoInfo.animeInfo = $('div[class="anime-info"]').find('a').text();

      },

      gotoEpisode( selectedEpisodeNumber ) {

        this.updateEpisodeNumber();
        this.$router.push({path: '/watch', query: { title: this.$route.query.title.replace( this.episodeNumber, selectedEpisodeNumber), totalEpisodes: this.totalEpisodes  }});
        
      },

      updateEpisodeNumber() {

        let query = this.$route.query.title.split('-')
        this.episodeNumber = parseInt( query[ query.length - 1 ] );

      },

      changeEpisode( val ) {

        this.gotoEpisode( this.episodeNumber + val );

        if( this.episodeNumber === 1 && val === -1 )  {

          alert('you have reached the beginning of the series.');

        } else if ( !this.paginatedEpisodeList.includes( this.episodeNumber ) ) {
          
          this.buildPaginatedEpisode();

        }

      },

      buildPaginatedEpisode() {
        //TODO: add handling for pagination when user reload s the page and the current episode is not in the list
        this.paginatedEpisodeList = this.episodeList.slice( this.start, this.end );

      }
     
    },

    mounted() {

      this.getVideo(); 
      this.updateEpisodeNumber();

      if ( this.$route.query.totalEpisodes ) {
          this.totalEpisodes = this.$route.query.totalEpisodes;
          this.episodeList = new Array( parseInt( this.totalEpisodes ) );
          this.episodeList = [ ...Array( parseInt( this.totalEpisodes ) ).keys() ]
          this.episodeList = this.episodeList.slice().reverse();
      }

      this.buildPaginatedEpisode();

    },

    beforeRouteUpdate( to, from, next ) {
      next();

    },

     watch: {
      $route( to, from ) {
        // react to route changes...
        if ( to.query != from.query ) {
          this.getVideo();
          this.updateEpisodeNumber();
        }
      },

      // watch the episode number to change the paginated episode list
      episodeNumber( nv ) {
        if (( nv - 1 ) === this.paginatedEpisodeList.slice( -1 )[0] ) {
          this.start = this.end - 1;
          this.episodeList.length > this.end + 20 ? this.end += 20 : this.end = this.episodeList.length
          this.buildPaginatedEpisode();
        }
      }
  }

  }
</script>

<style scoped>
  iframe {
    width: 100%;
    height: 500px;
  }

  ul > li {
    display: inline-block;
    /* You can also add some margins here to make it look prettier */
    zoom:1;
    *display:inline;
    /* this fix is needed for IE7- */
  }

  .m-5 {
    margin: 5px !important;
  }

  .mr-10 {
    margin-right: 10px
  }

</style>