<template>
  <q-dialog
    :model-value="modelValue"
    @update:model-value="val => $emit('update:modelValue', val)"
    transition-show="none"
    transition-hide="none"
    @hide="changeViewMode('SignInForm')"
  >
    <q-card :style="{ width: '400px' }">
      <q-card-section class="flex">
        <q-space />
        <q-btn icon="close" flat round dense v-close-popup />
      </q-card-section>
      <q-card-section class="q-px-xl q-pb-xl">
        <component
          :is="authViewComponents[viewMode]"
          @change-view="changeViewMode"
          @close-dialog="closeDialog"
        />
      </q-card-section>
    </q-card>
  </q-dialog>
</template>
<script setup>
import { defineAsyncComponent, ref } from 'vue';

defineProps({
  modelValue: {
    type: Boolean,
    default: false,
  },
});
const emit = defineEmits(['update:modelValue', 'openWebsocket']);
const viewMode = ref('SignInForm');
const changeViewMode = mode => (viewMode.value = mode);
const authViewComponents = {
  SignInForm: defineAsyncComponent(() => import('./SignInForm.vue')),
  SignUpForm: defineAsyncComponent(() => import('./SignUpForm.vue')),
  FindPasswordForm: defineAsyncComponent(() =>
    import('./FindPasswordForm.vue'),
  ),
};
const closeDialog = () => {
  emit('update:modelValue', false);
};
</script>

<style lang="scss" scoped></style>
