<template>
	<div data-tecourseware-box>
		<el-button type="primary" @click="addCourseware()">添加</el-button>
		<el-table :data="tableData" class="table">
			<el-table-column label="课件章节" prop="title" algin="center"></el-table-column>
			<el-table-column label="课件地址" prop="contentUrl" algin="center"></el-table-column>
			<el-table-column label="操作" algin="center">
				<template slot-scope="scope">
					<el-button type="text" @click="editData(scope.row)">编辑</el-button>
					<el-button type="text" @click="deleteData(scope.row)">删除</el-button>
				</template>
			</el-table-column>
		</el-table>
		
		<el-dialog :title="dialogData.dialogTitle" :visible.sync="dialogData.dialogVisible" :before-close="closeDialog" @close="closeDialog">
			<el-form :model="formData" ref="formData" :rules="rules" class="form" label-position="left">
				<el-form-item label="课件章节:" prop="title">
					<el-input v-model="formData.title"  placeholder="输入课件章节"></el-input>
				</el-form-item>
				<el-form-item label="排序" prop="sortId">
					<el-input v-model="formData.sortId" placeholder="输入排序序列"></el-input>
				</el-form-item>
				<el-form-item label="教学课件:" prop="contentUrl"  class="txt">
                    <el-upload
                        class="upload-demo"
                        :action="AJAX_ADMIN + '/upload'"
                        :accept="'application/pdf' || 'application/vnd.ms-powerpoint'"
                        :on-success="completeVideo"
                        :limit="1"
                        ref="upload"
                    >
                        <el-button size="small" type="primary">{{ formData.contentUrl ? '更换文件' : '点击上传'}}</el-button>
                        <div slot="tip" class="el-upload__tip">{{ formData.contentUrl ? '已上传' : '请选择教学文件'}}</div>
                    </el-upload>
                    <p>请上传ppt或pdf类型的课件</p>
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
	import {$v, AJAX_ADMIN, TIPS_TIEM, DateFormat} from '~/assets/js/const.js';
	
	import {Button,Table,TableColumn,MessageBox,Message,Dialog,Form,FormItem,Input,Upload} from 'element-ui';
	Vue.use(Button);
	Vue.use(Table);
	Vue.use(TableColumn);
	Vue.use(Dialog);
	Vue.use(Form);
	Vue.use(FormItem);
	Vue.use(Input);
	Vue.use(Upload);
	
	export default{
		data(){
			let sortNumber = (rule, value, callback) =>{
				let reg = /^[0-9]*[1-9][0-9]*$/i;
	            if (value == "") {
	                return callback(new Error("请输入排序"));
	            } else if (!reg.test(value)) {
	                return callback(new Error("请输入正整数"));
	            } else {
	                callback();
	            }
			}
			return{
				AJAX_ADMIN: AJAX_ADMIN,
				tableData: [],
				coursewareId: '',
				isEdit: false,
				dialogData: {
					dialogTitle: '',
					dialogVisible: false,
				},
				formData: {
					title: '',
					sortId: '',
					contentUrl: '',
				},
				rules: {
					title:[
						{ required: true, message: '请填写课程章节', trigger: 'blur' },
					],
					sortId:[
						{ required: true, validator: sortNumber, trigger: "blur" }
					],
				},	
			}
		},
		methods:{
			//新增
			addCourseware(){
				let self = this;
				self.isEdit = false;
				self.dialogData.dialogTitle = '新增课程课件';
				self.dialogData.dialogVisible = true;
			},
			//编辑
			editData(row){
				let self = this;
				self.isEdit = true;
				self.coursewareId = row._id;
				self.dialogData.dialogTitle = '编辑课程课件';
				self.dialogData.dialogVisible = true;
				self.$nextTick(()=>{
					for(let obj in row){
						self.formData[obj] = row[obj];
					}
				});
			},
			//提交表单
			sureDialog(formData){
				let self = this;
				self.$refs[formData].validate((valid) => {
					if(valid){
						let data = {};
						let url = self.isEdit ? '/courseware/upData' : '/courseware/addData';
						url = AJAX_ADMIN +url ;
                        data = self.formData;
						if(self.isEdit){
							//编辑
							data.coursewareId = self.coursewareId;
						}
//                      alert(JSON.stringify(data));
						$v.post(url, data, (d) => {
							Message({
					            	type: 'success',
					            	message: self.isEdit ? '编辑成功' : '添加成功',
					            	duration: TIPS_TIEM
                                  });
                            self.dialogData.dialogVisible = false;
                            self.getData();
						})
					}
				})
			},
			//删除
			deleteData(row){
				let self = this;
				MessageBox.confirm("此操作将永久删除文件，是否继续？", "提示", {
					cancelButtonText: "取消",
					confirmButtonText: "确定",
    				type: 'warning',
    			}).then(()=>{
    				let url = AJAX_ADMIN + '/courseware/delData'
    				let data = {};
    				data.coursewareId = row._id;
//  				console.log(data)
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
				let url = AJAX_ADMIN + '/courseware/getData';
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
				self.$refs['upload'].clearFiles();//清空添加之后的 upload组件中的文件列表
				self.$refs['formData'].resetFields();
				self.dialogData.dialogVisible = false;
				done ? done() : '';
			},
			completeVideo(response, file, fileList){
                if (response.meta.result == 'SUCCESS') {
                    this.formData.contentUrl = response.data.filename;
                    this.$forceUpdate();  //重新渲染
                } else {
                    Message.error({message: response.meta.message, duration: TIPS_TIEM});
                }
            },
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
	div[data-tecourseware-box]{
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