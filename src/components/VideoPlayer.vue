<template>
    <section id="video-player-panel">
        <div class="video-player">
            <div class="video">
                <video-window :source="id" :plStatus="isPlaylistChanged"></video-window>
            </div>
            <form class="playlist-select" v-on:submit.prevent v-if="!disabled">
                <span v-if="addPlaylist==1">
                    <input type="text" name="addfeature" v-model="playlistname" class="playlist-add" placeholder="playlist name">
                    <!-- <img class="add-icon" :src="require('@/assets/icons/add-playlist-button.svg')" @click="addNewPlaylist"> -->
                    <button class="playlist-add-button" @click="addNewPlaylist">Save</button>
                    <!-- <img class="cancel-icon" :src="require('@/assets/icons/cancel-button.svg')" @click="addPlaylist=0"> -->
                    <button class="playlist-add-button" @click="addPlaylist=0">Cancel</button>
                </span>
                <span v-else-if="addPlaylist==-1">
                    Are you sure you want to delete the playlist? It is permanent!
                    <button class="playlist-add-button" @click="removePlaylist">Yes</button>
                    <!-- <img class="remove-icon" :src="require('@/assets/icons/remove-button.svg')" @click="removePlaylist"> -->
                    <button class="playlist-add-button" @click="addPlaylist=0">Cancel</button> 
                </span>
                <span class="smallTxt" v-else>
                    <select class="select-channel" v-model="playN"  @click="changePlaylist">
                        <option :value="i"  v-for="(playlist,i) in playlists" :key="i" >{{playlist.name}}</option>
                    </select>
                    <!-- <img class="add-icon" :src="require('@/assets/icons/add-playlist-button.svg')" @click="addPlaylist=1"> -->
                    <button class="playlist-add-button" @click="addPlaylist=1" >Create a palylist</button>
                    <button class="playlist-add-button" @click="savePlaylist" :class="[isPlaylistChanged === 1 ? 'redPlayList' : 'greenPlayList']">Save palylist</button>
                    <!-- <img class="remove-icon" :src="require('@/assets/icons/remove-button.svg')" @click="removePlaylist"> -->
                    <button class="playlist-add-button" @click="addPlaylist=-1">Remove</button>
                </span>
            </form>
                <set-time class="timer"/>

            <draggable v-model="playlists[playN].videos" class="playlist" @end="isPlaylistChanged=1">
                <div class="video-item" v-for="(video,i) in playlists[playN].videos" :key="i">
                    <img :src="video.thumbnail">
                    <p class="video-title" v-html="video.title"  ></p>
                    <div class="video-playing" v-if="video.id === playingVideoId" >
                        <img style="width: 50%;margin: 0 auto;" src="@/assets/icons/nowPlayingIcon.png">
                    </div>
                    <div class="video-details" :title="video.title | requotes">
                        <div class="play" @click="playVideo(video, i)">
                            <img :alt="video.title" :src="require('@/assets/icons/play-button.svg')">
                        </div>
                        <div class="remove" v-on:click="removeFromPlaylist(i)" v-if="!disabled">
                            <img :src="require('@/assets/icons/remove-button.svg')">
                        </div>
                        <div class="search" @click="searchVideo(video.title)" v-if="!disabled">
                            <img :src="require('@/assets/icons/search-button.svg')">
                        </div>
                    </div>
                </div>
            </draggable>

            <div class="playlist-highlight" v-if="playlistHighlight">
                <div class="heading">
                    <h3>Playlist</h3>
                </div>
                <draggable v-model="playlists[playN].videos" class="playlist" @end="isPlaylistChanged=1">
                    <div class="video-item" v-for="(video,i) in playlists[playN].videos" :key="i">
                        <img :src="video.thumbnail">
                    <p class="video-title" v-html="video.title"  ></p>
                        <div class="video-details" >
                            <div class="play" @click="playVideo(video, i)">
                                <img :src="require('@/assets/icons/play-button.svg')">
                            </div>
                            <div class="remove" v-on:click="removeFromPlaylist(i)" v-if="!disabled">
                                <img :src="require('@/assets/icons/remove-button.svg')">
                            </div>
                            <div class="search" @click="searchVideo(video.title)" v-if="!disabled">
                                <img :src="require('@/assets/icons/search-button.svg')">
                            </div>
                        </div>
                    </div>
                </draggable>
            </div>
        </div>
    </section>    
</template>

<script>

import VideoWindow from './VideoPlayer/Video'
import axios from 'axios'
import draggable from 'vuedraggable'
import SetTime from './setTime.vue';

