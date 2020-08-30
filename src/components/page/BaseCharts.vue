<template> 
	<div class="app-container">
		<el-tabs v-model="activeName" class="tabs" @tab-click="handleClick">
			<el-tab-pane label="账号信息" name="first" class="tabs1 is-active">
				<el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
					<el-form-item label="头像" prop="img">
						<el-upload class="avatar-uploader" v-model="ruleForm.img" action="http://192.168.1.58:8082/ymzs/api/curriculum/addImage"
						 :show-file-list="false" :on-success="handleAvatarSuccess" :before-upload="beforeAvatarUpload">
							<img v-if="imageUrl" :src="imageUrl" class="avatar">
							<i v-else class="el-icon-plus avatar-uploader-icon"></i>
						</el-upload>
					</el-form-item>
					<el-form-item label="昵称" prop="name">
						<el-input v-model="ruleForm.name"></el-input>
					</el-form-item>

					<el-form-item>
						<el-button type="primary" @click="submitForm('ruleForm')">提交</el-button>
						<el-button @click="resetForm('ruleForm')">重置</el-button>
					</el-form-item>
				</el-form>
			</el-tab-pane>

			<el-tab-pane label="更改密码" name="Third" class="tabs1 is-active">
				<el-form :model="ruleForm2" :rules="rules" ref="ruleForm2" label-width="100px" class="demo-ruleForm">
					<el-form-item label="旧密码" prop="oldPassword">
						<el-input type="password" v-model="ruleForm2.oldPassword"></el-input>
					</el-form-item>
					<el-form-item label="新密码" prop="newPassword">
						<el-input type="password" v-model="ruleForm2.newPassword"></el-input>
					</el-form-item>
					<el-form-item label="确认密码" prop="checkPass">
						<el-input type="password" v-model="ruleForm2.checkPass"></el-input>
					</el-form-item>
					<el-form-item>
						<el-button type="primary" @click="submitForm2('ruleForm2')">提交</el-button>
						<el-button @click="resetForm('ruleForm2')">重置</el-button>
					</el-form-item>
				</el-form>
			</el-tab-pane>
		</el-tabs>

	</div>
