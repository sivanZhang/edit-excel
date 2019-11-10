# edit-excel

> 这是一个用表格形式编辑导入后的excel数据的组件，可以实现纯前端导入excel,在表格中编辑导入的数据，删除行列，在表头绑定字段，上传编辑后的表格数据。
### 在线demo
[在线访问](https://www.chidict.com/demos/table/)
### 使用说明
   使用时直接把[本git项目](https://github.com/sivanZhang/edit-excel)中components文件夹下的edit-excel文件夹复制到需要引用的项目中，然后inport使用：


template:
```html
<EditExcel
      :required-keys="requiredKeys"
      :not-required-keys="notRequiredKeys"
      @submit-table="submitTable"
    />
```
js:
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
        //规定必须在上传表格时绑定的字段
        name: "用户名",
        email: "邮箱"
      },
      notRequiredKeys: {
        //规定在上传表格时绑定的可选字段
        phone: "手机号码",
        dept: "部门",
        isactive: "是否使用"
      }
    };
  },
  methods: {
   /**
     * 点击“上传”按钮后出发
     *
     * @param {Object} data 返回当前表格数据
     *
     */
    submitTable(data) {
      //upload代码...
    }
  }
};
</script>
```
# DEMO
> 本项目可下载到本地，作为demo参考。
### 项目结构
**目录 (Table of Contents)**
``` bash
│ 
├── App.vue    # 引入了edit-excel组件，并在该页面有使用demo
└── components
    └── edit-excel    # 封装edit-excel组件文件夹
```
### 启动项目

```bash
# 克隆项目
git clone https://github.com/sivanZhang/edit-excel.git

# 进入项目目录
cd 项目目录

# 安装依赖
npm install

# 启动服务
npm run serve
```
### 视频预览
[![Watch the video](https://zjwvedio.oss-cn-beijing.aliyuncs.com/22.png)](https://zjwvedio.oss-cn-beijing.aliyuncs.com/vbz.mp4)

# 组件说明

### Attribute

| 属性名 | 说明 | 类型 | 默认值 |
| ------------------ | --------- | --------- | --------- |
| required-keys| 上传表格时必填字段，如果在表格编辑期间未绑定所有必填字段则无法通过提交验证| Object| — |
| not-required-keys | 非必填字段 | Object | {} |
| uploadLoading | “上传”按钮的loading状态 | Boolean | false |



### Events

| 事件名称 | 说明 | 回调参数 |
| ------------------ | --------- | --------- | 
| submit-table| 点击组件中“上传”按钮触发| submitTable：编辑完后的表格数据|
