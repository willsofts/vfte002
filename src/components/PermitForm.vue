<template>
  <DialogForm ref="permitDialog">
    <template v-slot:header>
        <h4 class="modal-title" id="modalheader">{{ labels.permit_dialog_title }}</h4>
    </template>
    <template v-slot:default>
        <div id="programinfolayer" class="row center-block">
            <div class="col-md-10">
                <label id="program_info" class="control-label">{{ localData.progname }}</label>
            </div>
        </div>
        <div id="permitinfolayer">
            <div v-for="(permit, index) in permitLists()" :key="index" class="row row-heighter">
                <div v-for="item in permit" :key="item.id" class="col-height col-md-4">
                    <div class="checkbox form-check">
                        <input type="checkbox" :id="item.id" :true-value="true" :false-value="false" v-model="localData.permits[item.id]" class="form-control input-md form-check-input"/>
                        <label :for="item.id" class="form-check-label">{{item.text}}</label>
                    </div>
                </div>
            </div>
        </div>
        <div class="row row-heighter">
            <div class="col-height col-md-3 col-label">
                <label>{{ labels.parameters_label }}</label>
            </div>
            <div class="col-height col-md-5">
                <input type="text" v-model="localData.parameters" class="form-control input-md" maxlength="100" />
            </div>                
        </div>
    </template>
    <template v-slot:footer>
        <button class="btn btn-dark btn-sm" @click="saveClick" v-if="insertMode"><em class="fa fa-save fa-btn-icon"></em>{{ labels.save_button }}</button>
        <button class="btn btn-dark btn-sm" @click="updateClick" v-if="updateMode"><em class="fa fa-save fa-btn-icon"></em>{{ labels.update_button }}</button>
        <button class="btn btn-dark btn-sm" data-dismiss="modal"><em class="fa fa-close fa-btn-icon"></em>{{ labels.cancel_button }}</button>
    </template>
</DialogForm>
</template>
<style>
#permitinfolayer { margin-left: 10px; margin-bottom: 10px; }
div.checkbox { padding-top: 5px; padding-bottom: 5px; }
</style>
<script>
import { ref } from 'vue';
import $ from "jquery";
import { DEFAULT_CONTENT_TYPE, getApiUrl }  from '@willsofts/will-app';
import { startWaiting, stopWaiting, submitFailure, detectErrorResponse }  from '@willsofts/will-app';
import { confirmUpdate, confirmSave, confirmDelete, serializeParameters } from '@willsofts/will-app';
import DialogForm from './DialogForm.vue';

const APP_URL = "/api/sfte002";
const defaultData = {
    groupname: "",
    programid: "",
    progname: "",
    parameters: "",
    seqno: 0,
    permname: [],
    permvalue: [],
    permits: {}
};

export default {
    components: {
        DialogForm
    },
    props: {
        modes: Object,
        labels: Object,
        dataCategory: Object,
    },
    emits: ["data-saved","data-updated","data-deleted"],
    setup(props) {
        const complete = ref({saveCallback: undefined, updateCallback: undefined });
        const mode = ref({action: "new", ...props.modes});
        const localData = ref({ ...defaultData }); 
        return { mode, localData, complete };
    },
    computed: {
        insertMode() {
            return this.mode.action=="insert" || this.mode.action=="new";
        },
        updateMode() {
            return this.mode.action=="update" || this.mode.action=="edit";
        },
    },
    methods: {
        permitLists() {
            let results = [];
            let chunkSize = 3;
            let tkpermit = this.$props.dataCategory.tkpermit;
            for (let i = 0; i < tkpermit.length; i += chunkSize) {
                results.push(tkpermit.slice(i, i + chunkSize));
            }
            return results;
        },
        reset(newData,newMode) {
            if(newData) {
                let permits = {};
                this.$props.dataCategory.tkpermit.forEach(item => { permits[item.id] = item.text });
                this.localData = {permits: permits, ...newData};
            }
            if(newMode) this.mode = {...newMode};
        },
        async saveClick() {
            console.log("click: save");
            let valid = await this.validateForm();
            if(!valid) return;
            this.startSaveRecord();
        },
        async updateClick() {
            console.log("click: update");
            let valid = await this.validateForm();
            if(!valid) return;
            this.startUpdateRecord();
        },
        async validateForm() {
            return true;
        },
        showDialog() {
            $(this.$refs.permitDialog.$el).modal("show");
        },  
        hideDialog() {
            $(this.$refs.permitDialog.$el).modal("hide");
        },
        scrapeParameters() {
            let permnames = [];
            let permvalues = [];
            for(let p in this.localData.permits) {
                let v = this.localData.permits[p];
                permnames.push(p);
                permvalues.push(String(v));
            }
            this.localData.permname = permnames;
            this.localData.permvalue = permvalues;
        },
        startInsertRecord(data,callback) {
            this.complete.saveCallback = callback;
            this.reset(data,{action:"insert"});
            this.showDialog();
        },
        startSaveRecord() {
            this.scrapeParameters();
            confirmSave(() => {
                this.saveRecord(this.localData);
            });
        },
        startUpdateRecord() {
            this.scrapeParameters();
            confirmUpdate(() => { 
                this.updateRecord(this.localData);
            });
        },
        startDeleteRecord(dataKeys) {
            confirmDelete(Object.values(dataKeys),() => {
                this.deleteRecord(dataKeys);
            });
        },
        saveRecord(dataRecord,callback) {
            let jsondata = {ajax: true};
            let formdata = serializeParameters(jsondata,dataRecord);
            startWaiting();
            $.ajax({
                url: getApiUrl()+APP_URL+"/proginsert",
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
                    this.hideDialog();
                    if(callback) callback(dataRecord,data);
                    this.$emit('data-saved',dataRecord,data);
                    if(this.complete.saveCallback) this.complete.saveCallback(data,dataRecord);
                }
            });
        },
        updateRecord(dataRecord,callback) {
            let jsondata = {ajax: true};
            let formdata = serializeParameters(jsondata,dataRecord);
            startWaiting();
            $.ajax({
                url: getApiUrl()+APP_URL+"/progupdate",
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
                    this.hideDialog();
                    if(callback) callback(data,dataRecord);
                    this.$emit('data-updated',dataRecord,data);
                    if(this.complete.updateCallback) this.complete.updateCallback(data,dataRecord);
                }
            });
        },
        deleteRecord(dataKeys,callback) {
            let jsondata = {ajax: true};
            let formdata = serializeParameters(jsondata,dataKeys);
            startWaiting();
            $.ajax({
                url: getApiUrl()+APP_URL+"/progremove",
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
                    if(callback) callback(dataKeys,data);
                    this.$emit('data-deleted',dataKeys,data);
                }
            });	
        },
        retrieveRecord(dataKeys,callback) {
            this.complete.updateCallback = callback;
            let jsondata = {ajax: true};
            let formdata = serializeParameters(jsondata,dataKeys);
            startWaiting();
            $.ajax({
                url: getApiUrl()+APP_URL+"/progpermit",
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
                        data.body.dataset.progname = data.body.dataset.programid + " - " + data.body.dataset.prognameen;
                        this.reset(data.body.dataset,{action:"edit"});
                        this.showDialog();
                    }
                }
            });	
        },
        removeProgramInGroup(pid,gid,callback) {
            let params = { programid : pid, groupname : gid };
            this.deleteRecord(params,callback);
        },
        editProgramInGroup(pid,gid,callback) {
            let params = { progid: pid, groupid: gid };
            this.retrieveRecord(params,callback);
        },
    },
};
</script>        
