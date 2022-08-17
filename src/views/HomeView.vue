<script setup>
import axios from 'axios'
import { onMounted, ref, computed } from "@vue/runtime-core"

const STATUS_PENDING = 'pending'
const STATUS_COMPLETED = 'completed'
const API_BASE = 'http://127.0.0.1:8000/api'

let tasks = ref([])

onMounted(() => {
  fetchTasks()
})

const tasksAmount = computed(() => tasks.value.length )

async function fetchTasks() {
  try {
    const response = await axios.get(`${API_BASE}/tasks`)
    tasks.value = response.data.data
  } catch (err) {
    console.warn(err)
  }
}

let isLoading = ref(false)
let task = ref({})
async function handleSubmit() {
  isLoading.value = true
  try {
    await axios.post(
      `${API_BASE}/tasks`,
      task.value
    )
    task.value = {}
    fetchTasks()
  } catch (err) {
    console.warn(err)
  }
  isLoading.value = false
}

async function handleComplete(taskSelected) {
  const newStatus = taskSelected.status === STATUS_COMPLETED ? STATUS_PENDING : STATUS_COMPLETED
  task.value.status = newStatus

  try {
    await axios.put(
      `${API_BASE}/tasks/${taskSelected.id}`,
      task.value
    )

    tasks.value = tasks.value.map( _task => {
      if (_task.id === taskSelected.id) {
        return {
          ..._task,
          status: newStatus
        }
      }

      return _task
    })
  } catch (err) {
    console.warn(err)
  }
}

async function handleDelete(taskId) {
  try {
    await axios.delete(
      `${API_BASE}/tasks/${taskId}`,
    )

    tasks.value = tasks.value.filter(task => task.id !== taskId)
  } catch (err) {
    console.warn(err)
  }
}
</script>

<template>
  <div class="w-full w-96">
    <div class="mb-5">
      <h1 class="text-4xl text-white">Tasks <span class="text-gray-400">({{tasksAmount}})</span></h1>
    </div>


    <div class="w-full flex justify-center mb-10">
      <input class="w-full text-black pl-2 " placeholder="New task" v-model="task.name" />
      <button class="bg-blue-500 p-2" :disabled="isLoading" @click="handleSubmit">+Add</button>
    </div>


    <div v-for="task in tasks" :key="task.id" class="flex bg-gray-200 my-3 cursor-pointer" :class="[
      task.status === STATUS_COMPLETED && 'bg-green-300'
    ]"
    @click="handleComplete(task)">
      <div class="w-3/4 text-black p-2" :class="[task.status === STATUS_COMPLETED && 'line-through']">{{ task.name }}</div>
      <button class="w-1/4 bg-red-500" @click.stop="handleDelete(task.id)">x</button>
    </div>
  </div>
</template>
