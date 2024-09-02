<script setup>
import { onMounted, ref } from "vue";

const tasksModal = ref({
  tasks: [],
  editing: null
})

const createTask = (data, form$) => {

  addToStorage(form$.data)

  syncFromStorage()

  form$.reset()

}

const addToStorage = (data) => {
  let tasks = localStorage.getItem("tasks")

  const storedTasks = tasks ? JSON.parse(tasks) : []

  storedTasks.push(data)

  localStorage.setItem("tasks", JSON.stringify(storedTasks))
}

const syncFromStorage = () => {
  let tasks = localStorage.getItem("tasks")

  const storedTasks = tasks ? JSON.parse(tasks) : []

  tasksModal.value.tasks = storedTasks
}

const syncToStorage = () => {
  localStorage.setItem("tasks", JSON.stringify(tasksModal.value.tasks))

}

const edit = (index) => {

  tasksModal.value.editing = index

}

const cancel = () => {
  tasksModal.value.editing = null
  syncFromStorage()
}

const save = () => {
  syncToStorage()
  tasksModal.value.editing = null
}

onMounted(() => {
  syncFromStorage()
})

</script>

<template>
  <div class="page">
    <div class="container">
      <h1>The Task List</h1>

      <Vueform size="lg" :endpoint="createTask">

        <TextElement name="task" placeholder="add a task" floating="task name" rules="required" />

        <RadiogroupElement name="type" :items="['personal', 'business']" view="tabs" default="personal" />

        <ButtonElement name="button" align="right" submits>
          Create Task
        </ButtonElement>

      </Vueform>


      <hr class="divider">

      <Vueform v-model="tasksModal" sync>

        <ListElement name="tasks" :controls="{
          add: false
        }" :add-class="{
          handle: 'task-sort-handle'
        }" sort @sort="syncToStorage" @remove="syncToStorage">

          <template #default="{
            index
          }">
            <ObjectElement :name="index" :add-class="[
              'task-container',
              tasksModal.tasks[index].type === 'personal' ? 'is-personal' : 'is-business'

            ]">
              <ButtonElement :label="`#${index + 1} - ${tasksModal.tasks[index].task}`" name="edit" align="right"
                :conditions="[
                  ['editing', '!=', index]
                ]" :columns="{
                  label: 8
                }" @click="edit(index)">
                edit
              </ButtonElement>

              <TextElement name="task" :conditions="[
                ['editing', index]
              ]" :columns="6">
              </TextElement>

              <RadiogroupElement name="type" :items="{
                'personal': 'P',
                'business': 'B',
              }"
              :columns="2"
              :conditions="[
                ['editing', index]
              ]"

              view="tabs" default="personal" />

              <ButtonElement
                :conditions="[
                  ['editing', index]
                ]" :columns="2" @click="cancel" danger full name="cancel">
                cancel
              </ButtonElement>
              <ButtonElement
                :conditions="[
                  ['editing', index]
                ]" :columns="2" @click="save"  full name="save">
                save
              </ButtonElement>

            </ObjectElement>


          </template>

        </ListElement>

        <HiddenElement name="editing"></HiddenElement>
      </Vueform>

    </div>
  </div>
</template>

<style scoped lang="scss">
.page {
  background: #f1f5f9;
  width: 100%;
  min-height: 100vh;
  padding-top: 2rem;
}

.container {
  max-width: 600px;
  margin: 0 auto;
}

h1 {
  font-size: 48px;
  margin-bottom: 2rem;
  font-weight: 600;
}

.divider {
  margin: 2rem 0;
  border-color: #e2e8f0;
}

.task-container {

  background: #fff;
  padding: 1rem;


  &.is-personal {
    border-left: 3px solid green;
  }

  &.is-business {
    border-left: 3px solid purple;
  }
}

.task-wrapper {
  display: flex;
  align-items: center;
}

.vf-list-handle.task-sort-handle {
  top: 1rem;
}
</style>
