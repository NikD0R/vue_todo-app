<script setup>
import { defineProps, defineEmits, ref, nextTick } from 'vue';

const props = defineProps(['todo']);
const emit = defineEmits(['delete', 'update']);
const editing = ref(false);
const newTitle = ref(props.todo.title);
const titleField = ref(null);

const startEditing = async () => {
  newTitle.value = props.todo.title;
  editing.value = true;

  await nextTick();

  if (titleField.value) {
    titleField.value.focus();
  }
};

const rename = () => {
  if (!editing.value) return;

  editing.value = false;

  if (newTitle.value === props.todo.title) {
    return;
  }

  if (!newTitle.value) {
    emit('delete');
    return;
  }

  emit('update', { ...props.todo, title: newTitle.value });
};

</script>

<template>
  <div data-cy="Todo" class="todo" :class="{ completed: todo.completed }">
    <label class="todo__status-label">
      <input data-cy="TodoStatus" type="checkbox" class="todo__status" :checked="todo.completed"
        @change="emit('update', { ...todo, completed: !todo.completed })" />
    </label>

    <form v-if="editing" @submit.prevent="rename">
      <input data-cy="TodoTitleField" type="text" class="todo__title-field" placeholder="Empty todo will be deleted"
        @keyup.escape="editing = false" ref="titleField" v-model.trim="newTitle" @blur="rename" />
    </form>

    <template v-else>
      <span data-cy="TodoTitle" class="todo__title" @dblclick="startEditing">
        {{ todo.title }}
      </span>
      <button type="button" class="todo__remove" data-cy="TodoDelete" @click="emit('delete')">
        ×
      </button>
    </template>


    <!-- overlay will cover the todo while it is being deleted or updated -->
    <div data-cy="TodoLoader" class="modal overlay" :class="{ 'is-active': false }">
      <div class="modal-background has-background-white-ter"></div>
      <div class="loader"></div>
    </div>
  </div>
</template>