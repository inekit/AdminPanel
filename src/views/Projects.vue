<template>
  <div>
    <AddProjectModal
      :visible="formVisible"
      :formData="formData"
      :mode="formMode"
    />
    <Table
      :fields="tableFieldNames"
      :postData="get"
      :actions="dataActions"
      :rows="rows"
      editMode="form"
      name="Проекты"
    />
  </div>
</template>

<script>
import AddProjectModal from '@/components/AddProjectModal.vue'
import Table from '@/components/Table.vue'
import eventBus from '../eventBus'

import axios from 'axios'
const myApi = axios.create({
  withCredentials: true,
})

export default {
  name: 'Posts',
  components: {
    AddProjectModal,
    Table,
  },
  data() {
    return {
      myApi: myApi,
      formVisible: false,
      formData: {},
      rows: [],
      dataActions: {
        Посты: { action: this.routeToPosts, color: 'primary' },
        Удалить: { action: this.delete, color: 'danger' },
      },
      tableFieldNames: [
        {
          name: 'name',
          title: 'Название',
        },
        {
          name: 'description',
          title: 'Описание',
        },
      ],
    }
  },
  created() {
    eventBus.$on('addNewProject', () => {
      this.formMode = 'new'
      this.formVisible = true
    })
    eventBus.$on('closeModal', () => {
      this.formVisible = false
      this.formData = {}
    })
    eventBus.$on('projectAdded', () => {
      this.formVisible = false
      this.get()
      this.formData = {}
    })
  },
  methods: {
    change(elObj) {
      this.formVisible = true
      this.formData = elObj
      this.formMode = 'edit'
    },
    get(take, page) {
      return myApi
        .get('http://127.0.0.1:3000/api/projects/', {
          params: {
            take: take ?? 10,
            page: page ?? 1,
          },
        })
        .then((res) => {
          if (res.data?.length > 0) this.rows = res.data
          return res.data
        })
        .catch((error) => {
          eventBus.emit('noresponse', error)
          //alert('Сервер не отвечает')
          return false
        })
    },
    delete(item) {

      const result = confirm('Вы действительно хотите удалить пользователя?')
      if (result)
        return myApi
          .delete('http://127.0.0.1:3000/api/admin/projects/', {
            data: { name: item.name },
          })
          .then(() => {
            this.get()
            //this.rows = this.rows.filter((el) => el.id !== id)
            //alert('Пользователь удален')
          })
          .catch(() => {
            alert('Сервер не отвечает')
          })
    },
    routeToPosts(item) {
      this.$router.push('/posts/project/' + item.name)
    },
  },
}
</script>

<style lang="scss">
button {
  margin-bottom: 20px;
}
</style>
