<template>
  <div id="app">
    <!-- App if Logged In -->

    <section class="section" style="padding:1em" v-if="logged_in==true">
      <div class="container is-fluid is-marginless">
        <div class="columns fixedHeight">
          <!-- Leftmost column: <sources>, Sources.vue -->
          <div
            class="column fixedHeight"
            style="max-width:380px;border:1px solid silver;background-color:whitesmoke"
          >
            <sources
              :sources="sources"
              :categories="categories"
              :posts="posts"
              :highlightedSource="highlightedSource"
              v-on:selectSource="selectSource"
            ></sources>
          </div>

          <!-- Middle Column: <post-titles>, PostTitles.vue -->
          <div
            class="column is-paddingless fixedHeight is-4"
            style="max-width:380px;border:1px solid silver"
          >
            <post-titles
              :posts="posts"
              :selectedPost="post"
              :highlightedSource="highlightedSource"
              v-on:clickedOnCircle="togglePostReadStatus"
              v-on:clickedOnPostTitle="selectPost"
            ></post-titles>
          </div>

          <!-- Rightmost Column: <post-content>, PostContent.vue -->
          <div class="column fixedHeight" style="padding:2.5rem; border:1px solid silver">
            <post :post="post"></post>
          </div>
        </div>
        <!-- /columns -->
      </div>
      <!-- /container -->
    </section>

    <!-- Logging In Loader -->

    <section v-if="logged_in=='attempting'">
      <progress class="progress is-small is-primary" max="100" style="height:3px"></progress>
      <!-- <h1 class="is-title is-size-3">Logging In</h1> -->
    </section>

    <!-- Login Form -->

    <section
      v-if="logged_in==false || logged_in=='failed_attempt'"
      class="hero is-fullheight is-primary"
    >
      <div class="hero-body">
        <div class="container">
          <div class="columns is-centered">
            <div class="column is-5-tablet is-4-desktop is-3-widescreen">
              <form action class="box">
                <div class="field">
                  <label for class="label">Email</label>
                  <div class="control">
                    <input
                      v-model="email"
                      type="email"
                      placeholder="e.g. bobsmith@gmail.com"
                      class="input"
                      required
                    />
                    <span class="icon is-small is-left">
                      <i class="fa fa-envelope"></i>
                    </span>
                  </div>
                </div>
                <div class="field">
                  <label for class="label">Password</label>
                  <div class="control">
                    <input
                      v-model="password"
                      type="password"
                      placeholder="*******"
                      class="input"
                      required
                    />
                    <span class="icon is-small is-left">
                      <i class="fa fa-lock"></i>
                    </span>
                  </div>
                </div>

                <div class="field">
                  <button @click.prevent="attempt_login" class="button is-success">Login</button>
                </div>
              </form>
            </div>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import Sources from "./components/Sources.vue";
import PostTitles from "./components/PostTitles.vue";
import Post from "./components/Post.vue";
import axios from "axios";

export default {
  name: "app",
  components: {
    Sources,
    PostTitles,
    Post
  },
  methods: {
    selectSource({ kind, value }) {
      this.highlightedSource = value;
      this.selectedPostIndex = 0;
    },
    selectPost(n, p) {
      this.selectedPostIndex = n;
      this.post = p;
      // this.markPostRead(p);
    },
    attempt_login() {
      axios
        .get(
          "https://infraread.mustapha.me/client/initialData?email=" +
            this.email +
            "&password=" +
            this.password
        )
        .then(res => {
          if (res.data.response == 200) {
            this.logged_in = true;
            this.sources = res.data.data.sources;
            this.categories = res.data.data.categories;
            this.posts = res.data.data.posts
              .sort((a, b) =>
                a.posted_at > b.posted_at
                  ? -1
                  : b.posted_at > a.posted_at
                  ? 1
                  : 0
              )
              .map(obj => ({ ...obj, seconds: Date.parse(obj.posted_at) }));
          }
        })
        .catch(res => {
          this.logged_in = "failed_attempt";
        });
      this.logged_in = "attempting";
    },
    togglePostReadStatus(post) {
      var targetPost = this.posts[this.getPostIndexByID(post.id)]; // We're using this because we're mutating original, unfiltered list of posts
      var oldReadStatus = targetPost.read;
      var newReadStatus = Math.abs(targetPost.read - 1);
      // First Toggle in the UI
      targetPost.read = newReadStatus;
      // Perform ajax request
      axios
        .get(
          "https://infraread.mustapha.me/client/toggleReadStatus?post=" +
            post.id +
            "&email=" +
            this.email +
            "&password=" +
            this.password
        )
        .then(res => {
          //nothing
        })
        .catch(res => {
          console.log("there was a problem with updating post status");
          //undo from UI
          targetPost.read = oldReadStatus;
        });
    },
    markPostRead(post) {
      // bounce back if post already read
      if (post.read == 1) {
        console.log("post already read");
        return;
      }
      // otherwise
      this.togglePostReadStatus(post);
    },
    selectPost(n, p) {
      this.selectedPostIndex = n;
      this.post = p;
      this.markPostRead(p);
      console.log(n);
    },

    // Utility functions
    // ===================
    getPostIndexByID(id) {
      // Get the array index of post from post id
      for (let index = 0; index < this.posts.length; index++) {
        if (this.posts[index].id == id) {
          return index;
        }
      }
      return false;
    }
  },
  data() {
    return {
      logged_in: false,
      email: null,
      password: null,
      temp: null,
      sources: null,
      categories: null,
      posts: null,
      post: null,
      highlightedSource: "allUnread", // Which source is highlighted on the first column
      selectedPostIndex: 0
    };
  }
};
</script>

<style lang="scss">
$primary: #da2525;
@import "../node_modules/bulma/bulma.sass";
a {
  color: $primary;
}
#app {
}
.fixedHeight {
  height: calc(100vh - 0.5em);
  overflow: scroll;
}
</style>
