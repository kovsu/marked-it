<script setup lang="ts">
import { computed } from "@vue/reactivity";
import { onMounted, ref, watch } from "vue";
import { marked } from "marked";
import hljs from "highlight.js";

const rendererMD = new marked.Renderer();
marked.setOptions({
  renderer: rendererMD,
  highlight: function (code) {
    return hljs.highlightAuto(code).value;
  },
  pedantic: false,
  gfm: true,
  breaks: false,
  sanitize: false,
  smartLists: true,
  smartypants: false,
  xhtml: false,
});

let input = ref<string>(
  'A First Level Header\n====================\n\nA Second Level Header\n---------------------\n\nNow is the time for all good men to come to\nthe aid of their country. This is just a\nregular paragraph.\n\nThe quick brown fox jumped over the lazy\ndog\'s back.\n\n### Header 3\n\n> This is a blockquote.\n> \n> This is the second paragraph in the blockquote.\n>\n> ## This is an H2 in a blockquote\n\n```ts\nfunction() {\n  let a = 1;\n  let b = 2;\n  console.log("a");\n}\n```\n\n-   Candy.\n-   Gum.\n-   Booze.\n\nThis is an [example link](http://example.com/).\n\nI get 10 times more traffic from [Google][1] than from\n[Yahoo][2] or [MSN][3].\n[1]: http://google.com/        "Google"\n[2]: http://search.yahoo.com/  "Yahoo Search"\n[3]: http://search.msn.com/    "MSN Search"\n\n\nI start my morning with a cup of coffee and\n[The New York Times][NY Times].\n[ny times]: http://www.nytimes.com/\n\n\n![alt text](https://images.unsplash.com/photo-1660092626225-f291ab2970b9?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxlZGl0b3JpYWwtZmVlZHw4fHx8ZW58MHx8fHw%3D&auto=format&fit=crop&w=800&q=60 "Title")\n\nI strongly recommend against using any `<blink>` tags.\n\nI wish SmartyPants used named entities like `&mdash;`\ninstead of decimal-encoded entites like `&#8212;`.\n\n'
);
let textarea = ref<HTMLTextAreaElement | null>(null);
let leftBox = computed(() => {
  return document.querySelector(".leftBox");
});

function setNum() {
  let n = input.value.split("\n");
  for (let i = 0; i < n.length; i++) {
    const li = document.createElement("li");
    li.innerText = i + "";
    leftBox.value?.appendChild(li);
  }
}

function appendN() {
  const arr = input.value.split("\n");
  const length = arr.length;
  // console.log(arr[length - 1].length);

  // 文字内容超过容器宽度，换行更新行数（添加 \n）
  if (length === 1 && arr[length - 1].length >= 41) {
    input.value += "\n";
    return;
  }

  if (length > 1 && arr[length - 1].length >= 41) {
    input.value += "\n";
    return;
  }
}

watch(
  () => input.value,
  () => {
    leftBox.value!.innerHTML = "";
    setNum();
    appendN();
  }
);

// 同步 textarea 和 leftBox 的滚动条
function setSameScroll() {
  textarea.value!.addEventListener("scroll", () => {
    leftBox.value!.scrollTop = textarea.value!.scrollTop;
  });
}

onMounted(() => {
  textarea.value!.addEventListener("keydown", (e) => {
    if (e.keyCode == 9) {
      e.preventDefault();
      const start = textarea.value!.selectionStart;
      const end = textarea.value!.selectionEnd;
      if (start === undefined || end === undefined) return;
      const txt = textarea.value!.value;
      const result = txt.substring(0, start) + "  " + txt.substring(end);
      textarea.value!.value = result;

      // 光标聚焦到按下 tab 之后的位置
      textarea.value!.focus();
      textarea.value!.selectionStart = start + 2;
      textarea.value!.selectionEnd = start + 2;
    }
  });

  setNum();
  setSameScroll();
});

// debounce
// let timer: number | undefined;
// function debounce(fn: Function, delay = 1000) {
//   if (timer) {
//     clearTimeout(timer);
//     console.log("---");
//   }

//   timer = setTimeout(() => {
//     fn();
//   }, delay);
// }

let compiledMarkdown = computed(() => {
  const tokens = marked.lexer(input.value);
  // console.log(tokens);
  return marked.parser(tokens);
});

function update() {
  const target = event!.target as HTMLInputElement;
  input.value = target.value;
}

let words = computed(() => {
  return input.value.replace("\n", "").length;
});
</script>

<template>
  <div class="editor">
    <div class="left">
      <ul class="leftBox"></ul>
      <textarea :value="input" @input="update" ref="textarea"></textarea>
      <div class="words">{{ words }} characters</div>
    </div>
    <div class="markdown-body" v-html="compiledMarkdown"></div>
  </div>
</template>

<style>
@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500&display=swap");

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.editor {
  display: flex;
  padding: 1rem;
}

.left {
  position: relative;
  width: 45vw;
  height: 95vh;
  border: 1px solid rgb(97, 97, 97);
  overflow: hidden;
}

.editor textarea {
  position: absolute;
  padding-left: 3rem;
  width: 45vw;
  height: 100%;
  /* overflow: hidden; */
  overflow-y: scroll;

  font-size: 1rem;
  font-weight: 500;
  line-height: 1.6rem;
  font-family: "Poppins", sans-serif;

  resize: none;
  outline: none;
  border: none;

  word-wrap: break-word;
  word-break: break-all;
}

.editor .markdown-body {
  width: 50vw;
  height: 95vh;
  padding: 0 1rem;
}

.leftBox {
  position: absolute;
  z-index: 1;
  left: 0;
  width: 3rem;
  height: 100%;
  text-align: left;
  list-style: none;
  overflow-y: scroll;
}

.leftBox::-webkit-scrollbar {
  display: none;
}

.leftBox li {
  text-align: center;
  font-size: 1rem;
  line-height: 1.6rem;
  color: rgba(128, 128, 128, 0.5);
}

.words {
  position: absolute;
  bottom: 0.5rem;
  right: 1.5rem;
  font-size: 0.8rem;
  color: rgb(164, 164, 164);
}

.editor textarea::-webkit-scrollbar {
  width: 10px;
  height: 10px;
}

.editor textarea::-webkit-scrollbar-track {
  width: 6px;
  background: rgba(#101f1c, 0.1);
  -webkit-border-radius: 2em;
  -moz-border-radius: 2em;
  border-radius: 2em;
}

.editor textarea::-webkit-scrollbar-thumb {
  background-color: rgba(144, 147, 153, 0.5);
  background-clip: padding-box;
  min-height: 28px;
  -webkit-border-radius: 2em;
  -moz-border-radius: 2em;
  border-radius: 2em;
  transition: background-color 0.3s;
  cursor: pointer;
}

.editor textarea::-webkit-scrollbar-thumb:hover {
  background-color: rgba(144, 147, 153, 0.3);
}
</style>
