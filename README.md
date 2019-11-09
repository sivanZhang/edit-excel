# vue-admin-template

> 这是一个用表格形式编辑导入后的excel数据的组件，可以实现纯前端导入excel,在表格中编辑导入的数据，删除行列，在表头绑定字段，上传编辑后的表格。

[线上地址](http://panjiachen.github.io/vue-admin-template)

## Extra

如果你想要根据用户角色来动态生成侧边栏和 router，你可以使用该分支[permission-control](https://github.com/PanJiaChen/vue-admin-template/tree/permission-control)

## 相关项目

[vue-element-admin](https://github.com/PanJiaChen/vue-element-admin)

[electron-vue-admin](https://github.com/PanJiaChen/electron-vue-admin)

[vue-typescript-admin-template](https://github.com/Armour/vue-typescript-admin-template)

写了一个系列的教程配套文章，如何从零构建后一个完整的后台项目:

- [手摸手，带你用 vue 撸后台 系列一(基础篇)](https://juejin.im/post/59097cd7a22b9d0065fb61d2)
- [手摸手，带你用 vue 撸后台 系列二(登录权限篇)](https://juejin.im/post/591aa14f570c35006961acac)
- [手摸手，带你用 vue 撸后台 系列三 (实战篇)](https://juejin.im/post/593121aa0ce4630057f70d35)
- [手摸手，带你用 vue 撸后台 系列四(vueAdmin 一个极简的后台基础模板,专门针对本项目的文章,算作是一篇文档)](https://juejin.im/post/595b4d776fb9a06bbe7dba56)
- [手摸手，带你封装一个 vue component](https://segmentfault.com/a/1190000009090836)

## Build Setup

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

浏览器访问 [http://localhost:9528](http://localhost:9528)


更多信息请参考 [使用文档](https://panjiachen.github.io/vue-element-admin-site/zh/)

## Demo

![demo](https://github.com/PanJiaChen/PanJiaChen.github.io/blob/master/images/demo.gif)

###  Attribute


| 属性名 | 说明 | 类型 | 默认值 |
| --------- | --------- | --------- | --------- |
| required-keys| 必填字段，如果在表格编辑期间未绑定所有必填字段则无法通过提交验证| Object| — |
| not-required-keys | 非必填字段 | Object | {} |
| uploadLoading | “上传”按钮的loading状态 | Boolean | false |



###  Events


| 事件名称 | 说明 | 回调参数 |
| --------- | --------- | --------- | 
| submit-table| 点击组件中“上传”按钮触发| submitTable：编辑完后的表格数据|
