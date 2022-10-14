<template>
    <CButton color="primary" @click="() => { visible = true }">Добавить новость</CButton>
  <CModal alignment="center" :visible="visible" @close="() => { visible = false }">
    <CModalHeader>
      <CModalTitle>Добавить новость</CModalTitle>
    </CModalHeader>
    <CModalBody>
    <form ref="add-file-form" class = "add-user"  style="display:'none'">
            <CFormInput class="mb-3" v-model="formData.title" placeholder="Никнейм"/>
            <CFormTextarea  v-model="formData.text" placeholder="Сооющение" />
            <CFormInput type="file" ref="file" v-on:change="handleFileUpload" class="mb-3" placeholder="Превью"/>
        </form>    
    </CModalBody>
    <CModalFooter>
      <CButton color="secondary" @click="() => { visible = false }">
        Отменить
      </CButton>
      <CButton color="primary" @click="addNewing" >Добавить новость</CButton>
    </CModalFooter>
  </CModal>
</template>

<script>
import axios from 'axios'
const myApi = axios.create({
    withCredentials: true, 
  })

export default({
    props:{
        visible: false,
        formData:{
            title: '',
            text: '',
            preview: '',
        }
        
    },
    methods: {
      handleFileUpload(event){
        this.formData.preview = event.target.files[0]
      },
        addNewing(){
          
          if (!this.formData.title || !this.formData.text) 
            return alert("Неверные данные")
          
          var formData = new FormData();

          formData.append("title", this.formData.title);
          formData.append("text", this.formData.text);
          formData.append("image", this.formData.preview);

          myApi.post(`http://127.0.0.1:3000/api/admin/posts`,formData,{headers: {
                'Content-Type': 'multipart/form-data'
              }})
            .then(()=>{
                alert("Пользователь добавлен")
            })
            .catch((error) => {
                alert("Ошибка добавления")
            })

        }
    }
})
</script>
