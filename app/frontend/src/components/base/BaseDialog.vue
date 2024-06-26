<script setup>
import { storeToRefs } from 'pinia';
import { computed } from 'vue';
import { useFormStore } from '~/store/form';

defineProps({
  modelValue: {
    default: false,
    type: Boolean,
  },
  type: {
    default: null,
    type: String,
  },
  showCloseButton: {
    default: false,
    type: Boolean,
  },
  width: {
    default: '500',
    type: String,
  },
  enableCustomButton: {
    default: false,
    type: Boolean,
  },
});

const emit = defineEmits([
  'update:modelValue',
  'close-dialog',
  'continue-dialog',
  'delete-dialog',
  'custom-dialog',
]);

const { isRTL, lang } = storeToRefs(useFormStore());

const RTL = computed(() => (isRTL.value ? 'ml-5' : 'mr-5'));

function closeDialog() {
  emit('close-dialog');
}

function continueDialog() {
  emit('continue-dialog');
}

function deleteDialog() {
  emit('delete-dialog');
}

function customDialog() {
  emit('custom-dialog');
}

defineExpose({ RTL });
</script>

<template>
  <v-dialog
    :max-width="width"
    persistent
    :model-value="modelValue"
    @click:outside="closeDialog"
    @keydown.esc="closeDialog"
  >
    <v-card>
      <div class="dialog-body" :class="{ 'dir-rtl': isRTL }">
        <div v-if="showCloseButton">
          <v-spacer />
          <v-icon
            color="primary"
            class="float-right m-3"
            icon="mdi-close"
            @click="closeDialog"
          ></v-icon>
        </div>
        <v-card-title class primary-title>
          <slot name="title"></slot>
        </v-card-title>
        <v-card-text>
          <div class="dialog-icon">
            <slot name="icon">
              <v-icon size="medium">default-icon</v-icon>
            </slot>
          </div>
          <div class="dialog-text" :lang="lang">
            <slot name="text">{{ $t('trans.baseDialog.defaultText') }}</slot>
          </div>
        </v-card-text>
      </div>
      <v-card-actions class="justify-center">
        <div v-if="type === 'OK'">
          <v-btn
            class="mb-5"
            color="primary"
            variant="flat"
            :title="$t('trans.baseDialog.ok')"
            @click="closeDialog"
          >
            <slot name="button-text">
              <span :lang="lang">{{ $t('trans.baseDialog.ok') }}</span>
            </slot>
          </v-btn>
        </div>
        <div v-else-if="type === 'CONTINUE'">
          <v-btn
            data-test="continue-btn-continue"
            class="mb-5"
            color="primary"
            variant="flat"
            :title="$t('trans.baseDialog.continue')"
            @click="continueDialog"
          >
            <slot name="button-text-continue">
              <span :lang="lang">{{ $t('trans.baseDialog.continue') }}</span>
            </slot>
          </v-btn>
          <v-btn
            data-test="continue-btn-cancel"
            class="mb-5"
            :class="RTL"
            variant="outlined"
            :title="$t('trans.baseDialog.cancel')"
            @click="closeDialog"
          >
            <slot name="button-text-cancel">
              <span :lang="lang">{{ $t('trans.baseDialog.cancel') }}</span>
            </slot>
          </v-btn>
        </div>
        <div v-else-if="type === 'SAVEDDELETE'">
          <v-btn
            class="mb-5 mr-5"
            :class="RTL"
            color="primary"
            variant="flat"
            :title="$t('trans.baseDialog.continue')"
            @click="continueDialog"
          >
            <slot name="button-text-continue">
              <span :lang="lang">{{ $t('trans.baseDialog.continue') }}</span>
            </slot>
          </v-btn>
          <v-btn
            data-test="saveddelete-btn-cancel"
            class="mb-5"
            variant="outlined"
            :title="$t('trans.baseDialog.cancel')"
            @click="deleteDialog"
          >
            <slot name="button-text-delete">
              <span :lang="lang">{{ $t('trans.baseDialog.cancel') }}</span>
            </slot>
          </v-btn>
        </div>
        <div v-else-if="type === 'CUSTOM'">
          <v-btn
            class="mb-5 mr-5"
            color="primary"
            variant="flat"
            :title="$t('trans.baseDialog.continue')"
            @click="continueDialog"
          >
            <slot name="button-text-continue">
              <span :lang="lang">{{ $t('trans.baseDialog.continue') }}</span>
            </slot>
          </v-btn>
          <v-btn
            v-if="enableCustomButton"
            data-test="custom-btn-custom"
            class="mb-5 mr-5"
            color="primary"
            variant="flat"
            :title="$t('trans.baseDialog.custom')"
            @click="customDialog"
          >
            <slot name="button-text-custom">
              <span :lang="lang">{{ $t('trans.baseDialog.custom') }}</span>
            </slot>
          </v-btn>
          <v-btn
            class="mb-5"
            variant="outlined"
            :title="$t('trans.baseDialog.cancel')"
            @click="closeDialog"
          >
            <slot name="button-text-cancel">
              <span :lang="lang">{{ $t('trans.baseDialog.cancel') }}</span>
            </slot>
          </v-btn>
        </div>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<style scoped>
.v-card-text {
  display: flex !important;
  padding: 1.5rem;
}
.dialog-icon {
  margin-right: 1rem;
  object-fit: contain;
  align-self: flex-start;
}
.dialog-text {
  flex: 1 1 auto;
  width: 90%;
}
</style>
