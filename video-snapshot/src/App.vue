<template>
  <div id="app">
    <VideoAnnotation :play-list="PlayList" :imgList.sync="imgList"/>
  </div>
</template>
<script>
import VideoAnnotation from "./components/VideoAnnotation";
export default {
  components: {
    VideoAnnotation
  },
  data() {
    return {
      PlayList: [
        { id: 1, name: "test1", url: "ge.mp4" },
        { id: 2, name: "test2", url: "47HK2MpfKwqx1510325093.mp4" }
      ],
      imgList:[]
    };
  }
};
</script>