<template>
    <div class="editor-container">
        <v-card class="rich-text-editor" outlined>
            <v-toolbar dense flat>
            <v-btn icon @click="editor?.chain().focus().toggleBold().run()" :color="editor?.isActive('bold') ? 'primary' : ''">
                <v-icon>mdi-format-bold</v-icon>
            </v-btn>
            <v-btn icon @click="editor?.chain().focus().toggleItalic().run()" :color="editor?.isActive('italic') ? 'primary' : ''">
                <v-icon>mdi-format-italic</v-icon>
            </v-btn>
            <v-btn icon @click="editor?.chain().focus().toggleUnderline().run()" :color="editor?.isActive('underline') ? 'primary' : ''">
                <v-icon>mdi-format-underline</v-icon>
            </v-btn>
            <v-btn icon @click="editor?.chain().focus().toggleStrike().run()" :color="editor?.isActive('strike') ? 'primary' : ''">
                <v-icon>mdi-format-strikethrough</v-icon>
            </v-btn>
            <v-btn icon @click="editor?.chain().focus().toggleHighlight().run()" :color="editor?.isActive('highlight') ? 'primary' : ''">
                <v-icon>mdi-format-color-highlight</v-icon>
            </v-btn>
            <v-btn icon @click="editor?.chain().focus().toggleBulletList().run()" :color="editor?.isActive('bulletList') ? 'primary' : ''">
                <v-icon>mdi-format-list-bulleted</v-icon>
            </v-btn>
            <v-btn icon @click="editor?.chain().focus().toggleOrderedList().run()" :color="editor?.isActive('orderedList') ? 'primary' : ''">
                <v-icon>mdi-format-list-numbered</v-icon>
            </v-btn>
            <v-btn icon @click="fileInput?.click()">
                <v-icon>mdi-image</v-icon>
            </v-btn>
            <input
                ref="fileInput"
                type="file"
                accept="image/*"
                style="display: none"
                @change="onFileInputChange"
            >
            </v-toolbar>
            <v-card-text class="pa-2">
            <editor-content :editor="editor" @drop.prevent="onDrop" />
            </v-card-text>
        </v-card>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { useEditor, EditorContent } from '@tiptap/vue-3'
import StarterKit from '@tiptap/starter-kit'
import Underline from '@tiptap/extension-underline'
import TextStyle from '@tiptap/extension-text-style'
import Highlight from '@tiptap/extension-highlight'
import Image from '@tiptap/extension-image'

const props = defineProps<{
  modelValue: string
}>()

const emit = defineEmits<{
  (e: 'update:modelValue', value: string): void
}>()

const editor = useEditor({
  content: props.modelValue,
  extensions: [
    StarterKit,
    Underline,
    TextStyle,
    Highlight,
    Image,
  ],
  onUpdate: ({ editor }) => {
    emit('update:modelValue', editor.getHTML())
  },
  editorProps: {
    handlePaste: (view, event) => {
      const items = Array.from(event.clipboardData?.items || [])
      for (const item of items) {
        if (item.type.indexOf('image') === 0) {
          event.preventDefault()
          const blob = item.getAsFile()
          if (blob) {
            const reader = new FileReader()
            reader.onload = (e) => {
              const url = e.target?.result
              if (typeof url === 'string') {
                view.dispatch(view.state.tr.replaceSelectionWith(
                  view.state.schema.nodes.image.create({ src: url })
                ))
              }
            }
            reader.readAsDataURL(blob)
          }
          return true
        }
      }
      return false
    },
  },
})

const fileInput = ref<HTMLInputElement | null>(null)

const addImage = (file: File) => {
  const reader = new FileReader()
  reader.onload = (e) => {
    const url = e.target?.result
    if (typeof url === 'string' && editor.value) {
      editor.value.chain().focus().setImage({ src: url }).run()
    }
  }
  reader.readAsDataURL(file)
}

const onFileInputChange = (event: Event) => {
  const input = event.target as HTMLInputElement
  if (input.files?.length) {
    addImage(input.files[0])
  }
}

const onDrop = (event: DragEvent) => {
  const file = event.dataTransfer?.files[0]
  if (file && file.type.startsWith('image/')) {
    addImage(file)
  }
}

</script>

<style>
.editor-container {
  max-width: 90%;
  margin: 20px auto;
  padding: 0 20px;
}

.rich-text-editor {
  width: 100%;
  border: 1px solid #e0e0e0;
  border-radius: 4px;
}

.rich-text-editor .ProseMirror {
  min-height: 100px;
  padding: 8px;
  line-height: 1.2;
}

.rich-text-editor .ProseMirror p {
  margin: 0 0 0.3em 0;
}

.rich-text-editor .ProseMirror p:last-child {
  margin-bottom: 0;
}

.rich-text-editor .ProseMirror:focus {
  outline: none;
}

.ProseMirror img {
  max-width: 100%;
  height: auto;
}
</style>