<template>
    <div>
        <div id="player"></div>
        <p v-if="isLoading">Loading</p>
        <p>{{playlistItems.length}}</p>
        <div>
            <button @click="playVideo(nowPlayingIndex-1)">⏮ Prev</button>
            <button @click="playVideo(nowPlayingIndex+1)">Next ⏭</button>
        </div>
        <div id="playlist-list" ref="playlistList">
            <div v-for="(video, index) in playlistItems" :key="video.etag" :ref="video.snippet.resourceId.videoId" class="playlist-item" :id="index === nowPlayingIndex ? 'playlist-item-active' : null" @click="playVideo(index)">
                <span v-if="index === nowPlayingIndex">▶ </span> {{video.snippet.title}}
            </div>
        </div>
    </div>
</template>

<script src="http://www.youtube.com/player_api"></script>
<script>
import axios from 'axios';

export default {
    name: 'Player',
    data(){
        return{
            apiKey: process.env.VUE_APP_API_KEY,
            playlistIds: typeof this.$route.query.id === 'string' ? [this.$route.query.id] : this.$route.query.id,
            isLoading: false,
            isError: false,
            loopPlaylist: true,
            playlistItems: [],
            nowPlayingIndex: 0,
            player: null,
        }
    },
    mounted() {
        // Loads the Youtube IFrame Player API code
        var tag = document.createElement('script');
        tag.src = "https://www.youtube.com/iframe_api";
        var firstScriptTag = document.getElementsByTagName('script')[0];
        firstScriptTag.parentNode.insertBefore(tag, firstScriptTag);

        

        this.getPlaylistItems(this.playlistIds);
    },
    methods: {
        async getPlaylistItems(playlistIds) {
            let nextPageAvailable = true;
            let nextPageToken;

            // Show loading screen
            this.isLoading = true;

            const $ = axios.create({
                baseURL: process.env.VUE_APP_ENDPOINT,
                headers: {}
            });

            for(let id of playlistIds) {
                await $.get(`/playlistItems?part=snippet&maxResults=50&playlistId=${id}&key=${this.apiKey}`).then(res=>{
                    // Get next page token
                    nextPageAvailable = res.data.nextPageToken ? true : false;
                    nextPageToken = nextPageAvailable ? res.data.nextPageToken : null;

                    this.playlistItems = this.playlistItems.concat(res.data.items);
                })
                .catch(()=>{
                    nextPageAvailable = false;
                    this.isLoading = false;
                    this.isError = true;
                });

                while(nextPageAvailable) {
                    await $.get(`/playlistItems?part=snippet&maxResults=50&pageToken=${nextPageToken}&playlistId=${id}&key=${this.apiKey}`).then(res=>{
                        // Get next page token
                        nextPageAvailable = res.data.nextPageToken ? true : false;
                        nextPageToken = nextPageAvailable ? res.data.nextPageToken : null;

                        this.playlistItems = this.playlistItems.concat(res.data.items);
                    })
                    .catch(()=>{
                        nextPageAvailable = false;
                        this.isLoading = false;
                        this.isError = true;
                    });
                }
            }

            // Stop loading
            this.isLoading = false;
            // Load player
            if(this.playlistItems.length > 0) {
                this.createPlayer();
            }
        },
        createPlayer() {
            // Create youtube player object
            this.player = new YT.Player('player', {
                height: '390',
                width: '640',
                videoId: this.playlistItems[0].snippet.resourceId.videoId,
                playerVars: {
                    'playsinline': 1
                },
                events: {
                    'onReady': this.onPlayerReady,
                    'onStateChange': this.onPlayerStateChange,
                    'onError': this.onPlayerError,
                }
            });
        },
        playVideo(index) {
            if(index > this.playlistItems.length-1) { 
                if(!this.loopPlaylist) {
                    return false;
                }
                index = 0;
            }
            
            if(index < 0) { 
                if(!this.loopPlaylist) {
                    return false;
                }
                index = this.playlistItems.length-1;
            }

            this.nowPlayingIndex = index;
            let videoId = this.playlistItems[index].snippet.resourceId.videoId;

            // Get item offset position
            let containerOffset = this.$refs.playlistList.offsetTop;
            let activePlaylistItem = this.$refs[videoId][0];
            let topPosition = activePlaylistItem.offsetTop - containerOffset;
            // Scroll item to top of container
            (this.$refs.playlistList).scrollTop = topPosition;

            this.player.loadVideoById(videoId, 0);
        },
        onPlayerReady(event) {
            event.target.playVideo();
        },
        onPlayerStateChange(event) {
            if (event.data == 0) {
                // Play next video
                this.playVideo(this.nowPlayingIndex+1);
            }

            if (event.data == -1) {
                // Play next video
                this.player.playVideo();
            }
        },
        onPlayerError(event) {
            // Play next video
            this.playVideo(this.nowPlayingIndex+1);
        }
    }
}
</script>

<style>
#playlist-list {
    height: 720px;
    max-width: 720px;
    overflow-y: scroll;
    background: #252525;
    color: #b6b6b6;
    border-radius: 2rem;
    overflow: -moz-scrollbars-none;
    -ms-overflow-style: none;
}

#playlist-list::-webkit-scrollbar { width: 0 !important }

.playlist-item {
    padding: .8rem 2rem;
    border-bottom: .5px solid #464646;
    cursor: pointer;
}
.playlist-item:hover {
    background-color: #464646;
}
.playlist-item:last-child {
    border-bottom: 0px;
}
#playlist-item-active {
    color: #fff;
}
</style>