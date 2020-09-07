<template>
	<div class="app-container">
		<el-tabs v-model="activeName" class="tabs" @tab-click="handleClick">

			<el-tab-pane label="绑定信息" name="first" class="tabs1 is-active">
				<el-form :model="ruleForm1" :rules="rules" ref="ruleForm1" label-width="100px" class="demo-ruleForm">
					<el-form-item label="账户类型" prop="type">
						<el-select v-model="ruleForm1.type" placeholder="请选择类型">
							<el-option label="支付宝" value="2"></el-option>
						</el-select>
					</el-form-item>
					<div>
						<el-form-item label="收款账号" prop="account">
							<el-input v-model="ruleForm1.account" placeholder="请输入收款账号"></el-input>
						</el-form-item>
						<el-form-item label="真实姓名" prop="name">
							<el-input v-model="ruleForm1.name" placeholder="请输入真实姓名"></el-input>
						</el-form-item>
					</div>
					<el-form-item>
						<el-button type="primary" @click="submitForm1('ruleForm1')">提交</el-button>
						<el-button @click="resetForm('ruleForm1')">重置</el-button>
					</el-form-item>
					<div class="dataclass">绑定记录</div>
					<el-table :data="MoneyDate" height="430" border style="width: 100%;padding-left:32px;">
						<el-table-column prop="createTime" label="创建时间">
						</el-table-column>
						<el-table-column prop="username" label="收款帐号">
						</el-table-column>
						<el-table-column prop="name" label="真实姓名">
						</el-table-column>
						<el-table-column prop="type" label="账户类型">
							<template slot-scope="scope">
								<p v-if="MoneyDate[scope.$index].type==2">支付宝</p>
							</template>
						</el-table-column>
						<el-table-column fixed="right" label="操作" width="100">
							<template slot-scope="scope">
								<el-button @click.native.prevent="deleteRow(scope.$index,scope.row.id, MoneyDate)" type="text" size="small">
									移除
								</el-button>
								<el-button type="text" size="small" @click="fnEdit(scope.row.name,scope.row.username,scope.row.id)">编辑</el-button>
							</template>
						</el-table-column>
					</el-table>
					<el-pagination background layout="prev, pager, next, sizes, total, jumper" :page-sizes="[5, 10, 15, 20]"
					 :page-size="listQuery1.pageSize" :total="total1" @current-change="handleCurrentChange1" @size-change="handleSizeChange1">
					</el-pagination>
				</el-form>
			</el-tab-pane>

			<el-dialog title="编辑" :visible.sync="dialogFormVisible">
				<el-form :model="Form" ref="Form">
					<el-form-item label="真实姓名" prop="account">
						<el-input v-model="Form.account"></el-input>

					</el-form-item>
					<el-form-item label="收款账号" prop="name">
						<el-input v-model="Form.name"></el-input>
					</el-form-item>
				</el-form>
				<div slot="footer" class="dialog-footer">
					<el-button @click="dialogFormVisible = false">取 消</el-button>
					<el-button type="primary" @click="fnDetermine()">确 定</el-button>
				</div>
			</el-dialog>

			<el-tab-pane label="余额提现" name="second" class="tabs1 is-active">
				<el-alert title="尊敬的用户,您的提现金额将于次月20日左右到账(节假日顺延),请耐心等待!" type="warning" show-icon></el-alert>
				<div class="marginright">
					<div class="grid-content bg-purple">
						<div>
							<el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
								<el-form-item label="余额" prop="">{{numbel||0}} (可提现)</el-form-item>
								<el-form-item label="账户类型">
									<el-select v-model="ruleForm.thirdPartyId" prop="thirdPartyId" placeholder="请选择类型">
										<el-option v-for="(item, index) in CardDate" :key="item.value" :value="item.value" :label="'支付宝' + item.label">
										</el-option>
									</el-select>
								</el-form-item>
								<el-form-item label="提现金额" prop="inputMoney">
									<el-input v-model="ruleForm.inputMoney" placeholder="请输入您要体现的金额"></el-input>
								</el-form-item>
								<el-button style="margin-left:40px;" type="primary" @click="submitForm('ruleForm')">提现</el-button>
							</el-form>
						</div>
					</div>
					<div class="dataclass">提现明细</div>
					<el-table :data="tableData" height="430" border style="width: 100%;padding-left:32px;">
						<el-table-column prop="createTime" label="提现时间" width="180">
						</el-table-column>
						<el-table-column prop="userId" label="用户id">
						</el-table-column>
						<el-table-column prop="accountName" label="账户名称" width="180">
						</el-table-column>
						<el-table-column prop="inputMoney" label="提现金额">
						</el-table-column>
						<el-table-column prop="scanMoney" label="扣税金额">
						</el-table-column>
						<el-table-column prop="balance" label="当前账户余额">
						</el-table-column>
						<el-table-column prop="accountCode" label="账号">
						</el-table-column>
						<el-table-column prop="type" label="账号类型">
							<template slot-scope="scope">
								<p v-if="tableData[scope.$index].type==2">支付宝</p>
								<p v-if="tableData[scope.$index].type==1">微信</p>
							</template>
						</el-table-column>

						<el-table-column prop="reason" label="驳回原因">
							<template slot-scope="scope">
								<p>{{tableData[scope.$index].reason||'暂无'}}</p>
							</template>
						</el-table-column>
						<el-table-column prop="serialNumber" label="流水号">
							<template slot-scope="scope">
								<p>{{tableData[scope.$index].serialNumber||'暂无'}}</p>
							</template>
						</el-table-column>
						<el-table-column prop="takeState" label="提现状态">
							<template slot-scope="scope">
								<p v-if="tableData[scope.$index].takeState==1">待审核</p>
								<p v-if="tableData[scope.$index].takeState==2">成功</p>
								<p v-if="tableData[scope.$index].takeState==3">驳回</p>
								<p v-if="tableData[scope.$index].takeState==4">已打款</p>
								<p v-if="tableData[scope.$index].takeState==5">打款失败</p>
							</template>
						</el-table-column>
						<el-table-column prop="partnerTradeNo" label="付款商户订单号" width="180">
							<template slot-scope="scope">
								<p>{{tableData[scope.$index].partnerTradeNo||'暂无'}}</p>
							</template>
						</el-table-column>
						<el-table-column prop="paymentNo" label="企业付款单号" width="180">
							<template slot-scope="scope">
								<p>{{tableData[scope.$index].paymentNo||'暂无'}}</p>
							</template>
						</el-table-column>
					</el-table>
					<el-pagination background layout="prev, pager, next, sizes, total, jumper" :page-sizes="[5, 10, 15, 20]"
					 :page-size="listQuery.pageSize" :total="total" @current-change="handleCurrentChange" @size-change="handleSizeChange">
					</el-pagination>
				</div>
			</el-tab-pane>
		</el-tabs>
	</div>
