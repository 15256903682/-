<template>
    <div>
        <el-button
            @click="fnFound(1)"
            type="primary"
            style="margin-left: 16px;margin:10px 0 20px"
        >创建课程</el-button>

        <el-table
            ref="classTable"
            :data="tableData"
            style="width: 100%"
            @expand-change="handleExpandChange"
        >
            <el-table-column type="expand">
                <template slot-scope="props">
                    <el-table :data="props.row.child" style="width: 100%">
                        <el-table-column label="课程章节" align="center">
                            <template slot-scope="scope">{{ scope.row.curriculumName }}</template>
                        </el-table-column>
                        <el-table-column label="视频上传" align="center">
                            <template slot-scope="scope">
                                <video
                                    v-if="scope.row.curriculumVideo"
                                    :src="scope.row.curriculumVideo"
                                    class="video-avatar"
                                    controls="controls"
                                ></video>
                                <div v-else>
                                    <el-upload
                                        action="http://192.168.1.58:8082/ymzs/api/curriculum/addVideo"
                                        v-bind:data="{id:scope.row.id}"
                                        v-bind:on-progress="uploadVideoProcess"
                                        v-bind:on-success="handleVideoSuccess"
                                        v-bind:before-upload="beforeUploadVideo.bind(this, scope.$index)"
                                        v-bind:show-file-list="false"
                                    >
                                        <el-button size="small" type="primary">点击上传</el-button>
                                    </el-upload>
                                </div>
                                <el-progress
                                    v-if="videoFlag == true && videoIndex ===scope.$index"
                                    type="circle"
                                    v-bind:percentage="videoUploadPercent"
                                    style="margin-top:7px;"
                                ></el-progress>
                            </template>
                        </el-table-column>

                        <el-table-column label="是否免费试看" align="center">
                            <template slot-scope="scope">{{scope.row.isFree === "true" ? '是' : '否'}}</template>
                        </el-table-column>
                        <el-table-column fixed="right" label="操作" width="150">
                            <template slot-scope="scope">
                                <el-button type="text" size="small" @click="fnAdd(2,scope.row)">编辑</el-button>
                                <el-button
                                    type="text"
                                    size="small"
                                    @click="fnSecondDelect(scope.$index,scope.row.id, props.row.child)"
                                >删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </template>
            </el-table-column>

            <el-table-column label="课程名称" align="center">
                <template slot-scope="scope">{{scope.row.parent.curriculumName }}</template>
            </el-table-column>
            <el-table-column label="讲师姓名" align="center">
                <template slot-scope="scope">{{ scope.row.parent.lecturerName ||"暂无"}}</template>
            </el-table-column>
            <el-table-column label="课程类型" align="center">
                <template slot-scope="scope">
                    <p v-if="tableData[scope.$index].parent.type == 1">医美咨询</p>
                    <p v-if="tableData[scope.$index].parent.type == 2">医美营销</p>
                    <p v-if="tableData[scope.$index].parent.type == 3">渠道开发</p>
                    <p v-if="tableData[scope.$index].parent.type == 4">运营管理</p>
                </template>
            </el-table-column>
            <el-table-column label="是否置顶" align="center" width="80">
                <template slot-scope="scope">
                    <p v-if="tableData[scope.$index].parent.isTop  == 0">否</p>
                    <p v-if="tableData[scope.$index].parent.isTop  == 1">是</p>
                </template>
            </el-table-column>
            <el-table-column label="综合推荐" align="center" width="80">
                <template slot-scope="scope">
                    <p v-if="tableData[scope.$index].parent.isRecommend  == 0">否</p>
                    <p v-if="tableData[scope.$index].parent.isRecommend  == 1">是</p>
                </template>
            </el-table-column>
            <el-table-column label="初始播放量" align="center">
                <template slot-scope="scope">{{scope.row.parent.viewCounts }}</template>
            </el-table-column>
            <el-table-column label="视频封面" align="center">
                <template slot-scope="scope">
                    <el-image
                        class="table-td-thumb"
                        :src="scope.row.parent.curriculumCoverPlan"
                        :preview-src-list="[scope.row.parent.curriculumCoverPlan]"
                    ></el-image>
                </template>
            </el-table-column>
            <el-table-column label="视频价格" align="center" width="80">
                <template slot-scope="scope">{{ scope.row.parent.price }}</template>
            </el-table-column>
            <el-table-column label="简介" align="center">
                <template slot-scope="scope">{{ scope.row.parent.curriculumIntroduce }}</template>
            </el-table-column>
            <el-table-column fixed="right" label="操作" width="150">
                <template slot-scope="scope">
                    <el-button type="text" size="small" @click="fnAdd(1,scope.row.parent)">添加</el-button>
                    <el-button type="text" size="small" @click="fnFound(2,scope.row.parent)">编辑</el-button>
                    <el-button
                        type="text"
                        size="small"
                        @click="fnMainDelect(scope.$index,scope.row.parent.id,tableData)"
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

        <el-dialog :title="title" :visible.sync="dialogFormVisible">
            <el-form
                :model="ruleForm"
                :rules="rules"
                ref="ruleForm"
                label-width="100px"
                class="demo-ruleForm"
            >
                <el-form-item label="课程名称" prop="curriculumName">
                    <el-input v-model="ruleForm.curriculumName" placeholder="请输入课程名称"></el-input>
                </el-form-item>
                <el-form-item label="讲师姓名" prop="lecturer">
                    <el-select v-model="ruleForm.lecturer" placeholder="请选择讲师">
                        <el-option
                            v-for="(item, index) in selLecturerList"
                            :key="item.value"
                            :value="item.value"
                            :label="item.label"
                        ></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="是否置顶" prop="isTop">
                    <el-select v-model="ruleForm.isTop" placeholder="请选择是否置顶">
                        <el-option label="是" value="1"></el-option>
                        <el-option label="否" value="0"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="是否综合推荐" prop="isRecommend">
                    <el-select v-model="ruleForm.isRecommend" placeholder="请选择是否综合推荐">
                        <el-option label="是" value="1"></el-option>
                        <el-option label="否" value="0"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="初始播放量" prop="viewCounts">
                    <el-input v-model="ruleForm.viewCounts" placeholder="请输入初始播放量"></el-input>
                </el-form-item>
                <el-form-item label="课程类型" prop="type">
                    <el-radio-group v-model="ruleForm.type">
                        <el-radio
                            v-for="(item, index) in selVideo"
                            :key="item.value"
                            :label="item.value"
                            :value="item.value"
                        >{{item.label}}</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="总价" prop="price">
                    <el-select v-model="ruleForm.price" placeholder="请选择总价">
                        <el-option label="0" value="0"></el-option>
                        <el-option label="1" value="1"></el-option>
                        <el-option label="6" value="6"></el-option>
                        <el-option label="30" value="30"></el-option>
                    </el-select>
                    <!-- <el-input v-model="ruleForm.price" placeholder="请输入总价"></el-input> -->
                </el-form-item>
                <el-form-item label="课程封面" prop="curriculumCoverPlan">
                    <el-upload
                        class="avatar-uploader"
                        action="http://192.168.1.58:8082/ymzs/api/curriculum/addImage"
                        :show-file-list="false"
                        :on-success="handleAvatarSuccess"
                        :before-upload="beforeAvatarUpload"
                    >
                        <img
                            v-if="ruleForm.curriculumCoverPlan"
                            :src="ruleForm.curriculumCoverPlan"
                            class="avatar"
                        />
                        <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                    </el-upload>
                </el-form-item>
                <el-form-item label="课程简介" prop="curriculumIntroduce">
                    <el-input
                        type="textarea"
                        v-model="ruleForm.curriculumIntroduce"
                        placeholder="请输入课程简介"
                    ></el-input>
                </el-form-item>

                <el-form-item>
                    <el-button type="primary" @click="submitForm('ruleForm')">{{CreateOrModify}}</el-button>
                    <el-button @click="resetForm('ruleForm')">重置</el-button>
                </el-form-item>
            </el-form>
        </el-dialog>

        <el-dialog :title="title" :visible.sync="dialogFormVisible1">
            <el-form
                :model="ruleForm1"
                :rules="rules"
                ref="ruleForm1"
                label-width="120px"
                class="demo-ruleForm"
            >
                <el-form-item label="上传课节的名称" prop="curriculumName">
                    <el-input v-model="ruleForm1.curriculumName" placeholder="请上传课节的名称"></el-input>
                </el-form-item>
                <el-form-item label="章节排序" prop="sortNum">
                    <el-input
                        v-model="ruleForm1.sortNum"
                        type="number"
                        :min="1"
                        placeholder="请上传章节排序,如果是第一章节，就输入纯数字：1"
                    ></el-input>
                </el-form-item>
                <el-form-item label="是否免费试看" prop="isFree">
                    <el-switch v-model="ruleForm1.isFree"></el-switch>
                </el-form-item>

                <el-form-item>
                    <el-button type="primary" @click="submitForm1('ruleForm1')">{{CreateOrModify}}</el-button>
                    <el-button @click="resetForm1('ruleForm1')">重置</el-button>
                </el-form-item>
            </el-form>
        </el-dialog>
    </div>
