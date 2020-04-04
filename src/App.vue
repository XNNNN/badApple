<template>
    <div id="app">
        <div class="main">
            <canvas class="canvas" id="canvas"></canvas>
        </div>
        <div class="aside">
            <input id="img-upload" type="file" accept="video/*" @change="uploadGetInformation" />
            <video
                id="beforVideo"
                width="100%"
                height="40%"
                :src="videoAddress"
                :controls="controls"
            ></video>
            <div id="setting">
                <el-form ref="form" :model="form">
                    <el-form-item label="模式选择:">
                        <el-radio-group v-model="form.resource" @change="modeSelect">
                            <el-radio label="grayMode">灰色模式</el-radio>
                            <el-radio label="blackWhiteMode">黑白模式</el-radio>
                            <el-radio label="colorMode">彩色模式</el-radio>
                        </el-radio-group>
                    </el-form-item>
                </el-form>
                <div class="block">
                    <div>阈值:</div>
                    <el-slider v-model="thresholdRange" show-input></el-slider>
                    <div class="scaleText">缩放率:</div>
                    <el-slider v-model="scale" show-input></el-slider>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: "app",
    data() {
        return {
            videoAddress: undefined,
            controls: "controls",
            form: {
                resource: ""
            },
            thresholdRange: 0,
            scale: 0
        };
    },
    methods: {
        uploadGetInformation(e) {
            // 获取上传文件的信息
            const _this = this;
            let file = e.target.files[0];
            let reader = new FileReader();

            if (!file.type.match("video.*")) {
                this.$message({
                    message: "文件格式不对哦。",
                    type: "warning"
                });
                return;
            }
            reader.readAsDataURL(file);
            reader.onload = function(arg) {
                _this.videoAddress = arg.target.result;
                _this.animetionFrame();
            };
            // reader.paly = this.play()
        },
        animetionFrame() {
            let video = document.getElementById("beforVideo");
            let canvas = document.getElementById("canvas");
            let ctx = canvas.getContext("2d");
            const self = this;
            requestAnimationFrame(function() {
                ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
                let imgData = ctx.getImageData(
                    0,
                    0,
                    canvas.width,
                    canvas.height
                );
                self.grayProcess(imgData, canvas);

                ctx.putImageData(imgData, 0, 0);
                self.animetionFrame();
            });
        },
        modeSelect(val) {
            this.animetionFrame(val);
            // switch (val) {
            //   case 'grayMode':
            //     console.log('111111')
            //     break;
            //   case 'blackWhiteMode':
            //     console.log('22222')
            //     break;
            //   case 'colorMode':
            //     console.log('333333')
            //     break;
            // }
        },
        grayProcess(imgData, canvas) {
            let data = imgData.data;

            for (let h = 0; h < canvas.height; h++) {
                //canvas高
                for (let w = 0; w < canvas.width; w++) {
                    //canvas宽
                    //每个像素rgb的平均值
                    let avg = Math.floor(
                        (data[(h * canvas.width + w) * 4] +
                            data[(h * canvas.width + w) * 4 + 1] +
                            data[(h * canvas.width + w) * 4 + 2]) /
                            3
                    );
                    //每个像素的rgb值都设为平均值，达到中间灰效果
                    data[(h * canvas.width + w) * 4] = data[
                        (h * canvas.width + w) * 4 + 1
                    ] = data[(h * canvas.width + w) * 4 + 2] = avg;
                }
            }
        }
    }
};
</script>

<style>
html,
body {
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
    background-image: linear-gradient(135deg, #52e5e7 10%, #130cb7 100%);
    flex: 2;
    display: flex;
    align-items: center;
}
.aside {
    flex: 1;
}
#canvas {
    width: 100%;
    height: 70%;
}
.el-form-item__content {
    margin-top: 2rem;
    /* display: flex;
    justify-content: space-between; */
}
.el-form-item__label {
    margin-left: 1rem;
    font-size: 1rem;
}
.block {
    margin: 1rem 3rem 1rem 3rem;
    display: flex;
    flex-direction: column;
}
.scaleText {
    margin-top: 2rem;
}
</style>
