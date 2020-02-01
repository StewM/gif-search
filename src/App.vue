<template>
  <v-app>
    <v-container>
      <v-layout text-center wrap>
        <v-flex xs12>
          <h1>GIF Search</h1>
        </v-flex>
        <v-flex xs12>
          <v-text-field
            solo 
            placeholder="Search here"
            v-model="search"
            @input="debouncedSearch()"
          ></v-text-field>
        </v-flex>
      </v-layout>
      <v-layout wrap ma-2>
        <v-flex xs12>
          <v-sheet elevation="3" color="grey lighten-3">
            <v-layout wrap pa-2 ma-2>
              <v-flex xs12 v-if="loading">
                <v-progress-linear indeterminate></v-progress-linear>
              </v-flex>
              <Gif v-for="gif in gifs" :key="gif.id" :gif="gif" v-on:copied-to-clipboard="notifyCopied()"></Gif>
            </v-layout>
          </v-sheet>
        </v-flex>
      </v-layout>
      <v-layout text-center wrap v-if="searched">
        <v-flex xs12>
          <v-btn @click="loadMore()">Load More</v-btn>
        </v-flex>
      </v-layout>

      <v-btn
            v-scroll="onScroll"
            v-show="fab"
            fab
            dark
            fixed
            bottom
            right
            color="primary"
            @click="toTop"
          >
            <v-icon>mdi-chevron-up</v-icon>
          </v-btn>
    </v-container>

    <v-snackbar
      v-model="snackbar"
      bottom
      :timeout=3000
    >
      {{ snackbarText }}
      <v-btn flat @click="snackbar = false">Close</v-btn>
    </v-snackbar>
  </v-app>
</template>

<script>
import Gif from './components/Gif';
import axios from 'axios';
import _ from 'lodash';

export default {
  name: 'App',

  components: {
    Gif,
  },

  data: () => ({
    gifs: [],
    loading: true,
    searched: false,
    search: '',
    snackbar: false,
    snackbarText: "Snackbar",
    offset: 0,
    fab: false
  }),

  methods: {
    debouncedSearch: _.debounce( function () {
      this.offset = 0;
      this.searchGif(false);
    }, 500),
    searchGif: function (append) {
      if(this.search != ''){
        this.loading = true;

        axios.get('https://api.giphy.com/v1/gifs/search', {
          params: {
            api_key: process.env.VUE_APP_GIPHY_KEY,
            q: this.search,
            limit: 9,
            offset: this.offset
          }
        })
        .then( response => {
          if (append){
            this.gifs = this.gifs.concat(response.data.data);
          } else {
            this.gifs = response.data.data;
          }
          this.searched = true;
          this.loading = false;
        }).catch( error => {
          if(error.response && error.response.status == 429){
            this.snackbarText = "I'm sorry, you've hit the API limit. Please try again later."
          } else {
            this.snackbarText = "I'm sorry, an error has occured"
          }
          this.snackbar = true;
          this.loading = false;
        })
      }
    },
    notifyCopied: function () {
      this.snackbarText = "Copied to clipboard!";
      this.snackbar = true;
    },
    loadMore: function () {
      this.offset += 9;
      this.searchGif(true);
    },
    onScroll: function (e) {
      if (typeof window === 'undefined') return
      const top = window.pageYOffset ||   e.target.scrollTop || 0
      this.fab = top > 20
    },
    toTop: function () {
      this.$vuetify.goTo(0)
    }
  },

  mounted: function () {
    // get featured gifs on page load
    axios.get('https://api.giphy.com/v1/gifs/trending', {
      params: {
        api_key: process.env.VUE_APP_GIPHY_KEY,
        limit: 3
      }
    })
    .then( response => {
      this.gifs = response.data.data;
      this.loading = false;
    }).catch( error => {
      if(error.response && error.response.status == 429){
        this.snackbarText = "I'm sorry, you've hit the API limit. Please try again later."
      } else {
        this.snackbarText = "I'm sorry, an error has occured"
      }
      this.snackbar = true;
      this.loading = false;
    })
  }
};
</script>
