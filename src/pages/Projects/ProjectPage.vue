<template>
  <div v-if="project" class="flex flex-col">
    <div class="flex flex-row items-center p-2">
      <div class="w-48 px-4">
        <div
          title="Switch board"
          class="w-full rounded-md p-2 text-gray-400 transition-colors hover:bg-gray-100 hover:text-gray-500 active:text-gray-600"
          @click="showBoardsSelector = true">
          <i class="fas fa-border-all fa-lg" />
          <span class="ml-2 font-bold">
            {{ board?.name }}
          </span>
        </div>
      </div>
      <h1 class="flex-grow text-center text-2xl">
        {{ project.name }}
      </h1>
      <div class="flex w-48 flex-row-reverse px-4">
        <RouterLink
          :to="
            $route.name !== 'project-settings-project' &&
            $route.name !== 'project-settings-board' &&
            $route.name !== 'project-settings-users'
              ? `/projects/${project.id}/settings`
              : `/projects/${project.id}`
          "
          class="block rounded-md p-2 text-gray-400 transition-colors hover:bg-gray-100 hover:text-gray-500 active:text-gray-600"
          :class="{
            'bg-gray-100 text-gray-600':
              $route.name === 'project-settings-project' ||
              $route.name === 'project-settings-board' ||
              $route.name === 'project-settings-users',
          }">
          <i class="fas fa-sliders fa-lg" />
        </RouterLink>
      </div>
    </div>

    <div class="flex-grow">
      <RouterView />
    </div>

    <ModalWindow v-if="showBoardsSelector" @close="showBoardsSelector = false">
      <BoardSelector
        :project="project"
        @switched="(boardId: number) => switchBoard(boardId)"
        @create="openBoardCreationModal" />
    </ModalWindow>

    <ModalWindow
      v-if="showCreateBoardModal"
      @close="showCreateBoardModal = false">
      <div
        class="w-1/3 min-w-max rounded-lg border border-gray-400 bg-white p-2 shadow-md">
        <form
          class="flex flex-col items-stretch space-y-6 p-8"
          @submit.prevent="createBoard">
          <h1 class="text-center text-2xl text-gray-800">New board</h1>
          <TextBox v-model="form.name" placeholder="Board name" />

          <div class="flex flex-row justify-center space-x-6">
            <AppButton type="submit" variant="primary" class="w-32"
              >Create</AppButton
            >
            <AppButton
              type="button"
              variant="secondary"
              class="w-32"
              @click="showCreateBoardModal = false"
              >Cancel</AppButton
            >
          </div>
        </form>
      </div>
    </ModalWindow>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import { mapStores } from 'pinia'
import { useProjectsStore } from '../../stores/projects/projects'
import BoardSelector from '../../components/BoardSelector.vue'
import ModalWindow from '../../components/ModalWindow.vue'
import { mapState } from 'pinia'
import { useTasksStore } from '../../stores/projects/tasks'

export default defineComponent({
  components: { BoardSelector, ModalWindow },
  data() {
    return {
      showBoardsSelector: false,
      showCreateBoardModal: false,
      form: {
        name: '',
      },
    }
  },
  computed: {
    ...mapStores(useProjectsStore, useTasksStore),
    ...mapState(useProjectsStore, ['project', 'board']),
  },
  async mounted() {
    await this.projectsStore.loadProject(this.$route.params.projectId as string)
    await this.projectsStore.loadBoard(this.$route.query.board as string)

    if (this.$route.query.task) {
      await this.tasksStore.openTask(Number(this.$route.query.task))
    }
  },
  methods: {
    async createBoard() {
      this.showCreateBoardModal = false

      this.projectsStore.createBoard({
        name: this.form.name,
      })

      this.form = {
        name: '',
      }
    },
    async switchBoard(boardId: number) {
      this.showBoardsSelector = false

      await this.projectsStore.loadBoard(boardId.toString())
    },
    openBoardCreationModal() {
      this.showBoardsSelector = false
      this.showCreateBoardModal = true
    },
  },
})
</script>
