<template>
  <!-- 로그인 폼의 메인 컨테이너 -->
  <div>
    <!-- 로그인 폼의 제목 -->
    <div class="text-h5 text-center text-weight-bold q-mb-xl">로그인</div>

    <!-- 로그인 폼 -->
    <q-form class="q-gutter-y-md" @submit.prevent="handleSignInEmail">
      <!-- 이메일 입력 필드 -->
      <q-input v-model="form.email" placeholder="이메일" outlined dense />
      <!-- 비밀번호 입력 필드 -->
      <q-input
        v-model="form.password"
        type="password"
        placeholder="비밀번호"
        outlined
        dense
      />
      <div>
        <!-- 로그인 버튼 -->
        <q-btn
          type="submit"
          label="로그인하기"
          class="full-width"
          unelevated
          color="primary"
          :loading="isLoading"
        />
        <div class="flex justify-between">
          <!-- 비밀번호 찾기 버튼 -->
          <q-btn
            label="비밀번호 찾기"
            color="secondary"
            flat
            dense
            size="13px"
            @click="$emit('changeView', 'FindPasswordForm')"
          />
          <!-- 회원가입 전환 버튼 -->
          <q-btn
            label="이메일 가입하기"
            color="secondary"
            flat
            dense
            size="13px"
            @click="$emit('changeView', 'SignUpForm')"
          />
        </div>
        <q-separator />
        <q-btn
          label="카카오 계정으로 로그인하기"
          class="full-width"
          unelevated
          color="primary"
          outline
          :href="getKakaoLoginUrl()"
        />
      </div>
      <!-- 구분선 -->
      <q-separator />
    </q-form>
  </div>
</template>

<script setup>
import { api } from 'src/boot/axios-config';
import { ref } from 'vue';
import { useQuasar } from 'quasar';
import { useAsyncState } from '@vueuse/core';
import { signInWithEmail } from 'src/services';

import { useAuthStore } from 'src/stores/auth';
const emit = defineEmits(['changeView', 'closeDialog', 'openWebsocket']);
const $q = useQuasar();
const authStore = useAuthStore();
const { isLoading, execute } = useAsyncState(
  async () => {
    await signInWithEmail(form.value);
  },
  null,
  {
    immediate: false,
    throwError: true,
    onSuccess: async () => {
      authStore.setAuthentication(true);
      $q.notify('환영합니다 :)');
      emit('closeDialog');
    },
    onError: err => {
      $q.notify({
        type: 'negative',
        message: err.response.data.message,
      });
    },
  },
);

// 로그인 폼 데이터 모델
const form = ref({
  email: '',
  password: '',
  username: null,
  passwordConfirm: null,
});
const getKakaoLoginUrl = () => {
  const kakaoParams = {
    client_id: 'f079d63a6ac2d6ce0971e8acfa3f0917',
    redirect_uri: 'http://localhost:9030/social/kakao',
    response_type: 'code',
  };
  const kakaoUrl = `https://kauth.kakao.com/oauth/authorize?${new URLSearchParams(
    kakaoParams,
  ).toString()}`;
  return kakaoUrl;
};
// 로그인 처리 함수
const handleSignInEmail = () => execute(form.value);
</script>

<style lang="scss" scoped></style>