</template>
<script>
	export default {
		name: 'financial',
		data() {
			const validateUsername = (rule, value, callback) => {
				if (value && (!(/^[1][34578]\d{9}$/).test(value) || !(/^[1-9]\d*$/).test(value) || value.length !== 11)) {
					callback(new Error('手机号码不符合规范'))
				} else {
					callback()
				}
			};
			return {
				activeName: 'first', //切换导航
				dialogFormVisible: false,
				MoneyDate: [], //银行卡列表
				tableData: [], //提现列表
				CardDate: [], //获取有哪些银行卡
				total1: null, //提现列表总页数
				total: null, ////银行卡列表总页数
				numbel: "", //可以提现的金额
				listQuery: {
					pageSize: 10,
					pageNum: 1
				},
				Form: {
					account: '',
					name: '',
					id: '',
				},
				listQuery1: {
					pageSize: 10,
					pageNum: 1,
					type: 2,
				},
				ruleForm1: {

					type: "",
					account: "",
					name: ""
				},
				ruleForm: {
					thirdPartyId: "",
					inputMoney: ""
				},
				rules: {
					type: [{
						required: true,
						message: '账户类型不能为空',
						trigger: 'blur'
					}],
					account: [{
						required: true,
						message: '收款账户不能为空',
						trigger: 'blur'
					}],
					name: [{
						required: true,
						message: '真实姓名不能为空',
						trigger: 'blur'
					}],
				}
			}
		},
		mounted() {

			this.bindingMesSel(); //获取绑定记录
			// var userToken = localStorage.getItem("userToken");
			// if (userToken == null || userToken == '' || userToken == undefined) {
			//   localStorage.removeItem("userToken");
			//   localStorage.removeItem("UserId");
			//   this.$router.push('/login');
			// }
		},
		methods: {
			//提交收款信息
			submitForm1(formName) {
				var userToken = localStorage.getItem("userToken");
				this.ruleForm1.token = userToken;
				this.$refs[formName].validate((valid) => {
					if (valid) {
						this.$axios({
							method: 'post',
							url: '/ymzs/api/curriculum/bindingAlipayAccount',
							params: this.ruleForm1
						}).then(res => {
							console.log()
							if (res.data.code == 0) {
								this.$message({
									showClose: true,
									message: '提交成功',
									type: 'success'
								});
								this.$refs[formName].resetFields();
								this.bindingMesSel(); //获取绑定记录
							} else if (res.data.code == 301) {
								localStorage.removeItem("userToken");
								localStorage.removeItem("UserId");
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
			//获取绑定记录
			bindingMesSel() {
				var userToken = localStorage.getItem("userToken");
				this.listQuery1.token = userToken,
					this.$axios({
						method: 'post',
						url: '/ymzs/api/curriculum/bindingMesSel',
						params: this.listQuery1
					}).then(res => {
						if (res.data.code == 0) {
							this.MoneyDate = res.data.data.data;
							this.total1 = res.data.data.num; //总条数
						} else if (res.data.code == 301) {
							// localStorage.removeItem("userToken");
							// localStorage.removeItem("UserId");
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
			//删除绑定记录的列表
			deleteRow(index, id, rows) {
				var userToken = localStorage.getItem("userToken");
				this.parmas = {
					token: userToken,
					id: id
				}
				this.$axios({
					method: 'post',
					url: '/ymzs/api/curriculum/delBindingAccount',
					params: this.parmas
				}).then(res => {
					if (res.data.code == 0) {
						rows.splice(index, 1);
					} else if (res.data.code == 301) {
						localStorage.removeItem("userToken");
						localStorage.removeItem("UserId");
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
			},

			//编辑
			fnEdit(account, name, id) {
				this.dialogFormVisible = true;
				this.Form.account = account;
				this.Form.name = name;
				this.Form.id = id;
			},
			//编辑点击确定
			fnDetermine() {
				var userToken = localStorage.getItem("userToken");
				this.Form.token = userToken;
				this.$axios({
					method: 'post',
					url: '/ymzs/api/curriculum/updateBindingAccount',
					params: this.Form
				}).then(res => {
					if (res.data.code == 0) {
						this.$message({
							showClose: true,
							message: '修改成功',
							type: 'success'
						});
						this.bindingMesSel();
						this.dialogFormVisible = false;

					} else if (res.data.code == 301) {
						localStorage.removeItem("userToken");
						localStorage.removeItem("UserId");
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
			},


			//获取银行卡信息
			selectAccoun() {
				var userToken = localStorage.getItem("userToken");
				this.parmas = {
					token: userToken
				}
				this.$axios({
					method: 'post',
					url: '/ymzs/api/curriculum/selectAccount',
					params: this.parmas
				}).then(res => {
					if (res.data.code == 0) {
						this.CardDate = res.data.data.data;
						this.numbel = res.data.data.money;
					} else if (res.data.code == 301) {
						localStorage.removeItem("userToken");
						localStorage.removeItem("UserId");
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
			},
			//提现
			submitForm(formName) {
				var userToken = localStorage.getItem("userToken");
				this.ruleForm.token = userToken;
				this.ruleForm.thirdPartyId = this.ruleForm.thirdPartyId;
				if (this.ruleForm.thirdPartyId == '') {
					this.$message({
						showClose: true,
						message: '请选择您要提现的账户类型',
						type: 'error'
					});
					return;
				} else if (this.ruleForm.inputMoney > this.numbel) {
					this.$message({
						showClose: true,
						message: '提现金额不能大于您的余额',
						type: 'error'
					});
					return;
				}
				// else if (this.ruleForm.inputMoney < 100) {
				//        this.$message({
				//          showClose: true,
				//          message: '您输入的金额不得少于100元',
				//          type: 'error'
				//        });
				//        return;
				//      }
				else {
					this.$refs[formName].validate((valid) => {
						if (valid) {
							this.$axios({
								method: 'post',
								url: '/ymzs/api/curriculum/insertYmzsUserMoney',
								params: this.ruleForm
							}).then(res => {
								if (res.data.code == 0) {
									this.$message({
										showClose: true,
										message: '提交成功',
										type: 'success'
									});
									this.selHistory();
									this.$refs[formName].resetFields();
								} else if (res.data.code == 301) {
									localStorage.removeItem("userToken");
									localStorage.removeItem("UserId");
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
				}
			},
			//提现明细
			selHistory() {
				var userToken = localStorage.getItem("userToken");
				this.listQuery.token = userToken,
					this.$axios({
						method: 'post',
						url: '/ymzs/api/curriculum/selHistory',
						params: this.listQuery
					}).then(res => {
						if (res.data.code == 0) {
							this.tableData = res.data.data.data;
							this.total = res.data.data.num; //总条数
						} else if (res.data.code == 301) {
							localStorage.removeItem("userToken");
							localStorage.removeItem("UserId");
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
			},

			// 提现记录列表前往第几页方法
			handleCurrentChange(cpage) {
				this.listQuery.pageNum = cpage;
				this.selHistory();
			},
			handleSizeChange(psize) {
				this.listQuery.pageSize = psize;
				this.selHistory();
			},

			// 绑定支付宝卡号列表前往第几页方法
			handleCurrentChange1(cpage) {
				this.listQuery1.pageNum = cpage;
				this.bindingMesSel();
			},
			handleSizeChange1(psize) {
				this.listQuery1.pageSize = psize;
				this.bindingMesSel();
			},
			//手机号中间四位变*
			starPhone(phoneNum) {
				let str = String(phoneNum),
					strLen = str.slice(-7, -3);
				return str.replace(strLen, "****");
			},
			//重置
			resetForm(formName) {
				this.$refs[formName].resetFields();
			},
			//导航切换
			handleClick(tab, event) {
				console.log(event.target.getAttribute('id'))
				if (event.target.getAttribute('id') == "tab-second") {
					this.selectAccoun() //获取可以提现的金额
					this.selHistory(); //提现明细

				} else {}
			},
		},
	}
</script>
<style rel="stylesheet/scss" lang="scss" scoped>
	.app-container {
		background-color: #e5e9f2;
		height: 885px;
		// padding-left: 2%;
	}

	.el-tabs__item {
		font-size: 18px;
	}

	.yue {
		font-size: 15px;
		margin-bottom: 15px;
		display: inline-block
	}

	.dataclass {
		width: 100%;
		height: 30px;
		font-size: 14px;
		line-height: 30px;
		color: #409EFF;
	}

	.el-pagination {
		margin-top: 20px;
	}

	.tixian {
		padding: 5px 20px;
		background-color: #36A3F7;
		color: #fff;
		margin-left: 20px;
		border-radius: 5px;
	}

	.inputtt {
		border: 1px solid #36A3F7;
		outline: none;
		font-size: 12px;
		height: 28px;
		padding-left: 10px;
		border-radius: 5px;
	}

	.el-input,
	.el-select {
		width: 15%;
	}

	.marginright {
		width: 100%;
		float: left;
		padding: 10px;
	}

	.iconimg {
		width: 80px;
		height: 80px;
		background-color: #40C9C6;
		float: left;
		text-align: center;
		margin: 0 auto;
		line-height: 80px;
		border-radius: 6px;
	}

	.iconimg:hover,
	.iconimg1:hover,
	.iconimg2:hover {
		transform: scale(0.8);
	}

	.iconimg1 {
		width: 80px;
		height: 80px;
		background-color: #36A3F7;
		float: left;
		text-align: center;
		margin: 0 auto;
		line-height: 80px;
		border-radius: 6px;
	}

	.iconimg2 {
		width: 80px;
		height: 80px;
		background-color: #F4516C;
		float: left;
		text-align: center;
		margin: 0 auto;
		line-height: 80px;
		border-radius: 6px;
	}

	.iconimg3 {
		width: 80px;
		height: 80px;
		background-color: #f4ea2a;
		float: left;
		text-align: center;
		margin: 0 auto;
		line-height: 80px;
		border-radius: 6px;
	}

	.iconfont {
		float: right;
		width: 400px;
		height: 80px;
		// line-height: 40px;
		font-size: 18px;
	}

	.icomi {
		vertical-align: middle;
	}

	.el-row {
		margin-bottom: 20px;

		&:last-child {
			margin-bottom: 0;
		}
	}

	.el-col {
		border-radius: 4px;
	}

	.grid-content {
		border-radius: 4px;
		min-height: 200px;
		padding: 10px;
		background-color: #fff;
	}
</style>
