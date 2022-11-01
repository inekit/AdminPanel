<template>
  <CButton color="primary" @click="addNewTag">Добавить тег</CButton>
  <CModal alignment="center" :visible="visible" @close="closeModal">
    <CModalHeader>
      <CModalTitle>Добавить тег</CModalTitle>
    </CModalHeader>
    <CModalBody>
      <form
        ref="add-file-form"
        @submit.prevent="addTag"
        class="add-user"
        style="display: 'none'"
      >
        <CFormInput class="mb-3" v-model="formData.name" placeholder="Тег" />
        <CFormTextarea v-model="formData.description" placeholder="Описание" />
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
      <CButton color="primary" @click="addTag">Добавить тег</CButton>
    </CModalFooter>
  </CModal>
</template>

<script>
import axios from 'axios'
const myApi = axios.create({
  withCredentials: true,
})
import eventBus from '../eventBus'

export default {
  props: {
    visible: false,
    formData: {
      name: '',
      description: '',
    },
  },

  methods: {
    addNewTag() {
      eventBus.$emit('addNewTag')
    },
    closeModal() {
      eventBus.$emit('closeModal')
    },
    constractFromData() {
      if (!this.formData.name || !this.formData.description) throw new Error()

      var formData = new FormData()

      formData.append('name', this.formData.name)
      formData.append('description', this.formData.description)

      return formData
    },
    addTag() {
      try {
        const formData = this.constractFromData()

        myApi
          .post(`http://127.0.0.1:3000/api/admin/tags`, formData, {
            headers: {
              'Content-Type': 'multipart/form-data',
            },
          })
          .then(() => {
            eventBus.$emit('tagAdded')
          })
          .catch((e) => {
            eventBus.$emit('tagAddingError', e)
          })
      } catch (e) {
        eventBus.$emit('wrongInputData', e)
      }
    },
  },
}
</script>
