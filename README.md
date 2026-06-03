<template>
  <div class="container">

    <h1>📋 業務改善タスク管理アプリ</h1>

    <!-- ダッシュボード -->
    <div class="stats-area">

      <div class="stat-card">
        <h3>総タスク</h3>
        <p>{{ tasks.length }}</p>
      </div>

      <div class="stat-card">
        <h3>完了</h3>
        <p>{{ completedTasks }}</p>
      </div>

      <div class="stat-card">
        <h3>未完了</h3>
        <p>{{ incompleteTasks }}</p>
      </div>

      <div class="stat-card">
        <h3>完了率</h3>
        <p>{{ completionRate }}%</p>
      </div>

    </div>

    <!-- 入力エリア -->
    <div class="input-area">

      <input
        v-model="task"
        placeholder="タスクを入力"
      />

      <select v-model="priority">

        <option value="高">高</option>
        <option value="中">中</option>
        <option value="低">低</option>

      </select>

      <input
        type="date"
        v-model="deadline"
      />

      <button @click="addTask">
        {{ editingIndex !== null ? '更新' : '追加' }}
      </button>

    </div>

    <!-- フィルター -->
    <div class="filter-area">

      <button @click="filter = '全て'">
        全て
      </button>

      <button @click="filter = '未完了'">
        未完了
      </button>

      <button @click="filter = '完了'">
        完了
      </button>

    </div>

    <!-- タスク一覧 -->
    <div
      v-for="(item, index) in filteredTasks"
      :key="index"
      class="task-card"
    >

      <div>

        <h2
          :class="[
            { done: item.done },
            item.priority
          ]"
        >
          {{ item.name }}
        </h2>

        <p>
          優先度：
          {{ item.priority }}
        </p>

        <p>
          締切：
          {{ item.deadline || '未設定' }}
        </p>

        <p>
          状態：
          {{ item.done ? '完了' : '未完了' }}
        </p>

      </div>

      <div class="button-area">

        <button @click="editTask(index)">
          編集
        </button>

        <button @click="toggleTask(index)">
          完了
        </button>

        <button @click="deleteTask(index)">
          削除
        </button>

      </div>

    </div>

  </div>
</template>

<script setup>
import {
  ref,
  watch,
  computed
} from 'vue'

const savedTasks =
  localStorage.getItem('tasks')

const tasks = ref(
  savedTasks
    ? JSON.parse(savedTasks)
    : []
)

const task = ref('')
const priority = ref('中')
const deadline = ref('')

const filter = ref('全て')

const editingIndex = ref(null)

const addTask = () => {

  if (!task.value) return

  if (editingIndex.value !== null) {

    tasks.value[editingIndex.value] = {
      ...tasks.value[editingIndex.value],
      name: task.value,
      priority: priority.value,
      deadline: deadline.value
    }

    editingIndex.value = null

  } else {

    tasks.value.push({
      name: task.value,
      done: false,
      priority: priority.value,
      deadline: deadline.value
    })

  }

  task.value = ''
  priority.value = '中'
  deadline.value = ''

}

const editTask = (index) => {

  const item = tasks.value[index]

  task.value = item.name
  priority.value = item.priority
  deadline.value = item.deadline

  editingIndex.value = index

}

const toggleTask = (index) => {

  tasks.value[index].done =
    !tasks.value[index].done

}

const deleteTask = (index) => {

  tasks.value.splice(index, 1)

}

const filteredTasks = computed(() => {

  if (filter.value === '未完了') {

    return tasks.value.filter(
      task => !task.done
    )

  }

  if (filter.value === '完了') {

    return tasks.value.filter(
      task => task.done
    )

  }

  return tasks.value

})

const completedTasks = computed(() => {

  return tasks.value.filter(
    task => task.done
  ).length

})

const incompleteTasks = computed(() => {

  return tasks.value.filter(
    task => !task.done
  ).length

})

const completionRate = computed(() => {

  if (tasks.value.length === 0) {
    return 0
  }

  return Math.round(
    (
      completedTasks.value /
      tasks.value.length
    ) * 100
  )

})

watch(
  tasks,
  (newTasks) => {

    localStorage.setItem(
      'tasks',
      JSON.stringify(newTasks)
    )

  },
  { deep: true }
)
</script>

<style>
body {
  margin: 0;
  font-family: sans-serif;
  background: #121212;
  color: white;
}

.container {
  max-width: 900px;
  margin: auto;
  padding: 30px;
}

h1 {
  text-align: center;
  margin-bottom: 30px;
}

.stats-area {
  display: grid;
  grid-template-columns:
    repeat(4, 1fr);

  gap: 15px;
  margin-bottom: 30px;
}

.stat-card {
  background: #1e1e1e;
  padding: 20px;
  border-radius: 12px;
  text-align: center;
}

.stat-card p {
  font-size: 24px;
  font-weight: bold;
}

.input-area {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.filter-area {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

input,
select,
button {
  padding: 10px;
  border: none;
  border-radius: 8px;
}

input {
  flex: 1;
}

button {
  cursor: pointer;
}

.task-card {
  background: #1e1e1e;
  padding: 20px;
  margin-bottom: 15px;
  border-radius: 12px;

  display: flex;
  justify-content: space-between;
  align-items: center;
}

.button-area {
  display: flex;
  gap: 10px;
}

.done {
  text-decoration: line-through;
  color: gray;
}

.高 {
  color: #ff6b6b;
}

.中 {
  color: #ffd43b;
}

.低 {
  color: #4dabf7;
}
</style>
