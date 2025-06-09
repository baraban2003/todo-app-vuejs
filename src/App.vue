<script setup>
import { computed, ref, TransitionGroup, onMounted } from 'vue';
import * as todoApi from './api/todos';
import StatusFilter from './components/StatusFilter.vue';
import TodoItem from './components/TodoItem.vue';
import Message from './components/Message.vue';

let initialTodos = [];

const todos = ref(initialTodos);

const title = ref('');

const status = ref('all');

const errorMessage = ref(null);

onMounted(async () => {
  try {
    todos.value = await todoApi.getTodos();
  } catch (error) {
    errorMessage.value.show('Unable to load todos');
  }
});

const addTodo = async () => {
  if (!title.value.trim()) {
    errorMessage.value.show('Title should not be empty');
    return;
  }

  try {
    const newTodo = await todoApi.createTodo(title.value.trim());

    todos.value.push(newTodo);
    title.value = '';
  } catch (error) {
    errorMessage.value.show('Unable to add a todo');
  }
};

const deleteTodo = async (todoId) => {
  try {
    await todoApi.deleteTodo(todoId);
    todos.value = todos.value.filter((todo) => todoId !== todo.id);
  } catch (error) {
    errorMessage.value.show('Unable to delete a todo');
  }
};

const updateTodo = async ({ id, title, completed }) => {
  try {
    const updatedTodo = await todoApi.updateTodo({ id, title, completed });
    const currentTodo = todos.value.find((todo) => todo.id === id);

    Object.assign(currentTodo, updatedTodo);
  } catch (error) {
    errorMessage.value.show('Unable to update a todo');
  }
};

const activeTodos = computed(() =>
  todos.value.filter((todo) => !todo.completed)
);

const toggleAll = () => {
  const allCompleted = activeTodos.value.length === 0;
  todos.value.forEach((todo) => {
    todo.completed = !allCompleted;
  });
};

const visibleTodos = computed(() => {
  if (status.value === 'active') {
    return activeTodos.value;
  }

  if (status.value === 'completed') {
    return todos.value.filter((todo) => todo.completed);
  }

  return todos.value;
});
</script>

<template>
  <div class="todoapp">
    <h1 class="todoapp__title">todos</h1>

    <header class="todoapp__header">
      <div class="todoapp__content">
        <button
          v-if="todos.length > 0"
          class="todoapp__toggle-all"
          :class="{ active: activeTodos.length === 0 }"
          @click="toggleAll"
        ></button>
        <form @submit.prevent="addTodo">
          <input
            class="todoapp__new-todo"
            placeholder="What needs to be done?"
            v-model="title"
          />
        </form>
      </div>
    </header>

    <TransitionGroup
      tag="section"
      name="todolist"
      class="todoapp__main"
      v-if="todos.length > 0"
    >
      <TodoItem
        v-for="todo of visibleTodos"
        :key="todo.id"
        :todo="todo"
        @delete="deleteTodo(todo.id)"
        @update="updateTodo($event)"
      />
    </TransitionGroup>

    <footer className="todoapp__footer">
      <div class="todo-count">{{ activeTodos.length }} items left</div>

      <StatusFilter :status="status" @change="status = $event" />

      <button
        class="todoapp__clear-completed"
        :disabled="todos.length === activeTodos.length"
        @click="todos = activeTodos"
      >
        Clear completed
      </button>
    </footer>

    <Message class="is-danger" ref="errorMessage">
      <template #header>
        <p>Server Error</p>
      </template>

      <template #default="{ text }">
        <p>{{ text }}</p>
      </template>
    </Message>
  </div>
</template>

<style scoped>
.todolist-enter-active,
.todolist-leave-active {
  max-height: 60px;
  transition: all 0.5s ease;
}
.todolist-enter-from,
.todolist-leave-to {
  opacity: 0;
  max-height: 0;
  transform: scaleY(0);
}
</style>
