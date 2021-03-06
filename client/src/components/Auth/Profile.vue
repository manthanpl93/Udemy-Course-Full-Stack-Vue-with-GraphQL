<template>
  <v-container class="text-xs-center">
    <!-- User Detail Card -->
    <v-flex sm6 offset-sm3>
      <v-card class="white--text" color="secondary">
        <v-layout>
          <v-flex xs5>
            <v-img height="125px" contain :src="user.avatar"> </v-img>
          </v-flex>
          <v-flex xs7>
            <v-card-title primary-title>
              <div>
                <div class="heading">{{ user.username }}</div>
                <div>{{ user.joinDate }}</div>
                <div class="hidden-xs-only font-weight-reqular">
                  {{ user.favorites.length }} Favorites
                </div>
                <div class="hidden-xs-only font-weight-reqular">
                  {{ userPosts.length }} Posts Added
                </div>
              </div>
            </v-card-title>
          </v-flex>
        </v-layout>
      </v-card>
    </v-flex>

    <!-- Posts Favorited By User -->
    <v-container v-if="!userFavorites.length">
      <v-layout row wrap>
        <v-flex xs12 class="text-center">
          <h2>You have no favorites currently. Go and add some.</h2>
        </v-flex>
      </v-layout>
    </v-container>

    <v-container class="mt-3 text-center" v-else>
      <v-flex xs12>
        <h2 class="font-weight-light">
          Favorited
          <span class="font-weight-regular">({{ userFavorites.length }})</span>
        </h2>
      </v-flex>
      <v-layout row wrap>
        <v-flex xs12 sm6 v-for="favorite in userFavorites" :key="favorite._id">
          <v-card class="mt-3 ml-1 mr-2" hover>
            <v-img height="30vh" :src="favorite.imageUrl"></v-img>
            <v-card-text>{{ favorite.title }}</v-card-text>
          </v-card>
        </v-flex>
      </v-layout>
    </v-container>

    <!-- Posts Created By User -->
    <v-container v-if="!userPosts.length">
      <v-layout row wrap>
        <v-flex xs12 class="text-center">
          <h2>You have no posts currently. Go and add some!</h2>
        </v-flex>
      </v-layout>
    </v-container>
    <v-container class="mt-3" v-else>
      <v-flex xs12 class="text-center">
        <h2 class="font-weight-light">Your Posts</h2>
        <span class="font-weight-regular">({{ userPosts.length }})</span>
      </v-flex>
      <v-layout row wrap>
        <v-flex xs12 sm6 v-for="post in userPosts" :key="post._id">
          <v-card
            class="mt-6 ml-1 mr-2 flex row justify-center align-center"
            hover
          >
            <v-btn
              @click="loadPost(post)"
              color="info"
              class="ml-3 mt-3 mb-3"
              floating
              fab
              small
              dark
            >
              <v-icon>edit</v-icon>
            </v-btn>
            <v-btn
              @click="handleDeleteUserPost(post)"
              color="error"
              class="ml-3 mt-3 mb-3"
              floating
              fab
              small
              dark
            >
              <v-icon>delete</v-icon>
            </v-btn>
            <v-img height="30vh" :src="post.imageUrl"></v-img>
            <v-card-text>{{ post.title }}</v-card-text>
          </v-card>
        </v-flex>
      </v-layout>
    </v-container>

    <!-- Edit Posts Dialog -->
    <v-dialog xs12 sm6 offset-sm3 persistent v-model="editPostDialog">
      <v-card>
        <v-card-title class="headline grey lighten-2">
          Update Post
        </v-card-title>

        <v-container>
          <v-form
            v-model="isFormValid"
            lazy-validation
            ref="form"
            @submit.prevent="handleUpdateUserPost"
          >
            <!--  Title Input -->
            <v-layout row>
              <v-flex xs12>
                <v-text-field
                  :rules="titleRules"
                  label="Post Title"
                  v-model="title"
                  type="text"
                  required
                ></v-text-field>
              </v-flex>
            </v-layout>

            <!-- Image Url Input -->
            <v-layout row>
              <v-flex xs12>
                <v-text-field
                  :rules="imageRules"
                  label="Image URL"
                  v-model="imageUrl"
                  type="text"
                  required
                ></v-text-field>
              </v-flex>
            </v-layout>

            <!-- Image Preview -->
            <v-layout row>
              <v-flex xs12>
                <img :src="imageUrl" height="300px" />
              </v-flex>
            </v-layout>

            <!-- Category Select -->
            <v-layout row>
              <v-flex xs12>
                <v-select
                  :rules="categoriesRules"
                  v-model="categories"
                  :items="[
                    'Art',
                    'Education',
                    'Travel',
                    'Photography',
                    'Technology',
                  ]"
                  multiple
                  label="Categories"
                ></v-select>
              </v-flex>
            </v-layout>

            <!-- Description Text Area  -->
            <v-layout row>
              <v-flex xs12>
                <v-textarea
                  :rules="descRules"
                  label="Description"
                  v-model="description"
                  type="text"
                  required
                ></v-textarea>
              </v-flex>
            </v-layout>

            <v-divider></v-divider>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                :disabled="!isFormValid"
                type="submit"
                class="success--text"
                text
                >Update</v-btn
              >
              <v-btn
                type="submit"
                class="error--text"
                text
                @click="editPostDialog = false"
                >Cancel</v-btn
              >
            </v-card-actions>
          </v-form>
        </v-container>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import { mapGetters } from "vuex";
export default {
  name: "Profile",
  data() {
    return {
      editPostDialog: false,
      isFormValid: true,
      title: "",
      imageUrl: "",
      categories: [],
      description: "",
      titleRules: [
        (title) => !!title || "Title is required",
        (title) =>
          title.length < 20 || "Title must have less than 20 characters",
      ],
      imageRules: [(image) => !!image || "Image is required"],
      categoriesRules: [
        (categories) =>
          categories.length >= 1 || "At least one category is required",
      ],
      descRules: [
        (desc) => !!desc || "Description is required",
        (desc) =>
          desc.length < 200 || "Description must have less than 200 characters",
      ],
    };
  },
  created() {
    this.handleGetUserPosts();
  },
  computed: {
    ...mapGetters(["user", "userFavorites", "userPosts"]),
  },
  methods: {
    handleGetUserPosts() {
      this.$store.dispatch("getUserPosts", {
        userId: this.user._id,
      });
    },
    handleUpdateUserPost() {
      if (this.$refs.form.validate()) {
        this.$store.dispatch("updateUserPost", {
          userId: this.user._id,
          postId: this.postId,
          title: this.title,
          imageUrl: this.imageUrl,
          categories: this.categories,
          description: this.description,
        });
        this.isFormValid = true;
        this.editPostDialog = false;
      } else {
        this.isFormValid = false;
      }
    },
    handleDeleteUserPost(post) {
      this.loadPost(post, false);
      const deletePost = window.confirm(
        "Are you sure you want to delete post?"
      );
      if (deletePost) {
        this.$store.dispatch("deleteUserPost", {
          postId: this.postId,
        });
      }
    },
    loadPost(
      { _id, title, imageUrl, categories, description },
      editPostDialog = true
    ) {
      this.editPostDialog = editPostDialog;
      this.postId = _id;
      this.title = title;
      this.imageUrl = imageUrl;
      this.categories = categories;
      this.description = description;
    },
  },
};
</script>