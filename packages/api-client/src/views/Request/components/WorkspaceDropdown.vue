<script setup lang="ts">
import DeleteSidebarListElement from '@/components/Sidebar/Actions/DeleteSidebarListElement.vue'
import RenameSidebarListElement from '@/components/Sidebar/Actions/RenameSidebarListElement.vue'
import { commandPaletteBus } from '@/libs/event-busses'
import { useWorkspace } from '@/store'
import {
  ScalarButton,
  ScalarContextMenu,
  ScalarDropdown,
  ScalarDropdownDivider,
  ScalarDropdownItem,
  ScalarIcon,
  ScalarModal,
  ScalarTooltip,
  useModal,
} from '@scalar/components'
import { computed, ref } from 'vue'
import { useRouter } from 'vue-router'

const { activeWorkspace, workspaces, workspaceMutators } = useWorkspace()
const { push } = useRouter()

const updateSelected = (uid: string) => {
  if (uid === activeWorkspace.value.uid) return
  push(`/workspace/${uid}`)
}

const isLastWorkspace = computed(() => Object.keys(workspaces).length === 1)

const createNewWorkspace = () =>
  commandPaletteBus.emit({ commandName: 'Create Workspace' })

const tempName = ref('')
const tempUid = ref('')
const renameModal = useModal()
const deleteModal = useModal()

const openRenameModal = (uid: string) => {
  tempName.value = workspaces[uid].name
  tempUid.value = uid
  renameModal.show()
}

const handleWorkspaceRename = () => {
  if (tempName.value.trim()) {
    workspaceMutators.edit(tempUid.value, 'name', tempName.value.trim())
    renameModal.hide()
  }
}

const openDeleteModal = (uid: string) => {
  tempName.value = workspaces[uid].name
  tempUid.value = uid
  deleteModal.show()
}

const deleteWorkspace = async () => {
  if (!isLastWorkspace.value) {
    const isActiveWorkspace = activeWorkspace.value.uid === tempUid.value
    const currentWorkspaces = { ...workspaces }
    delete currentWorkspaces[tempUid.value]

    workspaceMutators.delete(tempUid.value)

    if (isActiveWorkspace) {
      // Switch to another workspace if the active one is deleted
      const newWorkspaceUid = Object.keys(currentWorkspaces)[0]
      await push(`/workspace/${newWorkspaceUid}/`)
    }
  }
  deleteModal.hide()
}
</script>

<template>
  <div
    class="xl:min-h-header xl:py-2.5 py-1 flex items-center border-b-1/2 px-2.5 text-sm w-[inherit]">
    <ScalarDropdown>
      <ScalarButton
        class="font-normal h-full justify-start line-clamp-1 py-1.5 px-1.5 text-c-1 hover:bg-b-2 w-fit"
        fullWidth
        variant="ghost">
        <div class="font-medium m-0 text-sm flex gap-1.5 items-center">
          <h2 class="line-clamp-1 text-left w-[calc(100%-10px)]">
            {{ activeWorkspace.name }}
          </h2>
          <ScalarIcon
            class="size-2.5"
            icon="ChevronDown"
            thickness="3.5" />
        </div>
      </ScalarButton>

      <!-- Workspace list -->
      <template #items>
        <ScalarContextMenu
          v-for="(workspace, uid) in workspaces"
          :key="uid">
          <template #trigger>
            <ScalarDropdownItem
              class="flex gap-1.5 group/item items-center whitespace-nowrap text-ellipsis overflow-hidden w-full"
              @click.stop="updateSelected(uid)">
              <div
                class="flex items-center justify-center rounded-full p-[3px] w-4 h-4 group-hover/item:shadow-border"
                :class="
                  activeWorkspace.uid === uid
                    ? 'bg-blue text-b-1'
                    : 'text-transparent'
                ">
                <ScalarIcon
                  class="size-2.5"
                  icon="Checkmark"
                  thickness="3.5" />
              </div>
              <span class="text-ellipsis overflow-hidden">{{
                workspace.name
              }}</span>
            </ScalarDropdownItem>
          </template>
          <template #content>
            <ScalarDropdown static>
              <template #items>
                <ScalarDropdownItem
                  class="flex gap-2"
                  @mousedown="openRenameModal(uid)">
                  <ScalarIcon
                    class="inline-flex"
                    icon="Edit"
                    size="md"
                    thickness="1.5" />
                  <span>Rename</span>
                </ScalarDropdownItem>
                <ScalarTooltip
                  v-if="isLastWorkspace"
                  class="z-10"
                  side="bottom">
                  <template #trigger>
                    <ScalarDropdownItem
                      class="flex gap-2 w-full"
                      disabled
                      @mousedown.prevent>
                      <ScalarIcon
                        class="inline-flex"
                        icon="Delete"
                        size="md"
                        thickness="1.5" />
                      <span>Delete</span>
                    </ScalarDropdownItem>
                  </template>
                  <template #content>
                    <div
                      class="grid gap-1.5 pointer-events-none min-w-48 w-content shadow-lg rounded bg-b-1 z-100 p-2 text-xxs leading-5 z-10 text-c-1">
                      <div class="flex items-center text-c-2">
                        <span>Only workspace cannot be deleted.</span>
                      </div>
                    </div>
                  </template>
                </ScalarTooltip>
                <ScalarDropdownItem
                  v-else
                  class="flex !gap-2"
                  @mousedown.prevent="openDeleteModal(uid)">
                  <ScalarIcon
                    class="inline-flex"
                    icon="Delete"
                    size="sm"
                    thickness="1.5" />
                  <span>Delete</span>
                </ScalarDropdownItem>
              </template>
            </ScalarDropdown>
          </template>
        </ScalarContextMenu>
        <ScalarDropdownDivider />

        <!-- Add new workspace -->
        <ScalarDropdownItem
          class="flex items-center gap-1.5"
          @click="createNewWorkspace">
          <div class="flex items-center justify-center h-4 w-4">
            <ScalarIcon
              class="h-2.5"
              icon="Add"
              thickness="3" />
          </div>
          <span>Create new workspace</span>
        </ScalarDropdownItem>
      </template>
    </ScalarDropdown>
  </div>
  <ScalarModal
    :size="'xxs'"
    :state="deleteModal"
    title="Delete workspace">
    <DeleteSidebarListElement
      :variableName="tempName"
      warningMessage="This cannot be undone. You’re about to delete the workspace and everything inside it."
      @close="deleteModal.hide()"
      @delete="deleteWorkspace" />
  </ScalarModal>
  <ScalarModal
    :size="'xxs'"
    :state="renameModal"
    title="Rename Workspace">
    <RenameSidebarListElement
      :name="tempName"
      @close="renameModal.hide()"
      @rename="handleWorkspaceRename" />
  </ScalarModal>
</template>
