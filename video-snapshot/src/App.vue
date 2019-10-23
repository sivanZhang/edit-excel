<template>
  <div id="video-check">
    <el-row class="page" :gutter="15">
      <el-col :span="19" class="page-left" style="height:850px;">
        <div class="video-player" style="height：550px;">
          <!-- 播放器 -->
          <video-player
            ref="videoPlayer"
            @getCutImg="getMarkImage"
            @getCurrentVideoMode="getCurrentVideoMode"
            @getCurrentPlayId="getCurrentPlayId"
          />
        </div>
        <div class="video-list">
          <!-- 视频列表 -->
          <video-list ref="videoList" @initSource="initSource"></video-list>
        </div>
      </el-col>
      <el-col :span="8" class="page-right">
        <div id="videoTabs" class="video-tabs">
          <div class="mark-cont">
            <div class="mark-form">
              <ul class="img-list" v-if="imgList.length > 0">
                <li class="img-item" v-for="(item,index) in imgList" :key="index">
                  <el-image
                    :src="item.imgUrl"
                    :preview-src-list="ImageList(imgList)"
                    fit="cover"
                    style="height:45px;width:80px;"
                  ></el-image>
                  <div>
                    <span>{{item.currentProject.task.name}} (第{{item.currentFrame}}帧)</span>
                    <el-button
                      type="text"
                      class="del-btn"
                      icon="el-icon-delete-solid"
                      @click="delMarkImage(item,index)"
                    ></el-button>
                  </div>
                </li>
              </ul>
              <el-input placeholder="添加备注" v-model="markText" type="textarea"></el-input>
              <div class="btn-group">
                <el-radio-group v-model.number="approve_result">
                  <el-radio :label="0">拒绝</el-radio>
                  <el-radio :label="1">同意</el-radio>
                </el-radio-group>
                <div style="margin-top:5px">
                  <el-checkbox v-model="checked">是否提交客户审批</el-checkbox>
                </div>
                <el-input
                  v-if="checked"
                  type="textarea"
                  v-model="out_suggestion"
                  ref="outer-input"
                  placeholder="请输入提交客户审批的说明"
                  style="margin-top:5px"
                ></el-input>
                <el-input
                  v-if="checked"
                  v-model="out_path"
                  ref="outer-path"
                  placeholder="请输入提交客户审批的路径"
                  style="margin-top:5px"
                ></el-input>
                <template v-if="scoreShow">
                  <el-row type="flex" align="middle" style="margin-top:5px">
                    <el-col :span="5">评分</el-col>
                    <el-col :span="19" align="left">
                      <el-input-number v-model="score" :min="0" :max="100" :step="10"></el-input-number>
                    </el-col>
                  </el-row>
                </template>
                <div class="fr" style="margin-top:5px">
                  <el-button type="primary" class="btn add-btn" @click="commitApprove">提交</el-button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import VideoPlayer from "./components/VideoPlayer";
