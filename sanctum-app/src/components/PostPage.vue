<template>
  <nav class="navbar navbar-expand-lg bg-primary">
    <div class="container d-flex justify-content-between">
      <router-link to="/home" class="navbar-brand text-light">ForumApp</router-link>
      <button class="btn btn-light">
        <router-link to="/logout" class="nav-link text-primary">Logout</router-link>
      </button>
    </div>
  </nav>
  <div class="container mt-5">
    <h2 class="text-left mb-4">Posts</h2>
    <hr>
    <div class="row">
      <div class="col-lg-6">
        <div class="modern-container mb-3">
          <form @submit.prevent="createPost">
            <div class="form-group mb-2">
              <input type="text" class="form-control modern-input" v-model="newPost.title" placeholder="Title" required>
            </div>
            <div class="form-group">
              <textarea class="form-control modern-input" v-model="newPost.body" placeholder="Body" rows="5" required></textarea>
            </div>
            <div class="text-center mt-2 d-flex justify-content-end align-items-center">
              <button type="submit" class="btn btn-light modern-button">Create Post</button>
            </div>
          </form>
        </div>
      </div>
      <div class="col-lg-6">
        <div v-for="post in posts" :key="post.id" class="card mb-4 modern-card">
          <div class="card-body d-flex flex-column">
            <div class="d-flex justify-content-between align-items-center mb-3">
              <h5 class="card-title border-bottom pb-2 mb-3">{{ post.title }}</h5>
              <div>
                <button class="btn btn-sm edit-btn modern-btn" @click="openEditModal(post)">
                  <i class="fa-solid fa-pen"></i>
                </button>
                <button class="btn btn-sm delete-btn modern-btn" @click="deletePost(post.id)">
                  <i class="fa-solid fa-trash"></i>
                </button>
              </div>
            </div>
            <p class="card-text">{{ post.body }}</p>
            <hr class="my-4">
            <div class="mt-auto">
              <div class="d-flex justify-content-between align-items-center">
                <h6 class="text-left">Comments</h6>
                <button class="btn btn-sm text-light bg-primary btn-comment mb-2" @click="toggleCommentForm">
                  Add Comment
                </button>
              </div>
              <form v-if="showCommentForm" @submit.prevent="createComment(post.id)">
                <div class="form-group">
                  <textarea class="form-control modern-input" v-model="newComment" placeholder="Add a comment" rows="2" required></textarea>
                </div>
                <div class="text-right d-flex justify-content-start align-items-center">
                  <button type="submit" class="btn btn-primary text-light btn-comment mt-3">
                    <i class="fa-solid fa-paper-plane"></i>
                  </button>
                </div>
              </form>
              <div v-for="comment in post.comments" :key="comment.id" class="card mt-2 modern-comment-card">
                <div class="card-body">
                  <p class="card-text">{{ comment.body }}</p>
                  <div class="button-container">
                    <button class="btn btn-sm modern-btn" @click="openEditCommentModal(comment)">
                      <i class="fa-solid fa-pen"></i>
                    </button>
                    <button class="btn btn-sm modern-btn" @click="deleteComment(comment)">
                      <i class="fas fa-trash-alt"></i>
                    </button>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Edit Post Modal -->
    <div v-if="showEditModal" class="modal fade show" style="display: block;" tabindex="-1" role="dialog">
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Edit Post</h5>
          </div>
          <div class="modal-body">
            <form @submit.prevent="updatePost">
              <div class="form-group">
                <input type="text" class="form-control" v-model="editPostData.title" placeholder="Title" required>
              </div>
              <div class="form-group">
                <textarea class="form-control" v-model="editPostData.body" placeholder="Body" required></textarea>
              </div>
              <div class="text-right">
                <button type="submit" class="btn btn-success mr-2">Update</button>
                <button type="button" class="btn btn-light" data-dismiss="modal" @click="showEditModal = false">Cancel</button>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
    <!-- Edit Comment Modal -->
    <div v-if="showEditCommentModal" class="modal fade show" style="display: block;" tabindex="-1" role="dialog">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title">Edit Comment</h5>
                </div>
                <div class="modal-body">
                    <form @submit.prevent="updateComment">
                        <div class="form-group">
                            <textarea class="form-control" v-model="editCommentData.body" placeholder="Edit your comment" rows="3" required></textarea>
                        </div>
                        <div class="text-right">
                            <button type="submit" class="btn btn-success mr-2">Update</button>
                            <button type="button" class="btn btn-light" data-dismiss="modal" @click="showEditCommentModal = false">Cancel</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      posts: [],
      newPost: {
        title: '',
        body: ''
      },
      editPostData: {
        id: null,
        title: '',
        body: ''
      },
      editCommentData: {
          id: null,
          body: ''
      },
      postId: 0,
      showEditCommentModal: false,
      newComment: '',
      showPostForm: false,
      showEditModal: false,
      showCommentForm: false
    };
  },
  async mounted() {
    await this.fetchPosts();
  },
  methods: {
    toggleCommentForm() {
      this.showCommentForm = !this.showCommentForm;
    },
    async fetchPosts() {
      try {
        const response = await axios.get(`${this.$root.$data.apiUrl}/posts`, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem('token')}`
          }
        });
        this.posts = response.data;
        for (const post of this.posts) {
          const commentsResponse = await axios.get(`${this.$root.$data.apiUrl}/posts/${post.id}/comments`, {
            headers: {
              Authorization: `Bearer ${localStorage.getItem('token')}`
            }
          });
          post.comments = commentsResponse.data;
        }
      } catch (error) {
        console.error("Error fetching posts:", error);
      }
    },
    async createPost() {
      try {
        const response = await axios.post(`${this.$root.$data.apiUrl}/posts`, this.newPost, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem('token')}`
          }
        });
        this.posts.push(response.data);
        this.newPost.title = '';
        this.newPost.body = '';
        this.showPostForm = false;
      } catch (error) {
        console.error("Error creating post:", error);
      }
    },
    openEditModal(post) {
      this.editPostData.id = post.id;
      this.editPostData.title = post.title;
      this.editPostData.body = post.body;
      this.showEditModal = true;
    },
    async updatePost() {
      try {
        const response = await axios.put(`${this.$root.$data.apiUrl}/posts/${this.editPostData.id}`, {
          title: this.editPostData.title,
          body: this.editPostData.body
        }, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem('token')}`
          }
        });

        const updatedPost = this.posts.find(post => post.id === this.editPostData.id);
        if (updatedPost) {
          updatedPost.title = response.data.title;
          updatedPost.body = response.data.body;
        }

        this.showEditModal = false;
      } catch (error) {
        console.error("Error updating post:", error);
      }
    },
    async deletePost(postId) {
      try {
        await axios.delete(`${this.$root.$data.apiUrl}/posts/${postId}`, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem('token')}`
          }
        });
        this.posts = this.posts.filter(post => post.id !== postId);
      } catch (error) {
        console.error("Error deleting post:", error);
      }
    },
    async createComment(postId) {
      try {
        const response = await axios.post(
          `${this.$root.$data.apiUrl}/posts/${postId}/comments`,
          {
            body: this.newComment,
            post_id: postId
          },
          {
            headers: {
              Authorization: `Bearer ${localStorage.getItem('token')}`
            }
          }
        );
        const post = this.posts.find(post => post.id === postId);
        post.comments.push(response.data);
        this.newComment = '';
      } catch (error) {
        console.error("Error creating comment:", error);
      }
    },
    openEditCommentModal(comment) {
        this.editCommentData.id = comment.id;
        this.editCommentData.body = comment.body;
        this.showEditCommentModal = true;
        this.postId = comment.post_id;
    },
    async updateComment() {
      try {
        const response = await axios.put(
          `${this.$root.$data.apiUrl}/posts/${this.postId}/comments/${this.editCommentData.id}`,
          { body: this.editCommentData.body },
          {
            headers: {
              Authorization: `Bearer ${localStorage.getItem('token')}`
            }
          }
        );

        const postIndex = this.posts.findIndex(post => post.comments.some(c => c.id === this.editCommentData.id));
        const commentIndex = this.posts[postIndex].comments.findIndex(c => c.id === this.editCommentData.id);
        this.posts[postIndex].comments[commentIndex] = response.data;

        this.showEditCommentModal = false;
      } catch (error) {
        console.error("Error updating comment:", error);
      }
    },
    async deleteComment(comment) {
      try {
          await axios.delete(`${this.$root.$data.apiUrl}/posts/${comment.post_id}/comments/${comment.id}`, {
              headers: {
                  Authorization: `Bearer ${localStorage.getItem('token')}`
              }
          });
          
          const post = this.posts.find(post => post.id === comment.post_id);
          post.comments = post.comments.filter(c => c.id !== comment.id);
          console.log(post);
      } catch (error) {
          console.error("Error deleting comment:", error);
      }
  }


  }
};
</script>

