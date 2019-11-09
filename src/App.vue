<template>
  <div id="app">
    <UploadingTable
      :required-keys="requiredKeys"
      :not-required-keys="notRequiredKeys"
      @submit-table="submitTable"
    />
  </div>
</template>
<script>
import UploadingTable from "./components/UploadingTable";
export default {
  components: {
    UploadingTable
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