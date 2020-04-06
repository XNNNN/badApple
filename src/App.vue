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
                            <el-radio label="0">灰色模式</el-radio>
                            <el-radio label="1">黑白模式</el-radio>
                            <!-- <el-radio label="2">彩色模式</el-radio> -->
                        </el-radio-group>
                    </el-form-item>
                </el-form>
                <div class="block">
                    <div>阈值:</div>
                    <el-slider v-model="thresholdRange" show-input :max="255"></el-slider>
                    <!-- <div class="scaleText">缩放率:</div>
                    <el-slider v-model="scale" show-input></el-slider>-->
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
            thresholdRange: 128,
            scale: 0
        };
    },
    methods: {
        uploadGetInformation(e) {
            // 获取上传文件的信息
            const self = this;
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
                self.videoAddress = arg.target.result;
                self.animetionFrame();
            };
            // reader.paly = this.play()
        },
        animetionFrame(mode) {
            let video = document.getElementById("beforVideo");
            let canvas = document.getElementById("canvas");
            let ctx = canvas.getContext("2d");
            const self = this;
            let scale = self.scale === 0 ? 1 : self.scale * 0.01;
            let a = 0;
            let b = 0;
            console.log(video.videoWidth);
            a = canvas.width * scale;
            b = canvas.height * scale;
            requestAnimationFrame(function() {
                ctx.drawImage(video, 0, 0, a / scale, b / scale);
                console.log(canvas.height / scale);
                let imgData = ctx.getImageData(
                    0,
                    0,
                    canvas.width * scale,
                    canvas.height * scale
                );
                ctx.imageSmoothingEnabled = false;
                ctx.mozImageSmoothingEnabled = false;
                ctx.webkitImageSmoothingEnabled = false;
                ctx.msImageSmoothingEnabled = false;
                switch (mode) {
                    case "0": //灰色模式
                        self.grayProcess(imgData, canvas);
                        break;
                    case "1": //黑白模式
                        self.colorAndBlackWriteMode(
                            imgData,
                            self.thresholdRange,
                            mode
                        );
                        break;
                    case "2": // 彩色模式
                        self.colorAndBlackWriteMode(
                            imgData,
                            self.thresholdRange,
                            mode
                        );
                        break;
                }
                ctx.putImageData(imgData, 0, 0);
                self.animetionFrame(mode);
            });
        },
        modeSelect(mode) {
            this.animetionFrame(mode);
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
        },
        colorAndBlackWriteMode(imgData, threshold, mode) {
            let data = imgData.data;
            mode = Number(mode);
            for (let i = 0; i < data.length; i += 4) {
                let red = data[i];
                let green = data[i + 1];
                let blue = data[i + 2];
                let alpha = data[i + 3];

                // 灰度计算公式
                let gray =
                    0.299 * data[i] + 0.587 * data[i + 1] + 0.114 * data[i + 2];

                let color = gray >= threshold ? 255 : 0;

                data[i] = mode == 2 && color == 2 ? red : color; // red
                data[i + 1] = mode == 2 && color == 2 ? green : color; // green
                data[i + 2] = mode == 2 && color == 2 ? blue : color; // blue
                data[i + 3] = alpha >= threshold ? 255 : 0; // 去掉透明
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
