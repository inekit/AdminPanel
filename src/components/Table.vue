<template>
   <CRow>
      <CCol :md="12">
        <CCard class="mb-4">
          <CCardHeader> Пользователи </CCardHeader>
          <CCardBody>
            <CTable align="middle" class="mb-0 border" hover responsive>
              <CTableHead color="light">
                <CTableRow>
                  <CTableHeaderCell v-for="fn in fieldNames" :key="fn+'header'"  class="text-center">{{fn}}</CTableHeaderCell>
                  <CTableHeaderCell  class="text-center">Действия</CTableHeaderCell>
                </CTableRow>
              </CTableHead>
              <CTableBody>
                <CTableRow v-for="(row,i) in transformData(rows)" :key="i+'row'" >
                    <CTableDataCell v-for="(column,j) in row" :key="j+'row'" class="text-center" >
                        <CFormInput v-if="updatingId===rows[i]?.id"  v-model="formData[fields[j]?.name]"/>
                        <span v-else>{{column}}</span>
                    </CTableDataCell>
                    <CTableDataCell>
                        <div class="d-grid gap-2 d-md-flex justify-content-md-center">
                            <CButton v-if="updatingId===rows[i]?.id" :color="'primary'" size="md" @click="editRow(i)">Сохранить</CButton>       
                            <CButton v-if="updatingId===rows[i]?.id" :color="'light'" size="md" @click="updatingId=false">Отменить</CButton>                                                 
                            <CButton v-else v-for="(info, name) in actions" :key="name+'action'" :color="info?.color" size="sm" @click="chooseAction(name,info,i, j,column)">{{name}}</CButton>
                        </div>
                    </CTableDataCell>
                </CTableRow>
                <CTableRow> </CTableRow>
              </CTableBody>
            </CTable>
          </CCardBody>
        </CCard>
      </CCol>
    </CRow>
    <CPagination aria-label="Page navigation example">
        <CPaginationItem  @click="previousPage">Назад</CPaginationItem>
        <CPaginationItem @click="toPage(1)">1</CPaginationItem>
        <CPaginationItem  @click="nextPage">Далее</CPaginationItem>
    </CPagination>

</template>

<script>
export default({
    props:{
        fields: [],
        actions: [],
        rows: [],
        postData: {
            type: Function,
            default: ()=>{}
        },
        updateRow: {
            type: Function,
            default: ()=>{}
        },
        editMode: {
            type: String,
            default: "inline"
        },
    },
    data() {
    return {
      fieldNames: [],
      perPage: 10,
      page: 1,
      updatingId: false,
      formData: {},

    };
  },
    async mounted() {
        this.fieldNames = this.fields.map(el=>{if (typeof el === 'object') return el.title ?? el.name; else return el;})

        await this.postData(this.perPage, this.page);

    },
    methods: {
        chooseAction(name, info, rowId, colId,column){
            if (!info) return;
            console.log(name==="Изменить" && this.editMode === 'inline', rowId)
            if (name==="Изменить" && this.editMode === 'inline') {
                this.updatingId = this.rows[rowId]?.id; 
                this.formData = this.rows[rowId];
            }
            else info?.action(this.rows[rowId]?.id)
        },
        nextPage(){
            this.page++;
            this.postData(this.perPage, this.page);
        },
        previousPage(){
            if (this.page>1) this.page--;
            this.postData(this.perPage, this.page);
        },
        toPage(n){
            this.page = n;
            this.postData(this.perPage, this.page);
        },
        editRow(i){
            this.updatingId=false;
            this.updateRow(this.rows[i]?.id, this.formData);
        },
        transformData(data){
            return data?.map(this.transformDataEl)

        },
        transformDataEl(pair){
                let pairFormed = [];
                for (let f of this.fields) {
                    const children = f.name.split('.') ?? f.split('.')
                    let e = pair;
                    children?.forEach(c=>e=e?.[c])

                    pairFormed.push(e?? "")
                }
                return pairFormed;
        },
    },
})
</script>

<style lang="scss">
.btn-md{
    margin:0;
}
</style>