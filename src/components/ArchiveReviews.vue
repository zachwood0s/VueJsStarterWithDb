<template lang="pug">
  div
    NavBar
    .container
      .row.justify-content-md-center#searchArea
        .col-md-8.mt-5
          input.mt-5.mb-1.w-100.ft-xxsuperlarge.color-base2#userSearch(
            placeholder='Search'
            v-model='searchText'
            v-on:keyup.enter='search'
            )
      .row.justify-content-md-center
        .col.text-center.ft-medium Search for a review
      .row.justify-content-md-center.my-5
        ul.list-group.col-md#usersList(
          v-if="users && users.length"
          v-bind:class="{loaded: loaded}"
        ) 
          li.bg-base5.list-group-item.my-1(
            v-for="user of users"
          )
           .row
             .col-10 {{ user.Description }}
             a.col-2.font-weight-bold.bg-base9.color-base6.h-100.viewProfile.text-center(v-on:click="archive(user)") Archive Review
        ul(v-if="errors && errors.length")
          li(v-for="error of errors") {{ error.message }}
</template>

<script>
 import axios from 'axios'
 import NavBar from './NavBar.vue'
 const loadStartPadding = 300;

  export default {
   methods: {
     search () {
       var loader = this.loadMore
       console.log("no more results: ", this.outOfResults)
       if(this.searchText != this.lastSearchText){
         this.page = 1
         loader = this.loadNew
       }
       else if(this.outOfResults) {
         console.log("Out of results, ignoring")
         return;
       }

       var searchText = `/api/reviews?name=${this.searchText}&pageNumber=${this.page}&resultCount=${this.numberToLoad}`
       console.log(searchText)
       this.searching = true

       axios.get(searchText)

         .then(response => {
           this.searching = false;
           if (response.data.recordsets[0].length < this.numberToLoad){
             console.log('No more results')
             this.outOfResults = true
           }
           loader(response)
           this.lastSearchText = this.searchText
           window.setTimeout(() => this.loaded = true, 10)
         })
         .catch(e => {
           this.errors.push(e)
         })
     },

     archive (review) {
      console.log(review)
      axios.post(`/api/admin/archive/reviews?reviewId=${review.ReviewID}`)
        .then(response => {
          console.log("got response", response)
          if(response.data.output.archived == 1){
            this.archived=true
            this.lastSearchText = ""
            this.search()
          } else{
            this.notArchived=false
          }
        })
        .catch(e => {
          this.notArchived=false
        })
     },
     loadMore (response) {
       this.users = this.users.concat(response.data.recordsets[0])
     },
     loadNew (response) {
       this.users = response.data.recordsets[0]
     },
     onscroll () {
       var d = document.documentElement
       var offset = d.scrollTop + window.innerHeight
       var height = d.offsetHeight

       if (offset + loadStartPadding >= height) {
         if (!this.searching) {
           this.page++
           this.search()
         }
       }
     }
   },
   created () {
     window.addEventListener('scroll', this.onscroll)
   },
   components: {
     NavBar
   },
   data () {
     return {
       outOfResults: false,
       lastSearchText: '',
       searchText: '',
       searching: false,
       loaded: false,
       numberToLoad: 10,
       page: 1,
       users: [],
       errors: [],
       input: {
           reviewId: "",
       },
       archived: false,
       notArchived: true

     }
   }
 }
</script>