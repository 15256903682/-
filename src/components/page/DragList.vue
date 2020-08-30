<template> 
  <div class="app-container">
    <el-tabs v-model="activeName" class="tabs" @tab-click="handleClick">
      <el-tab-pane label="账号信息" name="first" class="tabs1 is-active">
        <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
          <el-form-item label="头像" prop="head">
            <el-upload class="avatar-uploader" action="https://upload.qiniup.com" :show-file-list="false" :data="dataObj"
              :on-success="handleAvatarSuccess" :before-upload="beforeAvatarUpload">
              <img v-if="imageUrl" :src="imageUrl" class="avatar">
              <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            </el-upload>
          </el-form-item>
          <el-form-item label="管理员" prop="adminName">
            <el-input v-model="ruleForm.adminName"></el-input>
          </el-form-item>
          <el-form-item label="公司名" prop="companyName">
            <el-input v-model="ruleForm.companyName" disabled="disabled"></el-input>
          </el-form-item>
          <el-form-item label="邀请码" prop="inviteCode">
            <el-input v-model="ruleForm.inviteCode" disabled="disabled"></el-input>
          </el-form-item>
          <el-form-item label="联系电话" prop="adminPhone">
            <el-input v-model="ruleForm.adminPhone"></el-input>
          </el-form-item>
          <el-form-item label="电子邮件" prop="email">
            <el-input v-model="ruleForm.email"></el-input>
          </el-form-item>
          <el-form-item label="公司网址" prop="companyWeb">
            <el-input v-model="ruleForm.companyWeb"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="submitForm('ruleForm')">提交</el-button>
            <el-button @click="resetForm('ruleForm')">重置</el-button>
          </el-form-item>
        </el-form>
      </el-tab-pane>

      <el-tab-pane label="更改密码" name="Third" class="tabs1 is-active">
        <el-form :model="ruleForm2" :rules="rules" ref="ruleForm2" label-width="100px" class="demo-ruleForm">
          <el-form-item label="旧密码" prop="oldPwd">
            <el-input type="password" v-model="ruleForm2.oldPwd"></el-input>
          </el-form-item>
          <el-form-item label="新密码" prop="newPwd">
            <el-input type="password" v-model="ruleForm2.newPwd"></el-input>
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
        } else if (value !== this.ruleForm2.newPwd) {
          callback(new Error('两次输入密码不一致!'));
        } else {
          callback();
        }
      };
      return {
        imageUrl: '', //头像
        activeName: 'first',
        ruleForm: {
          adminName: '',
          companyName: '',
          inviteCode: '',
          adminPhone: '',
          email: '',
          companyWeb: '',
        },
        ruleForm2: {
          oldPwd: '',
          newPwd: '',
          checkPass: '',
        },
        qiniutoken: '', //获取七牛云的token
        qiniukey: '', //七牛云返回的key、
        token: {
          key: ''
        },
        dataObj: {
          token: '',
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
          adminPhone: [{
            required: true,
            message: '请输入正确的联系方式',
            trigger: 'blur'
          }, {
            min: 11,
            max: 11,
            message: '长度在11 个字符',
            trigger: 'blur'
          }],
          email: [{
            required: true,
            message: '请输入邮箱地址',
            trigger: 'blur'
          }, {
            type: 'email',
            message: '请输入正确的邮箱地址',
            trigger: ['blur', 'change']
          }],
          companyWeb: [{
            required: true,
            message: '公司网址不能为空',
            trigger: 'blur'
          }, ],

          oldPwd: [{
            required: true,
            message: '旧密码不能为空',
            trigger: 'blur'
          }, ],
          newPwd: [{
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
      var userToken = sessionStorage.getItem("userToken");
      if (userToken == null || userToken == '' || userToken == undefined) {
        sessionStorage.removeItem("userToken");
        sessionStorage.removeItem("UserId");
        this.$router.push('/login');
      }
      this.getToken(); //获取七牛云的token
      this.fnInfor(); //获取账号信息

    },
    methods: {
      //提交用户信息
      submitForm(formName) {
        var userToken = sessionStorage.getItem("userToken");
        this.ruleForm.token = userToken,
          this.$refs[formName].validate((valid) => {
            if (valid) {
              this.$axios({
                method: 'post',
                url: '/organization/updateAccountInfo',
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
        var userToken = sessionStorage.getItem("userToken");
        this.parmas = {
          token: userToken
        }
        this.$axios({
          method: 'post',
          url: '/organization/getOrganization',
          params: this.parmas
        }).then(res => {
          if (res.data.code == 0) {
            this.ruleForm.adminName = res.data.data.adminName;
            this.ruleForm.companyName = res.data.data.companyName;
            this.ruleForm.inviteCode = res.data.data.inviteCode;
            this.ruleForm.adminPhone = res.data.data.adminPhone;
            this.ruleForm.email = res.data.data.email;
            this.ruleForm.companyWeb = res.data.data.companyWeb;
            this.imageUrl = res.data.data.avatar
          }else if (res.data.code == 301) {
              sessionStorage.removeItem("userToken");
              sessionStorage.removeItem("UserId");
              this.$router.push('/login');
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
      },


      //提交密码
      submitForm2(formName) {
        var userToken = sessionStorage.getItem("userToken");
        this.ruleForm2.token = userToken,
        this.ruleForm2.newPwd = this.$md5(this.ruleForm2.newPwd)
        this.ruleForm2.oldPwd = this.$md5(this.ruleForm2.oldPwd)
        this.ruleForm2.checkPass = this.$md5(this.ruleForm2.checkPass)

        this.$refs[formName].validate((valid) => {
          if (valid) {
            this.$axios({
              method: 'post',
              url: '/organization/editPwd',
              params: this.ruleForm2
            }).then(res => {
              if (res.data.code == 0) {
                 this.$refs[formName].resetFields();
                  this.$message({
                    showClose: true,
                    message: '提交成功',
                    type: 'success'
                  });
              }else if (res.data.code == 301) {
                  sessionStorage.removeItem("userToken");
                  sessionStorage.removeItem("UserId");
                  this.$router.push('/login');
              }else{
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
        var uuid = new Date().getTime();
        this.dataObj.token = this.qiniutoken;
        this.dataObj.key = "portrait/" + uuid;
        const isJPG = file.type === 'image/jpeg';
        const isLt2M = file.size / 1024 / 1024 < 2;
        if (!isJPG) {
          this.$message.error('上传头像图片只能是 JPG 格式!');
        }
        if (!isLt2M) {
          this.$message.error('上传头像图片大小不能超过 2MB!');
        }
        return isJPG && isLt2M;
      },
      handleAvatarSuccess(res, file) {
        this.imageUrl = URL.createObjectURL(file.raw);
        this.qiniukey = res.key;
        // console.log(this.qiniukey)
        this.fnAvater(this.qiniukey)
      },
      //上传头像到服务器
      fnAvater(key) {
        var userToken = sessionStorage.getItem("userToken");
        var url = 'http://img.1mei.vip/' + key
        this.$axios({
          method: 'post',
          url: '/user/editAvatar',
          params: {
            avatar: url,
            token: userToken
          }
        }).then(res => {
          if (res.data.code == 0) {
              alert("头像修改成功!")

          }else if (res.data.code == 301) {
              sessionStorage.removeItem("userToken");
              sessionStorage.removeItem("UserId");
              this.$router.push('/login');
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
      },
      handleClick(tab, event) {
        if (event.target.getAttribute('id') == "tab-second") {} else {}
      },
    }
  }
</script>

<style rel="stylesheet/scss"  scoped>
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
	width: 180px;
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
  	width: 180px;
  }
</style>
