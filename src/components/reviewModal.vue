<template>
    <CButton color="primary" @click="() => { visible = true }">Добавить ревью</CButton>
  <CModal alignment="center" scrollable size="lg" :visible="visible">
    <CModalHeader>
      <CModalTitle>Добавление ревью</CModalTitle>
    </CModalHeader>
    <CModalBody>
         <CLabel v-if="comments.length">Добавленные ревью</CLabel>

         <CRow v-for="(item,i) in comments" :key="'f-'+item" class="mb-3">
            <CCol sm="3">
                <CFormInput 
                 :value="comments[i]?.nick" 
                 @change=" editNick($event, i)"
                 :disabled="!(editing===i)"
                 />
                 
            </CCol>
            <CCol sm="2">
                <CFormSelect 
                 :value="comments[i]?.rate" 
                 @change=" editNick($event, i)"
                 :disabled="!(editing===i)"
                 :options="rateOptions"
                 />
            </CCol>
            <CCol sm="5">
                <CFormTextarea 
                 :value="comments[i]?.text" 
                 @change=" editText($event, i)"
                 rows="1"
                 :disabled="!(editing===i)"
                 maxlength="1000"

            ></CFormTextarea></CCol>
            <CCol sm="2">
                <CButton type ="button" class="w-100" v-show="editing===i" color="primary" @click="stopEditing(i)">Сохр</CButton>
                <CButton type ="button" class="w-100" v-show="editing!==i" color="light" @click="startEditing(i)">Изм</CButton>
            </CCol>
        </CRow> 
    <CLabel>Новое ревью</CLabel>               
    <form class = "add-user"  style="display:'none'">
        <CRow class="mb-3">
            <CCol sm="3">
                <CFormInput v-model="formData.nick" 
                 ref="first-input"
                 placeholder="Никнейм" 
                 aria-label="Username" 
                 aria-describedby="basic-addon1"
                 @blur="validateForm"

                 />
                 
            </CCol>
            <CCol sm="2">
                <CFormSelect 
                 v-model="formData.rate"
                 :options="rateOptions"
                 @blur="validateForm"
                 />
                 
            </CCol>
            <CCol sm="7"><CFormTextarea
                id="exampleFormControlTextarea1"
                label="Example textarea"
                placeholder="Текст"
                v-model="formData.text"
                rows="3"
                @blur="validateForm"
                maxlength="1000"
                
            ></CFormTextarea></CCol>
        </CRow>
        
    </form>    
    </CModalBody>
    <CModalFooter>
      <CButton color="primary" @click="validateForm">Добавить еще ревью</CButton>
      <CButton color="primary" @click="addFunc(comments)" >Отправить</CButton>
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
        addFunc: ()=>{}
    },
    data(){
        return {
            formData:{
                nick: '',
                text: '',
                rate: '5',
            },
            comments:[],
            editing: false,
            visible: false,
            rateOptions: [{ label: '*****', value: '5' },{ label: '****', value: '4' },{ label: '***', value: '3' },{ label: '**', value: '2' },{ label: '*', value: '1' },]
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
        
    }
})
</script>
