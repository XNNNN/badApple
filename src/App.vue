<template>
  <div id="app">
    <!-- <canvas id="canvas"></canvas> -->
    <!-- <div> -->
      <!-- <img id="source" src="../public/77818004_p0.jpg" width="300" height="227"> -->
      <!-- <video id="source" width="300" height="227" controls = "controls" muted src="../public/BADAPPLE 【高清重制版 2K 4K 60FPS】 - 1.1k_x264(Av47027321,P1).mp4"></video> -->
    <!-- </div> -->
    <el-container>
      <el-header>
         <el-switch
            class="colorControl"
            v-model="colorControl"
            active-color="#13ce66"
            inactive-color="#66B1FF"
            active-text="彩色"
            inactive-text="黑白"
            @change="changeColorMode()"
            >
          </el-switch>
        <span class="thresholdText">阈值：</span>  
        <el-slider
          class="threshold"
          v-model="Threshold"
          :min="0"
          :max="255"
          @input="getThresholdValue()"
          show-input>
        </el-slider>
        <span class="zoomText">像素大小：</span>
        <el-slider
          class="zoom"
          v-model="zoom"
          @input="getZoomValue()"
          show-input>
        </el-slider>
      </el-header>
      <el-container>
        <el-aside width="200px">
          <video
            id="beforVideo" 
            class="beforVideo" 
            width="200"
            height="112"
            :src="videoAddress" 
            :controls="controls"
            >
          </video>
          <input id="img-upload" type="file" accept="video/*" @change="uploadGetInformation
          ">
           <!-- <el-button class= "upload" type="file" accept="image/*">上传</el-button> -->
           <el-button class="download" type="primary">下载</el-button>
        </el-aside>
        <el-container>
          <el-main>
            <canvas id="output" width="1000" height="500"></canvas>
          </el-main>
        </el-container>
      </el-container>
    </el-container>
  </div>
</template>

<script>

export default {
  name: 'app',
  data () {
    return {
      colorControl: true, // 颜色模式
      threshold: 128, // 阈值
      zoom: 0,  // 缩放
      videoAddress: undefined, // 视频的本地地址
      controls: undefined  // 控制视频的控件属性
    }
  },
  mounted () {},
  methods: {
    uploadGetInformation (e) { // 获取上传文件的信息
      const _this = this
      let file = e.target.files[0]
      let reader = new FileReader()
      
      if(!file.type.match('video.*')) {
        this.$message({
          message: '文件格式不对哦。',
          type: 'warning'
        })
        return 
      }
      reader.readAsDataURL(file)
      reader.onload = function(arg) {
        let beforVideo = document.getElementById('beforVideo')  // 转码之前的元素
        let outputVideo = document.getElementById('output') // 转码后的视频元素
        let outputVideotx = outputVideo.getContext('2d') 
        let colorMode = _this.colorControl === true ? 0 : 1 

        _this.videoAddress = arg.target.result
        _this.controls = "controls" // 视频控件属性
        outputVideotx.scale(0.5,0.5) // canvas缩放
        setInterval(() => {
          outputVideotx.drawImage(beforVideo, 10, 10, 800, 448)
          let videoData = outputVideotx.getImageData(0, 0, 800, 448)
          _this.thresholdConvert(outputVideotx, videoData, colorMode)
        },16)
      }
    },
    thresholdConvert (ctx, imageData, mode) {
      console.log('mode', mode)
      let  data = imageData.data
      for (let i = 0; i < data.length; i += 4) {
            let red = data[i];
            let green = data[i + 1];
            let blue = data[i + 2];
            let alpha = data[i + 3];

            // 灰度计算公式
            let gray = 0.299 * data[i] + 0.587 * data[i + 1] + 0.114 *data[i + 2];
            let color = gray >= this.threshold ? 255 : 0;

            data[i]     = (mode == 0 && color == 0) ? red : color;    // red
            data[i + 1] = (mode == 0 && color == 0) ? green : color;  // green
            data[i + 2] = (mode == 0 && color == 0) ? blue : color;   // blue
            data[i + 3] = alpha >= this.threshold ? 255 : 0;               // 去掉透明
        }
        ctx.putImageData(imageData, 10, 100, 800, 448, 800, 448)
    },
    // getModeValue (ele) {
    //   for (let i = 0, len = ele.length; i < len; i++) {
    //         if (ele[i].checked) {
    //             return ele[i].value;
    //         }
    //     }
    // },
    getThresholdValue() { // 获取阈值
      console.log('this', this)
      console.log(this.Threshold)
    },
    getZoomValue() {
      console.log(this.zoom*0.01)
    },
    changeColorMode() {
      console.log(this.colorControl)

    }
  }
}
</script>

<style>
  html,body {
    height: 100%;
  }
  body {
    margin: 0px;
    padding: 0px;
    border: 0px;
  }
  .el-header, .el-footer {
    background-color: #B3C0D1;
    color: #333;
    line-height: 60px;
  }
  
  .el-aside {
    background-color: #D3DCE6;
    color: #333;
    text-align: center;
    line-height: 200px;
  }
  
  .el-main {
    height: calc(100vh - 60px);
    background-color: #E9EEF3;
    color: #333;
    text-align: center;
    line-height: 160px;
  }
  
  body > .el-container {
    margin-bottom: 40px;
  }
  
  .el-container:nth-child(5) .el-aside,
  .el-container:nth-child(6) .el-aside {
    line-height: 260px;
  }
  
  .el-container:nth-child(7) .el-aside {
    line-height: 320px;
  }
  .threshold {
    display: inline-block;
    width: 30%;
    margin-top: 10px;
  }
  .zoom {
   display: inline-block;
    width: 30%;
    margin-top: 10px;
  }
  .colorControl {
    display: inline-block;
    margin-left: 10px;
    margin-bottom: 30px;
  }
  .thresholdText {
    margin-left: 60px;
  }
  .zoomText {
    margin-left: 20px;
  }
  .beforVideo {
    background: #333;
  }
  /* #output {
    width: 800px;
    height: 448px;
  } */
</style>
