
<template>
	<div class="login-wrap">
		<div class="ms-login">
			<h2 class="ms-title">后台管理系统</h2>
			<!-- <h2 class="login-title color-main text-gradient">一美助手课程后台管理</h2> -->
			<el-form :model="param" :rules="rules" ref="login" label-width="0px" class="ms-content">
				<el-form-item prop="phone">
					<el-input v-model="param.phone" placeholder="用户名">
						<el-button slot="prepend" icon="el-icon-lx-people"></el-button>
					</el-input>
				</el-form-item>
				<el-form-item prop="password">
					<el-input type="password" placeholder="密码" v-model="param.password" @keyup.enter.native="submitForm()">
						<el-button slot="prepend" icon="el-icon-lx-lock"></el-button>
					</el-input>
				</el-form-item>
				<el-form-item prop="code">
					<el-input placeholder="请输入验证码" v-model="param.code" @keyup.enter.native="submitForm()" style="width:60%">
						<el-button slot="prepend" icon="el-icon-edit-outline"></el-button>
					</el-input>
					<img :src="captchaImg" @click="captchaClick"
					 class="graphical">
				</el-form-item>
				<el-form-item style="margin-bottom: 40px">
					<el-button style="width: 100%" type="primary" :loading="loading" @click="submitForm()">
						登录
					</el-button>
				</el-form-item>
			</el-form>
		</div>
	</div>
</template>

<script>
	export default {
		data: function() {
			return {
				param: {
					phone: '18508440083',
					password: '',
					code:""
				},
				rules: {
					phone : [{
						required: true,
						message: '请输入用户名',
						trigger: 'blur'
					}],
					password: [{
						required: true,
						message: '请输入密码',
						trigger: 'blur'
					}],
				},
				loading: false,
				captchaImg:"",
				uuid:""
			};
		},
		mounted(){
			this.captchaClick();//获取图形验证码
		},
		methods: {
			captchaClick() {
				this.$axios({
					method: 'post',
					url: '/ymzs/api/common/getVerify',
					params: {}
				}).then(res => {
					if (res.data.code == 0) {
						this.captchaImg= res.data.data;
						this.uuid = res.data.msg;
					}
				}).catch(error => {
					console.log(error);
				});
			},
			submitForm() {
				this.$refs.login.validate(valid => {
					if (valid) {
						// this.param.uuid = this.uuid;
						this.$axios({
							method: 'post',
							url: '/ymzs/api/curriculum/login',
							params: this.param
						}).then(res => {
							if (res.data.code == 0) {
							   this.$message({
							     showClose: true,
							     message: '登录成功',
							     type: 'success'
							   });
							   localStorage.setItem("userToken", res.data.data); //用户token
							   localStorage.setItem('ms_username', this.param.phone);
							   this.$router.push('/');
							  
							}else{
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
						this.$message.error('请输入账号和密码');
						console.log('error submit!!');
						return false;
					}
				});
			},
			
		},
	};
</script>

<style scoped>
	.login-wrap {
		position: relative;
		width: 100%;
		height: 100%;
		background-image: url(../../assets/img/login-bg.jpg);
		background-size: 100%;
	}

	.ms-title {
		width: 100%;
		line-height: 50px;
		text-align: center;
		font-size: 20px;
		color: #fff;
		border-bottom: 1px solid #ddd;
		text-align: center;
		color: #409EFF;
		font-family: '微软雅黑';
		background-image: -webkit-gradient(linear, 0 0, 0 bottom, from(rgba(36, 208, 169, 1)), to(rgba(92, 137, 250, 1)));
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
	}

	.ms-login {
		position: absolute;
		left: 50%;
		top: 50%;
		width: 350px;
		margin: -190px 0 0 -175px;
		border-radius: 5px;
		background: rgba(255, 255, 255, 0.3);
		overflow: hidden;
	}

	.ms-content {
		padding: 30px 30px;
	}

	.login-btn {
		text-align: center;
	}

	.login-btn button {
		width: 100%;
		height: 36px;
		margin-bottom: 10px;
	}

	.login-tips {
		font-size: 12px;
		line-height: 30px;
		color: #fff;
	}

	.el-button--primary {
		color: #FFF;
		background-color: #29DFD0;
		border-color: #29DFD0;
		background-image: linear-gradient(#0BD6E9, #409EFF);
	}

	.graphical {
		width: 40%;
		height: 32px;
	}
</style>
