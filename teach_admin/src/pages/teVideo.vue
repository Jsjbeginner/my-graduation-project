<template>
	<div data-tepublictly-box>
		<el-button type="primary" @click="addData('chapter')">添加</el-button>
        <!-- 二级列表 -->
        <el-table :data="tableData" border class="table" :expand-row-keys="expandData.expands" :row-key="expandData.getRowKeys" @expand-change="handleExpand">
            <el-table-column type="expand" width="80">
                <template slot-scope="scope">
                    <el-table class="childTable" :data="scope.row.sectionlist == '' ? [] : scope.row.sectionlist" :show-header="false" border >
                        <el-table-column label="章节名" prop="sectionName"></el-table-column>
                        <el-table-column label="章节标题" prop="sectionTitle"></el-table-column>
                        <el-table-column label="操作" prop="handle">
                            <template slot-scope="scope">
                                <el-button type="text" @click="editSection(scope.row, scope.$index)">编辑</el-button>
                                <el-button type="text" @click="del(scope.row,'section')">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </template>
            </el-table-column>
            <el-table-column label="章节名" prop="chapterName"></el-table-column>
            <el-table-column label="章节标题" prop="chapterTitle"></el-table-column>
            <el-table-column label="操作" prop="handle">
                <template slot-scope="scope">
                    <el-button type="text" @click="addData('section',scope.row)">添加</el-button>                    
    				<el-button type="text" @click="editChapter(scope.row, scope.$index)">编辑</el-button>
                    <el-button type="text" @click="del(scope.row,'chapter')">删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <!--章的dialog-->
        <el-dialog :title="dialogChapter.dialogTitle" :visible.sync="dialogChapter.dialogVisible" :before-close="closeChapter" @close="closeChapter">
        	<el-form :model="formChapter" ref="formChapter" :rules="rules2" class="formchapter" label-position="left">
				<el-form-item label="章名:" prop="chapterName">
					<el-input v-model="formChapter.chapterName" placeholder="请输入章名"></el-input>
				</el-form-item>
				<el-form-item label="章标题:" prop="chapterTitle">
					<el-input v-model="formChapter.chapterTitle" placeholder="请输入章标题"></el-input>
				</el-form-item>
			</el-form>
        	<div class="button_bottom">
				<el-button @click="sureData('formChapter','chapter')" type="primary">确 定</el-button>
				<el-button @click="closeChapter()">取 消</el-button>				
			</div>
		</el-dialog>
		<!--节的dialog-->
		<el-dialog :title="dialogSection.dialogTitle" :visible.sync="dialogSection.dialogVisible" :before-close="closeSection" @close="closeSection">
			<el-form :model="formSection" ref="formSection" :rules="rules" class="formsection"label-position="left">
				<el-form-item label="节名:" prop="sectionName">
					<el-input v-model="formSection.sectionName" placeholder="请输入节名"></el-input>
				</el-form-item>
				<el-form-item label="节标题:" prop="sectionTitle">
					<el-input v-model="formSection.sectionTitle" placeholder="请输入节标题"></el-input>
				</el-form-item>
				<el-form-item label="宣传视频:" prop="videoUrl"  class="txt">
                    <el-upload
                        class="upload-demo"
                        :action="AJAX_ADMIN + '/upload'"
                        :accept="'video/mp4'"
                        :on-success="completeVideo"
                        ref="upload"
                        :limit="1"
                    >
                        <el-button size="small" type="primary">{{ formSection.videoUrl ? '更换文件' : '点击上传'}}</el-button>
                        <div slot="tip" class="el-upload__tip">{{ formSection.videoUrl ? '已上传' : '请选择视频文件'}}</div>
                    </el-upload>
                    <p>视频格式支持MP4</p>
                </el-form-item>
			</el-form>
			<div class="button_bottom">
				<el-button @click="sureData('formSection','section')" type="primary">确 定</el-button>
				<el-button @click="closeSection()">取 消</el-button>				
			</div>
		</el-dialog>
		
		
	</div>
</template>

