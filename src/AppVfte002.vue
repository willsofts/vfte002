<!-- App.vue -->
<template>
  <div id="fswaitlayer" class="fa fa-spinner fa-spin"></div>
  <div class="pt-page pt-page-current pt-page-controller search-pager">
    <PageHeader ref="pageHeader" :labels="labels" pid="vfte002" version="1.0.0" showLanguage="true" @language-changed="changeLanguage" :multiLanguages="multiLanguages" :build="buildVersion" :visible="displayPageHeader" />
    <SearchForm ref="searchForm" :labels="labels" :dataCategory="dataCategory" @data-select="dataSelected" @data-insert="dataInsert" v-show="isShowModify == false" />
    <ModifyForm ref="modifyForm" :labels="labels" :dataCategory="dataCategory" @data-updated="modifyUpdated" @data-cancel="modifyCancel" v-show="isShowModify == true"/>
  </div>
  <teleport to="#modaldialog">
    <EntryForm ref="entryForm" :labels="labels" :dataCategory="dataCategory" @data-saved="dataSaved" @data-updated="dataUpdated" @data-deleted="dataDeleted" />
  </teleport>
  <teleport to="#permitdialog">
    <PermitForm ref="permitForm" :labels="labels" :dataCategory="dataCategory" @data-saved="permitSaved" @data-updated="permitUpdated"  @data-deleted="permitDeleted" />
  </teleport>
</template>
<script>
import { ref } from 'vue';
import $ from "jquery";
import { PageHeader } from '@willsofts/will-control';
import SearchForm from '@/components/SearchForm.vue';
import ModifyForm from './components/ModifyForm.vue';
import EntryForm from '@/components/EntryForm.vue';
import PermitForm from "@/components/PermitForm.vue";
import { getLabelModel, getMultiLanguagesModel, getMetaInfo } from "@willsofts/will-app";
import { DEFAULT_CONTENT_TYPE, getDefaultLanguage, setDefaultLanguage, getApiUrl } from "@willsofts/will-app";
import { startApplication, serializeParameters, loadAndMergeLabel } from "@willsofts/will-app";