export default {
    name: 'video-player',
    data(){
        return{
            id: null,
            playlists : [],
            isPlaylistChanged: 0,
            currentPlay: 0,
            disabled: false,
            playN: 0,
            addPlaylist: 0,
            playlistname: '',
            playlistHighlight: false,
            currentHoverItem : '',
            playingVideoId: null
        }
    },
    methods: {
        playVideo(item, i){
            this.checkPlayAllow(item.id, i)
            // this.$root.$emit('playerPlay')
        },
        removeFromPlaylist(i){
            this.playlists[this.playN].videos.splice(i, 1);
            this.isPlaylistChanged = 1
        },
        searchVideo(title){
            this.$root.$emit('searchRelated', title)
        },
        shuffle() {
            this.playlists[this.playN].videos.sort(() => Math.random() - 0.5);
        },
        changePlaylist(){
            
        },
        addNewPlaylist(){
            let playlist  = {
                name: this.playlistname,
                videos: []
            }

            this.playlists.push(playlist)
            this.addPlaylist = 0
            this.isPlaylistChanged = 1
        },
        removePlaylist(){
            this.playlists.splice(this.playN, 1);
            this.playN = 0
            this.isPlaylistChanged = 1
            this.addPlaylist = 0
        },
        savePlaylist() {
            let self = this
            axios({
              method: 'post',
              url: `https://imasparkle.org:8443/save/`,
              data: this.playlists,
              headers:{
                "Authorization" : "Token "+ localStorage.Token
              }
            })
            .then(function(response){
                self.isPlaylistChanged = 0
            })
            .catch(function(err){
                console.log("Login Required to save", err)
            });
        },
        checkPlayAllow(id, i) {
            let date = new Date()
            let savedTimer = localStorage.getItem('SetTimer') ? localStorage.getItem('SetTimer') : null
            if (savedTimer && savedTimer !== 'null') {
                let timerTime = Math.ceil((new Date(savedTimer).getTime() - date.getTime()) / 1000)
                if(timerTime > 0) {
                    this.currentPlay = i
                    this.playingVideoId = id
                    localStorage.setItem('playingVideoId', id)
                    this.id = id
                }
            } else {
                this.currentPlay = i
                this.playingVideoId = id
                localStorage.setItem('playingVideoId', id)
                this.id = id
            }
        }
    },
    components: {
        'video-window' : VideoWindow,
        SetTime,
        draggable
    },
    watch: {
        playN (newValue, oldValue) {
            if(newValue !== oldValue) {
                this.$root.$emit('setVideoToPlay', this.playlists[this.playN].videos[0].id)
                this.playingVideoId = this.playlists[this.playN].videos[0].id
                localStorage.setItem('playingVideoId', this.playlists[this.playN].videos[0].id)
            }
        }
    },
		filters:{
       	 requotes(val) {
       	     let res = val.replace("&quot;", "\"")
       	     res = res.replace("&#39;", "'")
       	     res = res.replace("&amp;", "&")
         	   return res
        	}
    	},
    mounted(){
        let self = this
        this.playingVideoId = localStorage.getItem('playingVideoId')
        this.$root.$on('shuffle', ()=>{
            this.shuffle(this.playlists[this.playN].videos)
            this.currentPlay = -1
        });


        this.$root.$on('play',(id)=>{
            this.checkPlayAllow(id)
        });


        this.$root.$on('addToPlaylist', (video)=>{
            let add = true;
            for(let i=0;i<this.playlists[this.playN].videos.length;i++){
                if(this.playlists[this.playN].videos[i].id == video.id){
                    console.log('same id')
                    add = false;
                    break;
                }
            }
            if(add){
                this.playlists[this.playN].videos.push(video);
                this.isPlaylistChanged = 1;
            }
        });


        axios({
          method: 'post',
          url: `https://imasparkle.org:8443/playlist/`,
          data: '',
          headers:{
            "Authorization" : "Token "+ localStorage.Token
          }
        })
        .then(function(response){
            self.playlists = JSON.parse(response.data)
            self.playVideo(self.playlists[self.playN].videos[self.currentPlay], self.currentPlay)
        })
        .catch(function(err){
            console.log("Login Required to save")
        });
        
        this.$root.$on('playNext',()=>{
            this.currentPlay += 1
            console.log(this.currentPlay)
            this.playVideo(this.playlists[this.playN].videos[this.currentPlay], this.currentPlay)
            if(this.currentPlay == this.playlists[this.playN].videos.length){
                this.currentPlay = 0
            }
        });

        this.$root.$on('playPrev',()=>{
            if(this.currentPlay > 0){    
                this.currentPlay -= 1
                this.playVideo(this.playlists[this.playN].videos[this.currentPlay], this.currentPlay)
            }
        });

        this.$root.$on('playChannel',channel=>{
            this.playlists = channel
            this.currentPlay = 0
            this.disabled = true
        });

        this.$root.$on('isSearch', (state)=>{
            this.playlistHighlight = state
        })

        this.$root.$on('changePlaylist', (_)=>{

                this.playN = this.playlists.findIndex(k => k.name == _ )
                this.playVideo(this.playlists[this.playN].videos[this.currentPlay], this.currentPlay)

        });

        this.$root.$on('playPlaylist', ()=>{
            let self = this
            axios({
              method: 'post',
              url: `https://imasparkle.org:8443/playlist/`,
              data: '',
              headers:{
                "Authorization" : "Token "+ localStorage.Token
              }
            })
            .then(function(response){
                self.playlists = JSON.parse(response.data)
                self.currentPlay = 0
                self.playN = 0
                self.playVideo(self.playlists[self.playN].videos[self.currentPlay], self.currentPlay)
                self.disabled = false
            })
            .catch(function(err){
                console.log(err)
            });
        });
    }
};
</script>

