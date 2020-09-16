<template>
	<div>

		<el-row :gutter="20" class="mgb20">
			<el-col :span="6">
				<el-card shadow="hover" :body-style="{padding: '0px'}">
					<div class="grid-content grid-con-1">
						<i class="el-icon-lx-people grid-con-icon"></i>
						<div class="grid-cont-right">
							<div class="grid-num">{{numbel.buyToday||0}}</div>
							<div>今日购买</div>
						</div>
					</div>
				</el-card>
			</el-col>
			<el-col :span="6">
				<el-card shadow="hover" :body-style="{padding: '0px'}">
					<div class="grid-content grid-con-2">
						<i class="el-icon-lx-notice grid-con-icon"></i>
						<div class="grid-cont-right">
							<div class="grid-num">{{numbel.monthlyIncome||0}}</div>
							<div>本月收入</div>
						</div>
					</div>
				</el-card>
			</el-col>
			<el-col :span="6">
				<el-card shadow="hover" :body-style="{padding: '0px'}">
					<div class="grid-content grid-con-3">
						<i class="el-icon-lx-goods grid-con-icon"></i>
						<div class="grid-cont-right">
							<div class="grid-num">{{numbel.totalRevenue}}</div>
							<div>总收入</div>
						</div>
					</div>
				</el-card>
			</el-col>
			<el-col :span="6">
				<el-card shadow="hover" :body-style="{padding: '0px'}">
					<div class="grid-content grid-con-4">
						<i class="el-icon-c-scale-to-original grid-con-icon"></i>
						<div class="grid-cont-right">
							<div class="grid-num">0</div>
							<div>余额</div>
						</div>
					</div>
				</el-card>
			</el-col>
		</el-row>

		<el-row :gutter="20">
			<el-col :span="12">
				<el-card shadow="hover">
					<schart ref="bar" class="schart" canvasId="bar" :options="options"></schart>
				</el-card>
			</el-col>
			<el-col :span="12">
				<el-card shadow="hover">
					<schart ref="line" class="schart" canvasId="line" :options="options2"></schart>
				</el-card>
			</el-col>
		</el-row>
		<el-row :gutter="10">
			<div class="dataclass">课程收入</div>
			<el-table :data="tableData" height="250" border style="width: 100%;padding-left:32px;">
			  <el-table-column prop="id" label="编号" width="180">
				  <template slot-scope="scope">
				      <p>{{scope.$index}}</p>
				  </template>
			  </el-table-column>
			  <el-table-column prop="name" label="讲师名称" width="180"></el-table-column>
			  <el-table-column prop="typeName" label="课程类型"></el-table-column>
			  <el-table-column prop="title" label="标题"></el-table-column>
			  <el-table-column prop="price" label="价格"></el-table-column>
			  <el-table-column prop="content" label="评论"></el-table-column>
			  <el-table-column prop="level" label="评分">
				  <template slot-scope="scope">
						<el-rate
							  v-model="tableData[scope.$index].level"
							  disabled
							  text-color="#ff9900"
						>
						</el-rate>
				  </template> 
			  </el-table-column>
			</el-table>
			<el-pagination background layout="prev, pager, next, sizes, total, jumper" :page-sizes="[5, 10, 15, 20]"
			 :page-size="params.pageSize" :total="total" @current-change="handleCurrentChange" @size-change="handleSizeChange">
			</el-pagination>
		</el-row>
	</div>
</template>

