<template>
  <DialogForm ref="dialogForm">
    <template v-slot:header>
      <h4 class="modal-title" v-if="insertMode">{{ labels.title_new_group }}</h4>
      <h4 class="modal-title" v-if="updateMode">{{ labels.title_edit_group }}</h4>
    </template>
    <template v-slot:default>
        <div class="row row-height">
          <div class="col-md-3 col-height col-label">
            <label for="groupnamedialog">{{ labels.groupname_label }}</label>
          </div>
          <div class="col-height col-md-4">
            <div class="input-group has-validation" :class="{'has-error': v$.groupname.$error}">
              <InputMask ref="groupname" v-model="localData.groupname" id="groupnamedialog" picture="(20)X" :disabled="disabledKeyField" /> 
              <label class="required">*</label>
            </div>
            <span v-if="v$.groupname.$error" class="has-error">{{ v$.groupname.$errors[0].$message }}</span>
          </div>
        </div>
        <div class="row row-height">
          <div class="col-md-3 col-height col-label">
            <label for="nameendialog">{{ labels.nameen_label }}</label>
          </div>
          <div class="col-height col-md-8">
            <div class="input-group has-validation" :class="{'has-error': v$.nameen.$error}">
              <input ref="nameen" type="text" v-model="localData.nameen" id="nameendialog" class="form-control input-md" maxlength="100" />
              <label class="required">*</label>
            </div>
            <span v-if="v$.nameen.$error" class="has-error">{{ v$.nameen.$errors[0].$message }}</span>
          </div>
        </div>
        <div class="row row-height">
          <div class="col-md-3 col-height col-label">
            <label for="namethdialog">{{ labels.nameth_label }}</label>
          </div>
          <div class="col-height col-md-8">
            <div class="input-group has-validation" :class="{'has-error': v$.nameth.$error}">
              <input ref="nameth" type="text" v-model="localData.nameth" id="namethdialog" class="form-control input-md" maxlength="100" />
              <label class="required">*</label>
            </div>
            <span v-if="v$.nameth.$error" class="has-error">{{ v$.nameth.$errors[0].$message }}</span>
          </div>
        </div>
        <div class="row row-height">
          <div class="col-height col-md-3 col-label">
            <label for="usertypedialog">{{ labels.usertype_label }}</label>
          </div>
          <div class="col-height col-md-5">
            <fieldset id="usertypedialogfieldset" :disabled="disabledKeyField"><legend class="fa-hidden"></legend>
              <div class="input-group has-validation" :class="{'has-error': v$.usertype.$error}">
                <select ref="usertype" v-model="localData.usertype" class="form-control input-md" id="usertypedialog">
                  <option v-for="item in dataCategory.tkusertype" :key="item.id" :value="item.id">{{item.text}}</option>
                </select>
              <label class="required">*</label>
            </div>
            </fieldset>
            <span v-if="v$.usertype.$error" class="has-error">{{ v$.usertype.$errors[0].$message }}</span>
          </div>
        </div>
        <div class="row row-height">
          <div class="col-height col-md-3 col-label">
            <label for="mobilegroupdialog">{{ labels.mobilegroup_label }}</label>
          </div>
          <div class="col-height col-md-5">
              <select ref="mobilegroup" v-model="localData.mobilegroup" class="form-control input-md" id="mobilegroupdialog">
                <option v-for="item in dataCategory.tkgroupmobile" :key="item.id" :value="item.id">{{item.text}}</option>
              </select>
          </div>
        </div>
        <div class="row row-height" id="privateflaglayer">
          <div class="col-height col-md-3 col-label">
          </div>
          <div class="col-height col-md-8">
            <div class="form-check">
              <input ref="privateflag" type="checkbox" id="privateflagdialog" :true-value="1" :false-value="0" v-model="localData.privateflag" class="form-control input-md form-check-input"/>
              <label for="privateflagdialog" class="form-check-label">{{ labels.privateflag_label }}</label>
            </div>
          </div>
        </div>
        <div class="row row-height">
          <div class="col-height col-md-3 col-label">
            <label>{{ labels.iconstyle_label }}</label>
          </div>
          <div class="col-height col-md-3">
            <div id="iconstyleswitcher"></div>
            <input type="hidden" id="iconstyledialog" v-model="localData.iconstyle" />
          </div>
        </div>
        <div class="row row-height">
          <div class="col-height col-md-3 col-label">
            <label for="seqnodialog">{{ labels.sequenceno_label }}</label>
          </div>
          <div class="col-height col-md-3">
            <div class="input-group has-validation" :class="{'has-error': v$.seqno.$error}">
              <InputNumber ref="seqno" v-model="localData.seqno" id="seqnodialog" /> 
              <label class="required">*</label>
            </div>
            <span v-if="v$.seqno.$error" class="has-error">{{ v$.seqno.$errors[0].$message }}</span>
          </div>
        </div>
    </template>
    <template v-slot:footer>
      <button ref="savebutton" id="savebutton" class="btn btn-dark btn-sm" @click="saveClick" v-if="insertMode"><em class="fa fa-save fa-btn-icon"></em>{{ labels.save_button }}</button>
      <button ref="updatebutton" id="updatebutton" class="btn btn-dark btn-sm" @click="updateClick" v-if="updateMode"><em class="fa fa-save fa-btn-icon"></em>{{ labels.update_button }}</button>
      <button id="canceldialogbutton" class="btn btn-dark btn-sm" data-dismiss="modal"><em class="fa fa-close fa-btn-icon"></em>{{ labels.cancel_button }}</button>
    </template>
  </DialogForm>