<style scoped>


.video-player{
    display: flex;
    flex-direction: column;
    text-align: center;
    margin-top: 40px;
}


.playlist{
    /* display: grid; */
    /* grid-template-columns: repeat(4, 1fr); */
    width: 100%;
    /* grid-gap: 15px 10px; */
    padding: 10px;
    height: 250px;
    background: #535353;
    overflow-y: scroll;
    margin: 0 auto;
    overflow-x: hidden;
    -webkit-overflow-scrolling: touch;
    box-sizing: border-box;
}
.playlist::-webkit-scrollbar {
    -webkit-appearance: none;
  width: 20px;
}

/*@media only screen  and (max-width: 1000px){
    .playlist{
        width: 91%;
        grid-template-columns: repeat(3, 1fr);
        grid-template-rows: repeat(8, 8.5vw);
    }
    .playlist .video-item{
    }
}*/

.playlist .video-item{
    width: 80px;
    position: relative;
    height: 75px;
    float: left;
    margin-left: 12px;
    
}

.playlist .video-item img{
    width: inherit;
}

.playlist .video-item img:hover{
    cursor: pointer;
}
.redPlayList {
    background: red;
    color: white;
}
.greenPlayList {
    background: green;
    color: white;
}
.playlist .video-item .video-details {
  transition: .5s ease;
  position: absolute;
  opacity: 0;
  left: 50%;
  top: 30%;
  transform: translate(-50%, -50%);
  -ms-transform: translate(-50%, -50%);
  text-align: center;
  width: 100%;
  height: 61%;
  display: flex;
  align-items: center;
  background: rgba(0,0,0,0.5);
  z-index: -1;
}

.playlist .video-item:hover > .video-details{
    z-index: 0;
    opacity: 1;
}

.playlist .video-item .video-playing {
  transition: .5s ease;
  position: absolute;
  left: 50%;
  top: 30%;
  transform: translate(-50%, -50%);
  -ms-transform: translate(-50%, -50%);
  text-align: center;
  width: 100%;
  height: 61%;
  display: flex;
  align-items: center;
  background: rgb(255, 255, 255, 0.7);
}

.playlist .video-item .video-details div{
    width: 100%;
    justify-content: center;
}
.playlist .video-item .video-details div img{
    width: 15px;
}

.playlist-select{
    margin-bottom: 10px;
}

.playlist-add-button{
    padding: 4px 6px;
    margin-left: 5px;
    font-size: 0.8em;
    border-radius: 12px;
    border: 1px solid gray;
    cursor: pointer;
}

.playlist-button{
    margin-left: 5px;
}

.add-icon{
    width: 25px;
    margin-left: 10px;
    cursor: pointer;
    position: absolute;
}

.cancel-icon{
    width: 28px;
    margin-left: 40px;
    margin-top: 2px;
    cursor: pointer;
    position: absolute;
}

.playlist-add{
    background: #535353;
    border: none;
    padding: 5px 10px;
    border-radius: 20px;
    color: #FFF;
}

.timer {
    position: absolute;
    top: 89px;
}

.remove-icon{
    width: 28px;
    margin-left: 40px;
    margin-top: 2px;
    cursor: pointer;
    position: absolute; 
}
p.video-title {
    font-size: 8px;
    height: 20px;
    overflow: hidden;
    color: white;
    text-align: left;
    line-height: 10px;
}
@media (max-width: 400px){
    .smallTxt{
        font-size: 13px;
    }
}
@media (max-width: 900px){
    section{
        padding: 0px;
    }
    .timer {
        top: 142px;
    }
    .playlist{
        /* display: flex;
        flex-wrap: wrap;
        justify-content: flex-start; */
        /* flex-basis:calc(50% - 10px); */
        /* align-items: flex-start; */
        /* justify-content: center; */
        /* align-items: center; */
        /* flex-direction: row; */

        /* display: grid; */
        /* grid-template-columns: repeat(4, 1fr); */
        padding: 5px;
        /* grid-gap: 15px; */
        padding-right: 30px;

    }
    .playlist .video-item{
        /* margin-right: 5px; */
        width: 70px ;
            height: 65px;
    }
    .playlist-highlight .playlist{
        padding: 0px;
        padding-top: 10px;
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        grid-gap: 10px 0;
        overflow-y: auto;
        /* padding-left: 25px; */
    }

    .playlist-highlight .playlist .video-item{
        width: 80px;

    }
    p.video-title {
    height: 20px;
    margin-top: -3px;
}

}
@media (min-width:901px){
    .playlist-highlight{
        display: none;
    }
}
.playlist-highlight{
    position: fixed;
    top: 60%;
    height: 42vh;
    width: 100%;
}

.playlist-highlight .playlist{
    height: 85%;
}

.playlist-highlight .heading{
    background: #303030;
    color: #FFF;
}

</style>