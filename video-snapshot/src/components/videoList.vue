<template>
  <div class="video-cont">
    <div id="videoSliderList">
      <el-button
        size="mini"
        v-for="(item,index) in selectProjects"
        :key="index"
        :type="currentPlayId==item.task.id?'success':''"
        @click="initSource(index),getCurrentPlayId(item.task.id)"
        icon="el-icon-video-play"
        style="margin: 10px 2%;"
      >{{item.task.name}}</el-button>
    </div>
    <el-divider content-position="left">选中播放审核：</el-divider>
    <el-checkbox-group v-model="selectProjectIds">
      <div class="list">
        <div class="item" v-for="(item,index) in projectList" :key="index">
          <el-checkbox
            :label="index"
            :key="index"
            @change="changeCheckedProject($event,item,index)"
          >
            <p class="pro-name">{{item.task.name}}</p>
          </el-checkbox>
          <div style="margin-top:5px">
            <el-image
              :src="item.project.image?$store.state.BASE_URL+item.project.image:''"
              fit="cover"
            >
              <div
                slot="error"
                style="height: 100%;display: flex;justify-content: center;align-items: center;font-size: 56px;background: #dcdfe6;"
              >
                <i class="el-icon-picture" style="color:#909399"></i>
              </div>
            </el-image>
          </div>
        </div>
      </div>
    </el-checkbox-group>
  </div>
</template>
<script>
export default {
  name: "VideoList",
  data() {
    return {
      projectList: [],
      selectProjectIds: [],
      selectProjects: [],
      currentPlayId: -1
    };
  },
  created() {
    this.initInfo();
  },
  methods: {
    initInfo() {
      //读取AJAX获取数据
      this.projectList = [...this.$store.state.video.Selection];
    },
    getCurrentPlayId(id) {
      this.currentPlayId = id;
    },
    initSource(index) {
      let selectProject = this.selectProjects[index];
      /* this.currentPlayId = selectProject.task.id; */
      let projectList = [];
      projectList[0] = selectProject;
      if (index + 1 != this.selectProjects.length) {
        projectList[1] = this.selectProjects[index + 1];
      } else {
        projectList[1] = null;
      }
      this.$emit("initSource", projectList, index, this.selectProjects);
    },
    /**
     * 卡片选中状态切换 ，更新展示的播放列表
     * @param {Boolean} e 点击卡片的多选按钮返回的状态表示是否选中
     * @param {Object} item 点击卡片多选按钮时返回的资产对象
     */
    changeCheckedProject(e, item, index) {
      if (e) {//&& item.path
        let url;
        switch (index) {
          case 0:
            url = "47HK2MpfKwqx1510325093.mp4";
            break;
          case 0:
            url = "banner02.mp4";
            break;
          case 0:
            url = "47HK2MpfKwqx1510325093.mp4";
            break;
          default:
            url = "sEzdz3fIXgqc1512572926.mp4";
            break;
        }
        item = {
          ...item,
          url
        };
        //videoImage:`${this.$store.state.BASE_URL}${item.asset.image}`
        this.selectProjects.push(item);
      } /* else if (e && !item.path) {
        e = false;
        this.$message.warning("镜头路径为空");
      } */ else {
        this.selectProjects = this.selectProjects.filter(t => {
          return t.task.id !== item.task.id;
        });
      }
    },
    addVideoList() {}
  }
};
</script>
<style lang="scss" scoped>
.el-divider{
  margin: 20px 0 10px;
}
#videoSliderList {
  display: block;
  white-space: nowrap;
  width: 100%;
  height: 50px;
  overflow: auto;
  /* li {
    width: 20%;
    display: inline-block;
    margin: 10px 2%;
    .color-slider {
      display: block;
      width: 100%;
      height: 25px;
      background: red;
      cursor: pointer;
      text-align: center;
      color: #fff;
      line-height: 25px;
    }
    .on {
      background: #67c23a;
    }
  } */
}
.video-cont {
  width: 100%;
  height: 100%;
  .list {
    list-style: none;
    padding: 10px;
    margin: 0;
    overflow: hidden;
    height: 100%;
    height: calc(100% - 90px);
    display: flex;
    .item {
      width: 19%;
      border: 1px dotted #ddd;
      margin: 0 0.5%;
      background: #fff;
      padding: 10px;
      .el-image {
        width: 100%;
        height: 90px;
      }
    }
  }
}
</style>
