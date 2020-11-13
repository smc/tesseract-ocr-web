<template>
  <header>
    <h1>Optical Character Recognition</h1>
  </header>
  <main class="container">
    <div class="img-container">
      <ImageLoader />
    </div>
    <div class="actions">
      <progress v-if="showProgress" :value="progress" />
      <div class="status" v-if="showProgress">{{ status }}</div>
      <button @click="doOCR">Recognize</button>
    </div>
    <div class="text-container">
      <editor
        tinymce-script-src="/js/tinymce/tinymce.min.js"
        v-model="text"
        :init="editorConfig"
      ></editor>
    </div>
  </main>
</template>

<script lang="ts">
import { defineComponent, reactive, toRefs } from "vue";
import ImageLoader from "@/components/ImageLoader.vue";
import { createWorker, PSM, OEM, Worker } from "tesseract.js";
import axios from "axios";
import Editor from "@tinymce/tinymce-vue";

const recognize = async (
  worker: Worker,
  img: HTMLElement,
  language: string
) => {
  await worker.load();
  await worker.loadLanguage(language);
  await worker.initialize(language, OEM.LSTM_ONLY);
  await worker.setParameters({
    tessedit_pageseg_mode: PSM.SINGLE_BLOCK,
  });
  const {
    data: { text },
  } = await worker.recognize(img);
  return text;
};

async function spellcheck(method, text, success, failure) {
  if (method === "spellcheck") {
    const language = this.getLanguage();
    const api = `https://spell.toolforge.org/spellcheck/${language}`;
    axios
      .post(api, {
        text: text,
      })
      .then((response) => {
        const results = response.data;
        const misspellings = {};
        for (let i = 0; i < results.length; i++) {
          if (!results[i].spellcheck) {
            misspellings[results[i].word] = results[i].suggestions;
          }
        }

        success({ words: misspellings, dictionary: [] });
      })
      .catch((error) => {
        failure("Spellcheck error:" + error);
      });
  } else if (method === "addToDictionary") {
    success();
  }
}

export default defineComponent({
  name: "App",
  data: () => ({}),
  components: {
    ImageLoader,
    Editor,
  },
  setup() {
    const state = reactive({
      progress: 0,
      status: "",
      showProgress: false,
      text: "",
      language: "eng+mal",
      editorConfig: {
        height: 600,
        menubar: true,
        toolbar_mode: "sliding",
        toolbar_sticky: true,
        mobile: {
          menubar: true,
        },
        plugins: [
          "advlist autolink lists link image charmap print preview anchor",
          "searchreplace visualblocks code fullscreen",
          "insertdatetime media table paste code help wordcount spellchecker",
        ],
        spellchecker_languages: "English=en,Malayalam=ml",
        toolbar:
          "spellchecker| undo  redo | formatselect | fontselect | bold italic underline strikethrough codeformat | backcolor forecolor | alignleft aligncenter  alignright alignjustify | bullist numlist outdent indent | removeformat| help",
        spellchecker_callback: spellcheck,
      },
    });

    const worker = createWorker({
      logger: (m) => {
        console.log(m);
        state.progress = m.progress;
        state.status = m.status;
      },
    });

    const doOCR = async () => {
      const img = document.getElementById("ocr-img");
      if (img) {
        state.showProgress = true;
        state.text = await recognize(worker, img, state.language);
        state.showProgress = false;
      }
    };
    return { ...toRefs(state), doOCR, spellcheck };
  },
});
</script>

<style lang="less">
@import url(https://smc.org.in/fonts/manjari.css);

:root {
  --primary-color-h: 192;
  --primary-color-s: 100%;
  --primary-color-l: 41%;
  /* default */
  --primary-color: hsl(
    var(--primary-color-h),
    var(--primary-color-s),
    var(--primary-color-l)
  );

  /* darken */
  --primary-color--dark: hsl(
    var(--primary-color-h),
    var(--primary-color-s),
    calc(var(--primary-color-l) - 30%)
  );
}

body {
  display: flex;
  height: 100%;
  flex-direction: column;
  padding: 0;
  margin: 0;
}

#app {
  font-family: Helvetica, "Manjari", Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: var(--primary-color--dark);
}

header,
footer {
  flex: 1;
  background-color: var(--primary-color);
  color: #ffffff;
  padding: 4px;
}

header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  .title {
    color: #ffffff;
    text-decoration: none;
  }
  .title h1 {
    display: inline-flex;
  }
}

main {
  display: flex;
  flex: 1 0 auto;
  flex-direction: row;
  flex-wrap: wrap;
  padding: 0;
  margin: 0;
  justify-content: space-between;
  .img-container {
    width: 49vw;
    padding: 4px;
  }
  .text-container {
    width: 49vw;
    padding: 4px;
    min-height: 600px;
    textarea {
      font-size: 1.2em;
    }
  }
  .actions {
    order: 2;
  }
}

.actions {
  height: 100px;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  progress {
    height: 8px;
    width: 100%;
  }
  button {
    font-size: 1em;
  }
}
footer {
  min-height: fit-content;
  a {
    color: #ffffff;
  }
}

.content {
  flex: 1;
  padding: 8px;
  margin: 0;
}

@media (max-width: 600px) {
  main {
    flex-direction: column;
    .img-container {
      width: 100vw;
    }
    .text-container {
      width: 100vw;
    }
    .actions {
      order: 0;
    }
  }
}
</style>