<script>
	import Vue from 'vue';
	import {$v,TIPS_TIEM,AJAX_ADMIN} from '~/assets/js/const.js';
	
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
			return{
				AJAX_ADMIN: AJAX_ADMIN,
				tableData: [],
				isEdit: false, // true:编辑,false:新增
				fatherId: '',  //存储对应节的父集Id
				expandData: {
                    // 获取row的key值
                    getRowKeys(row) {
                        return row._id;
                    },
                    // 要展开的行，数值的元素是row的key值
                    expands: []
                },
				chapterId: '', //章ID
				sectionlist: [], //对应章的节数组
                //章的dialog
				dialogChapter: {
					dialogTitle: '',
					dialogVisible: false,
				},
				//章的formData
				formChapter: {
					chapterName: '',
					chapterTitle: '',
				},
				rules2: {
					chapterName:[
						{ required: true, message: '请填写章名称', trigger: 'blur' },
					],
					chapterTitle:[
						{ required: true, message: '请填写章标题', trigger: 'blur' },
					],
				},
				
				sectionId: '', //节唯一标识
				//节的dialog
				dialogSection: {
					dialogTitle: '',
					dialogVisible: false,
				},
				//节的formData
				formSection: {
					videoUrl: '',
					sectionName:'',//节名
					sectionTitle:'',//节标题
				},
				rules: {
					sectionName:[
						{ required: true, message: '请填写节名称', trigger: 'blur' },
					],
					sectionTitle:[
						{ required: true, message: '请填写节标题', trigger: 'blur' },
					],
				}
			}
		},
		methods:{
			//新增,type=chapter为章,否则为节
			addData(type,row){
				let self = this;
				self.isEdit = false;
				if(type == 'chapter'){
					self.dialogChapter.dialogTitle = '新增章';
					self.dialogChapter.dialogVisible = true;
				} else {
					self.fatherId = row._id;
					self.dialogSection.dialogTitle = '新增节';
					self.dialogSection.dialogVisible = true;
				}
			},
			//编辑章
			editChapter(row,index){
				let self = this;
				self.isEdit = true;
				self.chapterId = row._id;
				self.sectionlist = row.sectionlist;
				self.dialogChapter.dialogTitle = '编辑章';
				self.dialogChapter.dialogVisible = true;
				self.$nextTick(()=>{
					for(let obj in row){
						if(obj =='chapterName' || obj =='chapterTitle') {
							self.formChapter[obj] = row[obj];
						}
					}
				});
			},
			//编辑节
			editSection(row, index){
//				alert(JSON.stringify(row))
				let self = this;
				self.isEdit = true;
				self.sectionId = row.time;
				self.dialogSection.dialogTitle = '编辑小节';
				self.dialogSection.dialogVisible = true;
				self.$nextTick(()=>{
					for(let obj in row){
						self.formSection[obj] = row[obj];
					}
				});
			},
			//提交章
//			sureChapter(formChapter){
//				let self = this;
//				self.$refs[formChapter].validate((valid) => {
//					if(valid){
//						let data = {};
//						let url = self.isEdit ? '/business/updVideo' : '/business/addVideo';
////						url = AJAX_CMS_ADMIN +url ;
//                      data = self.formChapter;
//                      alert(JSON.stringify(data))
//						if(self.isEdit){
//							//编辑
//							data.chapterId = self.chapterId;
//						}
//						data.type = 'chapter';
//                      alert(JSON.stringify(data))
////						$v.post(this,url, data, (d) => {
////							Message({
////					            	type: 'success',
////					            	message: self.isEdit ? '编辑成功' : '添加成功',
////					            	duration: TIPS_TIEM
////                                });
////                          self.dialogChapter.dialogVisible = false;
////                          self.getData();
////						})
//					}
//				})
//			},
////			提交节
//			sureSection(formSection){
//				let self = this;
//				self.$refs[formSection].validate((valid) => {
//					if(valid){
//						let data = {};
//						let url = self.isEdit ? '/business/updVideo' : '/business/addVideo';
////						url = AJAX_CMS_ADMIN +url ;
//                      data = self.formSection;
//                      alert(JSON.stringify(data))
//						if(self.isEdit){
//							//编辑
//							data.sectionId = self.sectionId;
//						}
//						data.type = 'section';
//                      alert(JSON.stringify(data))
//
//					}
//				})
//			},
			//提交数据
			sureData(refName,type){
//				alert(type)
				let self = this;
				self.$refs[refName].validate((valid) => {
					if(valid){
						let data = {};
						let url = '';
//						let url = self.isEdit ? '/business/upVideo' : '/business/addVideo';
						if(type == 'chapter'){
							data = self.formChapter;
							if(self.isEdit){
								url = "/tevideo/upChapterData"
								data.chapterId = self.chapterId;
								data.sectionlist = self.sectionlist;
							} else{
								url = "/tevideo/addChapterData";
								data.sectionlist = [];
							}
						} else {
							data = self.formSection;
							data.fatherId = this.fatherId; //新增或编辑都需要父ID
							if(self.isEdit){
								url = "/tevideo/upSectionData"
								data.time = self.sectionId;
							} else {
								let time = new Date().getTime();
								data.time = String(time); //时间撮
								url = "/tevideo/addSectionData"
							}
						}
//						alert(JSON.stringify(data));
						url = AJAX_ADMIN + url;
						$v.post(url, data, (d) => {
							Message({
					            	type: 'success',
					            	message: self.isEdit ? '编辑成功' : '添加成功',
					            	duration: TIPS_TIEM
                                  });
							if(type == 'chapter'){
								self.dialogChapter.dialogVisible = false;
							} else {
								self.dialogSection.dialogVisible = false;
							}
                            self.getData();
						})
					}
				})
			},
			//删除
			del(row,type){
				let self = this;
				MessageBox.confirm("此操作将永久删除文件，是否继续？", "提示", {
					cancelButtonText: "取消",
					confirmButtonText: "确定",
    				type: 'warning',
    			}).then(()=>{
    				let url = '';
    				let data = {};
    				if(type == 'chapter'){
    					//章的删除
    					url = AJAX_ADMIN + '/tevideo/delChapterData';
    					data.chapterId = row._id;
    				} else {
    					//节的删除
    					url = AJAX_ADMIN + '/tevideo/delSectionData';
    					data.fatherId = self.fatherId;
    					data.time = row.time;
    				}
//  				alert(JSON.stringify(data))
    				$v.post(url, data, (d)=>{
    					Message({
			            	type: 'success',
			            	message: '删除成功！',
			            	duration: 2000
			          	});
			          	this.getData();
    				})
    			}).catch((e)=>{
    				console.log(e.message)
    				Message({
					    type: 'info',
					    message: '已取消删除',
		            	duration: 2000
					});
    			})
			},
			//获取数据
			getData(){
				let url = AJAX_ADMIN + '/tevideo/getData';
				$v.get(url, (d)=>{
					this.tableData = d.data;
				})
			},
			/******************* Dialog 清除数据************/
			//章的dialog关闭时清除数据
			closeChapter(done){
				let self = this;
				for(let i in self.formChapter){
					self.formChapter[i] = '';
				}
//				self.$refs['upload'].clearFiles();
				self.$refs['formChapter'].resetFields();
				self.dialogChapter.dialogVisible = false;
				done ? done() : '';
			},
			//节的dialog关闭时清除数据
			closeSection(done){
				let self = this;
				for(let i in self.formSection){
					self.formSection[i] = '';
				}
				self.$refs['upload'].clearFiles();
				self.$refs['formSection'].resetFields();
				self.dialogSection.dialogVisible = false;
				done ? done() : '';
			},
			//视频上传函数
			completeVideo(response, file, fileList){
                if (response.meta.result == 'SUCCESS') {
                    this.formSection.videoUrl = response.data.filename;
                    this.$forceUpdate();  //重新渲染
                } else {
                    Message.error({message: response.meta.message, duration: TIPS_TIEM});
                }
            },
            //table的某一行展开或者关闭的触发该事件
            handleExpand(row, expanded) {
            	//实现单选
              	let isHave = null; //true 为展开row.false为关闭row
            	expanded.map((item)=>{
            		if(item == row){
            			isHave = true;
            		} else {
            			isHave = false;
            		}
            	})
                if (isHave) {
                    this.expandData.expands = [row._id];
                } else {
                	this.expandData.expands = [];
                }
                this.fatherId = this.expandData.getRowKeys(row);
            	//实现多选,但是暂时不支持可以获取到节的父Id
//          	let isHave = null; //true 为展开row.false为关闭row
//          	expanded.map((item)=>{
//          		if(item == row){
//          			isHave = true;
//          		} else {
//          			isHave = false;
//          		}
//          	})
//              if (isHave) {
//                  this.expandData.expands.push(row.chapterId);
//              } else {
//                  let arr = [];
//                  arr.push(row.chapterId);
//                  this.expandData.expands = this.expandData.expands.filter((item) => {
//                      if (String(arr).indexOf(item) == -1) {
//                          return item;
//                      }
//                  })
//              }
                
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
	/*@import '~/assets/css/public.scss';*/
	div[data-tepublictly-box]{
		.table{
			margin-top: 20px;
		}
        .el-table__header-wrapper {
            thead {
                tr th:last-child {
                    border-right: none;
                }
            }
        }
        .el-table__body-wrapper{
        	.el-table__row{
        		td {
	    			padding: 5px 0;
	        	}
        	}
        	tr .el-table__expanded-cell{
    		    padding: 20px 0px 20px 80px;
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