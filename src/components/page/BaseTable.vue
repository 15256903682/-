<template>
    <div>
        <div class="container">
            <el-form :inline="true" :model="listQuery" ref="listQuery" class="demo-form-inline">
                <el-form-item label prop="nickname">
                    <el-input v-model="listQuery.curriculumName" placeholder="课程名称"></el-input>
                </el-form-item>
                <template>
                    <el-select v-model="listQuery.type" filterable placeholder="请选择">
                        <el-option
                            v-for="item in options"
                            :key="item.value"
                            :label="item.label"
                            :value="item.value"
                        ></el-option>
                    </el-select>
                </template>
                <el-form-item>
                    <el-button type="primary" @click="onSubmit('listQuery')">查询</el-button>
                    <!-- <el-button @click="resetForm('listQuery')">重置</el-button> -->
                </el-form-item>
            </el-form>
            <el-table :data="tableData" border style="width: 100%" >
                <el-table-column prop="id" label="编号" width="80" align="center">
                    <template slot-scope="scope">
                        <p>{{scope.$index}}</p>
                    </template>
                </el-table-column>
                <el-table-column prop="id" label="课程id" width="100" align="center"></el-table-column>
                <el-table-column prop="createTime" label="购买日期" align="center"></el-table-column>
                <el-table-column prop="curriculumName" label="课程名称" align="center"></el-table-column>
                <el-table-column prop="lecturerName" label="讲师姓名" align="center"></el-table-column>
                <el-table-column prop="price" label="价格" align="center"></el-table-column>
                <el-table-column prop="coverAddres" label="课程封面" align="center">
                    <template slot-scope="scope">
                        <el-image
                            class="table-td-thumb"
                            :src="scope.row.coverAddres"
                            :preview-src-list="[scope.row.coverAddres]"
                            style="width:40px;height:40px"
                        ></el-image>
                    </template>
                </el-table-column>
                <el-table-column prop="type" label="课程类型" align="center">
                    <template slot-scope="scope">
                        <p v-if="tableData[scope.$index].type==1">医美咨询</p>
                        <p v-if="tableData[scope.$index].type==2">医美营销</p>
                        <p v-if="tableData[scope.$index].type==3">渠道开发</p>
                        <p v-if="tableData[scope.$index].type==4">运营管理</p>
                    </template>
                </el-table-column>
                <el-table-column prop="content" label="评论" align="center"></el-table-column>
            </el-table>
            <el-pagination
                background
                layout="prev, pager, next, sizes, total, jumper"
                :page-sizes="[5, 10, 15, 20]"
                :page-size="listQuery.pageSize"
                :total="total"
                @current-change="handleCurrentChange"
                @size-change="handleSizeChange"
            ></el-pagination>
        </div>
    </div>
</template>

<script>
export default {
    name: 'basetable',
    data() {
        return {
            listQuery: {
                pageSize: 10,
                pageNum: 1,
                token: '',
                type: '',
                curriculumName: ''
            },
            tableData: [],
            total: null,
            options: [
                {
                    value: '1',
                    label: '医美咨询'
                },
                {
                    value: '2',
                    label: '医美营销'
                },
                {
                    value: '3',
                    label: '渠道开发'
                },
                {
                    value: '4',
                    label: '运营管理'
                }
            ]
        };
    },
    mounted() {
        this.selBuyList();
    },
    methods: {
        selBuyList() {
            var userToken = localStorage.getItem('userToken');
            this.listQuery.token = userToken;
            this.$axios({
                method: 'post',
                url: '/ymzs/api/curriculum/selBuyList',
                params: this.listQuery
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
        onSubmit() {
            this.selBuyList();
        },
        // 提现记录列表前往第几页方法
        handleCurrentChange(cpage) {
            this.listQuery.pageNum = cpage;
            this.selBuyList();
        },
        handleSizeChange(psize) {
            this.listQuery.pageSize = psize;
            this.selBuyList();
        },
        resetForm(formName) {
            this.$refs[formName].resetFields();
        }
    }
};
</script>

<style scoped>
.el-select {
    margin-right: 10px;
}
</style>