</template>
<script>
	export default {
		data() {
			const validateUsername = (rule, value, callback) => {
				if (value && (!(/^[1][34578]\d{9}$/).test(value) || !(/^[1-9]\d*$/).test(value) || value.length !== 11)) {
					callback(new Error('手机号码不符合规范'))
				} else {
					callback()
				}
			};
			var validatePass = (rule, value, callback) => {
				if (value === '') {
					callback(new Error('请输入密码'));
				} else {
					if (this.ruleForm2.checkPass !== '') {
						this.$refs.ruleForm2.validateField('checkPass');
					}
					callback();
				}
			};
			var validatePass2 = (rule, value, callback) => {
				if (value === '') {
					callback(new Error('请再次输入密码'));
				} else if (value !== this.ruleForm2.newPassword) {
					callback(new Error('两次输入密码不一致!'));
				} else {
					callback();
				}
			};
			return {
				imageUrl: '', //头像
				activeName: 'first',
				ruleForm: {
					name: '',
					img: ""
				},
				ruleForm2: {
					oldPassword: '',
					newPassword: '',
					checkPass: '',
				},
				qiniutoken: '', //获取七牛云的token
				qiniukey: '', //七牛云返回的key、
				token: {
					key: ''
				},
				rules: {
					adminName: [{
						required: true,
						message: '请输入活动名称',
						trigger: 'blur'
					}, {
						min: 2,
						max: 5,
						message: '长度在 2 到 5 个字符',
						trigger: 'blur'
					}],

					oldPassword: [{
						required: true,
						message: '旧密码不能为空',
						trigger: 'blur'
					}, ],
					newPassword: [{
						required: true,
						validator: validatePass,
						trigger: 'blur'
					}, ],
					checkPass: [{
						required: true,
						validator: validatePass2,
						trigger: 'blur'
					}, ],
				}
			};
		},
		mounted() {
			var userToken = localStorage.getItem("userToken");
			// if (userToken == null || userToken == '' || userToken == undefined) {
			//   sessionStorage.removeItem("userToken");
			//   sessionStorage.removeItem("UserId");
			//   this.$router.push('/login');
			// }
			// this.getToken(); //获取七牛云的token
			this.fnInfor(); //获取账号信息

		},
		methods: {
			//提交用户信息
			submitForm(formName) {
				var userToken = localStorage.getItem("userToken");
				this.ruleForm.token = userToken,
					this.$refs[formName].validate((valid) => {
						if (valid) {
							this.$axios({
								method: 'post',
								url: '/ymzs/api/curriculum/updateBasicInf',
								params: this.ruleForm
							}).then(res => {
								if (res.data.code == 0) {
									this.$message({
										showClose: true,
										message: '提交成功',
										type: 'success'
									});
								} else if (res.data.code == 301) {
									sessionStorage.removeItem("userToken");
									sessionStorage.removeItem("UserId");
									this.$router.push('/login');
								} else {
									this.$message({
										showClose: true,
										message: res.data.msg,
										type: 'error'
									});
								}
							}).catch(error => {
								console.log(error);
							});
						} else {
							console.log('error submit!!');
							return false;
						}
					});
			},
			//获取基本信息
			fnInfor() {
				var userToken = localStorage.getItem("userToken");
				this.parmas = {
					token: userToken
				}
				this.$axios({
					method: 'post',
					url: '/ymzs/api/curriculum/selBasicInf',
					params: this.parmas
				}).then(res => {
					if (res.data.code == 0) {
						this.ruleForm.name = res.data.data.name;
						this.imageUrl = res.data.data.avatar
					} else if (res.data.code == 301) {
						// sessionStorage.removeItem("userToken");
						// sessionStorage.removeItem("UserId");
						// this.$router.push('/login');
					} else {
						this.$message({
							showClose: true,
							message: res.data.msg,
							type: 'error'
						});
					}
				}).catch(error => {
					console.log(error);
				});
			},


			//提交密码
			submitForm2(formName) {
				var userToken = localStorage.getItem("userToken");
				this.ruleForm2.token = userToken,
					// this.ruleForm2.newPwd = this.$md5(this.ruleForm2.newPwd)
					// this.ruleForm2.oldPwd = this.$md5(this.ruleForm2.oldPwd)
					// this.ruleForm2.checkPass = this.$md5(this.ruleForm2.checkPass)

					this.$refs[formName].validate((valid) => {
						if (valid) {
							this.$axios({
								method: 'post',
								url: '/ymzs/api/user/upPassword',
								params: this.ruleForm2
							}).then(res => {
								if (res.data.code == 0) {
									this.$refs[formName].resetFields();
									this.$message({
										showClose: true,
										message: '提交成功',
										type: 'success'
									});
								} else if (res.data.code == 301) {
									sessionStorage.removeItem("userToken");
									sessionStorage.removeItem("UserId");
									this.$router.push('/login');
								} else {
									this.$refs[formName].resetFields();
									this.$message({
										showClose: true,
										message: res.data.msg,
										type: 'error'
									});
								}
							}).catch(error => {
								console.log(error);
							});

						} else {
							console.log('error submit!!');
							return false;
						}
					});
			},
			//重置
			resetForm(formName) {
				this.$refs[formName].resetFields();
			},
			getToken() {
				this.$axios({
					method: 'get',
					url: '/file/uptoken',
					params: this.token
				}).then(res => {
					if (res.data.code == 0) {
						this.qiniutoken = res.data.msg
					}
				}).catch(error => {
					console.log(error);
				});
			},
			beforeAvatarUpload(file) {
				// const isJPG = file.type === 'image/jpeg';
				// const isPNG = file.type === 'image/png';
				const isLt2M = file.size / 1024 / 1024 < 2;
				// if (!isJPG ) {
				//   this.$message.error('上传图片必须是JPG/PNG 格式!');
				// }
				if (!isLt2M) {
					this.$message.error('上传头像图片大小不能超过 2MB!');
				}
				return isLt2M;
			},
			handleAvatarSuccess(res, file) {
				this.imageUrl = URL.createObjectURL(file.raw);
				this.ruleForm.img = res.data;
			},

			handleClick(tab, event) {
				if (event.target.getAttribute('id') == "tab-second") {} else {}
			},
		}
	}
</script>

<style>
	.app-container {
		
		padding-left: 10%;
	}

	.dataclass {
		width: 100%;
		height: 30px;
		font-size: 18px;
		color: #333;
		font-weight: bold;

	}


	.avatar-uploader .el-upload {
		border: 1px dashed #d9d9d9;
		border-radius: 6px;
		cursor: pointer;
		position: relative;
		overflow: hidden;
		width: 180px
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
		border: 1px solid #ddd;
	}

	.avatar {
		width: 178px;
		height: 178px;
		display: block;
	}
</style>
