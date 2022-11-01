<template>
  <CButton color="primary" @click="addNewPost">Добавить новость</CButton>
  <CModal
    size="xl"
    backdrop="static"
    alignment="center"
    :visible="visible"
    @close="closeModal"
  >
    <CModalHeader>
      <CModalTitle>Добавить новость</CModalTitle>
    </CModalHeader>
    <CModalBody>
      <form
        ref="add-file-form"
        @submit.prevent="mode === 'new' ? addNewing() : editNewing()"
        class="add-user"
        style="display: 'none'"
      >
        <CFormInput
          class="mb-3"
          v-model="formData.title"
          placeholder="Заголовок"
        />
        <div class="tags-cloud">
          <span>Теги</span>
          <CFormCheck
            v-for="tag in tags"
            :key="tag.name"
            :id="tag.name"
            :checked="formData.tags_array?.has(tag.name)"
            @change="changeT"
            type="checkbox"
            :value="tag.name"
            :label="tag.name"
          />
        </div>
        <div class="projects-list">
          <span>Проект</span>
          <CFormCheck
            v-for="project in projects"
            :key="project.name"
            :id="project.name"
            :checked="project.name === formData.project_name"
            @change="changeP"
            type="radio"
            name="project-name"
            :value="project.name"
            :label="project.name"
          />
          <CFormCheck
            id="null-name"
            :checked="!formData.project_name"
            @change="changeP"
            type="radio"
            name="project-name"
            value=""
            label="Вне проекта"
          />
        </div>
        <QuillEditor
          theme="snow"
          toolbar="essential"
          ref="postTextEditor"
          id="postTextEditor"
          placeholder="Текст статьи"
        />
      </form>
    </CModalBody>
    <CModalFooter>
      <CButton
        color="secondary"
        @click="
          () => {
            visible = false
          }
        "
      >
        Отменить
      </CButton>
      <CButton v-show="mode === 'new'" color="primary" @click="addNewing"
        >Добавить пост</CButton
      >
      <CButton v-show="mode === 'edit'" color="primary" @click="editNewing"
        >Редактировать пост</CButton
      >
    </CModalFooter>
  </CModal>
</template>

<script>
import axios from 'axios'
const myApi = axios.create({
  withCredentials: true,
})
import eventBus from '../eventBus'
import { QuillEditor } from '@vueup/vue-quill'
import '@vueup/vue-quill/dist/vue-quill.snow.css'

export default {
  components: { QuillEditor },
  props: {
    mode: {
      required: true,
      type: String,
      validator: (value) => ['new', 'edit'].includes(value.toLowerCase()),
    },
    visible: false,
    formData: {
      title: '',
      text: '',
      preview: '',
      tags_array: new Set(),
    },
  },
  updated() {
    this.formData.text && this.$refs.postTextEditor?.setHTML(this.formData.text)
    console.log(this.formData)
  },
  async mounted() {
    this.tags = await this.getTagCloud()
    this.projects = await this.getProjects()
    console.log(this.tags, this.projects)
  },
  methods: {
    async getTagCloud() {
      const res = await myApi
        .get('http://127.0.0.1:3000/api/tags/')
        .catch((error) => {
          eventBus.emit('noresponse', error)
        })

      return res.data
    },
    async getProjects() {
      const res = await myApi
        .get('http://127.0.0.1:3000/api/projects/')
        .catch((error) => {
          eventBus.emit('noresponse', error)
        })

      return res.data
    },
    addNewPost() {
      eventBus.$emit('addNewPost')
    },
    changeP(e){
      console.log(e.target.value)
      this.formData.project_name = e.target.value
    },
    changeT(e){
      console.log(e.target.checked)
      if (e.target.checked) this.formData.tags_array.add(e.target.value)
      else this.formData.tags_array.delete(e.target.value)
    },
    closeModal() {
      eventBus.$emit('closeModal')
    },
    handleFileUpload(event) {
      this.formData.preview = event.target.files[0]
    },
    constractFromData(isEdit) {
      if (!this.formData.title || !this.$refs.postTextEditor.getHTML())
        throw new Error()

      var formData = new FormData()

      formData.append('title', this.formData.title)
      formData.append('text', this.$refs.postTextEditor.getHTML())

      this.formData.project_name && formData.append('projectName', this.formData.project_name)
      const tags_array = Array.from(this.formData.tags_array)
      for (var i = 0; i < tags_array.length; i++) {
        console.log(tags_array[i])
        formData.append(`tagsArray`, tags_array[i])
      }

      isEdit && formData.append('id', this.formData.id)

      return formData
    },
    addNewing() {
      console.log(this.$refs.postTextEditor.getHTML())
      try {
        const formData = this.constractFromData()

        myApi
          .post(`http://127.0.0.1:3000/api/admin/posts`, formData, {
            headers: {
              'Content-Type': 'multipart/form-data',
            },
          })
          .then(() => {
            eventBus.$emit('postAdded')
          })
          .catch((e) => {
            eventBus.$emit('postAddingError', e)
          })
      } catch (e) {
        eventBus.$emit('wrongInputData', e)
      }
    },
    editNewing() {
      try {
        const formData = this.constractFromData(true)

        myApi
          .put(`http://127.0.0.1:3000/api/admin/posts`, formData, {
            headers: {
              'Content-Type': 'multipart/form-data',
            },
          })
          .then(() => {
            eventBus.$emit('postEdited')
          })
          .catch((e) => {
            eventBus.$emit('postEditingError', e)
          })
      } catch (e) {
        console.log(e)
        eventBus.$emit('wrongInputData', e)
      }
    },
  },
}
</script>

<style lang="scss" scoped>
.tags-cloud,
.projects-list {
  display: flex;
  flex-wrap: wrap;
  & > * {
    margin-right: 20px;
  }
  & > span {
    flex: 0 0 100%;
    margin-bottom: 10px;
  }
}
</style>
