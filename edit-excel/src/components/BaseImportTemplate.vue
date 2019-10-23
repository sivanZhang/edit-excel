<template>
  <div class="import-table-template">
    <div class="text-right">
      共
      <span>{{tableData.length}}</span> 条数据
    </div>
    <div
      v-loading="tableLoading"
      :element-loading-text="tableLoadingText"
      element-loading-spinner="el-icon-loading"
      element-loading-background="rgba(0, 0, 0, 0.8)"
      style="height:calc(100vh - 168px)"
    >
      <el-table
        :data="tableData"
        border
        @cell-dblclick="dblhandleCurrentChange"
        style="width: 100%;"
        height="100%"
        @selection-change="selected"
      >
        <el-table-column v-if="tableData.length" type="selection" fixed width="50" align="center"></el-table-column>
        <template v-for="(col ,index) in tableCols">
          <el-table-column
            :index="index"
            v-bind:key="index"
            :prop="col.prop"
            :label="col.label"
            align="center"
            min-width="130"
          >
            <template slot="header" slot-scope="scope">
              <div @click="handelClickHeader(scope)" style="cursor:pointer;">
                <p style="height:15px;">{{scope.column.label.split(",")[0]}}</p>
                <p style="height:30px">{{scope.column.label.split(",")[1]}}</p>
              </div>
            </template>
            <template slot-scope="scope">
              <el-input
                v-if="scope.row.isEdit  && index!=0"
                size="small"
                v-model="scope.row[col.prop]"
                placeholder="请输入内容"
                v-on:blur="inputblur"
              ></el-input>
              <span
                v-if="!scope.row.isEdit && !isImage(scope.row[col.prop])"
              >{{scope.row[col.prop]}}</span>
              <el-image
                v-if="isImage(scope.row[col.prop])"
                :src="$store.state.BASE_URL+scope.row[col.prop]"
                fit="cover"
                style="width:80px;height:45px"
              ></el-image>
            </template>
          </el-table-column>
        </template>
      </el-table>
    </div>
    <el-dialog
      :title="'字段绑定['+selectCurrentCol.label+']'"
      :visible.sync="dialogVisible"
      width="30%"
    >
      <el-row type="flex" justify="space-between">
        <el-col>
          <el-select v-model="selectKey" placeholder="请选择" filterable @change="changeHandlerRadio">
            <el-option-group v-for="group in keysMap" :key="group.label" :label="group.label">
              <el-option
                v-for="(item,index) in group.options"
                :key="index"
                :label="allKeysMap[item]"
                :value="item"
              ></el-option>
            </el-option-group>
          </el-select>
        </el-col>
      </el-row>
      <div style="margin-top:20px">
        <el-button type="danger" @click="deleteCol">删除本列</el-button>
        <el-button type="warning" @click="cancelMapping">清空字段</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
