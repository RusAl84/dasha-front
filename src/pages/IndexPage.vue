<template>
  <q-page>
    <q-card class="chatmessages">
      <div class="flexrow">
        <div>
          <q-card-section>
            <div class="text-h6">Загрузка данных:</div>
          </q-card-section>
        </div>
        <div>
          <q-card-section>
            <q-uploader
              :url="hostae_uploadae"
              label="Загрузите свои данные .json, chatMessages/*"
              square
              flat
              bordered
              single
              @uploaded="fileUploaded"
              accept=".json, chatMessages/*"
              style="min-width: 600px; max-width: 600px"
            />
          </q-card-section>
        </div>
      </div>
      <q-card-section>
        <div class="editorclass">
          <q-editor
            v-model="chatmessages"
            max-height="300px"
            :dense="$q.screen.lt.md"
            :toolbar="[
              [
                {
                  label: $q.lang.editor.align,
                  icon: $q.iconSet.editor.align,
                  fixedLabel: true,
                  list: 'only-icons',
                  options: ['left', 'center', 'right', 'justify'],
                },
                {
                  label: $q.lang.editor.align,
                  icon: $q.iconSet.editor.align,
                  fixedLabel: true,
                  options: ['left', 'center', 'right', 'justify'],
                },
              ],
              [
                'bold',
                'italic',
                'strike',
                'underline',
                'subscript',
                'superscript',
              ],
              ['token', 'hr', 'link', 'custom_btn'],
              ['print', 'fullscreen'],
              [
                {
                  label: $q.lang.editor.formatting,
                  icon: $q.iconSet.editor.formatting,
                  list: 'no-icons',
                  options: ['p', 'h1', 'h2', 'h3', 'h4', 'h5', 'h6', 'code'],
                },
                {
                  label: $q.lang.editor.fontSize,
                  icon: $q.iconSet.editor.fontSize,
                  fixedLabel: true,
                  fixedIcon: true,
                  list: 'no-icons',
                  options: [
                    'size-1',
                    'size-2',
                    'size-3',
                    'size-4',
                    'size-5',
                    'size-6',
                    'size-7',
                  ],
                },
                {
                  label: $q.lang.editor.defaultFont,
                  icon: $q.iconSet.editor.font,
                  fixedIcon: true,
                  list: 'no-icons',
                  options: [
                    'default_font',
                    'arial',
                    'arial_black',
                    'comic_sans',
                    'courier_new',
                    'impact',
                    'lucida_grande',
                    'times_new_roman',
                    'verdana',
                  ],
                },
                'removeFormat',
              ],
              ['quote', 'unordered', 'ordered', 'outdent', 'indent'],

              ['undo', 'redo'],
              ['viewsource'],
            ]"
            :fonts="{
              arial: 'Arial',
              arial_black: 'Arial Black',
              comic_sans: 'Comic Sans MS',
              courier_new: 'Courier New',
              impact: 'Impact',
              lucida_grande: 'Lucida Grande',
              times_new_roman: 'Times New Roman',
              verdana: 'Verdana',
            }"
          />
        </div>
      </q-card-section>
    </q-card>
    <q-card class="chatmessages">
      <div class="flexrow">
        <div>
          <q-card-section>
            <div class="text-h6">Поиск целевой информации:</div>
          </q-card-section>
          <q-input
            class="inputtext"
            v-model="find_text"
            type="textarea"
            autogrow
            float-label="Введите текст для анализа"
          />
          <q-card-section>
            <q-btn color="primary" label="Поиск" @click="onFind_data" />
          </q-card-section>
          <q-card-section>
            <div class="text-h6">Полученная сигнатура (сжатый портрет):</div>
          </q-card-section>
          <q-input
            class="inputtext"
            v-model="sig_text"
            type="textarea"
            readonly
            autogrow
          />
          <q-card-section>
            <q-btn
              color="primary"
              label="Получить сигнатуры (сжатый портрет)"
              @click="onGet_Sig"
            />
          </q-card-section>
        </div></div
    ></q-card>

    <q-card class="chatmessages">
      <div class="flexrow">
        <div>
          <q-card-section>
            <div class="text-h6">
              Найденные активные эксплоиты в корпоративных диалоговых текстах:
            </div>
          </q-card-section>
          <q-card-section class="q-gutter-lg">
            <div>
              <q-select
                filled
                v-model="ttype"
                :options="options"
                label="Выберите алгоритм"
              />
            </div>
            <div>
              <q-btn
                color="primary"
                label="Поиск активных эксплоитов"
                @click="onFindCL"
              />
            </div>
          </q-card-section>
          <!-- <q-card-section>
            {{ filename }}
          </q-card-section> -->
        </div>
      </div>
      <q-card-section v-if="ae_data.length > 0">
        <div v-for="(line, i) in ae_data" :key="i">
          <div>
            <b>Найденный фрагмент &nbsp; {{ i + 1 }}</b>
          </div>
          <div><b>Исходный текст сообщения:</b> {{ line.text }}</div>
          <div><b>Сжатый портрет: </b>{{ line.RAKE }}</div>
          <br />
        </div> </q-card-section
    ></q-card>
  </q-page>
</template>

<script>
import { defineComponent, ref } from "vue";
import axios from "axios";

export default {
  data() {
    return {
      file: "",
      find_data: "http://127.0.0.1:5000/find_data",
      data_proc: "http://127.0.0.1:5000/data_proc",
      get_sig: "http://127.0.0.1:5000/get_sig",
      chatmessages: "",
      find_text: ref(""),
      ttype: ref(""),
      sig_text: ref(""),
      resp_data: ref(""),
      all_data: ref([{}]),
      ae_data: ref([{}]),
      filename: ref(""),
      options: ["RAKE", "YAKE", "BERT"],
    };
  },
  methods: {
    handleFileUpload() {
      this.file = this.$refs.file.files[0];
    },
    fileUploaded({ files, xhr }) {
      let data = JSON.parse(xhr.response);
      this.chatmessages = data["text"];
      this.filename = data["filename"];
    },
    async onFind_data() {
      const response = await axios({
        method: "post",
        url: this.find_data,
        data: {
          find_text: this.find_text,
        },
      });
      console.log(response);
      this.resp_data = response.data;
      this.resp_text = response.data.print_text;
    },
    async onGet_Sig() {
      const response = await axios({
        method: "post",
        url: this.get_sig,
        data: {
          text: this.find_text,
        },
      });
      console.log(response);
      this.sig_text = response.data;
    },
    async onProcAdd() {
      const response = await axios({
        method: "post",
        url: "get_pattern_add",
        data: this.resp_data,
      });
      this.all_data = response.data;
      console.log("this.resp_data");
      console.log(this.all_data);
    },
  },
};
</script>

<style lang="sass">
.page
  padding-bottom: 10px
  padding-top: 10px
  padding-left: 10px
  padding-r: 10px

.chatmessages
  padding: 10px
  margin: 10px
.flexrowЁ
  display: flex
  flex-flow: row wrap
  justify-content: flex-start
.editorclass
  max-height: 500px
.inputtext
  width: 800px
</style>
