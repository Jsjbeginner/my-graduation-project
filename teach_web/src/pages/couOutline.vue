<template>
	<!--课程大纲-->
	<div data-cououtline-box>
		<TeachBanner :modelChild = "modelFather" class="banner"></TeachBanner>
		<!--<pre>{{data}}</pre>-->
		<div class="content">
			<div class="top">
				<span class="title">{{data.courseName}}</span>
				<span class="title titleEn">{{data.courseNameEn}}</span>
				<div class="item">
					<span class="name">课程代码：</span><span class="cntent">{{data.courseCode}}</span>
				</div>
				<div class="item itemtwo">
					<div class="leftDiv"><span class="name">学时数：</span><span class="cntent">{{data.period}}</span></div>
					<div class="rightDiv"><span class="name">学分数：</span><span class="cntent">{{data.credit}}</span></div>
				</div>
				<div class="item itemtwo">
					<div class="leftDiv"><span class="name">执笔人：</span><span class="cntent">{{data.penner}}</span></div>
					<div class="rightDiv rightDivTwo"><span class="name">讨论参加人：</span><span class="cntent">{{data.discussants}}</span></div>
				</div>
				<div class="item">
					<span class="name">审核人：</span><span class="cntent">{{data.auditor}}</span>
				</div>
			</div>
			<div class="bottom">
				<span class="title">大纲详情</span>
				<Richtext :address="data.textUrl" v-if="data.textUrl"></Richtext>
			</div>
		</div>
		<Footer></Footer>
        <Copyright></Copyright>
	</div>
</template>

<script>
	import Vue from 'vue';
	import TeachBanner from '~/component/common/TeachBanner';
	import Richtext from '~/component/common/RichText';
	import Footer from '~/component/common/Footer';
    import Copyright from '~/component/common/Copyright';
    import {$v, srcUrl, AJAX_ADMIN, TIPS_TIEM, DateFormat} from '~/assets/js/const.js';
	export default {
		components:{
			TeachBanner,
			Richtext,
			Footer,
       		Copyright,
		},
		data(){
			return{
				data:{},
				modelFather: {
					srcUrl: require('../assets/img/banner_outline.jpg'),
					title: '课程大纲',
				}
			}
		},
		methods: {
			
		},
		mounted(){
			let url = AJAX_ADMIN + '/outline/getData';
			$v.get(url, (d) => {
				this.data = d.data[0];
			});
		},
	}
</script>

<style lang="scss">
	@import '~/assets/css/response.scss';
	div[data-cououtline-box]{
		.banner{
			background-color: rgba(63, 81, 181, 0.09);
		}
		.content{
			width: 100%;
			.top{
				background-color: rgba(63, 81, 181, 0.09);
				padding: 40px 0px;
				.title{
					display: block;
					width: 80%;
	    			margin: auto;
					text-align: center;
					padding: 20px 0px;
					font-size: 18px;
					line-height: 26px;
					white-space: nowrap;
				    overflow: hidden;
				    text-overflow: ellipsis;
				}
				.titleEn{
					width: 85%;
		    	    padding-top: 0px;
			    }
				.item{
				    width: 60%;
				    margin: 20px auto;
				    display: flex;
				    align-items: center;
				    &:first-child{
				    	background-color: red;
				    }
				    /*justify-content: space-between;*/
					>span{
						line-height: 24px;
	    				font-size: 16px;
	    				/*white-space: nowrap;
				    	overflow: hidden;
				    	text-overflow: ellipsis;*/
					}
					.name{
						color: #4CAF50;
					    width: 100px;
					}
					.more{
						width: 100%;
						text-align: right;
	   					padding-right: 15px;
						&:hover{
							color: red;
						    cursor: pointer;
						}
					}
					.leftDiv{
						width: 50%;
					    display: flex;
	    				align-items: center;
						.name{
							line-height: 24px;
	    					font-size: 16px;
						    width: 100px;
	    					display: inline-block;
						}
						.cntent{
							line-height: 24px;
	    					font-size: 16px;
						    width: calc(100% - 100px);
	    					display: inline-block;
							white-space: nowrap;
					    	overflow: hidden;
					    	text-overflow: ellipsis;
						}
					}
					.rightDiv{
						padding-left: 10px;
						display: flex;
	    				align-items: center;
	    				>span{
	    					line-height: 24px;
	    					font-size: 16px;
	    					width: 100px;
	    					display: inline-block;
	    				}
					}
					.rightDivTwo{
						.name{
							width: 120px;
						}
						.cntent{
							width: calc(100% - 120px);
	    					display: inline-block;
							white-space: nowrap;
					    	overflow: hidden;
					    	text-overflow: ellipsis;
	    				}
					}
				}   
				.itemtwo{
					justify-content: space-between;
				}
			}
			.bottom{
				width: 100%;
				background-color: rgba(63,81,181,.09);
				padding-bottom: 40px;
				.title{
					/*color: #4CAF50;*/
					display: block;
					width: 80%;
	    			margin: auto;
					text-align: center;
					padding: 20px 0px;
					font-size: 18px;
					line-height: 26px;
				}
			}
		}
		
		@include media-sm(){
			.title{
				width: 90% !important;
			}
			.item{
				width: 85% !important;
			}
		}
		
		
		@include media-xs(){
			.content{
				.top{
					padding: 20px 0px;
					.title{
						font-size: 16px;
						line-height: 22px;
						padding: 10px 0px;
						width: 95%;
					}
					.titleEn{
						padding-top: 0px;
					}
					.item{
						display: block;
						width: 95%;
						margin: 10px 0px 10px 20px;
						.name{
							width: 100px;
							display: inline-block;
						}
						.leftDiv{
							width: 100%;
							margin-bottom: 10px;
							.name{
							    width: 100px;
							}
						}
						.rightDiv{
							padding-left: 0px;
						}
						.rightDivTwo{
							.name{
								width: 100px;
							}
						}
					}
				}
				.bottom{
					width: 100%;
					padding-bottom: 20px;
					.title{
						font-size: 16px;
						line-height: 22px;
						padding: 10px 0px;
						width: 95%;
					}
				}
			}
		}
	}
</style>