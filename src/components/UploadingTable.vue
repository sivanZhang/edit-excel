<template>
  <div id="asset-list">
    <div style="padding-bottom:15px;">
      <input
        class="file_inp"
        ref="file_inp"
        accept=".xlsx, .xls, .csv"
        type="file"
        @change="importExcel($event.target)"
      />
      <el-button
        icon="el-icon-circle-plus"
        type="success"
        @click="openFile"
        class="pan-btn green-btn"
        size="mini"
      >添加Excel</el-button>
      <!-- openFile -->
      <el-button
        icon="el-icon-upload"
        type="primary"
        @click="getAsset"
        class="pan-btn green-btn"
        size="mini"
        :loading="uploadLoading"
        :disabled="uploadDisabled"
      >上传</el-button>
      <el-button
        icon="el-icon-refresh-left"
        type="danger"
        @click="importAsset(1)"
        class="pan-btn green-btn"
        size="mini"
      >清空表格</el-button>
      <el-button
        icon="el-icon-delete-solid"
        type="danger"
        @click="deleteTableRow()"
        class="pan-btn red-btn"
        v-show="$refs.tableTemplate?$refs.tableTemplate.tableData.length : false"
        size="mini"
      >删除已选</el-button>
    </div>
    <import-table-template ref="tableTemplate" @returnAssemblingData="returnAssemblingData"></import-table-template>
  </div>
</template>

<script>
import XLSX from "xlsx";
import ImportTableTemplate from "./BaseImportTemplate";
export default {
  neme: "excel-import",
  props: {
    requiredKeys: {
      type: Object,
    },
    notRequiredKeys: {
      type: Object,
    },
    uploadLoading:{
        type:Boolean,
        default:false
    }
  },
  data() {
    return {
      uploadDisabled: true,
      testDataJSON: []
    };
  },
  components: { ImportTableTemplate },
  methods: {
    //点击隐藏的上传文件按钮
    openFile() {
      this.$refs.file_inp.click();
    },
    //导入excel 变异为数组
    importExcel(obj) {
      let _self = this;
      if (!obj.files) {
        return;
      }
      let file = obj.files[0],
        types = file.name.split(".")[1],
        fileType = ["xlsx", "xlc", "xlm", "xls", "xlt", "xlw", "csv"].some(
          item => item === types
        );
      if (!fileType) {
        this.$message.error("格式错误！请重新选择");
        return;
      }
      _self.testDataJSON = [];
      _self.$refs.tableTemplate.openLoading("数据导入中");
      //异步等到解析文件后调用其他方法
      file2Xce(file).then(tabJson => {
        //这里可判断数据是否为空
        _self.testDataJSON = [...tabJson];
        _self.importAsset();
        _self.uploadDisabled = false;
        obj.value = null; //可以重新导入同一个表
      });
      function file2Xce(file) {
        return new Promise(function(resolve) {
          let reader = new FileReader();
          reader.onload = function(e) {
            let data = e.target.result;
            let wb = XLSX.read(data, {
              type: "binary"
            });
            resolve(
              XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]], {
                header: 1, //二维数组展示
                raw: false,
                skipHeader: true
              })
            );
          };
          reader.readAsBinaryString(file);
        });
      }
    },
    //获得编辑后的数据
    getAsset() {
      this.$refs.tableTemplate.getAssemblingData();
    },
    /**
     * 获取表格组装好的数据
     * 组件中必须 @returnAssemblingData="returnAssemblingData"
     */
    returnAssemblingData(data) {
      this.$emit('submit-table',data)
    },
    deleteTableRow() {
      this.$refs.tableTemplate.deleteRow();
    },
    //导入数据 type=1表示重置
    importAsset(type) {
      if (type === 1) {
        this.$refs.tableTemplate.initData({
          reset: true,
          datas: [],
          notRequiredKeys: {},
          requiredKeys: {}
        });
        return;
      }
      let data = {
        reset: false,
        datas: this.testDataJSON,
        notRequiredKeys: this.notRequiredKeys,
        requiredKeys: this.requiredKeys
      };
      this.$refs.tableTemplate.initData(data);
    }
  }
};
</script>
<style scoped>
.file_inp {
  display: none;
}
</style>
