<template>
	<div>
	<div v-if="videoForm.showVideoPath ==''">
		<el-upload action="http://192.168.1.58:8082/ymzs/api/curriculum/addVideo" 
			v-bind:data="{id:27}" 
			v-bind:on-progress="uploadVideoProcess"
			v-bind:on-success="handleVideoSuccess" 
			v-bind:before-upload="beforeUploadVideo" 
			v-bind:show-file-list="false">
			<el-button size="small" type="primary">点击上传</el-button>
		</el-upload>
	</div>
	<video src="https://oss.shal.club/curriculum/video/09e3ecc2-0f7e-4711-b94e-d7ca670216b4.mp4" class="avatar video-avatar"
	      controls="controls">
	</video>
	</div>
</template>


<script>
	export default {
		data() {
			return {
				//参数
				videoFlag: false,
				//是否显示进度条
				videoUploadPercent: "",
				//进度条的进度，
				isShowUploadVideo: false,
				//显示上传按钮
				videoForm: {
					showVideoPath: ''
				},
			}
		},
		methods: {
			beforeUploadVideo(file) {
				var fileSize = file.size / 1024 / 1024 < 5;
				if (['video/mp4', 'video/ogg', 'video/flv', 'video/avi', 'video/wmv', 'video/rmvb', 'video/mov'].indexOf(file.type) ==
					-1) {
					this.$alert('请上传正确的视频格式', '提示1', {
						confirmButtonText: '确定',
						callback: action => {
							this.$message({
								type: 'info',
								message: `请上传正确的格式`
							});
						}
					});
					return false;
				}
				if (!fileSize) {
					this.$alert('视频大小不能超过5MB', '提示2', {
						confirmButtonText: '确定',
						callback: action => {
							this.$message({
								type: 'info',
								message: `视频大小不超过5MB`
							});
						}
					});
					return false;
				}
				this.isShowUploadVideo = false;
			},
			//进度条
			uploadVideoProcess(event, file, fileList) {
				this.videoFlag = true;
				this.videoUploadPercent = file.percentage.toFixed(0) * 1;
			},
			//上传成功回调
			handleVideoSuccess(res, file) {
				
				this.isShowUploadVideo = true;
				this.videoFlag = false;
				this.videoUploadPercent = 0;
				//后台上传地址
				if (res.code == 1) {
					console.log('执行了', res.data.image_url)
					this.curgimg = res.data.image_url
					this.videoForm.showVideoPath = (this.httpurl + res.data.image_url);
				} else {
					this.$alert("是否上传", '提示', {
						confirmButtonText: '确定',
						callback: action => {
							this.$message({
								type: 'info',
								message: `上传成功`
							});
						}
					});
				}
			},
		}
	}
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
		border-color: #409EFF;
	}

	.avatar-uploader-icon {
		font-size: 28px;
		color: #8c939d;
		width: 178px;
		height: 178px;
		line-height: 178px;
		text-align: center;
	}

	.avatar {
		width: 178px;
		height: 178px;
		display: block;
	}
	.el-upload--text{
		width: 80px;
		height: 32px;
		border:1px solid #fff;
	}
</style>