export default {
  name: "ImportTableTemplate",
  data() {
    return {
      isShowOptionBar: false,
      dialogVisible: false, //弹出框控制
      assemblingData: {},
      tableLoading: false,
      tableLoadingText: "",
      keysMap: [], //绑定字段所需要的数据
      allKeysMap: [], //用来缓存的字段
      dealKeys: [], //原始keys
      tableCols: [], //列[{...}]
      tableData: [], //表格数据 [{},{},{}]
      selectKey: null,
      selectCurrentCol: { label: "" }, //选中的当前列
      BoundkeyCol: [], //已绑定字段列的存储
      selection: []
    };
  },
  methods: {
    //是否为图片正则
    isImage(str) {
      return /(.*)\.(jpg|bmp|gif|ico|pcx|jpeg|tif|png|raw|tga)$/.test(str);
    },
    //点击表格头
    handelClickHeader({ column }) {
      if (this.tableCols[column.index]) {
        this.selectCurrentCol = column;
        this.dialogVisible = true;
        this.selectKey = "";
      }
    },
    //删除列
    deleteCol() {
      this.tableCols.splice(this.selectCurrentCol.index, 1);
        this.cancelBind();
        this.dialogVisible = false;
    },
    //缓存勾选中的行
    selected(e) {
      this.selection = [...e];
    },
    //删除勾选的行
    deleteRow() {
      if (!this.selection.length) {
        this.$message.warning("请勾选行");
      }
      this.tableData = this.tableData.filter(t => {
        return !this.selection.includes(t);
      });

      this.selection = [];
    },
    /**
     * 绑定资产字段
     */
    changeHandlerRadio(value) {
      let isBind = this.BoundkeyCol.findIndex(t => t.field === value);

      if (isBind < 0) {
        //this.selectCurrentCol点击的列的信息
        let label = this.tableCols[this.selectCurrentCol.index].label; //label是选中列的lable为了截取ABCD.....
        this.tableCols[this.selectCurrentCol.index].label =
          //大写英文字母 + 传过来的中文字段
          label.split(",")[0] + "," + this.allKeysMap[value];
        this.tableCols[this.selectCurrentCol.index].name = value;
        //缓存已选择的 keys
        this.BoundkeyCol.push({
          Node: this.selectCurrentCol.property,
          field: value
        });
        this.dialogVisible = false;
      } else {
        this.$message.error("该字段已有绑定过");
      }
    },
    //取消绑定
    cancelMapping() {
      this.cancelBind();
      let label = this.tableCols[this.selectCurrentCol.index].label;
      this.tableCols[this.selectCurrentCol.index].label =
        label.split(",")[0] + ",未绑定字段";
      this.tableCols[this.selectCurrentCol.index].name = "";
      this.dialogVisible = false;
    },
    //导入后初始化
    initData(data) {
      //表格数据 = 空
      this.BoundkeyCol = [];
      this.tableData = [];
      this.tableLoading = true;
      this.tableLoadingText = "数据组装中";
      //原始数据= [[],[],[],[]]
      const dealDatas = data.datas;
      //绑定字段= {}
      this.allKeysMap = { ...data.requiredKeysMap, ...data.keysMap };
      this.keysMap = [
        {
          label: "必填字段",
          options: Object.keys(data.requiredKeysMap),
          obj: { ...data.requiredKeysMap }
        },
        {
          label: "非必填字段",
          options: Object.keys(data.keysMap)
        }
      ];

      //原始keys
      this.dealKeys = [];
      //绑定字段 的 key组成一个数组 push到 原始keys中
      for (let key in this.allKeysMap) {
        this.dealKeys.push(key);
      }
      this.getTableHeader(data.reset, dealDatas);
    },
    //获取传递的数据
    getAssemblingData() {
      this.assemblingData.values = [];
      /**
       * 已经绑定列的下下标
       */
      let bindKeys = new Set(),
        values = [];
      //绑定的镜头赋值
      this.tableData.forEach((t, i) => {
        let value = [];
        this.BoundkeyCol.forEach(assetItem => {
          bindKeys.add(assetItem.field);
          value.push(this.tableData[i][assetItem.Node]);
        });
        values.push(value);
      });
      bindKeys = [...bindKeys];
      //如果有绑定环节
      this.assemblingData.keys = bindKeys;
      this.assemblingData.values = values;
      //必填字段验证
      for (const t of this.keysMap[0].options) {
        if (!bindKeys.includes(t)) {
          this.$message.warning(this.keysMap[0].obj[t] + "是必填字段");
          return false;
        }
      }
      this.$emit("returnAssemblingData", this.assemblingData);
    },
    /**
     * 组装表格头  bl = 是否重置表格
     */
    getTableHeader(bl, dealDatas) {
      this.tableCols = [];
      //加载中文字
      this.tableLoadingText = "数据组装表头中";
      //
      if (dealDatas.length > 0) {
        this.tableLoading = true;
        let firstData = dealDatas[0];
        //用第一列原始data进行循环
        for (let i = 0; i < firstData.length; i++) {
          this.letterIndex = 65 + i;
          let label = {
            label: String.fromCharCode(65 + i) + ",未绑定字段",
            prop: "node" + i,
            name: ""
          };
          //列数据
          this.tableCols.push(label);
        }
        this.isShowOptionBar = true;
        //开始组装数据
        this.getTableData(dealDatas);
      } else {
        this.tableLoading = false;
        this.isShowOptionBar = false;
        if (bl) return;
        this.$message.error("导入的数据是空数据");
      }
    },
    //转化成el tabale用的数据格式
    getTableData(dealDatas) {
      this.tableLoadingText = "数据组装数据中";
      dealDatas.forEach(t => {
        let data = {};
        t.forEach((ct, ci) => {
          data["node" + ci] = t[ci];
        });
        data["isEdit"] = false;
        this.tableData.push(data);
      });
      this.tableLoading = false;
    },
    dblhandleCurrentChange(row) {
      row.isEdit = true;
    },
    inputblur() {
      let tableD = this.tableData;
      tableD.forEach(function(item) {
        item.isEdit = false;
      });
    },
    //取消绑定某列的镜头字段
    cancelBind() {
      this.BoundkeyCol.forEach((t, i, arr) => {
        if (t.Node === this.selectCurrentCol.property) {
          arr.splice(i, 1);
        }
      });
    },
    openLoading(msg) {
      this.tableLoading = true;
      this.tableLoadingText = msg;
    },
    closeLoading() {
      this.tableLoading = false;
    }
  }
};
</script>
<style scoped>
.import-table-template {
  position: relative;
}
.text-right {
  width: 100%;
  position: absolute;
  right: 0;
  top: -40px;
  text-align: right;
  font-size: 16px;
  font-weight: 600;
  z-index: -1;
}
.text-right span {
    color: #ed4014;
    font-size: 18px;
  }
</style>
