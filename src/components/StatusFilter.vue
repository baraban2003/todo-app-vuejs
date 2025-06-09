<script setup>
import { defineProps, defineEmits } from 'vue';

const { status } = defineProps({
  status: String,
});

const emit = defineEmits(['change']);
</script>

<template>
  <nav class="filter">
    <a
      href="#/"
      class="filter__link"
      :class="{ selected: status === 'all' }"
      @click="emit('change', 'all')"
    >
      All
    </a>
    <a
      href="#/active"
      class="filter__link"
      :class="{ selected: status === 'active' }"
      @click="emit('change', 'active')"
    >
      Active
    </a>
    <a
      href="#/completed"
      class="filter__link"
      :class="{ selected: status === 'completed' }"
      @click="emit('change', 'completed')"
    >
      Completed
    </a>
  </nav>
</template>