import VideoList from "./components/videoList";
export default {
  components: { VideoPlayer, VideoList },
  data() {
    return {
      out_suggestion: "",
      checked: false,
      approve_result: 0,
      score: null,
      imgList: [], //  视频截图列表
      activeTab: "first",
      markText: "",
      currentVideoIsEdit: false,
      pWidth: 0,
      pHeight: 0,
      submitList: [],
      currentId: null,
      scoreShow: false,
      out_path:null
    };
  },
  mounted() {
    let bH = document.body.offsetHeight;
    let videoTabsH = document.getElementById("videoTabs").offsetHeight;
    let videoPlayer = document.getElementsByClassName("video-player");
    this.pHeight = videoPlayer[0].offsetHeight;
    this.pWidth = videoPlayer[0].offsetWidth;
    document.getElementById("videoComment").style.height =
      bH - (videoTabsH + 20 + 20 + 94 + 165) + "px";
  },
  methods: {
    commitApprove() {
      if (!this.submitList.length) {
        this.$message.warning("请选择镜头");
        return false;
      } else if (!this.markText) {
        this.$message.warning("请添加备注");
        return false;
      }
      let RequestList = this.submitList.map((t, i, arr) => {
        let data = {
          task_id: t.task.id,
          approve_result: this.approve_result,
          suggestion: this.markText,
          key: []
        };
        if (this.checked) {
          //添加提交外网审核字段
          data = {
            ...data,
            click: "",
            out_suggestion: this.out_suggestion,
            path :this.out_path
          };
        }
        if (this.scoreShow === true) {
          data = { ...data, score: this.score };
        }
        this.imgList.forEach(k => {
          if (k.task === t.task.id) {
            data["key"].push({
              image: k.imgUrl,
              frame: k.currentFrame
            });
          }
        });
        return postApprove(data);
      });
      //并发请求，避免重复处理后逻辑
      AXIOS.all(RequestList).then(
        AXIOS.spread((...arg) => {
          this.submitList.forEach((t, i) => {
            this.$message({
              message: t.task.name + " " + arg[i].data.msg,
              showClose: true,
              duration: 0
            });
          });
          this.$refs["approvelogs"].getApproveLog(this.currentId);
          this.imgList = [];
          this.markText = "";
        })
      );
    },
    //点击播放列表回传  projectLists播放列表   index 当前点击的item 下标
    initSource(projectList, index, projectLists) {
      this.currentId = projectList[0].task.id;
      if (projectList[0].project.pro_type === 0) {
        this.scoreShow = true;
      } else {
        this.scoreShow = false;
      }
      this.$refs["approvelogs"].getApproveLog(projectList[0].task.id);
      this.submitList = [...projectLists];
      if (this.currentVideoIsEdit) {
        this.$message.error("处于视频标注模式");
      } else {
        this.$refs.videoPlayer.initVideoUrl(
          projectList[0],
          this.pWidth,
          this.pHeight
        );
        this.$refs.videoPlayer.initNextVideo(index, projectList);
      }
    },
    getMarkImage(obj) {
      this.imgList.push(obj);
      let bH = document.body.offsetHeight;
      let videoTabsH = document.getElementById("videoTabs").offsetHeight;
      if (bH - (165 + videoTabsH + 20 + 20 + 53) <= 0) {
        document.getElementById("videoComment").style.height = 0 + "px;";
      } else {
        document.getElementById("videoComment").style.height =
          bH - (165 + videoTabsH + 20 + 20 + 53) + "px";
      }
    },
    delMarkImage(data, index) {
      this.imgList.splice(index, 1);
      let bH = document.body.offsetHeight;
      let videoTabsH = document.getElementById("videoTabs").offsetHeight;
      if (this.imgList.length > 0) {
        if (bH - (165 + videoTabsH + 20 + 20 - 82) <= 0) {
          document.getElementById("videoComment").style.height = 0 + "px;";
        } else {
          document.getElementById("videoComment").style.height =
            bH - (165 + videoTabsH + 20 + 20 - 82) + "px";
        }
      } else {
        if (bH - (165 + videoTabsH + 20 + 20 - 110) <= 0) {
          document.getElementById("videoComment").style.height = 0 + "px;";
        } else {
          document.getElementById("videoComment").style.height =
            bH - (165 + videoTabsH + 20 + 20 - 110) + "px";
        }
      }
    },
    ImageList(imgList) {
      return imgList.map(t => t.imgUrl);
    },
    //传递 获取视频是否属于编辑中
    getCurrentVideoMode(mode) {
      this.currentVideoIsEdit = !mode;
      // console.log("currentVideoIsEdit", this.currentVideoIsEdit);
    },
    getCurrentPlayId({ task }) {
      this.$refs.videoList.getCurrentPlayId(task.id);
      this.$refs["approvelogs"].getApproveLog(task.id);
    }
  }
};
</script>

<style lang="scss" scoped>
@mixin scrollStyle {
  &::-webkit-scrollbar {
    /*滚动条整体样式*/
    width: 6px; /*高宽分别对应横竖滚动条的尺寸*/
    height: 6px;
    cursor: pointer;
  }
  &::-webkit-scrollbar-thumb {
    /*滚动条里面小方块*/
    border-radius: 5px;
    box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
    background: rgba(0, 0, 0, 0.2);
    cursor: pointer;
  }
  &::-webkit-scrollbar-track {
    /*滚动条里面轨道*/
    box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
    border-radius: 0;
    background: rgba(0, 0, 0, 0.1);
  }
}
#videoComment {
  overflow-y: scroll;
}
#video-check {
  width: 100%;
  min-height: calc(100vh - 84px);
  background-color: #eee;
  .page {
    width: 100%;
    height: 100%;
    min-width: 1200px;
    overflow-y: hidden;
    background-color: #eeeeee;
    .page-left {
      height: 100%;
      width: 62%;
      margin-right: 0.5%;
      .video-player {
        width: 100%;
        height: 65%;
        background: #fff;
      }
      .video-list {
        width: 100%;
        margin-top: 1%;
        background: #fff;
      }
    }
    .page-right {
      height: 100%;
      margin-left: 0.5%;
      .video-tabs {
        padding: 10px;
        background: #fff;
        margin-bottom: 10px;
        min-height: 50%;
        max-height: 100%;
        overflow: auto;
      }
      .video-comment {
        padding: 10px;
        background: #fff;
        height: 100%;
        overflow: auto;
      }
      .video-info,
      .video-tabs,
      .video-comment {
        @include scrollStyle;
      }
    }
  }
}
.mark-list {
  width: 100%;
  height: 100%;
  padding: 0 10px;
  margin: 0;
  list-style: none;

  .item {
    float: left;
    width: 100px;
    height: 100px;
    margin-right: 12px;
    margin: 12px 12px 12px 0;

    img {
      width: 100%;
      height: 100%;
    }
  }
}

.mark-cont {
  width: 100%;

  .mark-form {
    .img-list {
      @include scrollStyle;
      list-style: none;
      padding: 0;
      margin: 0;
      max-height: 195px;
      overflow-y: scroll;
      margin-bottom: 5px;

      .img-item {
        margin: 6px 0;
        height: 62px;
        background: #eee;
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 0 8px;

        .del-btn {
          font-size: 18px;
          color: #f56c6c;
          margin-left: 10px;
        }
      }
    }

    .btn-group {
      margin-top: 10px;
      overflow: hidden;

      .fr {
        float: right;
      }

      .btn {
        padding: 6px 10px;
        font-size: 12px;
      }
    }
  }
}
</style>