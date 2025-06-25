<template>
  <div class="todo-app">
    <h1>Todo List dengan Axios & JSON Server (CRUD lengkap)</h1>

    <!-- Form tambah todo baru -->
    <form @submit.prevent="addTodo">
      <input v-model="newTodo" placeholder="Tambah todo baru..." required />
      <button type="submit" :disabled="loading">Tambah</button>
    </form>

    <p v-if="loading">Memuat data...</p>
    <p v-if="error" style="color: red;">{{ error }}</p>

    <ul v-if="todos.length">
      <li v-for="todo in todos" :key="todo.id">
        <input
          type="checkbox"
          :checked="todo.completed"
          @change="toggleCompleted(todo)"
        />

        <!-- Tampilan normal -->
        <template v-if="editId !== todo.id">
          <span :style="{ textDecoration: todo.completed ? 'line-through' : 'none' }">
            {{ todo.title }}
          </span>
          <button @click="startEdit(todo)">Edit</button>
        </template>

        <!-- Form edit -->
        <template v-else>
          <input v-model="editTitle" />
          <button @click="updateTodo(todo)">Simpan</button>
          <button @click="cancelEdit">Batal</button>
        </template>

        <button @click="deleteTodo(todo.id)">Hapus</button>
      </li>
    </ul>

    <p v-else>Tidak ada todo.</p>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

// State
const todos = ref([]);
const newTodo = ref('');
const loading = ref(false);
const error = ref('');
const editId = ref(null);
const editTitle = ref('');

// URL API
const API_URL = 'http://localhost:3000/todos';

// Load data todo saat halaman dimuat
const loadTodos = async () => {
  loading.value = true;
  error.value = '';
  try {
    const response = await axios.get(API_URL);
    todos.value = response.data;
  } catch (err) {
    error.value = 'Gagal memuat data todo.';
    console.error(err);
  } finally {
    loading.value = false;
  }
};

// Tambah todo
const addTodo = async () => {
  if (!newTodo.value.trim()) return;

  loading.value = true;
  error.value = '';
  try {
    const newData = {
      title: newTodo.value,
      completed: false,
    };
    const response = await axios.post(API_URL, newData);
    todos.value.push(response.data);
    newTodo.value = '';
  } catch (err) {
    error.value = 'Gagal menambah todo.';
    console.error(err);
  } finally {
    loading.value = false;
  }
};

// Toggle todo selesai/belum
const toggleCompleted = async (todo) => {
  loading.value = true;
  error.value = '';
  try {
    const response = await axios.put(`${API_URL}/${todo.id}`, {
      ...todo,
      completed: !todo.completed,
    });
    const idx = todos.value.findIndex(t => t.id === todo.id);
    if (idx !== -1) todos.value[idx] = response.data;
  } catch (err) {
    error.value = 'Gagal memperbarui todo.';
    console.error(err);
  } finally {
    loading.value = false;
  }
};

// Hapus todo
const deleteTodo = async (id) => {
  loading.value = true;
  error.value = '';
  try {
    await axios.delete(`${API_URL}/${id}`);
    todos.value = todos.value.filter(todo => todo.id !== id);
  } catch (err) {
    error.value = 'Gagal menghapus todo.';
    console.error(err);
  } finally {
    loading.value = false;
  }
};

// Edit todo
const startEdit = (todo) => {
  editId.value = todo.id;
  editTitle.value = todo.title;
};

const cancelEdit = () => {
  editId.value = null;
  editTitle.value = '';
};

const updateTodo = async (todo) => {
  if (!editTitle.value.trim()) return;

  loading.value = true;
  error.value = '';
  try {
    const response = await axios.put(`${API_URL}/${todo.id}`, {
      ...todo,
      title: editTitle.value,
    });
    const idx = todos.value.findIndex(t => t.id === todo.id);
    if (idx !== -1) todos.value[idx] = response.data;
    editId.value = null;
    editTitle.value = '';
  } catch (err) {
    error.value = 'Gagal memperbarui todo.';
    console.error(err);
  } finally {
    loading.value = false;
  }
};

// Jalankan saat komponen dimount
onMounted(loadTodos);
</script>

<style scoped>
.todo-app {
  max-width: 500px;
  margin: 40px auto;
  padding: 20px 25px;
  background: #f9f9f9;
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

h1 {
  text-align: center;
  font-size: 1.6rem;
  margin-bottom: 20px;
  color: #333;
}

form {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

input[type="text"],
input[type="checkbox"],
input {
  flex-grow: 1;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 6px;
  font-size: 1rem;
  transition: border 0.2s ease-in-out;
}

input[type="text"]:focus {
  border-color: #007bff;
  outline: none;
}

button {
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 6px;
  padding: 10px 15px;
  font-size: 0.95rem;
  transition: background-color 0.2s ease-in-out;
}

button:hover:not(:disabled) {
  background-color: #0056b3;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

ul {
  list-style: none;
  padding-left: 0;
}

li {
  display: flex;
  align-items: center;
  background: #ffffff;
  padding: 10px 12px;
  border-radius: 8px;
  margin-bottom: 10px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
}

li span {
  flex-grow: 1;
  margin-left: 12px;
  font-size: 1rem;
  color: #333;
}

li input[type="text"] {
  flex-grow: 1;
  font-size: 1rem;
  padding: 8px;
}

li button {
  margin-left: 6px;
  font-size: 0.9rem;
  padding: 8px 10px;
}

p {
  text-align: center;
  font-size: 1rem;
  color: #666;
}

p[style*="color: red;"] {
  color: #e74c3c;
  font-weight: bold;
}
</style>

<style>
body {
  background-color: #eef2f7;
  margin: 0;
  padding: 0;
}
</style>

