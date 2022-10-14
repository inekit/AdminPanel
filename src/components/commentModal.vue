<template>
    <CButton color="primary" @click="() => { visible = true }">Добавить пользователя</CButton>
  <CModal alignment="center" :visible="visible">
    <CModalHeader>
      <CModalTitle>Добавление комментариев</CModalTitle>
    </CModalHeader>
    <CModalBody>
    
         <CRow v-for="(item,i) in comments" :key="'f-'+item" class="mb-3">
            <CCol sm="4 ">
                <CFormInput 
                 :value="comments[i]?.nick" 
                 @change=" editNick($event, i)"
                 :disabled="!(editing===i)"
                 />
                 
            </CCol>
            <CCol sm="6">
                <CFormTextarea 
                 :value="comments[i]?.text" 
                 @change=" editText($event, i)"
                 rows="3"
                 :disabled="!(editing===i)"
            ></CFormTextarea></CCol>
            <CCol sm="2">
                <CButton type ="button" v-show="editing===i" color="primary" @click="stopEditing(i)">Сохр</CButton>
                <CButton type ="button" v-show="editing!==i" color="light" @click="startEditing(i)">Изм</CButton>
            </CCol>
        </CRow> 
                   
    <form class = "add-user"  style="display:'none'">
        <CRow class="mb-3">
            <CCol sm="4 ">
                <CFormInput v-model="formData.nick" 
                 ref="first-input"
                 placeholder="Никнейм" 
                 aria-label="Username" 
                 aria-describedby="basic-addon1"
                 @blur="validateForm"

                 />
                 
            </CCol>
            <CCol sm="8"><CFormTextarea
                id="exampleFormControlTextarea1"
                label="Example textarea"
                placeholder="Текст"
                v-model="formData.text"
                rows="3"
                @blur="validateForm"
                text="Must be 8-20 words long."
                
            ></CFormTextarea></CCol>
        </CRow>
        
    </form>    
    </CModalBody>
    <CModalFooter>
      <CButton color="primary" @click="validateForm">Добавить еще одного</CButton>
      <CButton color="primary" @click="add" >Отправить</CButton>
      <CButton color="secondary" @click="() => { visible = false }">
        Отменить
      </CButton>
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
        
    },
    data(){
        return {
            formData:{
                nick: '',
                text: '',
            },
            comments:[],
            editing: false,
            visible: false
        }
    },
    methods: {
        validateForm(){
            if (this.formData.nick && this.formData.text) {
                this.comments.push(this.formData)
                this.formData = {}
                this.$refs['first-input'].$el.focus()
            }
        },
        editNick(e, i){
            this.comments[i].nick=e.target.value
        },
        editText(e, i){
            this.comments[i].nick=e.target.value
        },
        startEditing(i){
        
            console.log(1,i);
            this.editing=i
        },
        stopEditing(i){
        
            console.log(1,i);
            this.editing=false
        },
        add(){

            if (!this.comments || this.comments.length === 0) return alert("Неверные данные")

          myApi.post(`http://127.0.0.1:3000/api/admin/comments`,{comments: this.comments, postId: this.$route.params.id,})
            .then(()=>{
                alert("Пользователи добавлены");
                this.visible = false;
            })
            .catch((error) => {
                 alert("Пользователи не добавлены")

            })

        }
    }
})
</script>
