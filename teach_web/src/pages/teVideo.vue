<template>
	<!--教学视频-->
	<div data-teVideo-box>
		<TeachBanner :modelChild = "modelFather"></TeachBanner>
		<div class="videoDiv">
			<el-collapse class="collapse" accordion v-model="chapterName">
			  	<el-collapse-item v-for="(item , index) in chapterlist" :title="item.chapterName" :name="item.chapterName" :key="index" >
			  		<span class="chapterTitle">{{item.chapterTitle}}</span>
	  				<div class="sectionDiv" v-for="(item2 , index2) in item.sectionlist" :key="index2">
	  					<span class="sectionName">{{item2.sectionName}}</span>
	  					<span class="sectionTitle">{{item2.sectionTitle}}</span>
	  					<span class="videoPlay" @click="goPlay(item2)">播放</span>
	  				</div>
			  	</el-collapse-item>
			</el-collapse>
		</div>
		<Footer></Footer>
        <Copyright></Copyright>
	</div>
</template>

<script>
	import Vue from 'vue';
	import TeachBanner from '~/component/common/TeachBanner';
	import Footer from '~/component/common/Footer';
    import Copyright from '~/component/common/Copyright';
    
    import {Collapse,CollapseItem} from 'element-ui';
    Vue.use(Collapse);
	Vue.use(CollapseItem);
    import {$v, srcUrl, AJAX_ADMIN, TIPS_TIEM, DateFormat} from '~/assets/js/const.js';
    
	export default {
		components:{
			TeachBanner,
			Footer,
       		Copyright,
		},
		data(){
			return{
				chapterlist: [],
				chapterName:'',
				modelFather: {
					srcUrl: require('../assets/img/banner_video.jpg'),
					title: '教学视频',
				}
			}
		},
		methods: {
			goPlay(item2){
				item2.chapterName = this.chapterName;
//				alert(JSON.stringify(item2))
				this.$router.push({path: '/teVideoItem',query:{obj:JSON.stringify(item2)}});
			}
		},
		mounted(){
			let url = AJAX_ADMIN + '/tevideo/getData'
			$v.get(url, (d)=>{
				this.chapterlist = d.data;
			})
		},
	}
</script>

<style lang="scss">
	@import '~/assets/css/response.scss';
	div[data-teVideo-box]{
		.videoDiv{
		    width: 90%;
			margin: 40px auto;
			.collapse{
			    position: relative;
			    border: 2px solid rgba(33,150,243,.43);
			    border-radius: 25px;
			    padding: 30px;
			    .el-collapse-item{
			    	.el-collapse-item__header{
		    		    border-bottom-color: #ebeef5;
					    height: 48px;
					    line-height: 48px;
					    font-size: 16px;
			    	}
			    	.is-active{
			    		color: rgba(0,8,255,.89);
			    	}
			    	.el-collapse-item__wrap{
			    		.el-collapse-item__content{
		    			    padding: 20px;
			    			.chapterTitle{
		    				    font-size: 16px;
							    line-height: 22px;
							    display: inline-block;
							    padding-bottom: 10px;
						        color: rgba(0,8,255,.89);
			    			}
			    			.sectionDiv{
		    				    padding: 10px 20px;
	    				        line-height: 24px;
	    				        display: flex;
		    				    .sectionName{
		    				    	display: inline-block;
		    				    	width: 80px;
		    				    }
		    				    .sectionTitle{
		    				    	display: inline-block;
		    				    	width: 100px;
		    				    	white-space: nowrap;
				    				overflow: hidden;
								    text-overflow: ellipsis;
		    				    }
		    				    .videoPlay{
		    				    	display: inline-block;
		    				    	width: 80px;
		    				    	padding-left: 20px;
		    				    	&:hover{
		    				    		cursor: pointer;
		    				    		color: rgba(0,8,255,.89);
		    				    	}
		    				    }
			    			}
			    		}
			    	}
			    }
			}
		}
		
		@include media-xs(){
			.videoDiv{
				width: 95%;
			    margin: 20px auto;
			    .collapse{
			    	padding: 15px 10px;
			    	.el-collapse-item{
			    		.el-collapse-item__header{
		    			    /*height: 40px;
		    			    line-height: 40px;
		    			    .el-icon-arrow-right{
		    			    	line-height: 40px;
		    			    }*/
			    		}
			    		.el-collapse-item__wrap{
			    			.el-collapse-item__content{
			    				padding: 10px;
			    			}
			    		}
			    	}
			    }
			}
		}
	}
</style>