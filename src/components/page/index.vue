<template>
    <div>
        <!-- 单图片上传 -->
        <el-upload
            class="avatar-uploader"
            action="#"
            list-type="picture"
            :auto-upload="false"
            :show-file-list="false"
            :on-change="handleCrop"
            :http-request="upload"
        >
            <img
                v-if="imageUrl"
                :src="imageUrl"
                class="avatar"
                ref="singleImg"
                @mouseenter="mouseEnter"
                @mouseleave="mouseLeave"
                :style="{width:width+'px',height:height+'px'}"
            />
            <i
                v-else
                class="el-icon-plus avatar-uploader-icon"
                :style="{width:width+'px',height:height+'px','line-height':height+'px','font-size':height/6+'px'}"
            ></i>
            <div
                id="uploadIcon"
                v-if="imageUrl"
                ref="reupload"
                @mouseenter="mouseEnter"
                @mouseleave="mouseLeave"
                :style="{width:'100%'}"
            >
                <i
                    class="el-icon-zoom-in"
                    @click.stop="handlePreviewSingle"
                    :style="{color:'#2E2E2E',fontSize:'25px',display:'inline-block',paddingRight:'15px'}"
                ></i>
                <i
                    class="el-icon-upload"
                    :style="{color:'#2E2E2E',fontSize:'25px',display:'inline-block'}"
                ></i>
            </div>
        </el-upload>
        <!-- 剪裁组件弹窗 -->
        <el-dialog :visible.sync="cropperModel" width="1100px" :before-close="beforeClose">
            <Cropper
                :img-file="file"
                ref="vueCropper"
                :fixed-number="fixedNumber"
                :is-disabled="isDisabled"
                @upload="upload"
            ></Cropper>
        </el-dialog>
        <!-- 预览大图 -->
        <el-dialog :visible.sync="dialogVisible" width="800px">
            <img width="760px" :src="dialogImageUrl" />
        </el-dialog>
    </div>
</template>
<script>
import Cropper from './cropper';
// import axios from '@/assets/js/axios'
export default {
    name: 'index',
    props: {
        fixedNumber: {
            // 剪裁框比例设置
            default: function () {
                return [1, 1];
            }
        },
        width: {
            // 单图剪裁框宽度
            type: Number,
            default: 178
        },
        height: {
            // 单图剪裁框高度
            type: Number,
            default: 178
        }
    },
    data() {
        return {
            file: '', // 当前被选择的图片文件
            imageUrl: '', // 单图情况框内图片链接
            dialogImageUrl: '', // 多图情况弹窗内图片链接
            uploadList: [], // 上传图片列表
            reupload: true, // 控制"重新上传"开关
            dialogVisible: false, // 展示弹窗开关
            cropperModel: false, // 剪裁组件弹窗开关
            isDisabled: false
        };
    },
    updated() {
        if (this.$refs.vueCropper) {
            this.$refs.vueCropper.Update();
        }
    },
    mounted() {},
    methods: {
        /** **************************** single单图情况 **************************************/
        handlePreviewSingle(file) {
            // 点击进行图片展示
            console.log(this.file);
            this.dialogImageUrl = this.file.url;
            this.dialogVisible = true;
        },
        mouseEnter() {
            // 鼠标划入显示“重新上传”
            this.$refs.reupload.style.display = 'block';
            // if (this.$refs.failUpload.style.display === 'block') {
            //     this.$refs.failUpload.style.display = 'none';
            // }
            this.$refs.singleImg.style.opacity = '0.6';
        },
        mouseLeave() {
            // 鼠标划出隐藏“重新上传”
            this.$refs.reupload.style.display = 'none';
            this.$refs.singleImg.style.opacity = '1';
        },
        handleCrop(file, files) {
            // 点击弹出剪裁框
            this.cropperModel = true;
            this.file = file;
            // this.imageUrl = file.url
        },

        /************************************************************************************/

        async upload(data) {
            this.isDisabled = true;
            this.$axios({
                method: 'post',
                url: 'http://192.168.1.58:8082/ymzs/api/curriculum/addImageBase64',
                params: {
                    base64: data,
                    type: ""
                }
            })
                .then((res) => {
                    if (res.data.code == 0) {
                        this.imageUrl = res.data.data; 
                        this.cropperModel = false;
                        this.$message({
                            showClose: true,
                            message: res.data.msg,
                            type: 'success'
                        });
                    }
                })
                .catch((error) => {
                    console.log(error);
                })
                .finally(() => {
                    this.isDisabled = false;
                    this.cropperModel = false;
                    // this.imageUrl = this.file.url;
                });
        },

        beforeClose() {
            // this.uploadList.pop();
            this.cropperModel = false;
        }
        // 压缩图片
        // compress(img) {
        //     let canvas = document.createElement('canvas');
        //     let ctx = canvas.getContext('2d');
        //     // let initSize = img.src.length;
        //     let width = img.width;
        //     let height = img.height;
        //     canvas.width = width;
        //     canvas.height = height;
        //     // 铺底色
        //     ctx.fillStyle = '#fff';
        //     ctx.fillRect(0, 0, canvas.width, canvas.height);
        //     ctx.drawImage(img, 0, 0, width, height);
        //     // 进行压缩
        //     let ndata = canvas.toDataURL('image/jpeg', 0.8);
        //     return ndata;
        // },
        // // base64转成bolb对象
        // dataURItoBlob(base64Data) {
        //     let byteString;
        //     if (base64Data.split(',')[0].indexOf('base64') >= 0) {
        //         byteString = atob(base64Data.split(',')[1]);
        //     } else {
        //         byteString = unescape(base64Data.split(',')[1]);
        //     }
        //     let mimeString = base64Data.split(',')[0].split(':')[1].split(';')[0];
        //     let ia = new Uint8Array(byteString.length);
        //     for (let i = 0; i < byteString.length; i++) {
        //         ia[i] = byteString.charCodeAt(i);
        //     }
        //     return new Blob([ia], { type: mimeString });
        // }
    },
    components: {
        Cropper
    }
};
</script>
<style>
.avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
}
.avatar-uploader .el-upload:hover {
    border-color: #409eff;
}
.avatar-uploader-icon {
    color: #8c939d;
    text-align: center;
}
.avatar {
    display: block;
}
.reupload {
    border-radius: 50%;
    position: absolute;
    color: #fff;
    background-color: #000000;
    opacity: 0.6;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    display: none;
}
#uploadIcon {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    display: none;
}
</style>