</template>
<style>
#usertypedialogfieldset { width: 100%; }
#privateflaglayer { margin-bottom: 10px; }
</style>
<script>
import { ref, computed, watch } from 'vue';
import { useVuelidate } from '@vuelidate/core';
import { required, helpers } from '@vuelidate/validators';
import $ from "jquery";
import { DEFAULT_CONTENT_TYPE, getApiUrl, disableControls }  from '@willsofts/will-app';
import { startWaiting, stopWaiting, submitFailure, detectErrorResponse }  from '@willsofts/will-app';
import { confirmUpdate, confirmSave, confirmDelete, successbox, serializeParameters } from '@willsofts/will-app';
import { InputNumber, InputMask } from '@willsofts/will-control';
import DialogForm from './DialogForm.vue';

const APP_URL = "/api/sfte002";
const defaultData = {
  groupname: "",
  nameen: "",
  nameth: "",
  usertype: "",
  mobilegroup: "",
  privateflag: "0",
  iconstyle: "fa fa-desktop",
  seqno: 0,
};

export default {
  components: {
    DialogForm, InputNumber, InputMask
  },
  props: {
    modes: Object,
    labels: Object,
    dataCategory: Object
  },
  emits: ["data-saved","data-updated","data-deleted"],
  setup(props) {
    const mode = ref({action: "new", ...props.modes});
    const localData = ref({ ...defaultData }); 
    const disabledKeyField = ref(false);
    const reqalert = ref(props.labels.empty_alert);
    const requiredMessage = () => {
      return helpers.withMessage(reqalert, required);
    }
    const validateRules = computed(() => { 
      return {
        groupname: { required: requiredMessage() },
        nameen: { required: requiredMessage() },
        nameth: { required: requiredMessage() },
        usertype: { required: requiredMessage() },
        seqno: { required: requiredMessage() },
      } 
    });
    const v$ = useVuelidate(validateRules, localData, { $lazy: true, $autoDirty: true });
    return { mode, v$, localData, disabledKeyField, reqalert };
  },
  created() {
    watch(this.$props, (newProps) => {      
      this.reqalert = newProps.labels.empty_alert;
    });
  },
  computed: {
    insertMode() {
      return this.mode.action=="insert" || this.mode.action=="new";
    },
    updateMode() {
      return this.mode.action=="update" || this.mode.action=="edit";
    },
  },
  mounted() {
    this.$nextTick(function () {
      $("#modaldialog_layer").find(".modal-dialog").draggable();
      $("#iconstyleswitcher").styleswitcher({$styleInput: $("#iconstyledialog"), width: 200, styleURL : getApiUrl()+"/api/style/category" });	
    });
  },
  methods: {
    reset(newData,newMode) {
      if(newData) this.localData = {...newData};
      if(newMode) this.mode = {...newMode};
    },
    submit() {      
      this.$emit('update:formData', this.localData);
    },
    async saveClick() {
      console.log("click: save");
      disableControls($("#savebutton"));
      let valid = await this.validateForm();
      if(!valid) return;
      this.startSaveRecord();
    },
    async updateClick() {
      console.log("click: update");
      disableControls($("#updatebutton"));
      let valid = await this.validateForm();
      if(!valid) return;
      this.startUpdateRecord();
    },
    async validateForm(focusing=true) {
      const valid = await this.v$.$validate();
      console.log("validate form: valid",valid);
      console.log("error:",this.v$.$errors);
      if(!valid) {
        if(focusing) {
          this.focusFirstError();
        }
        return false;
      }
      return true;
    },
    focusFirstError() {
      if(this.v$.$errors && this.v$.$errors.length > 0) {
        let input = this.v$.$errors[0].$property;
        let el = this.$refs[input];
        if(el) el.focus(); //if using ref
        else $("#"+input+"dialog").trigger("focus"); //if using id
      }
    },
    showDialog(callback) {
      //$("#modaldialog_layer").modal("show");
      if(callback) $(this.$refs.dialogForm.$el).on("shown.bs.modal",callback);
      $(this.$refs.dialogForm.$el).modal("show");
    },  
    hideDialog() {
      //$("#modaldialog_layer").modal("hide");
      $(this.$refs.dialogForm.$el).modal("hide");
    },
    resetRecord() {
      //reset to default data 
      this.reset(defaultData,{action:"insert"});
      //reset validator
      this.v$.$reset();
      //enable key field
      this.disabledKeyField = false;
    },
    startInsertRecord() {
      this.resetRecord();
      this.showDialog(() => { this.$refs.groupname.focus(); });
      $("#iconstyleswitcher").styleupdate();
    },
    startSaveRecord() {
      confirmSave(() => {
        this.saveRecord(this.localData);
      });
    },
    startUpdateRecord() {
      confirmUpdate(() => { 
        this.updateRecord(this.localData);
      });
    },
    startDeleteRecord(dataKeys) {
      confirmDelete(Object.values(dataKeys),() => {
        this.deleteRecord(dataKeys);
      });
    },
    saveRecord(dataRecord) {
        let jsondata = {ajax: true};
        let formdata = serializeParameters(jsondata,dataRecord);
        startWaiting();
        $.ajax({
          url: getApiUrl()+APP_URL+"/insert",
          data: formdata.jsondata,
          headers : formdata.headers,
          type: "POST",
          dataType: "json",
          contentType: DEFAULT_CONTENT_TYPE,
          error : function(transport,status,errorThrown) {
            console.error("error: status",status,"errorThrown",errorThrown);
            submitFailure(transport,status,errorThrown);
          },
          success: (data) => {
            stopWaiting();
            console.log("success",data);
            if(detectErrorResponse(data)) return;
            successbox(() => {
              //reset data for new record insert
              this.resetRecord();
              setTimeout(() => { this.$refs.groupname.focus(); },100);
              this.$emit('data-saved',dataRecord,data);
            });
          }
      });
    },
    updateRecord(dataRecord) {
        let jsondata = {ajax: true};
        let formdata = serializeParameters(jsondata,dataRecord);
        startWaiting();
        $.ajax({
          url: getApiUrl()+APP_URL+"/update",
          data: formdata.jsondata,
          headers : formdata.headers,
          type: "POST",
          dataType: "json",
          contentType: DEFAULT_CONTENT_TYPE,
          error : function(transport,status,errorThrown) {
            console.error("error: status",status,"errorThrown",errorThrown);
            submitFailure(transport,status,errorThrown);
          },
          success: (data) => {
            stopWaiting();
            console.log("success",data);
            if(detectErrorResponse(data)) return;
            successbox(() => {
              this.hideDialog();
              this.$emit('data-updated',dataRecord,data);
            });
          }
      });
    },
    retrieveRecord(dataKeys) {
      let jsondata = {ajax: true};
      let formdata = serializeParameters(jsondata,dataKeys);
      startWaiting();
      $.ajax({
        url: getApiUrl()+APP_URL+"/retrieve",
        data: formdata.jsondata,
        headers : formdata.headers,
        type: "POST",
        dataType: "json",
        contentType: DEFAULT_CONTENT_TYPE,
        error : function(transport,status,errorThrown) {
          console.error("retrieveRecord: error: status",status,"errorThrown",errorThrown);
          submitFailure(transport,status,errorThrown);
        },
        success: (data) => {
          stopWaiting();
          console.log("retrieveRecord: success",data);
          if(data.body.dataset) {
            this.reset(data.body.dataset,{action:"edit"});
            this.v$.$reset();
            this.disabledKeyField = true;
            this.showDialog(() => { this.$refs.nameen.focus(); });
            $("#iconstyleswitcher").styleupdate(this.localData.iconstyle);
          }
        }
      });	
    },
    deleteRecord(dataKeys) {
      let jsondata = {ajax: true};
      let formdata = serializeParameters(jsondata,dataKeys);
      startWaiting();
      $.ajax({
        url: getApiUrl()+APP_URL+"/remove",
        data: formdata.jsondata,
        headers : formdata.headers,
        type: "POST",
        dataType: "json",
        contentType: DEFAULT_CONTENT_TYPE,
        error : function(transport,status,errorThrown) {
          console.error("deleteRecord: error: status",status,"errorThrown",errorThrown);
          submitFailure(transport,status,errorThrown);
        },
        success: (data) => {
          stopWaiting();
          console.log("deleteRecord: success",data);
          if(detectErrorResponse(data)) return;
          successbox(() => {
            this.$emit('data-deleted',dataKeys,data);
          });
        }
      });	
    },
  }
};
</script>
