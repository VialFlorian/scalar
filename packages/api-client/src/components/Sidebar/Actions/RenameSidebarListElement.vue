<script setup lang="ts">
import SidebarListElementForm from '@/components/Sidebar/Actions/SidebarListElementForm.vue'
import { ScalarTextField } from '@scalar/components'
import { ref } from 'vue'

const props = defineProps<{
  name: string
}>()

const emit = defineEmits<{
  (e: 'close'): void
  (e: 'rename', newName: string): void
}>()

const newName = ref(props.name)
</script>
<template>
  <SidebarListElementForm
    @cancel="emit('close')"
    @submit="emit('rename', newName)">
    <ScalarTextField
      v-model="newName"
      autofocus />
  </SidebarListElementForm>
</template>
