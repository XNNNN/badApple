<template>
  <div id="app">
    <div class="main">
      <canvas class="canvas" id="canvas"></canvas>
    </div>
    <div class="aside">
      <input id="img-upload" type="file" accept="video/*" @change="uploadGetInformation">
      <video
        id="beforVideo" 
        class="beforVideo" 
        width="100%"
        height="40%"
        :src="videoAddress" 
        :controls="controls"
      ></video>
    </div>
  </div>
</template>

<script>

export default {
  name: 'app',
  data () {
    return {
      videoAddress: undefined,
      controls: "controls"
    }
  },
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
        _this.videoAddress = arg.target.result
        _this.animetionFrame()
      }
      // reader.paly = this.play()
    },
    animetionFrame() {
      let video = document.getElementById('beforVideo') 
      let canvas = document.getElementById('canvas')
      let ctx = canvas.getContext('2d')
      const self = this
      requestAnimationFrame(function() {
        ctx.drawImage(video, 0, 0, canvas.width, canvas.height)
        var imgData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        var data = imgData.data;
        for(var h=0; h<canvas.height; h++){     //canvas高
            for(var w=0; w<canvas.width; w++){      //canvas宽
                //每个像素rgb的平均值
                var avg = Math.floor((data[(h*canvas.width+w)*4]+data[(h*canvas.width+w)*4+1]+data[(h*canvas.width+w)*4+2])/3);
                //每个像素的rgb值都设为平均值，达到中间灰效果
                data[(h*canvas.width+w)*4]=data[(h*canvas.width+w)*4+1]=data[(h*canvas.width+w)*4+2] = avg;
            }
        }
        ctx.putImageData(imgData, 0, 0)
        self.animetionFrame()
      })
      // let imageData = ctx.getImageData(0, 0, canvas.width, canvas.height)
      // ctx.putImageData(imageData, 0, 0)
      // this.animetionFrame()
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
  #app {
    display: flex;
    height: 100%;
  }
  .main {
    background-image: linear-gradient( 135deg, #52E5E7 10%, #130CB7 100%);
    flex: 2;
    display: flex;
    align-items: center;
  }
  .aside {
    flex: 1;
    border-color: #409EFF;
    border-style: solid;
    border-width: 0px 0px 0px 2px;
  }
  #canvas {
    width: 100%;
    height: 70%;
  }
</style>
