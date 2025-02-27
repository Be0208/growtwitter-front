<script setup lang="ts">
import type { TweetType } from '@/types';
import default_avatar from '@/assets/default-avatar.png';
import { tempoDesdeCriacao } from '@/utils/PastTime';
import { postComment, postLike, postRetweet } from '@/services/api';
import { onMounted, ref } from 'vue';

interface TweetTypeProps {
  data: TweetType;
}

const props = defineProps<TweetTypeProps>();
const liked = ref(false);
const artificialLike = ref(0);
const dropdown = ref(false);
const retweetModal = ref(false);
const comment = ref('');
const retweetLoading = ref(false);
const commentInput = ref<string>('');
const showDiv = ref(false);

function like() {
  if (liked.value === false) {
    artificialLike.value++;
    liked.value = true;
  } else {
    artificialLike.value--;
    liked.value = false;
  }
}

async function handleSubmit(id: number) {
  await postComment(id, commentInput.value);
  window.location.reload();
}
const toogleDiv = () => {
  showDiv.value = !showDiv.value;
};
const toggleDropdown = () => {
  dropdown.value = !dropdown.value;
};
const handleRetweet = async (id: number) => {

  const response = await postRetweet(id);
  if (response) {
    dropdown.value = false;
  }


};
const handleRetweetWithComment = async (id: number, content: string) => {
  retweetLoading.value = true;
  const response = await postRetweet(id, content);
  if (response) {
    dropdown.value = false;
    retweetModal.value = false;
  }

};

const toggleModalRetweet = () => {
  retweetModal.value = true;
  comment.value = '';

};



async function handlePostLike(id: number) {
  like();
  let resp = await postLike(id);
  if (!resp) {
    like();
  }
}

onMounted(() => {
  const user = localStorage.getItem('userData');
  if (user) {
    liked.value = props.data.likes.some(
      (like: any) => like.userId == JSON.parse(user).id
    );
  }

});
</script>

<template>
  <div class="card-principal rounded-0">
    <v-card-actions class="ga-2">
      <div class="d-block align-self-start">
        <RouterLink :to="`/profile/${data.user.id}`"><v-avatar :image="data.user.avatar_url ?? default_avatar"
            size="50"></v-avatar></RouterLink>



      </div>
      <div class="tweet-body">
        <div class="tweet-header">
          <div>
            <RouterLink :to="`/profile/${data.user.id}`">
              <strong>{{ data.user.name }}</strong> <span>@{{ data.user.username }}</span>
            </RouterLink>

            <span> ·</span> <span>{{ tempoDesdeCriacao(data.created_at) }}</span>
          </div>
          <div style="display: flex; align-items: end; flex-direction: column; position: relative;">
            <v-btn icon small @click="toggleDropdown"> <v-icon icon="mdi-menu"></v-icon></v-btn>
            <div v-if="dropdown" class="dropdown">
              <v-btn small @click="handleRetweet(data.id)"> Retweet</v-btn>
              <v-btn small @click="toggleModalRetweet()"> Retweet com Comentário</v-btn>
            </div>
          </div>

        </div>
        <p class="tweet-content">{{ data.content }}</p>
        <div class="tweet-actions">
          <v-btn icon small @click="toogleDiv()">💬{{ data.comments_count }}</v-btn>
          <v-btn icon small class="btn-like" @click="handlePostLike(data.id)">
            {{ liked ? '❤️' : '🤍' }}
            {{ data.likes_count + artificialLike }}
          </v-btn>
        </div>
        <div v-if="showDiv">
          <hr>
          <div v-for="comment in props.data.comments" :key="comment.id">

            <div style="display: flex; align-items: center; justify-content: end; width: 100%; margin: 5px 0;">
              <RouterLink :to="`/profile/${comment.user.id}`">
                <v-avatar :image="comment.user.avatar_url ?? default_avatar" size="25"></v-avatar>
              </RouterLink>
              <div>
                <RouterLink :to="`/profile/${comment.user.id}`">
                  <strong>{{ comment.user.name }}</strong> <span>@{{ comment.user.username }}</span>
                </RouterLink>

                <span> ·</span> <span>{{ tempoDesdeCriacao(comment.created_at) }}</span>
              </div>
            </div>
            <div style="font-size: 12px; width: 100%; text-align: end; font-weight: bold;">{{
              comment.content }}</div>


          </div>
          <form @submit.prevent="handleSubmit(data.id)">

            <div class="text-box">
              <div class="box-container">
                <textarea placeholder="Comentar" v-model="commentInput"></textarea>

                <div class="formatting">
                  <button type="submit" class="send" title="Send">
                    <svg fill="none" viewBox="0 0 24 24" height="18" width="18" xmlns="http://www.w3.org/2000/svg">
                      <path stroke-linejoin="round" stroke-linecap="round" stroke-width="2.5" stroke="#ffffff"
                        d="M12 5L12 20"></path>
                      <path stroke-linejoin="round" stroke-linecap="round" stroke-width="2.5" stroke="#ffffff"
                        d="M7 9L11.2929 4.70711C11.6262 4.37377 11.7929 4.20711 12 4.20711C12.2071 4.20711 12.3738 4.37377 12.7071 4.70711L17 9">
                      </path>
                    </svg>
                  </button>
                </div>

              </div>
            </div>

          </form>

        </div>
      </div>
    </v-card-actions>
  </div>

  <v-dialog v-model="retweetModal" max-width="500px">
    <v-card>
      <v-card-title>
        <span class="text-h5">Retweetar com comentário</span>
      </v-card-title>
      <v-card-text>
        <v-textarea v-model="comment" label="Adicionar um comentário..." auto-grow rows="3"></v-textarea>
        <v-card class="mt-4" outlined>
          <v-card-text>
            <div class="d-flex align-center">
              <v-avatar :image="data.user.avatar_url ?? default_avatar" size="50"></v-avatar>
              <div class="ml-3">
                <div class="font-weight-bold">{{ data.user.name }}</div>
                <div class="text--secondary">@{{ data.user.username }}</div>
              </div>
            </div>
            <div class="mt-3 ml-13">{{ data.content }}</div>
          </v-card-text>
        </v-card>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="blue darken-1" @click="retweetModal = false">Cancelar</v-btn>
        <v-btn :loading="retweetLoading" color="blue darken-1"
          @click="handleRetweetWithComment(data.id, comment)">Retweet</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>

