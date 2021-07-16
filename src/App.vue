<template>
  <div class="container">
    <div class="editor" ref="json"></div>
    <div class="editor" ref="ts"></div>
  </div>
</template>

<script setup="props" lang="ts">
import { onMounted, ref } from 'vue';
import { editor } from 'monaco-editor';
import EditorWorker from 'monaco-editor/esm/vs/editor/editor.worker?worker';
import JsonWorker from 'monaco-editor/esm/vs/language/json/json.worker?worker';
import TsWorker from 'monaco-editor/esm/vs/language/typescript/ts.worker?worker';
import JsonToTS from 'json-to-ts';
import IStandaloneCodeEditor = editor.IStandaloneCodeEditor;

declare global {
  interface Window {
    MonacoEnvironment: {
      getWorker: (workerId: number, label: string) => any
    }
  }
}

self.MonacoEnvironment = {
  getWorker(workerId, label) {
    if (label === 'json') {
      return new JsonWorker();
    } else if (label === 'typescript') {
      return new TsWorker();
    }
    return new EditorWorker();
  },
};

const json = ref();
const ts = ref();

let jsonEdit: IStandaloneCodeEditor;
let tsEdit: IStandaloneCodeEditor;

onMounted(() => {
  const jsonModel = editor.createModel(
    '',
    'json',
  );

  const tsModel = editor.createModel(
    '',
    'typescript',
  );

  jsonEdit = editor.create(json.value, {
    model: jsonModel,
    automaticLayout: true,
    tabSize: 2,
    theme: 'vs-dark',
    fontSize: 16,
  });

  tsEdit = editor.create(ts.value, {
    model: tsModel,
    automaticLayout: true,
    tabSize: 2,
    theme: 'vs-dark',
    fontSize: 16,
  });

  jsonEdit.onDidChangeModelContent(() => {
    try {
      tsEdit.setValue(JsonToTS(JSON.parse(jsonEdit.getValue())).join('\r\r'));
    } catch (e) {
      tsEdit.setValue(e.toString());
    }
  });
});
</script>

<style scoped>
.container {
  position: fixed;
  height: 100%;
  width: 100%;
  display: flex;
}

.editor {
  flex: 1;
  height: 100%;
  max-width: 50%;
}
</style>