<style scoped>
.card {
  border-radius: 12px;
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.15);
}

.btn {
  border-radius: 5px;
  margin: 5px;
}

.navbar-brand {
  font-size: 26px;
  font-weight: bold;
}

.edit-btn, .delete-btn {
  background-color: transparent;
  border: none;
  color: #007bff;
}

.edit-btn.i:hover, .delete-btn i:hover {
  color: #0056b3;
}

.custom-textarea {
  resize: none;
}

.card-title {
  border-bottom: 2px solid #007bff;
  padding-bottom: 6px;
}

.card-body {
  padding: 1.5rem;
}

.comment-card {
  border-radius: 8px;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.15);
}

.button-container {
  position: absolute;
  bottom: 10px;
  right: 10px;
}

hr {
  border: 1px solid #007bff;
}

.modern-container {
  padding: 18px;
  border-radius: 10px;
  background: #e9ecef;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.1), inset 0 -3px 6px rgba(0, 0, 0, 0.05);
}

.modern-input .form-control {
  border: none;
  border-radius: 8px;
  padding: 14px;
  margin-bottom: 15px;
  background: #ffffff;
  box-shadow: inset 1px 1px 3px rgba(0, 0, 0, 0.1), inset -1px -1px 3px rgba(255, 255, 255, 0.4);
}

