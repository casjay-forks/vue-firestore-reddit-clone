<template>
<section>
  <div class="container">
    <button
        v-if="isLoggedIn"
        @click="showCommentForm = !showCommentForm"
        class="button is-success">
        Toggle Comment Form
      </button>
      
      <form 
        v-if="showCommentForm && isLoggedIn"
        @submit.prevent="onCreateComment()">
          <b-field label="Comment">
              <b-input
                v-model="comment.content">
              </b-input>
          </b-field>
          <b-field label="URL">
              <b-input v-model="comment.URL"
                  type="url"></b-input>
          </b-field>
          <button class="button is-info">Post Comment</button>
      </form>
          <div class="card">
                  <div class="card-content">
                      <div class="media">
                          <div class="media-content">
                            <figure class="image is-48x48">
                                  <img :src="post[0].author[0].photoURL"> 
                              </figure>
                               Posted by <strong class="is-size-5">{{post[0].author[0].displayName}}</strong>  
                        <time class="is-size-6"> {{ post[0].created_at.seconds |  moment("from", "now", true) }} ago </time>
                            <hr>
                              <p class="title is-size-2"
                                  v-if="!post[0].URL">{{post[0].title}}</p>
                              <p class="title is-size-2"
                                  v-if="post[0].URL">
                                  <a :href="post[0].URL"
                                      target="_blank">{{post[0].title}} </a>
                              </p>
                          </div>
                      </div>
                               <div class="content is-size-3" v-if="post[0].description">
                          {{post[0].description}}
                             </div>
                      <div class="card-image">
                      <figure>
                          <img :src="post[0].URL"
                              v-if="post[0].URL && isImage(post[0].URL)">
                      </figure>
                  </div>
                    <span class="tag is-dark is-medium">
                      <h5 class="is-size-5" v-if="post[0].likes == 1">{{post[0].likes}} like</h5>
                             <h5 class="is-size-5" v-else>{{post[0].likes}} likes</h5>
                 </span>
                          <br>
                          <hr>
                           <button v-if="isLoggedIn && !alreadyLiked(post[0].liked_by,user.id)"
                            @click="likePost(post[0].id)"
                            class="button is-primary">
                            Like 👍
                          </button>
                           <button v-if="isLoggedIn && alreadyLiked(post[0].liked_by,user.id)"
                            @click="unlikePost(post[0].id)"
                            class="button is-primary">
                            Unlike 👎
                          </button>
                          <br>
                          <br>
                           <button @click="deletePost(post[0].id)" class="button is-danger"
                            v-if="user && user.id == post[0].user_id">Delete Post
                            </button>
                  </div>
              </div>
          </div>
          <hr>
      <b class="is-size-2" v-if="comments > 0">Comments ({{comments.length}})</b>
    <br>
    
    <br>
          <article class="card" v-for="comment in comments" :key="comment.id">
  <figure class="media-left">
    <p class="image is-64x64">
          <img :src="comment.author[0].photoURL"
        alt="Placeholder image">
    </p>
  </figure>
  <div class="media-content">
    <div class="content">
      <p>
        <strong class="is-size-5">{{comment.author[0].displayName}}</strong><small class="is-size-5"> {{comment.created_at.seconds |  moment("from", "now", true) }} ago</small>
        <br>
        <h5 class="is-size-5">{{comment.content}}</h5>
        <br>
        <p class="title is-4"
                                  v-if="comment.URL && !isImage(comment.URL)">
                                  <a :href="comment.URL"
                                      target="_blank">{{comment.URL}}</a>
                              </p>
    </div>
     <div class="comment-img">
                      <figure class="image">
                          <img :src="comment.URL"
                              v-if="comment.URL && isImage(comment.URL)" >
                      </figure>
                  </div>
                  <!--
     <h5 class="is-size-6" v-if="comment.likes == 1">{{comment.likes}} like</h5>
                          <h5 class="is-size-6" v-else>{{comment.likes}} likes</h5>
                           <button v-if="isLoggedIn && !alreadyLiked(comment.liked_by,user.id)"
                            @click="likeComment(comment.id)"
                            class="button is-primary">
                            Like 👍
                          </button>
                           <button v-if="isLoggedIn && alreadyLiked(comment.liked_by,user.id)"
                            @click="unlikeComment(comment.id)"
                            class="button is-primary">
                            Unlike 👎
                          </button>
                          -->
                           <button
                            @click="deleteComment(comment.id)"
                            v-if="user && user.id == comment.user_id"
                            class="button is-danger">
                            Delete Comment
                          </button>     
      </div>
    </article>
    </section>
</template>

<script>
import TimeAgo from 'vue2-timeago'

import {
  mapState,
  mapGetters,
  mapActions,
} from 'vuex';


export default {
    components: {
    TimeAgo,
  },
  data: () => ({
    showCommentForm: false,
 
    searchTerm: '',
    comment: {
      content: '',
      URL: '',
    },
  }),
  mounted() {
    this.initUsers();
    this.initPost(this.$route.params.post_id);
    this.initComments(this.$route.params.post_id);
  },
  watch: {
    '$route.params.post_id': function () {
      this.initPost(this.$route.params.post_id);
    },

  },
  computed: {
    ...mapState('post', ['post','comments']),
    ...mapState('auth', ['isLoggedIn', 'user']),
  },
  methods: {
    isImage(url) {
      return url.match(/(png|jpg|jpeg|gif)$/);
    },
    alreadyLiked(arr, id){
      const found = arr.find(element => element == id)
      console.log(found);
      if(found) return true;
      else return false;
    },
    ...mapActions('post', ['createComment', 'initPost','initComments', 'deletePost',
    'deleteComment','likePost','unlikePost',
    'likeComment','unlikeComment']),
    ...mapActions('users', {
      initUsers: 'init',
    }),
    async onCreateComment() {
      if (this.comment.content  || this.comment.URL) {
        this.createComment(this.comment);
        
        this.comment = {
          content: '',
          URL: '',
        };
        this.showForm = false;
      }
    },
  },
};
</script>

<style>
    .search-form {
        margin-top: 2em;
    }
    .posts {
        margin-top: 2em;
    }
    .card {
        padding:1%;
        height: 100%;
        margin: 1%;
        border-radius: 6px;
    }
    
    .card img {
        border-radius: 5px;
        width:70%;
    }
    .comment-img img{
      width:20%;
    }
    .media-content {
    overflow-x: visible; 
}
</style>

