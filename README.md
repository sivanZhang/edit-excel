# edit-excel

> 这是一个用表格形式编辑导入后的excel数据的组件，可以实现纯前端导入excel,在表格中编辑导入的数据，删除行列，在表头绑定字段，上传编辑后的表格数据。



## 使用说明
   使用时直接把[本git项目](http://panjiachen.github.io/vue-admin-template)中components文件夹下的edit-excel文件夹复制到需要引用的项目中，然后inport使用：
    ```html
     <EditExcel
      :required-keys="requiredKeys"
      :not-required-keys="notRequiredKeys"
      @submit-table="submitTable"
    />
     ```
   ```js
<script>
import EditExcel from "@/components/edit-excel";
export default {
  components: {
    EditExcel
  },
  data() {
    return {
      requiredKeys: {
        //必填字段
        name: "用户名",
        email: "邮箱"
      },
      notRequiredKeys: {
        //非必填字段
        phone: "手机号码",
        dept: "部门",
        isactive: "是否使用"
      }
    };
  },
  methods: {
    submitTable(data) {
      this.$notify({
        title: "提交服务器的数据为：",
        message: data,
        duration: 0,
        type: "warning"
      });
    }
  }
};
</script>
```
# DEMO    
## Demo结构

``` bash
│ 
├── App.vue    # 引入了edit-excel下面组件，并在该页面有使用demo
└── components
    └── edit-excel    # edit-excel组件封装
```
## 启动demo项目

```bash
# 克隆项目
git clone 项目git地址

# 进入项目目录
cd 项目目录

# 安装依赖
npm install

# 启动服务
npm run serve
```


# 组件说明

##  Attribute


| 属性名 | 说明 | 类型 | 默认值 |
| --------- | --------- | --------- | --------- |
| required-keys| 必填字段，如果在表格编辑期间未绑定所有必填字段则无法通过提交验证| Object| — |
| not-required-keys | 非必填字段 | Object | {} |
| uploadLoading | “上传”按钮的loading状态 | Boolean | false |



##  Events


| 事件名称 | 说明 | 回调参数 |
| --------- | --------- | --------- | 
| submit-table| 点击组件中“上传”按钮触发| submitTable：编辑完后的表格数据|
