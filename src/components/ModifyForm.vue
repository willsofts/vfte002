<template>
    <div id="entrylayer" class="entry-layer">
      <div class="portal-area sub-entry-layer">
        <div class="row row-height">
            <div class="col-md-3 col-height col-label">
                <label>{{ labels.groupname_label }}</label>
            </div>
            <div class="col-height col-md-3">
                <input type="text" ref="groupname" v-model="localData.groupname" class="form-control input-md" disabled /> 
            </div>
        </div>
        <div class="row row-height">
            <div class="col-md-3 col-height col-label">
                <label for="nameen">{{ labels.nameen_label }}</label>
            </div>
            <div class="col-height col-md-6">
                <div class="input-group has-validation" :class="{'has-error': v$.nameen.$error}">
                <input ref="nameen" type="text" v-model="localData.nameen" id="nameen" class="form-control input-md" maxlength="100" />
                <label class="required">*</label>
                </div>
                <span v-if="v$.nameen.$error" class="has-error">{{ v$.nameen.$errors[0].$message }}</span>
            </div>
        </div>
        <div class="row row-height">
            <div class="col-md-3 col-height col-label">
                <label for="nameth">{{ labels.nameth_label }}</label>
            </div>
            <div class="col-height col-md-6">
                <div class="input-group has-validation" :class="{'has-error': v$.nameth.$error}">
                <input ref="nameth" type="text" v-model="localData.nameth" id="nameth" class="form-control input-md" maxlength="100" />
                <label class="required">*</label>
                </div>
                <span v-if="v$.nameth.$error" class="has-error">{{ v$.nameth.$errors[0].$message }}</span>
            </div>
        </div>
        <div class="row row-height">
            <div class="col-height col-md-3 col-label">
                <label for="usertype">{{ labels.usertype_label }}</label>
            </div>
            <div class="col-height col-md-3">
                <fieldset id="usertypefieldset" disabled><legend class="fa-hidden"></legend>
                    <select ref="usertype" v-model="localData.usertype" class="form-control input-md" id="usertype">
                    <option v-for="item in dataCategory.tkusertype" :key="item.id" :value="item.id">{{item.text}}</option>
                    </select>
                </fieldset>
            </div>
        </div>
        <div class="row row-height">
            <div class="col-height col-md-3 col-label">
                <label for="mobilegroup">{{ labels.mobilegroup_label }}</label>
            </div>
            <div class="col-height col-md-3">
                <select ref="mobilegroup" v-model="localData.mobilegroup" class="form-control input-md" id="mobilegroup">
                    <option v-for="item in dataCategory.tkgroupmobile" :key="item.id" :value="item.id">{{item.text}}</option>
                </select>
            </div>
        </div>
        <div class="row row-height" id="privateflaglayer">
            <div class="col-height col-md-3 col-label">
            </div>
            <div class="col-height col-md-8">
                <div class="form-check">
                    <input ref="privateflag" type="checkbox" id="privateflag" :true-value="1" :false-value="0" v-model="localData.privateflag" class="form-control input-md form-check-input"/>
                    <label for="privateflag" class="form-check-label">{{ labels.privateflag_label }}</label>
                </div>
            </div>
        </div>
        <div class="row row-height">
            <div class="col-height col-md-3 col-label">
                <label>{{ labels.iconstyle_label }}</label>
            </div>
            <div class="col-height col-md-3">
                <div id="iconstylelayer">
                  <button class="btn btn-light icon-style-btn"><em :class="localData.iconstyle" aria-hidden='true'></em></button>
                </div>
            </div>
        </div>
        <div class="row row-height">
            <div class="col-height col-md-3 col-label">
                <label for="seqnodialog">{{ labels.sequenceno_label }}</label>
            </div>
            <div class="col-height col-md-2">
                <div class="input-group has-validation" :class="{'has-error': v$.seqno.$error}">
                    <InputNumber ref="seqno" v-model="localData.seqno" id="seqnodialog" /> 
                    <label class="required">*</label>
                </div>
                <span v-if="v$.seqno.$error" class="has-error">{{ v$.seqno.$errors[0].$message }}</span>
            </div>
        </div>
        <div class="row row-height">
            <div class="col-md-12 pull-right text-right btn-control-layer">
                <button id="updatemodifybutton" class="btn btn-dark btn-sm" @click="updateClick"><em class="fa fa-save fa-btn-icon"></em>{{ labels.update_button }}</button>
                <button class="btn btn-dark btn-sm" @click="cancelClick"><em class="fa fa-close fa-btn-icon"></em>{{ labels.cancel_button }}</button>
            </div>
        </div>
      </div>

			<div id="programcontrollayer" class="row">
			<div id="proglayerheader" class="pt-page-header pt-page-corser"><label id="progtrans_label">{{ labels.progtrans_label }}</label><a href="javascript:void(0)" class="pull-right up" @click="toggleCollapseExpand($event)"><em class="fa fa-chevron-circle-up fa-toggle-collapse"></em></a></div>
				<div id="proglayer" class="portal-area portal-area-layer">
					<div id="programrowlayer" class="row">
						<div class="col-md-3 col-height col-label text-right">
							<label id="prog_progingroup_label" class="control-label">{{ labels.prog_progingroup_label }}</label>
						</div>
						<div class="col-md-3 col-height">
              <select ref="progingroup" v-model="localData.progingroup" class="form-control input-md" id="progingroup">
                <option v-for="item in dataCategory.tprog" :key="item.id" :value="item.id">{{item.id}} - {{item.text}}</option>
              </select>
						</div>
						<div class="col-md-1 col-height">
							<button id="addprogbutton" @click="addProgClick" class="btn btn-dark btn-sm" title="Add Program"><em class="fa fa-plus" aria-hidden="true"></em></button>
						</div>
					</div>
					<table id="progtable" class="table table-bordered table-hover table-striped tablesorter">
						<thead id="progtableheader">
							<tr>
								<th width="10%" class="text-center th-sequence"><label id="prog_seqno_headerlabel">{{ labels.prog_seqno_headerlabel }}</label></th>
								<th width="30%" class="text-center th-data"><label id="prog_progid_headerlabel">{{ labels.prog_progid_headerlabel }}</label></th>
								<th width="50%" class="text-center th-data"><label id="prog_progname_headerlabel">{{ labels.prog_progname_headerlabel }}</label></th>
								<th width="10%" class="text-center th-action"><em class="fa fa-bolt" aria-hidden="true"></em></th>
							</tr>
						</thead>
						<tbody id="aprogtablebody">
						</tbody>
					</table>
					<div id="progtablelayer">
						<ul id="progtablebody" class="ul-table-listing"></ul>
					</div>					
				</div>
			</div>

    </div>
  </template>
  <style>
  #usertypefieldset { width: 100%; }
  #privateflaglayer { margin-bottom: 10px; }
  #updatemodifybutton { margin-right: 15px; }
  button.icon-style-btn { width:55px; }
  </style>
  <script>
  import { ref, computed, watch } from 'vue';
  import { useVuelidate } from '@vuelidate/core';
  import { required, helpers } from '@vuelidate/validators';
  import $ from "jquery";
  import { DEFAULT_CONTENT_TYPE, getApiUrl }  from '@willsofts/will-app';
  import { startWaiting, stopWaiting, submitFailure, detectErrorResponse }  from '@willsofts/will-app';
  import { confirmUpdate, confirmRemove, confirmCancel, successbox, serializeParameters, alertbox } from '@willsofts/will-app';
  import { InputNumber } from '@willsofts/will-control';
  
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
    progid: [],
  };
  
  export default {
    components: {
      InputNumber
    },
    props: {
      labels: Object,
      dataCategory: Object
    },
    emits: ["data-updated","data-cancel"],
    setup(props) {
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
          seqno: { required: requiredMessage() },
        } 
      });
      const v$ = useVuelidate(validateRules, localData, { $lazy: true, $autoDirty: true });
      return { v$, localData, disabledKeyField, reqalert };
    },
    created() {
      watch(this.$props, (newProps) => {      
        this.reqalert = newProps.labels.empty_alert;
      });
    },
    mounted() {
      this.$nextTick(function () {
        $("#modaldialog_layer").find(".modal-dialog").draggable();
        $("#progtablebody").sortable({
          stop: () => {
            this.displaySequenceTableLists("#progtablebody");
          }
        });	
      });
    },
    methods: {
      reset(newData) {
        if(newData) this.localData = {...newData};
      },
      submit() {      
        this.$emit('update:formData', this.localData);
      },
      clearingFields() {
        $("#progtablebody").empty();
      },
      async cancelClick() {
        console.log("click: cancel");
        confirmCancel(() => {
          this.clearingFields();
          this.$emit('data-cancel',{action:"cancel"});
        });
      },
      async updateClick() {
        console.log("click: update");
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
      resetRecord() {
        this.reset(defaultData);
        this.v$.$reset();
      },
      startUpdateRecord() {
        confirmUpdate(() => {
          let progids = [];
          $("input.progidclass",$("#progtablebody")).each((index,element) => {
            progids.push($(element).val());
          });
          this.localData.progid = progids;
          this.updateRecord(this.localData);
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
              successbox();
              this.$emit('data-updated',dataRecord,data);
            }
        });
      },
      retrieveRecord(dataKeys,callback) {
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
              this.reset(data.body.dataset);
              this.v$.$reset();
              this.clearingFields();
              this.readProgramInfo({groupname: data.body.dataset.groupname});
              if(callback) callback();
            }
          }
        });	
      },
      readProgramInfo(dataKeys) {
        let jsondata = {ajax: true};
        let formdata = serializeParameters(jsondata,dataKeys);
        startWaiting();
        $.ajax({
          url: getApiUrl()+APP_URL+"/proglist",
          data: formdata.jsondata,
          headers: formdata.headers,
          type: "POST",
          dataType: "json",
          contentType: DEFAULT_CONTENT_TYPE,
          error : function(transport,status,errorThrown) { 
            console.error("retrieveRecord: error: status",status,"errorThrown",errorThrown);
            submitFailure(transport,status,errorThrown);
          },
          success: (data) => { 
            stopWaiting();
            if(data.body["rows"]) {              
              $(data.body.rows).each((index,element) => { 
                this.displayProgramInfo(element["programid"],element["progname"],element["seqno"]);
              });
              this.displaySequenceTableLists("#progtablebody");
              this.adjustProgTableHeight();
            }
          }
        });	
      },
      displaySequenceTableLists(tablelists) {
        $("li",$(tablelists)).each(function(index) { 
          let $table = $(this).find("table").eq(0);
          $table.find("tr").eq(0).find("td").eq(0).html(""+(index+1));
        });
      },
      adjustProgTableHeight() {
        let totalHeight = 0;
        $("#progtablebody li").each(function() {
          totalHeight += $(this).outerHeight(); 
        });
        $("#progtablelayer").height(totalHeight+5);	
      },
      toggleCollapseExpand(event) {
        let $src = $(event.target).parent();
        if($src.is(".up")) {
          $src.removeClass("up").addClass("down");
          $("#proglayer").hide();
          $src.find(".fa").removeClass("fa-chevron-circle-up").addClass("fa-chevron-circle-down");
        } else {
          $src.removeClass("down").addClass("up");
          $("#proglayer").show();
          $src.find(".fa").removeClass("fa-chevron-circle-down").addClass("fa-chevron-circle-up");
        }
      },
      addProgClick() {
        console.log("add new program");
        let pid = $("#progingroup").val();
        if($.trim(pid)=="") return;
        let found = $("input[value='"+pid+"']",$("#progtablebody")).length>0;
        if(found) {
          alertbox("Duplicate program entry");
          return;
        }
        let idx = $("tr",$("#progtablebody")).length;
        let text = $("option:selected",$("#progingroup")).text();
        let params = { groupname: this.localData.groupname, programid: $("#progingroup").val(), progname: text, parameters: "", seqno: idx+1 };
        this.$root.$refs.permitForm.startInsertRecord(params,() => {
          this.displayProgramInfo(pid,text,idx+1);
          this.adjustProgTableHeight();
        });
      },
      displayProgramInfo(pid,pname,seqno) {
        if(!seqno) seqno = "";
        let $table = $("<table class='prog-item-table-class'></table>");
        let $item = $('<li class="prog-item-class ui-state-active"></li>');	
        let $tr = $("<tr></tr>");
        let $td1 = $('<td class="cclass progno-column" align="center"></td>').append(seqno);
        let $td2 = $('<td class="cclass progid-column" align="center"></td>').append(pid);
        let $td3 = $('<td class="cclass progname-column">&nbsp;</td>').append(pname);
        let $td4 = $('<td class="cclass progctrl-column" align="center"></td>');
        let $ebtn = $('<button class="btn-edit fa-btn fa fa-pencil"></button>');
        $ebtn.attr("title","Edit "+pid);
        $ebtn.on("click",() => { 
          this.editProgramInGroup(pid,this.localData.groupname);
        });	
        let $btn = $('<button class="btn-delete fa-btn fa fa-trash"></button>');
        $btn.attr("title","Delete "+pid);
        $btn.on("click",() => { 
          confirmRemove([pid],() => { 
            this.removeProgramInGroup(pid,this.localData.groupname,() => { 
              $item.remove();
              this.displaySequenceTableLists("#progtablebody");
            });
          });
          return false;
        });
        $td4.append($ebtn).append("&nbsp;&nbsp;").append($btn);
        let $uinput = $('<input type="hidden" name="progid" class="progidclass"></input>');
        $uinput.val(pid);
        $td2.append($uinput)
        $tr.append($td1).append($td2).append($td3).append($td4);
        $table.append($tr);
        $item.append($table);
        $("#progtablebody").append($item);
      },
      editProgramInGroup(pid,gid,callback) {
        this.$root.$refs.permitForm.editProgramInGroup(pid,gid,callback);
      },
      removeProgramInGroup(pid,gid,callback) {
        this.$root.$refs.permitForm.removeProgramInGroup(pid,gid,callback);
      },
    }
  };
  </script>
  