<script setup>
import { computed, ref, onMounted, TransitionGroup } from 'vue';
import StatusFilter from "./components/StatusFilter.vue";
import TodoItem from './components/TodoItem.vue';
import Message from './components/Message.vue';
import * as todoApi from './api/todos';

const todos = ref([]);
const title = ref('');
const errorMessage = ref(null);
const status = ref('all');

const activeTodos = computed(() =>
  todos.value.filter((todo) => !todo.completed)
);
const visibleTodos = computed(() => {
  if (status.value === 'active') {
    return activeTodos.value;
  }

  if (status.value === 'completed') {
    return todos.value.filter(todo => todo.completed);
  }

  return todos.value;
});

onMounted(async () => {
  try {
    todos.value = await todoApi.getTodos();
  } catch (error) {
    errorMessage.value.show('Unable to load todos');
  }
});
async function addTodo() {
  if (!title.value) {
    errorMessage.value.show("Title should not be empty");
    return;
  }

  try {
    const newTodo = await todoApi.createTodo(title.value);

    todos.value.push(newTodo);
    title.value = '';
  } catch (error) {
    errorMessage.value.show('Unable to add a todo');
  }
}

const deleteTodo = async todoId => {
  try {
    await todoApi.deleteTodo(todoId);
    todos.value = todos.value.filter(todo => todoId !== todo.id);
  } catch (error) {
    errorMessage.value.show('Unable to delete a todo');
  }
};

const updateTodo = async ({ id, title, completed }) => {
  try {
    const updatedTodo = await todoApi.updateTodo({ id, title, completed });
    const currentTodo = todos.value.find(todo => todo.id === id);

    Object.assign(currentTodo, updatedTodo);
  } catch (error) {
    errorMessage.value.show('Unable to update a todo');
  }
};
</script>

<template>
  <div class="todoapp">
    <h1 class="todoapp__title">todos {{ todos.length }}</h1>

    <div class="todoapp__content">
      <header class="todoapp__header">
        <!-- this button should have `active` class only if all todos are completed -->
        <button type="button" class="todoapp__toggle-all" data-cy="ToggleAllButton" v-if="todos.length > 0"
          :class="{ active: activeTodos.length === 0 }"></button>

        <!-- Add a todo on form submit -->
        <form @submit.prevent="addTodo">
          <input data-cy="NewTodoField" type="text" class="todoapp__new-todo" placeholder="What needs to be done?"
            v-model="title" @input="errorMessage = ''" />
        </form>
      </header>

      <TransitionGroup class="todoapp__main" data-cy="TodoList" tag="section" v-if="todos.length > 0" name="todolist">
        <TodoItem v-for="todo of visibleTodos" :key="todo.id" :todo="todo" @delete="deleteTodo(todo.id)"
          @update="updateTodo($event)" />
      </TransitionGroup>

      <!-- Hide the footer if there are no todos -->
      <footer class="todoapp__footer" data-cy="Footer" v-if="todos.length > 0">
        <span class="todo-count" data-cy="TodosCounter">
          {{ activeTodos.length }} items left
        </span>

        <StatusFilter v-model="status" />

        <!-- this button should be disabled if there are no completed todos -->
        <button type="button" class="todoapp__clear-completed" data-cy="ClearCompletedButton"
          :disabled="todos.length === activeTodos.length" @click="todos = activeTodos">
          Clear completed
        </button>
      </footer>
    </div>

    <!-- DON'T use conditional rendering to hide the notification -->
    <!-- Add the 'hidden' class to hide the message smoothly -->
    <div data-cy="ErrorNotification" class="notification is-danger is-light has-text-weight-normal" v-if="errorMessage">
      <button data-cy="HideErrorButton" type="button" class="delete" @click="errorMessage = ''"></button>
      <Message class="is-warning" ref="errorMessage">
        <template #header>
          <p>Server Error</p>
        </template>

        <template #default="{ text }">
          <p>{{ text }}</p>
        </template>
      </Message>
    </div>
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
