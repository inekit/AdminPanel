<template>
  <div>
   <AddUserModal :visible="formVisible" :formData="formData"/>
    <Table :fields="tableFieldNames" :postData="getUsers" :actions="dataActions" :rows="rows"/>
  </div>
</template>

<script>
import AddUserModal from '@/components/AddUserModal.vue'
import Table from '@/components/Table.vue'

import axios from 'axios'
const myApi = axios.create({
    withCredentials: true, 
  })

export default {

  
  name: 'Users',
  components: {
      AddUserModal,
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
          "Бан": {action:  this.banUser, color: "light"},
          "Изменить": {action: this.changeUser, color: "warning"},
          "Удалить": {action: this.deleteUser, color: "danger"},
        },
        tableFieldNames: [
          {
            name: 'nick',
            title: 'Никнейм'
          },
          {
            name: 'email',
            title: 'Почта'

          },
        ],
      }
  },
  methods: {
    changeUser(userObj){this.formVisible=true; this.formData=userObj},
    getUsers (perPage,page) {
      return myApi.get('http://127.0.0.1:3000/api/admin/users/',{
        perPage: perPage ?? 0,
        page: page ?? 0
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
    deleteUser (id){
      console.log(id)
      const result = confirm('Вы действительно хотите удалить пользователя?');
      if (result) return myApi.delete('http://127.0.0.1:3000/api/admin/users/',{
        data: {id}
        })
        .then((res)=>{
            this.rows = this.rows.filter(el=>el.id !== id);
            alert("Пользователь удален"); 
          })
        .catch((error) => { alert('Сервер не отвечает') })
    },
    banUser (id){
      const result = confirm('Вы действительно хотите забанить пользователя?');
      if (result) return myApi.put('http://127.0.0.1:3000/api/admin/users/',{id, ban: true})
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