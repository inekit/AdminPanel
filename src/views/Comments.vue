<template>
  <div>
   <CommentModal :visible="formVisible" :formData="formData"/>
    <Table :fields="tableFieldNames" :postData="get" :actions="dataActions" :rows="rows" :updateRow="update" editMode="inline"/>
  </div>
</template>

<script>
import CommentModal from '@/components/commentModal.vue'
import Table from '@/components/Table.vue'

import axios from 'axios'
const myApi = axios.create({
    withCredentials: true, 
  })

export default {

  
  name: 'Users',
  components: {
      CommentModal,
      Table
  },
  data(){
      return {
        myApi:myApi,
        formVisible:false,
        formData: {

        },
        rows: [],
        dataActions: {
          "Изменить": {action: this.change, color: "warning"},
          "Удалить": {action: this.delete, color: "danger"},
        },
        tableFieldNames: [
          {
            name: 'author.nick',
            title: 'Автор'
          },
          {
            name: 'text',
            title: 'Текст'

          },
          {
            name: 'publication_date',
            title: 'Дата публикации'

          },
        ],
      }
  },
  methods: {
    change(elObj){this.formVisible=true; this.formData=elObj},
    get(take,page) {
      return myApi.get('http://127.0.0.1:3000/api/comments/',{
        params: {
          postId: this.$route.params.id,
          take: take ?? 10,
          page: page ?? 1}
        })
      .then((res)=>{
          console.log(res)
          this.rows=res.data;
          return res.data
      })
      .catch((error) => {
      //eventBus.emit('noresponse',error)
      alert('Сервер не отвечает')
      return false
      })
    },
    update(id, formData){
      return myApi.put('http://127.0.0.1:3000/api/admin/exchangers/',formData)
        .then((res)=>{
            this.rows = this.rows.map(el=>{if (el.id===id) return formData; else return el});
            alert("Пользователь изменен"); 
          })
        .catch((error) => { alert('Сервер не отвечает') })
    },
    delete(id){
      console.log(id)
      const result = confirm('Вы действительно хотите удалить пользователя?');
      if (result) return myApi.delete('http://127.0.0.1:3000/api/admin/exchangers/',{
        data: {id}
        })
        .then((res)=>{
            this.rows = this.rows.filter(el=>el.id !== id);
            alert("Пользователь удален"); 
          })
        .catch((error) => { alert('Сервер не отвечает') })
    },
    ban(id){
      const result = confirm('Вы действительно хотите забанить пользователя?');
      if (result) return myApi.put('http://127.0.0.1:3000/api/admin/exchangers/',{id, ban: true})
        .then((res)=>{ alert("Пользователь забанен"); 
        })
        .catch((error) => { alert('Сервер не отвечает') })
    }
  }

}
</script>

<style lang="scss">
    button{
        margin-bottom:20px;
    }
</style>