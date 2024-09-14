<template>
    <div class="container mx-auto px-4 py-8">
      <h1 class="text-3xl font-bold mb-8 text-gray-800">Todo List</h1>
      
      <!-- Add Task Form -->
      <form @submit.prevent="addTask" class="mb-8 bg-white shadow-md rounded px-8 pt-6 pb-8 mb-4">
        <div class="mb-4">
          <input v-model="newTask.name" placeholder="Task Name" required
                 class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline">
        </div>
        <div class="mb-4">
          <select v-model="newTask.status"
                  class="block appearance-none w-full bg-white border border-gray-400 hover:border-gray-500 px-4 py-2 pr-8 rounded shadow leading-tight focus:outline-none focus:shadow-outline">
            <option value="pending">Pending</option>
            <option value="in_progress">In Progress</option>
            <option value="completed">Completed</option>
          </select>
        </div>
        <div class="mb-4 w-1/5">
  <input v-model="newTask.deadline" type="date" required
         @input="formatDate"
         class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline">
</div>
        <div class="flex items-center justify-between">
          <button type="submit"
                  class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline">
            Add Task
          </button>
        </div>
      </form>
  
      <!-- Tasks Table -->
      <div class="bg-white shadow-md rounded my-6">
        <table class="text-left w-full border-collapse">
          <thead>
            <tr>
              <th class="py-4 px-6 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">Name</th>
              <th class="py-4 px-6 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">Status</th>
              <th class="py-4 px-6 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">Deadline</th>
              <th class="py-4 px-6 bg-grey-lightest font-bold uppercase text-sm text-grey-dark border-b border-grey-light">Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="task in tasks" :key="task.id" class="hover:bg-grey-lighter">
              <td class="py-4 px-6 border-b border-grey-light">{{ task.name }}</td>
              <td class="py-4 px-6 border-b border-grey-light">
                <span :class="getStatusClass(task.status)">{{ task.status }}</span>
              </td>
              <td class="py-4 px-6 border-b border-grey-light">{{ formatDate(task.deadline) }}</td>
              <td class="py-4 px-6 border-b border-grey-light">
                <button @click="editTask(task)" class="text-blue-600 font-bold py-1 px-3 rounded text-xs bg-blue-100 hover:bg-blue-200 mr-2">Edit</button>
                <button @click="deleteTask(task.id)" class="text-red-600 font-bold py-1 px-3 rounded text-xs bg-red-100 hover:bg-red-200">Delete</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
  
      <!-- Edit Task Modal -->
      <div v-if="editingTask" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full" id="my-modal">
        <div class="relative top-20 mx-auto p-5 border w-96 shadow-lg rounded-md bg-white">
          <h3 class="text-lg font-bold mb-4">Edit Task</h3>
          <form @submit.prevent="updateTask">
            <div class="mb-4">
              <input v-model="editingTask.name" required
                     class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline">
            </div>
            <div class="mb-4">
              <select v-model="editingTask.status"
                      class="block appearance-none w-full bg-white border border-gray-400 hover:border-gray-500 px-4 py-2 pr-8 rounded shadow leading-tight focus:outline-none focus:shadow-outline">
                <option value="pending">Pending</option>
                <option value="in_progress">In Progress</option>
                <option value="completed">Completed</option>
              </select>
            </div>
            <div class="mb-4">
              <input v-model="editingTask.deadline" type="date" required
                     class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline">
            </div>
            <div class="flex items-center justify-between">
              <button type="submit"
                      class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline">
                Update
              </button>
              <button @click="cancelEdit"
                      class="bg-gray-500 hover:bg-gray-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline">
                Cancel
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

const tasks = ref([]);
const newTask = ref({
  name: '',
  status: 'pending',
  deadline: '',
});
const editingTask = ref(null);

const fetchTasks = async () => {
  try {
    const response = await axios.get('/tasks');
    tasks.value = response.data;
  } catch (error) {
    console.error('Error fetching tasks:', error);
  }
};

const addTask = async () => {
  try {
    const response = await axios.post('/tasks', newTask.value);
    tasks.value.push(response.data);
    newTask.value = { name: '', status: 'pending', deadline: '' };
  } catch (error) {
    console.error('Error adding task:', error);
  }
};

const editTask = (task) => {
  editingTask.value = { 
    ...task,
    deadline: formatDateForInput(task.deadline)
  };
};

const updateTask = async () => {
  try {
    const response = await axios.put(`/tasks/${editingTask.value.id}`, editingTask.value);
    const index = tasks.value.findIndex(t => t.id === editingTask.value.id);
    tasks.value.splice(index, 1, response.data);
    editingTask.value = null;
  } catch (error) {
    console.error('Error updating task:', error);
  }
};

const deleteTask = async (id) => {
  if (confirm('Are you sure you want to delete this task?')) {
    try {
      await axios.delete(`/tasks/${id}`);
      tasks.value = tasks.value.filter(t => t.id !== id);
    } catch (error) {
      console.error('Error deleting task:', error);
    }
  }
};

const cancelEdit = () => {
  editingTask.value = null;
};

const formatDate = (date) => {
  const d = new Date(date);
  const month = (d.getMonth() + 1).toString().padStart(2, '0');
  const day = d.getDate().toString().padStart(2, '0');
  const year = d.getFullYear();
  return `${month}/${day}/${year}`;
};

const formatDateForInput = (date) => {
  const d = new Date(date);
  const day = d.getDate().toString().padStart(2, '0');
  const month = (d.getMonth() + 1).toString().padStart(2, '0');
  const year = d.getFullYear();
  return `${year}-${month}-${day}`;
};

const formatDates = (event) => {
  const date = new Date(event.target.value);
  const month = (date.getMonth() + 1).toString().padStart(2, '0');
  const day = date.getDate().toString().padStart(2, '0');
  const year = date.getFullYear();
  newTask.value.deadline = `${year}-${month}-${day}`;
};

const getStatusClass = (status) => {
  switch (status) {
    case 'pending':
      return 'bg-yellow-200 text-yellow-800 py-1 px-3 rounded-full text-xs';
    case 'in_progress':
      return 'bg-blue-200 text-blue-800 py-1 px-3 rounded-full text-xs';
    case 'completed':
      return 'bg-green-200 text-green-800 py-1 px-3 rounded-full text-xs';
    default:
      return '';
  }
};

onMounted(fetchTasks);
</script>
