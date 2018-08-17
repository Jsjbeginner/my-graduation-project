<template>
	<!--教学课件-->
	<div data-teCourseware-box>
		<TeachBanner :modelChild = "modelFather"></TeachBanner>
		<TeachCourseware></TeachCourseware>
		<Footer></Footer>
        <Copyright></Copyright>
	</div>
</template>

<script>
	import Vue from 'vue';
	import TeachBanner from '~/component/common/TeachBanner';
	import TeachCourseware from '~/component/common/TeachCourseware';
	import Footer from '~/component/common/Footer';
    import Copyright from '~/component/common/Copyright';
    
	export default {
		components:{
			TeachBanner,
			TeachCourseware,
			Footer,
       		Copyright,
		},
		data(){
			return{
				data:{},
				modelFather:{
					srcUrl: require('../assets/img/banner_courseware.jpg'),
					title: '教学课件',
				}
			}
		},
		methods: {
			
		},
		mounted(){
			
		},
	}
</script>

<style lang="scss">
	@import '~/assets/css/response.scss';
	div[data-teCourseware-box]{
		
		
		
		
		@include media-xs(){
			
		}
	}
</style>