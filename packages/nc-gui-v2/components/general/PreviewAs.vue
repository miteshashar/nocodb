<script lang="ts" setup>
import { useI18n } from 'vue-i18n'
import { onUnmounted, useEventListener, useGlobal, useState, watch } from '#imports'
import MdiAccountStar from '~icons/mdi/account-star'
import MdiAccountHardHat from '~icons/mdi/account-hard-hat'
import MdiAccountEdit from '~icons/mdi/account-edit'
import MdiEyeOutline from '~icons/mdi/eye-outline'
import MdiCommentAccountOutline from '~icons/mdi/comment-account-outline'

const { float } = defineProps<{ float?: boolean }>()

const position = useState('preview-as-position', () => ({
  y: `${window.innerHeight - 100}px`,
  x: `${window.innerWidth / 2 - 250}px`,
}))

const { t } = useI18n()

const roleList = [
  { value: 'editor', label: t('objects.roleType.editor') },
  { value: 'commenter', label: t('objects.roleType.commenter') },
  { value: 'viewer', label: t('objects.roleType.viewer') },
]

const { previewAs } = useGlobal()

const roleIcon = {
  owner: MdiAccountStar,
  creator: MdiAccountHardHat,
  editor: MdiAccountEdit,
  viewer: MdiEyeOutline,
  commenter: MdiCommentAccountOutline,
}

const divMove = (e: MouseEvent) => {
  position.value = { y: `${e.clientY - 10}px`, x: `${e.clientX - 18}px` }
}
const mouseUp = () => {
  window.removeEventListener('mousemove', divMove, true)
}

useEventListener(window, 'mouseup', mouseUp, false)

const mouseDown = () => {
  window.addEventListener('mousemove', divMove, true)
}

onUnmounted(() => {
  window.removeEventListener('mousemove', divMove, true)
})

/** reload page on previewas change */
watch(previewAs, () => window.location.reload())
</script>

<template>
  <div
    v-if="float"
    v-show="previewAs"
    class="floating-reset-btn nc-floating-preview-btn p-4"
    :style="{ top: position.y, left: position.x }"
  >
    <MdiDrag style="cursor: move" class="text-white" @mousedown="mouseDown" />

    <div class="divider" />

    <div class="pointer flex items-center gap-4">
      <!-- Preview as -->
      <span>{{ $t('activity.previewAs') }}</span>

      <a-radio-group v-model:value="previewAs" name="radioGroup">
        <a-radio v-for="role of roleList" :key="role.value" class="capitalize !text-white" :value="role.value"
          >{{ role.label }}
        </a-radio>
      </a-radio-group>

      <div class="divider -ml-4" />

      <!-- Close -->
      <div class="flex items-center gap-2 cursor-pointer" @click="previewAs = null">
        <MdiExitToApp />
        {{ $t('general.close') }}
      </div>
    </div>
  </div>

  <template v-else>
    <template v-for="role of roleList" :key="role.value">
      <a-menu-item @click="previewAs = role.value">
        <div class="nc-project-menu-item group">
          <component :is="roleIcon[role.value]" class="group-hover:text-accent" />

          <span class="capitalize" :class="{ 'x-active--text': role.value === previewAs }">
            {{ role.label }}
          </span>
        </div>
      </a-menu-item>
    </template>

    <template v-if="previewAs">
      <a-menu-item @click="previewAs = null">
        <div class="nc-project-menu-item group">
          <MdiClose class="group-hover:text-accent" />
          <!-- Reset Preview -->
          <span class="text-capitalize text-xs whitespace-nowrap">
            {{ $t('activity.resetReview') }}
          </span>
        </div>
      </a-menu-item>
    </template>
  </template>
</template>

<style scoped>
.floating-reset-btn {
  @apply z-1000 index-100 fixed text-white
  @apply flex items-center overflow-hidden whitespace-nowrap gap-4 rounded shadow-md;
  background-color: #4351e7;
}

:deep(.ant-radio) {
  @apply transform scale-80;
}

.divider {
  @apply h-5 w-2px bg-white/50;
}
</style>
