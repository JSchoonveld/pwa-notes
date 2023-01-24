<script setup>
</script>

<template>
  <div class="flex w-screen h-screen text-gray-700">
    <div class="flex flex-col flex-shrink-0 w-64 border-r border-gray-300 bg-gray-100 p-5 items-center">
      <a v-for="note in notes" :key="note.created_at" href="#" class="my-2">{{new Date(note.created_at).toLocaleString()}}</a>
    </div>
    <div class="flex flex-col flex-grow">
      <div class="flex flex-col flex-grow overflow-auto container mx-auto">
        <div class="border-2 my-10 border-black rounded">
          <div class="border-b-2 border-black">
            <button @click="setBold" class="hover:bg-black focus:bg-black focus:text-white hover:text-white text-black rounded m-1 font-bold p-1 w-10">b</button>
            <button @click="setHeading('H1')" class="hover:bg-black focus:bg-black focus:text-white hover:text-white text-black rounded m-1 font-bold p-1 w-10">H1</button>
            <button @click="setHeading('H2')" class="hover:bg-black focus:bg-black focus:text-white hover:text-white text-black rounded m-1 font-bold p-1 w-10">H2</button>
          </div>
          <editor-content :editor="editor"></editor-content>
        </div>
      </div>
      <div class="h-16 bg-gray-100 border-t border-gray-300 flex items-center justify-end">
        <button @click="saveNote" class="border-2 border-black rounded hover:bg-black hover:text-white px-3 py-1 mx-10 text-xl font-semibold">Save Note</button>
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
      editor: null,
      heading: null,
      database: null,
      notes: [],
    }
  },
  async created() {
    this.database = await this.getDatabase();
    let notes = await this.getNotes();
    this.notes = notes.reverse();
  },
  mounted() {
    this.editor = new Editor({
      content: '',
      extensions: [
          StarterKit
      ],
      editorProps: {
        attributes: {
          class: "prose lg:prose-xl focus:outline-none p-5 w-full"
        }
      }
    })
  },
  beforeUnmount() {
    this.editor.destroy();
  },
  methods: {
    setBold() {
      this.editor.commands.setBold()
    },
    setHeading(heading) {
      this.editor.commands.insertContent(`<${heading}></${heading}>`)
    },
    async saveNote() {
      return new Promise((resolve, reject) => {
        let transaction = this.database.transaction(["notes"], 'readwrite');

        transaction.oncomplete = e => {
          resolve();
        }

        let now = new Date();

        let note = {
          content: this.editor.getHTML(),
          created_at: now.getTime(),
        }
        this.notes.unshift(note)
        transaction.objectStore('notes').add({
          content: this.editor.getHTML(),
          created_at: now.getTime(),
        })
      })
    },
    async getDatabase() {
      return new Promise((resolve, reject) => {
        let db = window.indexedDB.open("notes", 1.15);

        db.onerror = ev => {
          console.log('db.onerror' , ev)
          reject('Error opening the database');
        }

        db.onsuccess = ev => {
          console.log('db.onsuccess' , ev)
          resolve(ev.target.result);
        }

        db.onupgradeneeded = ev => {
          console.log('db.onupgradeneeded' , ev)
          ev.target.result.deleteObjectStore("notes");
          ev.target.result.createObjectStore("notes", { keyPath: "created_at" });
        }
      })
    },
    async getNotes() {
      return new Promise((resolve, reject) => {
        let notes = this.database.transaction('notes')
          .objectStore('notes')
          .getAll()
          .onsuccess = e => {
          resolve(e.target.result);
        }
      })
    }
  },
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
