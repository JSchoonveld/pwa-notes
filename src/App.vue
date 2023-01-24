<script setup>
import Tiptap from "./components/Tiptap.vue";
</script>

<template>
  <div class="flex w-screen h-screen text-gray-700">
    <div class="flex flex-col flex-shrink-0 w-64 border-r border-gray-300 bg-gray-100">

    </div>
    <div class="flex flex-col flex-grow">
      <div class="flex flex-col flex-grow overflow-auto">
        <editor-content :editor="editor"></editor-content>
      </div>
    </div>
  </div>
</template>

<script>
import { Editor, EditorContent } from "@tiptap/vue-3";
import StarterKit from "@tiptap/starter-kit";

export default {
  name: 'App',
  components: {
    EditorContent
  },
  data() {
    return {
      editor: null
    }
  },
  mounted() {
    this.editor = new Editor({
      content: '',
      extensions: [
          StarterKit
      ],
      editorProps: {
        attributes: {
          class: "prose lg:prose-xl my-6 mx-auto focus:outline-none"
        }
      }
    })
  },
  beforeUnmount() {
    this.editor.destroy();
  }
}
</script>

<style>
h1 {
  font-size: 24px;
  font-weight: bold;
  color: red;
}

h2 {
  font-size: 20px;
  color: blue;
}
</style>