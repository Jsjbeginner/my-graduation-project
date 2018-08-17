<template>
	<div data-tecode-box>
		<el-button type="primary" @click="addInform()">添加</el-button>
		<el-table :data="tableData" class="table">
			<el-table-column label="实例标题" prop="title" algin="center"></el-table-column>
			<el-table-column label="实例描述" prop="content" algin="center"></el-table-column>
			<el-table-column label="实例地址" prop="textUrl" algin="center"></el-table-column>
			<el-table-column label="操作" algin="center">
				<template slot-scope="scope">
					<el-button type="text" @click="editData(scope.row)">编辑</el-button>
					<el-button type="text" @click="deleteData(scope.row)">删除</el-button>
				</template>
			</el-table-column>
		</el-table>
		
		<el-dialog :title="dialogData.dialogTitle" :visible.sync="dialogData.dialogVisible" :before-close="closeDialog" @close="closeDialog">
			<el-form :model="formData" ref="formData" :rules="rules" class="form"label-position="left">
				<el-form-item label="实例标题:" prop="title">
					<el-input v-model="formData.title"  placeholder="输入实例标题"></el-input>
				</el-form-item>
				<el-form-item label="实例描述:" prop="content">
					<el-input v-model="formData.content" placeholder="输入实例描述" type="textarea"  :autosize="{ minRows: 1, maxRows: 3}"></el-input>
				</el-form-item>
				<el-form-item label="实例代码" prop="textUrl" class="txt">
					<Editor v-if="dialogData.dialogVisible" :url="formData.textUrl" @get-url="getUrl" flag="1" ref="editor"></Editor>
				</el-form-item>
			</el-form>
			<div class="button_bottom">
				<el-button @click="sureDialog('formData')" type="primary">确 定</el-button>
				<el-button @click="closeDialog()">取 消</el-button>				
			</div>
		</el-dialog>
	</div>
</template>

<script>
	import Vue from 'vue';
	import {$v,videoUrl,TIPS_TIEM,AJAX_ADMIN} from '~/assets/js/const.js';
	import Editor from '~/component/common/Editor'; 
	import {Button,Table,TableColumn,Dialog,Form,FormItem,Input,MessageBox,Message} from 'element-ui';

	Vue.use(Button);
	Vue.use(Table);
	Vue.use(TableColumn);
	Vue.use(Dialog);
	Vue.use(Form);
	Vue.use(FormItem);
	Vue.use(Input);
	
	export default{
		components:{
			Editor,
		},
		data(){
			return{
				tableData: [],
				dialogData: {
					dialogTitle: '',
					dialogVisible: false,
				},
				isEdit: false,
				codeId: '',
				formData: {
					title: '',
					content: '',
					textUrl: '',
				},
				rules: {
					title:[
						{ required: true, message: '请填写实例标题', trigger: 'blur' },
					],
					content:[
						{ required: true, message: '请填写实例描述', trigger: 'blur' },
					],
				},
			}
		},
		methods:{
			//添加
			addInform(){
				let self = this;
				self.isEdit = false;
				self.dialogData.dialogTitle = '新增课程实例';
				self.dialogData.dialogVisible = true;
			},
			//编辑
			editData(row){
				let self = this;
				self.isEdit = true;
				self.codeId = row._id;
				self.dialogData.dialogTitle = '编辑课程大纲';
				self.dialogData.dialogVisible = true;
				self.$nextTick(()=>{
					for(let obj in row){
						if(obj != 'codeId'){
							self.formData[obj] = row[obj];
						}
					}
				});
			},
			//提交表单
			sureDialog(formData){
				let self = this;
				self.$refs[formData].validate((valid) => {
					if(valid){
						self.$refs['editor'].getUrl();
					}
				})
			},
			//获取信息
			getUrl(urll, flag, result){
				let self = this;
//              let index = Number(flag)
                if(result == 'SUCCESS'){
                    self.formData.textUrl = urll;
                } else {
                    self.formData.textUrl = '';
                }
                let data = self.formData;
                let url = self.isEdit ? '/tecode/upData' : '/tecode/addData';
                if(self.isEdit){
                	//编辑
                	data.codeId = self.codeId;
                }
                url = AJAX_ADMIN + url;
				$v.post(url, data, (d) => {
					Message({
			            	type: 'success',
			            	message: self.isEdit ? '编辑成功' : '添加成功',
			            	duration: TIPS_TIEM
			          	});
					self.closeDialog();
					self.getData();
				})
			},
			//删除数据
			deleteData(row){
				let self = this;
				MessageBox.confirm("此操作将永久删除文件，是否继续？", "提示", {
					cancelButtonText: "取消",
					confirmButtonText: "确定",
    				type: 'warning',
    			}).then(()=>{
    				let url = AJAX_ADMIN + '/tecode/delData'
    				let data = {};
    				data.codeId = row._id;
    				$v.post(url,data,(d)=>{
    					Message({
			            	type: 'success',
			            	message: '删除成功！',
			            	duration: 2000
			          	});
			          	this.getData();
    				})
    			}).catch(()=>{
    				Message({
					    type: 'info',
					    message: '已取消删除',
		            	duration: 2000
					});
    			})
			},
			//获取数据
			getData(){
				let url = AJAX_ADMIN + '/tecode/getData';
				$v.get(url, (d) => {
					this.tableData = d.data;
				});
			},
			//dialog关闭时清除数据
			closeDialog(done){
				let self = this;
				for(let i in self.formData){
					self.formData[i] = '';
				}
				self.$refs['formData'].resetFields();
				self.dialogData.dialogVisible = false;
				done ? done() : '';
			}
		},
		mounted(){
			if(!sessionStorage.getItem("islogin")){
	    		this.$router.push({path: '/login'});
	    	}
			this.getData();
		}
	}
</script>

<style lang="scss">
	div[data-tecode-box]{
		@import '~/assets/css/public.scss';
		.el-table{
		    padding-top: 20px;
		    .cell{
	    	    text-align: center;
	    	    white-space: nowrap;
				overflow: hidden;
			    text-overflow: ellipsis;
		    }
		}
		.el-dialog__wrapper{
    		overflow: hidden; 
		    .el-dialog{
			    margin: auto !important;
		        top: 15%;
		        overflow-y: auto;
    			max-height: 80%;
			    .el-dialog__header{
		    	    padding: 20px 20px 10px 20px;
			    	.el-dialog__title{
			    		font-size: 16px;
			    	}
			    }
			    .el-dialog__body{
			    	padding: 10px 20px;
			    	.el-form{
			    		.el-form-item{
			    			padding: 10px 0px;
			    			.el-form-item__label{
			    				display: block;
			    			}
			    			.el-form-item__content{
			    				display: block;
							    width: 100%;
							    /*overflow: hidden;*/
			    			}
			    		}
			    		.txt{
			    			.el-form-item__label{
			    				font-weight: bold;
    							font-size: 16px;
			    			}
			    			.el-form-item__content{
			    				overflow: hidden;
			    			}
			    		}
			    	}
			    	.button_bottom{
			    		padding: 20px 0px;
		    		    display: flex;
    					justify-content: flex-end;
			    	}
			    }
			}
		}
	}
</style>