</template>

<script>
export default {
    name: 'Table',
    data() {
        return {
            imageUrl: '',
            //参数
            videoIndex: '',
            videoFlag: false,
            //是否显示进度条
            videoUploadPercent: 0,
            //进度条的进度，
            isShowUploadVideo: false,
            //显示上传按钮
            videoForm: {
                showVideoPath: ''
            },
            title: '创建课程',
            CreateOrModify: '立即创建',
            dialogFormVisible: false,
            dialogFormVisible1: false,
            // editdialogFormVisible:false,
            // editdialogFormVisible1:false,
            selLecturerList: [], //获取教师姓名
            selVideo: [], //获取视频类型
            total: null,
            ruleForm: {
                curriculumName: '',
                lecturer: '',
                isTop: '1',
                isRecommend: '1',
                type: '',
                price: '',
                viewCounts: '',
                curriculumIntroduce: '',
                curriculumCoverPlan: '',
                token: '',
                id: ''
            },
            page: {
                pageSize: 10,
                pageNum: 1,
                token: ''
            },
            ruleForm1: {
                curriculumName: '',
                isFree: '',
                parentId: '',
                token: '',
                sortNum: '',
                token: '',
                id: ''
            },
            rules: {
                curriculumName: [{ required: true, message: '请输入活动名称', trigger: 'blur' }],
                lecturer: [{ required: true, message: '请输入讲师项目', trigger: 'blur' }],
                type: [{ required: true, message: '请选择课程类型', trigger: 'blur' }],
                price: [{ required: true, message: '请输入价格', trigger: 'blur' }],
                curriculumCoverPlan: [{ required: true, message: '请上传课程封面', trigger: 'blur' }],
                curriculumIntroduce: [{ required: true, message: '请输入课程简介', trigger: 'blur' }],
                classperiod: [{ required: true, message: '请输入课程节数', trigger: 'blur' }],
                viewCounts: [{ required: true, message: '请输入初始播放量', trigger: 'blur' }]
            },
            tableData: [],
            ismianshow: false
        };
    },
    mounted() {
        this.selLecturer(); //获取教师列表
        this.selVideoType(); //获取视频类型
        this.getClassList2(); //获取上传课程列表;
    },
    // created() {},
    methods: {
        //创建or编辑课程
        fnFound(el, arr) {
            if (el == 1) {
                this.dialogFormVisible = true;
                //  this.ruleForm = {};//清空ruleForm的值
                this.ruleForm.curriculumCoverPlan = '';
                this.ruleForm.curriculumName = '';
                this.ruleForm.lecturer = '';
                this.ruleForm.isTop = '1';
                this.ruleForm.isRecommend = '1';
                this.ruleForm.type = '';
                this.ruleForm.price = '';
                this.ruleForm.viewCounts = '';
                this.ruleForm.curriculumIntroduce = '';
                this.ruleForm.curriculumCoverPlan = '';
                this.title = '创建课程';
                this.CreateOrModify = '立即创建';
            } else {
                this.dialogFormVisible = true; //打开弹框
                this.title = '编辑课程';
                this.CreateOrModify = '立即修改';
                this.ruleForm.isTop = String(arr.isTop);
                this.ruleForm.isRecommend = String(arr.isRecommend);
                this.ruleForm.type = String(arr.type);
                this.ruleForm.curriculumName = arr.curriculumName;
                this.ruleForm.lecturer = arr.lecturer;
                this.ruleForm.price = arr.price;
                this.ruleForm.viewCounts = arr.viewCounts;
                this.ruleForm.curriculumIntroduce = arr.curriculumIntroduce;
                this.ruleForm.curriculumCoverPlan = arr.curriculumCoverPlan;
                this.ruleForm.id = arr.id;
            }
        },

        //添加事件
        fnAdd(el, arr) {
            // console.log(arr)
            if (el == 1) {
                this.dialogFormVisible1 = true;
                this.ruleForm1 = {}; //清空ruleForm的值
                this.ruleForm1.parentId = arr.id;
                this.CreateOrModify = '立即创建';
            } else {
                this.dialogFormVisible1 = true;
                this.CreateOrModify = '立即修改';
                this.ruleForm1.curriculumName = arr.curriculumName;
                this.ruleForm1.isFree = arr.isFree;
                this.ruleForm1.parentId = arr.parentId;
                this.ruleForm1.token = arr.token;
                this.ruleForm1.sortNum = arr.sortNum;
                this.ruleForm1.token = arr.token;
                this.ruleForm1.id = arr.id;
            }
        },
        //创建课程
        submitForm(formName) {
            if (this.CreateOrModify == '立即创建') {
                var userToken = localStorage.getItem('userToken');
                (this.ruleForm.token = userToken),
                    this.$refs[formName].validate((valid) => {
                        if (valid) {
                            this.$axios({
                                method: 'post',
                                url: '/ymzs/api/curriculum/uploadClassVideo',
                                params: this.ruleForm
                            })
                                .then((res) => {
                                    if (res.data.code == 0) {
                                        this.dialogFormVisible = false;
                                        this.$refs[formName].resetFields();
                                        this.getClassList2();
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
                                url: '/ymzs/api/curriculum/upTheCourse',
                                params: this.ruleForm
                            })
                                .then((res) => {
                                    if (res.data.code == 0) {
                                        this.dialogFormVisible = false;
                                        this.$refs[formName].resetFields();
                                        this.getClassList2();
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
        //添加章数
        submitForm1(formName) {
            var userToken = localStorage.getItem('userToken');
            this.ruleForm1.token = userToken;
            if (this.ruleForm1.isFree == true) {
                this.ruleForm1.isFree = 1;
            } else {
                this.ruleForm1.isFree = 0;
            }
            if (this.CreateOrModify == '立即创建') {
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                        this.$axios({
                            method: 'post',
                            url: '/ymzs/api/curriculum/uploadClassVideo',
                            params: this.ruleForm1
                        })
                            .then((res) => {
                                if (res.data.code == 0) {
                                    this.dialogFormVisible1 = false;
                                    this.getClassList2();
                                    this.$refs[formName].resetFields();
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
                this.$axios({
                    method: 'post',
                    url: '/ymzs/api/curriculum/upTheCourse',
                    params: this.ruleForm1
                })
                    .then((res) => {
                        if (res.data.code == 0) {
                            this.dialogFormVisible1 = false;
                            this.$refs[formName].resetFields();
                            this.getClassList2();
                        }
                    })
                    .catch((error) => {
                        console.log(error);
                    });
            }
        },

        //
        //上传封面
        handleAvatarSuccess(res, file) {
            this.ruleForm.curriculumCoverPlan = URL.createObjectURL(file.raw);
            // this.ruleForm.curriculumCoverPlan = "https://oss.shal.club/avatarRandom/18.jpg";
            this.ruleForm.curriculumCoverPlan = res.data;
            console.log(this.ruleForm.curriculumCoverPlan);
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
        //获取教师的名称
        selLecturer() {
            this.$axios({
                method: 'post',
                url: '/ymzs/api/curriculum/selLecturerList',
                params: {}
            })
                .then((res) => {
                    if (res.data.code == 0) {
                        this.selLecturerList = res.data.data;
                    }
                })
                .catch((error) => {
                    console.log(error);
                });
        },
        //获取视频类型
        selVideoType() {
            this.$axios({
                method: 'post',
                url: '/ymzs/api/curriculum/selVideoType',
                params: {}
            })
                .then((res) => {
                    if (res.data.code == 0) {
                        this.selVideo = res.data.data;
                    }
                })
                .catch((error) => {
                    console.log(error);
                });
        },
        //获取课程的list
        getClassList2() {
            var userToken = localStorage.getItem('userToken');
            (this.page.token = userToken),
                this.$axios({
                    method: 'post',
                    url: '/ymzs/api/curriculum/getClassList2',
                    params: this.page
                })
                    .then((res) => {
                        if (res.data.code == 0) {
                            this.tableData = res.data.data.data.data;
                            this.total = res.data.data.num;
                        }
                    })
                    .catch((error) => {
                        console.log(error);
                    });
        },

        //上传视频
        beforeUploadVideo(index, file) {
            var fileSize = file.size / 1024 / 1024 < 10;
            if (['video/mp4', 'video/ogg', 'video/flv', 'video/avi', 'video/wmv', 'video/rmvb', 'video/mov'].indexOf(file.type) == -1) {
                this.$message({
                    type: 'info',
                    message: `请上传正确的格式`
                });

                return false;
            }
            if (!fileSize) {
                this.$message({
                    type: 'info',
                    message: `视频大小不超过10MB`
                });
                return false;
            }
            this.videoIndex = index;
            this.isShowUploadVideo = false;
        },
        //进度条
        uploadVideoProcess(event, file, fileList) {
            console.log('文件上传时' + JSON.stringify(event) + '||' + JSON.stringify(file) + '||' + JSON.stringify(fileList));
            this.videoFlag = true;
            this.videoUploadPercent = Number(file.percentage.toFixed(0) * 1);
        },
        //上传成功回调
        handleVideoSuccess(res, file) {
            console.log('文件上传之后' + JSON.stringify(res) + '||' + JSON.stringify(file));
            this.isShowUploadVideo = true;
            this.videoFlag = false;
            this.videoUploadPercent = 0;
            //后台上传地址
            if (res.code == 0) {
                this.$message({
                    type: 'info',
                    message: `上传成功`
                });
                this.getClassList2();
            }
        },
        //删除课程每一章的记录
        fnMainDelect(index, id, rows) {
            var userToken = localStorage.getItem('userToken');
            this.parmas = {
                token: userToken,
                id: id
            };
            this.$axios({
                method: 'post',
                url: '/ymzs/api/curriculum/delTheCourse',
                params: this.parmas
            })
                .then((res) => {
                    if (res.data.code == 0) {
                        rows.splice(index, 1);
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
        //删除每一节的
        fnSecondDelect(index, id, rows) {
            var userToken = localStorage.getItem('userToken');
            this.parmas = {
                token: userToken,
                id: id
            };
            this.$axios({
                method: 'post',
                url: '/ymzs/api/curriculum/delTheCourse',
                params: this.parmas
            })
                .then((res) => {
                    if (res.data.code == 0) {
                        rows.splice(index, 1);
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
            this.getClassList2();
        },
        handleSizeChange(psize) {
            this.page.pageSize = psize;
            this.getClassList2();
        },

        //重置
        resetForm(formName) {
            this.$refs[formName].resetFields();
        },
        //重置
        resetForm1(formName) {
            this.$refs[formName].resetFields();
        },
        //折叠面板
        handleExpandChange(row, expandRows) {
            const $classTable = this.$refs.classTable;

            if (expandRows.length > 1) {
                expandRows.forEach((expandRow) => {
                    if (row.id !== expandRow.id) {
                        $classTable.toggleRowExpansion(expandRow, false);
                    }
                });
            }
        }
    }
};
</script>

<style rel="stylesheet/scss" scoped>
.demo-table-expand {
    font-size: 0;
}

.demo-table-expand label {
    width: 90px;
    color: #99a9bf;
}

.demo-table-expand .el-form-item {
    margin-right: 0;
    margin-bottom: 0;
    width: 50%;
}

/* 视频上传 */

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

.table-td-thumb {
    height: 62px;
}
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
.video-avatar {
    width: 150px;
    height: 130px;
    display: block;
    margin: 0 auto;
}
</style>