const buildVersion = process.env.VUE_APP_BUILD_DATETIME;
export default {
  components: {
    PageHeader, SearchForm, EntryForm, PermitForm, ModifyForm
  },
  setup() {
    const dataChunk = {};
    const dataCategory = {
      tprog: [],
      tkgroupmobile: [],
      tkusertype: [],
      tkpermit: [],
    };
    let isShowModify = ref(false);
    let labels = ref(getLabelModel());
    let alreadyLoading = ref(false);
    const multiLanguages = ref(getMultiLanguagesModel());
    const displayPageHeader = !(String(getMetaInfo().DISPLAY_PAGE_HEADER)=="false");
    return { displayPageHeader, buildVersion, multiLanguages, labels, dataCategory, dataChunk, alreadyLoading, isShowModify };
  },
  mounted() {
    console.log("App: mounted ...");
    this.$nextTick(async () => {
      //ensure ui completed then invoke startApplication 
      startApplication("vfte002",(data) => {
        console.log("vueapp: message",data);
        if(data.type=="language") {
          let lang = data.language;
          if(lang) {
            this.changeLanguage(lang);
          }
        } else {
          this.multiLanguages = getMultiLanguagesModel();
          this.messagingHandler(data);
          this.loadDataCategories(!this.alreadyLoading,() => {
            this.$refs.pageHeader.changeLanguage(getDefaultLanguage());
          });
        }
          loadAndMergeLabel("vfte002", (success) => {
            if (success) {
              this.changeLanguage(getDefaultLanguage());
            }
          });
      });
      //try to find out parameters from url
      const searchParams = new URLSearchParams(window.location.href);
      console.log("param: authtoken=",searchParams.get("authtoken"),", language=",searchParams.get("language"));      
    });
  },
  methods: {
    messagingHandler(data) {
      console.log("messagingHandler: data",data); 
    },
    changeLanguage(lang) {
      setDefaultLanguage(lang);
      let labelModel = getLabelModel(lang);
      this.labels = labelModel;
      this.resetDataCategories(lang);
    },
    loadDataCategories(loading,callback) {
      console.log("loadDataCategories: loading",loading);
      if(!loading) return;
      let jsondata = {names: ["tprog", "tkgroupmobile", "tkusertype", "tkpermit"]};
      let formdata = serializeParameters(jsondata);
      $.ajax({
        url: getApiUrl()+"/api/category/lists",
        data: formdata.jsondata,
        headers : formdata.headers,
        type: "POST",
        dataType: "json",
        contentType: DEFAULT_CONTENT_TYPE,
        error : function(transport,status,errorThrown) {
          console.error("loadDataCategories: error: status",status,"errorThrown",errorThrown);
        },
        success: (data) => {
          this.alreadyLoading = true;
          console.log("loadDataCategories: success",data);
          if(data.body) {
            for(let item of data.body) {
              if(item.category && item.resultset && item.resultset.rows) {
                this.dataChunk[item.category] = item.resultset.rows;
              }
            }
            console.log("data chunk",this.dataChunk);
            this.resetDataCategories();
            if(callback) callback();
          }
        }
      });	
    },
    resetDataCategories(lang) {
      if(!lang) lang = getDefaultLanguage();
      if(!lang || lang.trim().length==0) lang = "EN";
      let tprog;
      let tkgroupmobile;
      let tkusertype;
      let tkpermit;
      let tk_categories = this.dataChunk["tprog"];
      if(tk_categories) {
        tprog = tk_categories.map((item) => { return { id: item.programid, text: "TH"==lang?item.prognameth:item.progname } });
      }
      tk_categories = this.dataChunk["tkgroupmobile"];
      if(tk_categories) {
        tkgroupmobile = tk_categories.map((item) => { return { id: item.typeid, text: "TH"==lang?item.nameth:item.nameen } });
      }
      tk_categories = this.dataChunk["tkusertype"];
      if(tk_categories) {
        tkusertype = tk_categories.map((item) => { return { id: item.typeid, text: "TH"==lang?item.nameth:item.nameen } });
      }
      tk_categories = this.dataChunk["tkpermit"];
      if(tk_categories) {
        tkpermit = tk_categories.map((item) => { return { id: item.typeid, text: "TH"==lang?item.nameth:item.nameen } });
      }
      if(tprog) this.dataCategory.tprog = tprog;
      if(tkgroupmobile) this.dataCategory.tkgroupmobile = tkgroupmobile;
      if(tkusertype) this.dataCategory.tkusertype = tkusertype;
      if(tkpermit) this.dataCategory.tkpermit = tkpermit;
    },
    dataSelected(item,action) {
      //listen action from search form
      console.log("App: dataSelected",item,"action",action);
      if("edit"==action) {
        console.log("do edit");
        this.$refs.modifyForm.retrieveRecord({groupname: item.groupname},() => { this.isShowModify = true; });
      } else if("delete"==action) {
        this.$refs.entryForm.startDeleteRecord({groupname: item.groupname});
      } else if("modify"==action) {
        this.$refs.entryForm.retrieveRecord({groupname: item.groupname});        
      }
    },
    dataInsert(filters) {
      //listen action from search form
      console.log("App: record insert",filters);
      this.$refs.entryForm.startInsertRecord();
    },
    dataSaved(data,response) {
      //listen action from entry form when after saved
      console.log("App: record saved");
      console.log("data",data,"response",response);
      this.$refs.searchForm.search();
    },
    dataUpdated(data,response) {
      //listen action from entry form when after updated
      console.log("App: record updated");
      console.log("data",data,"response",response);
      this.$refs.searchForm.search();
    },
    dataDeleted(data,response) {
      //listen action from entry form when after deleted
      console.log("App: record deleted");
      console.log("data",data,"response",response);
      this.$refs.searchForm.search(true);
    },
    permitSaved(data,response) {
      console.log("App: permit saved");
      console.log("data",data,"response",response);
    },
    permitUpdated(data,response) {
      console.log("App: permit updated");
      console.log("data",data,"response",response);
    },
    permitDeleted(data,response) {
      console.log("App: permit deleted");
      console.log("data",data,"response",response);
    },
    modifyUpdated(data,response) {
      console.log("App: modify updated",data,"response",response);
      this.isShowModify = false;
    },
    modifyCancel(data) {
      console.log("App: modify cancel",data);
      this.isShowModify = false;
    }
  }
};
</script>