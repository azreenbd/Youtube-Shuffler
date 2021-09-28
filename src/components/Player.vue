<template>
    <div>
        <div>
            <button>⏮ Prev</button>
            <button>Next ⏭</button>
        </div>
        <div id="playlist-list" ref="playlistList">
            <div v-for="video in playlistItems" :key="video.etag" :ref="video.snippet.resourceId.videoId" class="playlist-item" :id="video.snippet.resourceId.videoId === nowPlaying ? 'playlist-item-active' : null" @click="playVideo(video.snippet.resourceId.videoId)">
                <span v-if="video.snippet.resourceId.videoId === nowPlaying">▶ </span> {{video.snippet.title}}
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    name: 'HelloWorld',
    data(){
        return{
            apiKey: process.env.VUE_APP_API_KEY,
            isLoading: false,
            isError: false,
            playlistItems: [],
            nowPlaying: '',
        }
    },
    mounted() {
        this.getPlaylistItem();
    },
    methods: {
        async getPlaylistItem(playlistId) {
            let nextPageAvailable = true;
            let nextPageToken;

            // Show loading screen
            this.isLoading = true;

            const $ = axios.create({
                baseURL: process.env.VUE_APP_ENDPOINT,
                headers: {}
            });

            await $.get(`/playlistItems?part=snippet&maxResults=50&playlistId=${playlistId}&key=${this.apiKey}`).then(res=>{
                if(!res.data.nextPageToken) {
                    nextPageAvailable = false;
                }
                nextPageToken = res.data.nextPageToken;
                this.playlistItems = this.playlistItems.concat(res.data.items);
            })
            .catch(()=>{
                nextPageAvailable = false;
                this.isLoading = false;
                this.isError = true;
            });

            while(nextPageAvailable) {
                await $.get(`/playlistItems?part=snippet&maxResults=50&pageToken=${nextPageToken}&playlistId=${playlistId}&key=${this.apiKey}`).then(res=>{
                    if(!res.data.nextPageToken) {
                        nextPageAvailable = false;
                    }
                    nextPageToken = res.data.nextPageToken;
                    this.playlistItems = this.playlistItems.concat(res.data.items);
                })
                .catch(()=>{
                    nextPageAvailable = false;
                    this.isLoading = false;
                    this.isError = true;
                });
            }

            // Stop loading
            this.isLoading = false;
        },
        playVideo(videoId) {
            this.nowPlaying = videoId;

            // Get item offset position
            let containerOffset = this.$refs.playlistList.offsetTop;
            let activePlaylistItem = this.$refs[videoId][0];
            let topPosition = activePlaylistItem.offsetTop - containerOffset;
            // Scroll item to top of container
            (this.$refs.playlistList).scrollTop = topPosition;


            console.log(videoId);
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