<script>
	import Schart from 'vue-schart';
	import bus from '../common/bus';
	export default {
		name: 'dashboard',
		data() {
			return {
				numbel: {
				  buyToday: '',
				  monthlyIncome: '',
				  totalRevenue: '',
				  balance: '',
				},
				params:{
				  pageSize: 10,
				  pageNum: 1
				},
				name: localStorage.getItem('ms_username'),
				tableData: [],
				total:null,
				options: {
					type: 'bar',
					title: {
						text: '最近一周各视频销售图'
					},
					xRorate: 25,
					labels: ['周一', '周二', '周三', '周四', '周五','周六', '周日'],
					datasets: [{
							label: '医美咨询',
							data: [234, 278, 270, 190, 230]
						},
						{
							label: '医美营销',
							data: [164, 178, 190, 135, 160]
						},
						{
							label: '渠道开发',
							data: [144, 198, 150, 235, 120]
						},{
							label: '渠道',
							data: [144, 100, 150, 235, 120]
						}
					]
				},
				options2: {
					type: 'line',
					title: {
						text: '最近几个月各品类销售趋势图'
					},
					labels: ['6月', '7月', '8月', '9月', '10月'],
					datasets: [
						{
							label: '整体销售数量',
							data: [234, 278, 270, 190, 230]
						},
						{
							label: '收藏数量',
							data: [164, 178, 150, 135, 160]
						},
						
					]
				}
			};
		},
		mounted(){
			this.homePageincome()//今日购买
			this.classIncome()//课程收入
			this.nearAWeekHistogram()//圆柱图
			this.statisticalLineChart();//折线图
		},
		components: {
			Schart
		},
		computed: {
			
		},
		
		methods: {
			homePageincome(){
				this.$axios({
					method: 'post',
					url: '/ymzs/api/curriculum/homePageincome',
					params: {}
				}).then(res => {
					try {
						const {buyToday, monthlyIncome, totalRevenue} = res.data.data
						if (res.data.code == 0) {
							this.numbel.buyToday= buyToday
							this.numbel.monthlyIncome = monthlyIncome
							this.numbel.totalRevenue = totalRevenue
						}
					} catch (err) {
						this.$message.error(err)
					}
				}).catch(error => {
					console.log(error);
				});
			},
			classIncome(){
				this.$axios({
					method: 'post',
					url: '/ymzs/api/curriculum/classIncome',
					params: this.params
				}).then(res => {
					if (res.data.code == 0) {
						this.tableData=res.data.data.data;
						this.total = res.data.data.num;
					}
				}).catch(error => {
					console.log(error);
				});
			},
			nearAWeekHistogram(){
				this.$axios({
					method: 'post',
					url: '/ymzs/api/curriculum/nearAWeekHistogram',
					params: {}
				}).then(res => {
					if (res.data.code == 0) {
						
						var typeName =[];
						var arr = res.data.data;
						arr.forEach((item,index) =>{ 
						  typeName.push(item)
						});
						this.options.datasets = typeName
					}
				}).catch(error => {
					console.log(error);
				});
			},
			statisticalLineChart(){
				this.$axios({
					method: 'post',
					url: '/ymzs/api/curriculum/statisticalLineChart',
					params: {}
				}).then(res => {
					if (res.data.code == 0) {
						var time=[]
						var arr = res.data.data;
						this.options2.labels= arr[1];
						this.options2.datasets = arr[0];
					}
				}).catch(error => {
					console.log(error);
				});
			},
			// 提现记录列表前往第几页方法
			handleCurrentChange(cpage) {
				this.params.pageNum = cpage;
				this.classIncome();
			},
			handleSizeChange(psize) {
				this.params.pageSize = psize;
				this.classIncome();
			},
		
		}
	};
</script>


<style scoped>
	.el-row {
		margin-bottom: 20px;
	}

	.grid-content {
		display: flex;
		align-items: center;
		height: 100px;
	}

	.grid-cont-right {
		flex: 1;
		text-align: center;
		font-size: 14px;
		color: #999;
	}

	.grid-num {
		font-size: 30px;
		font-weight: bold;
	}

	.grid-con-icon {
		font-size: 50px;
		width: 100px;
		height: 100px;
		text-align: center;
		line-height: 100px;
		color: #fff;
	}
	.grid-con-1 .grid-con-icon {
		background-color: #408CCA;
	}
	.grid-con-1 .grid-num {
		color: rgb(45, 140, 240);
	}

	.grid-con-2 .grid-con-icon {
		 background-color: #EFAD4D;
	}

	.grid-con-2 .grid-num {
		color: rgb(45, 140, 240);
	}

	.grid-con-3 .grid-con-icon {
		background-color: #5EB75B;
	}
	.grid-con-3 .grid-num {
		color: rgb(242, 94, 67);
	}
	.grid-con-4 .grid-con-icon {
		background-color: #FE6383;
	}
	
	.grid-con-4 .grid-num {
		color: rgb(242, 94, 67);
	}
	.user-info {
		display: flex;
		align-items: center;
		padding-bottom: 20px;
		border-bottom: 2px solid #ccc;
		margin-bottom: 20px;
	}
	.user-avator {
		width: 120px;
		height: 120px;
		border-radius: 50%;
	}

	.user-info-cont {
		padding-left: 50px;
		flex: 1;
		font-size: 14px;
		color: #999;
	}

	.user-info-cont div:first-child {
		font-size: 30px;
		color: #222;
	}

	.user-info-list {
		font-size: 14px;
		color: #999;
		line-height: 25px;
	}

	.user-info-list span {
		margin-left: 70px;
	}

	.mgb20 {
		margin-bottom: 20px;
	}

	.todo-item {
		font-size: 14px;
	}

	.todo-item-del {
		text-decoration: line-through;
		color: #999;
	}

	.schart {
		width: 100%;
		height: 300px;
	}
	.dataclass {
		width: 98%;
		height: 30px;
		font-size: 16px;
		padding-left: 15px;
		color: #408CCA;
   }

</style>