.modern-input .form-control:focus {
  outline: none;
  box-shadow: inset 2px 2px 4px rgba(0, 0, 0, 0.1), inset -2px -2px 4px rgba(255, 255, 255, 0.4);
}

.modern-button {
  border: none;
  border-radius: 16px;
  padding: 12px 28px;
  box-shadow: 4px 4px 8px rgba(0, 0, 0, 0.1), -4px -4px 8px rgba(255, 255, 255, 0.4);
  color: #333;
  font-size: 15px;
  cursor: pointer;
  transition: transform 0.2s ease-in-out;
}

.modern-button:hover {
  transform: translateY(-1px);
}

.modern-button:active {
  transform: translateY(0.5px);
}

.btn-comment {
  color: #fff;
  border: none;
  border-radius: 16px;
  padding: 7px 26px;
  box-shadow: 4px 4px 8px rgba(0, 0, 0, 0.1), -4px -4px 8px rgba(255, 255, 255, 0.4);
  color: #333;
  font-size: 15px;
  cursor: pointer;
  transition: transform 0.2s ease-in-out;
}

.modern-card {
  border-radius: 12px;
  background: #ffffff;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1), inset 0 -4px 8px rgba(0, 0, 0, 0.05);
}

.modern-btn {
  font-size: 12px;
  border: none;
  border-radius: 50%;
  background: #007bff;
  padding: 9px;
  color: #fff;
  box-shadow: 3px 3px 6px rgba(0, 0, 0, 0.1), -3px -3px 6px rgba(255, 255, 255, 0.4);
  transition: transform 0.2s ease-in-out;
}

.modern-btn:hover {
  transform: translateY(-1px);
}

.modern-btn:active {
  transform: translateY(0.5px);
}

.modern-input {
  border: none;
  border-radius: 8px;
  padding: 14px;
  background: #ffffff;
  box-shadow: inset 1px 1px 3px rgba(0, 0, 0, 0.1), inset -1px -1px 3px rgba(255, 255, 255, 0.4);
  transition: box-shadow 0.3s ease;
}

.modern-input:focus {
  outline: none;
  box-shadow: inset 2px 2px 4px rgba(0, 0, 0, 0.1), inset -2px -2px 4px rgba(255, 255, 255, 0.4);
}

</style>
