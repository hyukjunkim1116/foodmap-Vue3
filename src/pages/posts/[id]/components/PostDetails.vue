<template>
  <BaseCard class="q-pa-lg">
    <div class="flex q-mb-md">
      <q-btn
        icon="sym_o_arrow_back"
        flat
        round
        dense
        color="grey"
        size="16px"
        @click="$router.back()"
      />
      <q-space />
      <q-btn
        :icon="post.is_liked ? 'favorite' : 'sym_o_favorite'"
        flat
        round
        dense
        color="red"
        size="16px"
        @click="executeHandleLike(handleLike, post.id)"
      />
      <q-btn
        :icon="post.is_disliked ? 'thumb_down' : 'sym_o_thumb_down'"
        flat
        round
        dense
        color="black"
        size="16px"
        @click="executeHandleDislike(handleDislike, post.id)"
      />
      <q-btn
        :icon="post.is_bookmarked ? 'bookmark' : 'sym_o_bookmark'"
        flat
        round
        dense
        color="blue"
        size="16px"
        @click="executeHandleBookmark(handleBookmark, post.id)"
      />
      <q-btn
        v-if="!authStore.hasOwnContent(post.author?.uid)"
        :icon="'sym_o_flag'"
        flat
        round
        dense
        color="black"
        size="16px"
        @click="openReportForm"
      />
    </div>
    <div class="flex items-center">
      <q-avatar>
        <img
          :src="post.author?.image || generateDefaultPhotoURL(post.author?.uid)"
        />
      </q-avatar>
      <div class="q-ml-md">
        <div>{{ post.author?.username }}</div>
        <div class="text-grey-6">
          {{ date.formatDate(post.created_at, 'YYYY. MM. DD HH:mm:ss') }}
        </div>
      </div>
      <q-space />
      <q-btn
        v-if="authStore.hasOwnContent(post.author?.uid)"
        icon="more_horiz"
        round
        flat
      >
        <q-menu>
          <q-list style="min-width: 100px">
            <q-item clickable v-close-popup :to="`/posts/${post.id}/edit`">
              <q-item-section>수정하기</q-item-section>
            </q-item>
            <q-item clickable v-close-popup @click="handleDeletePost">
              <q-item-section>삭제하기</q-item-section>
            </q-item>
          </q-list>
        </q-menu>
      </q-btn>
    </div>

    <div class="q-mt-md text-h5 text-weight-bold">{{ post.title }}</div>

    <q-separator class="q-my-lg" />

    <TiptapViewer v-if="post.content" :content="post.content" />
  </BaseCard>
  <!-- 인증 관련 다이얼로그 컴포넌트 -->
  <ReportForm
    v-model="reportForm"
    :postId="post.id"
    @close-dialog="reportForm = false"
  />
</template>

<script setup>
import { ref } from 'vue';
import { date, useQuasar } from 'quasar';
import { useRoute, useRouter } from 'vue-router';
import { useAuthStore } from 'stores/auth';
import { useAsyncState } from '@vueuse/core';

import {
  deletePost,
  getPostDetails,
  generateDefaultPhotoURL,
  handleLike,
  handleDislike,
  handleBookmark,
} from 'src/services';
import BaseCard from 'src/components/base/BaseCard.vue';
import TiptapViewer from 'src/components/tiptap/TiptapViewer.vue';
import ReportForm from 'src/components/report/ReportForm.vue';

const route = useRoute();
const router = useRouter();
const $q = useQuasar();
const authStore = useAuthStore();
const post = ref({});
const reportForm = ref(false);
const openReportForm = () => {
  if (!authStore.isLogin) {
    $q.notify({
      type: 'negative',
      message: '로그인이 필요합니다!',
    });
  } else {
    reportForm.value = true;
  }
};
const { error } = useAsyncState(
  getPostDetails(route.params.id),
  {},
  {
    immediate: true,
    onSuccess: response => {

      post.value = response.data;

    },
    onError: err => {
      $q.notify({
        type: 'negative',
        message: err.response.data.message,
      });
    },
  },
);
const { execute: executeDeletePost } = useAsyncState(deletePost, null, {
  immediate: false,
  onSuccess: () => {
    $q.notify('삭제완료!');
    router.push('/');
  },
  onError: err => {
    $q.notify({
      type: 'negative',
      message: err.response.data.message,
    });
  },
});
const handleDeletePost = async () => {
  $q.dialog({
    title: '알림',
    message: '삭제 하시겠어요?',
    persistent: true,
    cancel: true,
    ok: {
      push: true,
    },
    cancel: {
      push: true,
      color: 'negative',
    },
  }).onOk(async () => {
    await executeDeletePost(deletePost, route.params.id);
  });
};

const { execute: executeHandleLike } = useAsyncState(handleLike, [], {
  immediate: false,
  throwError: true,
  onSuccess: response => {
    post.value.is_liked = response.data.is_liked;
  },
  onError: err => {
    $q.notify({
      type: 'negative',
      message: err.response.data.message,
    });
  },
});
const { execute: executeHandleDislike } = useAsyncState(handleDislike, [], {
  immediate: false,
  throwError: true,
  onSuccess: response => {
    post.value.is_disliked = response.data.is_disliked;
  },
  onError: err => {
    $q.notify({
      type: 'negative',
      message: err.response.data.message,
    });
  },
});
const { execute: executeHandleBookmark } = useAsyncState(handleBookmark, [], {
  immediate: false,
  throwError: true,
  onSuccess: response => {
    post.value.is_bookmarked = response.data.is_bookmarked;
  },
  onError: err => {
    $q.notify({
      type: 'negative',
      message: err.response.data.message,
    });
  },
});
</script>

<style lang="scss" scoped></style>
