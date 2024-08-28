<template>
    <div>
      <h1>Rich Text Editor</h1>
      <RichTextEditor v-model="content" />
      <div class="output">
        <h2>Output:</h2>
        <div v-html="content" class="content"></div>
      </div>
    </div>
  </template>
  
  <script setup lang="ts">
  import { ref } from 'vue'
  
  const content = ref('<p>Hello, world!</p>')
  </script>
  
  <style>
.output {
  margin-top: 20px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.output h2 {
  margin-top: 0;
  margin-bottom: 10px;
}

.output .content {
  padding: 8px;
  line-height: 1.2; /* Adjusted to match input */
}

.output .content p {
  margin: 0;
  padding: 0;
  line-height: 1.2; /* Adjusted to match input */
}

/* Override any default paragraph styles */
.output .content p + p {
  margin-top: 0.3em; /* Add a small space between paragraphs */
}

/* Ensure inline elements don't add extra height */
.output .content * {
  vertical-align: baseline;
}
</style>