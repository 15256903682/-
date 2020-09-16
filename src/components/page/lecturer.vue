<template>
    <div>
        <el-button
            @click="Newbuild(1)"
            type="primary"
            style="margin-left: 16px;margin:10px 0 20px"
        >新建讲师</el-button>
        <el-form :inline="true" :model="page" class="demo-form-inline">
            <el-form-item label="讲师姓名">
                <el-input v-model="page.lecturerName" placeholder="讲师姓名"></el-input>
            </el-form-item>
            <el-form-item>
                <el-button type="primary" @click="onSubmit()">查询</el-button>
                <el-button @click="resetForm1('page')">重置</el-button>
            </el-form-item>
        </el-form>
        <template>
            <el-table :data="tableData" border style="width: 100%">
                <el-table-column prop="id" label="编号" width="80" align="center">
                    <template slot-scope="scope">
                        <p>{{scope.$index}}</p>
                    </template>
                </el-table-column>
                <el-table-column prop="lecturerName" label="讲师姓名" width="180" align="center"></el-table-column>
                <el-table-column prop="lecturerLabel" label="讲师标签" width="180" align="center"></el-table-column>
                <el-table-column prop="ymExperience" label="医美经验" align="center"></el-table-column>
                <el-table-column prop="courseCharacteristic" label="课程特色" align="center"></el-table-column>
                <el-table-column prop="workingTime" label="从业时长" align="center"></el-table-column>
                <el-table-column prop="lecturerAvatar" label="讲师头像" align="center">
                    <template slot-scope="scope">
                        <el-image
                            class="table-td-thumb"
                            :src="scope.row.lecturerAvatar"
                            :preview-src-list="[scope.row.lecturerAvatar]"
                        ></el-image>
                    </template>
                </el-table-column>
                <el-table-column prop="personalAdvantage" label="个人优势" align="center"></el-table-column>
                <el-table-column prop="Remark" label="备注" align="center">
                    <template slot-scope="scope">
                        <p>{{tableData[scope.$index].remark||'暂无'}}</p>
                    </template>
                </el-table-column>
                <el-table-column fixed="right" label="操作" width="150">
                    <template slot-scope="scope">
                        <el-button type="text" size="small" @click="Newbuild(2,scope.row)">编辑</el-button>
                        <el-button
                            type="text"
                            size="small"
                            @click="fnMainDelect(scope.$index,scope.row.id,tableData)"
                        >删除</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <el-pagination
                background
                layout="prev, pager, next, sizes, total, jumper"
                :page-sizes="[5, 10, 15, 20]"
                :page-size="page.pageSize"
                :total="total"
                @current-change="handleCurrentChange"
                @size-change="handleSizeChange"
            ></el-pagination>
        </template>
        <el-dialog :title="title" :visible.sync="dialogFormVisible">
            <el-form
                :model="ruleForm"
                :rules="rules"
                ref="ruleForm"
                label-width="100px"
                class="demo-ruleForm"
            >
                <el-form-item label="讲师姓名" prop="lecturerName">
                    <el-input v-model="ruleForm.lecturerName"></el-input>
                </el-form-item>
                <el-form-item label="讲师标签" prop="lecturerLabel">
                    <el-input v-model="ruleForm.lecturerLabel"></el-input>
                </el-form-item>
                <el-form-item label="医美经验" prop="ymExperience">
                    <el-input v-model="ruleForm.ymExperience"></el-input>
                </el-form-item>
                <el-form-item label="课程特色" prop="courseCharacteristic">
                    <el-input v-model="ruleForm.courseCharacteristic"></el-input>
                </el-form-item>
                <el-form-item label="从业时长" prop="workingTime">
                    <el-input v-model="ruleForm.workingTime"></el-input>
                </el-form-item>

                <el-form-item label="讲师头像" prop="lecturerAvatar">
                    <el-upload
                        class="avatar-uploader"
                        action="http://api.11mei.cn/ymzs/api/curriculum/addImage"
                        :show-file-list="false"
                        :on-success="handleAvatarSuccess"
                        :before-upload="beforeAvatarUpload"
                    >
                    <img
                        v-if="ruleForm.lecturerAvatar"
                        :src="ruleForm.lecturerAvatar"
                        class="avatar"
                    />
                    <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                    </el-upload>
                </el-form-item>
                <el-form-item label="个人优势" prop="personalAdvantage">
                    <el-input v-model="ruleForm.personalAdvantage"></el-input>
                </el-form-item>
                <el-form-item label="备注" prop="Remark">
                    <el-input v-model="ruleForm.Remark"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="submitForm('ruleForm')">{{CreateOrModify}}</el-button>
                    <el-button @click="resetForm('ruleForm')">重置</el-button>
                </el-form-item>
            </el-form>
        </el-dialog>
        <el-dialog title="提示" :visible.sync="centerDialogVisible" width="20%" center>
            <span>确定删除该讲师？</span>
            <span slot="footer" class="dialog-footer">
                <el-button @click="centerDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="fnDefine()">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    name: 'Lecturer',
    data() {
        return {
            dialogFormVisible: false,
            centerDialogVisible: false,
            title: '新建讲师',
            CreateOrModify: '立即创建',
            tableData: [],
            total: null,
            ruleForm: {
                id: '',
                lecturerName: '',
                lecturerLabel: '',
                ymExperience: '',
                courseCharacteristic: '',
                workingTime: '',
                lecturerAvatar: '',
                personalAdvantage: '',
                Remark: '',
                token: ''
            },
            page: {
                pageSize: 10,
                pageNum: 1,
                token: '',
                lecturerName: ''
            },
            delect: {
                index: '',
                id: '',
                rows: ''
            },
            rules: {
                lecturerLabel: [{ required: true, message: '请输入讲师标签', trigger: 'blur' }],
                ymExperience: [{ required: true, message: '请输入医美经验', trigger: 'blur' }],
                courseCharacteristic: [{ required: true, message: '请输入课程特色', trigger: 'blur' }],
                workingTime: [{ required: true, message: '请输入从业时长', trigger: 'blur' }],
                lecturerName: [{ required: true, message: '请输入讲师姓名', trigger: 'blur' }],
                lecturerAvatar: [{ required: true, message: '请上传讲师头像', trigger: 'blur' }]
            }
        };
    },
    mounted() {
        this.selLecturerList();
    },
    methods: {
        Newbuild(el, arr) {
            if (el == 1) {
                this.dialogFormVisible = true;
                this.title = '新增讲师';
                this.ruleForm = {}; //清空ruleForm的值
                this.CreateOrModify = '立即创建';
            } else {
                this.dialogFormVisible = true; //打开弹框
                this.title = '编辑讲师';
                this.CreateOrModify = '立即修改';
                this.ruleForm.id = arr.id;
                this.ruleForm.lecturerName = arr.lecturerName;
                this.ruleForm.lecturerLabel = arr.lecturerLabel;
                this.ruleForm.ymExperience = arr.ymExperience;
                this.ruleForm.courseCharacteristic = arr.courseCharacteristic;
                this.ruleForm.workingTime = arr.workingTime;
                this.ruleForm.lecturerAvatar = arr.lecturerAvatar;
                this.ruleForm.personalAdvantage = arr.personalAdvantage;
                this.ruleForm.Remark = arr.remark;
                this.ruleForm.token = arr.token;
            }
        },
        selLecturerList() {
            var userToken = localStorage.getItem('userToken');
            (this.page.token = userToken),
                this.$axios({
                    method: 'post',
                    url: '/ymzs/api/introduction/selLecturerList',
                    params: this.page
                })
                    .then((res) => {
                        if (res.data.code == 0) {
                            this.tableData = res.data.data.data;
                            this.total = res.data.data.num;
                        }
                    })
                    .catch((error) => {
                        console.log(error);
                    });
        },
        //搜索
        onSubmit() {
            this.selLecturerList();
        },
        handleAvatarSuccess(res, file) {
            this.ruleForm.lecturerAvatar = URL.createObjectURL(file.raw);
            this.ruleForm.lecturerAvatar = res.data;
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
        submitForm(formName) {
            if (this.CreateOrModify == '立即创建') {
                var userToken = localStorage.getItem('userToken');
                (this.ruleForm.token = userToken),
                    this.$refs[formName].validate((valid) => {
                        if (valid) {
                            this.$axios({
                                method: 'post',
                                url: '/ymzs/api/introduction/insertLecturer',
                                params: this.ruleForm
                            })
                            .then((res) => {
                                if (res.data.code == 0) {
                                    this.dialogFormVisible = false;
                                    this.$refs[formName].resetFields();
                                    this.selLecturerList();
                                }
                            })
                            .catch((error) => {
                                console.log(error);
                            });
                        } else {
                            console.log('error submit!!');
                            return false;
                        }
                    });
            } else {
                var userToken = localStorage.getItem('userToken');
                (this.ruleForm.token = userToken),
                    this.$refs[formName].validate((valid) => {
                        if (valid) {
                            this.$axios({
                                method: 'post',
                                url: 'ymzs/api/introduction/updateLecturer',
                                params: this.ruleForm
                            })
                                .then((res) => {
                                    if (res.data.code == 0) {
                                        this.dialogFormVisible = false;
                                        this.$refs[formName].resetFields();
                                        this.selLecturerList();
                                    }
                                })
                                .catch((error) => {
                                    console.log(error);
                                });
                        } else {
                            console.log('error submit!!');
                            return false;
                        }
                    });
            }
        },
        //删除课程每一章的记录
        fnMainDelect(index, id, rows) {
            this.centerDialogVisible = true;
            this.delect.index = index;
            this.delect.id = id;
            this.delect.rows = rows;
        },
        //删除确定按钮
        fnDefine() {
            // console.log(this.delect.index+"||"+this.delect.id+"||"+JSON.stringify(this.delect.rows) )
            var userToken = localStorage.getItem('userToken');
            this.parmas = {
                token: userToken,
                id: this.delect.id
            };
            this.$axios({
                method: 'post',
                url: '/ymzs/api/introduction/delLecturer',
                params: this.parmas
            })
            .then((res) => {
                if (res.data.code == 0) {
                    this.delect.rows.splice(this.delect.index, 1);
                    this.centerDialogVisible = false;
                    this.$message({
                        showClose: true,
                        message: res.data.msg,
                        type: 'success'
                    });
                } else if (res.data.code == 301) {
                    localStorage.removeItem('userToken');
                    localStorage.removeItem('UserId');
                    this.$router.push('/login');
                } else {
                    this.$message({
                        showClose: true,
                        message: res.data.msg,
                        type: 'error'
                    });
                }
            })
            .catch((error) => {
                console.log(error);
            });
        },
        // 获取课程的列表
        handleCurrentChange(cpage) {
            this.page.pageNum = cpage;
            this.selLecturerList();
        },
        handleSizeChange(psize) {
            this.page.pageSize = psize;
            this.selLecturerList();
        },
        resetForm(formName) {
            this.$refs[formName].resetFields();
        },
        resetForm1(formName) {
            this.$refs[formName].resetFields();
        }
    }
};
</script>
<style rel="stylesheet/scss" scoped>
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
    margin: 0 auto;
}
.table-td-thumb {
    height: 62px;
    width: 62px;
}
</style>  