</template>

<style scoped>
.card-principal {
  border-top: 1px solid #ebe8e8;
  transition: background-color 0.3s ease;
}


.dropdown {
  display: flex;
  flex-direction: column;
  position: absolute;
  top: 50px;
  right: 0;
  background-color: white;
  border: 1px solid #ebe8e8;
  padding: 10px;
  border-radius: 5px;
  width: max-content;
  z-index: 10;
}



.tweet-body {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  width: 100%;
}

.tweet-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;

}

.tweet-header strong {
  font-weight: bold;
}

.tweet-header span {
  color: #657786;
  font-size: 14px;
}

.tweet-content {
  overflow-wrap: break-word;
  word-break: break-word;
}

.v-btn {
  text-transform: none !important;
}

.btn-like:hover {
  filter: drop-shadow(1px 1px 1px red);
  color: rgb(135, 0, 0);
}

.text-box {
  width: 100%;
  height: fit-content;

  padding: 8px;
}

.text-box .box-container {
  background-color: #f1f1f1;
  border-radius: 8px 8px 21px 21px;
  padding: 8px;
}

.text-box textarea {
  width: 100%;
  height: 40px;
  resize: none;
  border: 0;
  border-radius: 6px;
  padding: 12px 12px 10px 12px;
  font-size: 13px;
  outline: none;
  caret-color: #0a84ff;
}



.text-box .formatting button {
  width: 30px;
  height: 30px;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: transparent;
  border-radius: 50%;
  border: 0;
  outline: none;
}

.text-box .formatting button:hover {
  background-color: #f1f1f1;
}

.text-box .formatting .send {
  width: 30px;
  height: 30px;
  background-color: #0a84ff;
  margin: 0 0 0 auto;
}

.text-box .formatting .send:hover {
  background-color: #026eda;
}

@keyframes ripple {
  0% {
    transform: scale(0);
    opacity: 0.6;
  }

  100% {
    transform: scale(1);
    opacity: 0;
  }
}
</style>
