<template>
  <div id="app">
    <v-header :seller="seller"></v-header>
    <div class="tab border-1px-before border-1px-after">
    	<div class="tab-item">
    		 <router-link to="/goods">商品</router-link>
    	</div>
    	<div class="tab-item">
    		<router-link to="/ratings">评论</router-link>
    	</div>
    	<div class="tab-item">
    		<router-link to="/seller">商家</router-link>
    	</div>
    </div>
    <!--<v-goods></v-goods>-->
    <router-view :seller="seller"></router-view>
  </div>
</template>

<script>
	import header from "@/components/header/header.vue"
	import goods from "@/components/goods/goods.vue"
//	import ratings from "@/components/ratings/ratings.vue"
	
	const ERR_OK=0;
export default {
  name: 'app',
  data(){
  	return{
  		seller:{}
  	};
  },
  created(){
  	this.$http.get('/api/seller').then(function(res){
  		res=res.body;
  		if(res.errno===ERR_OK){
  			this.seller=res.data;
//			console.log(this.seller);
  		}
  	})
  },
  components:{
  	'v-header':header,
  	'v-goods':goods,
//	'ratings':ratings
  }
}
</script>

<style lang="less">
	@import "./common/stylus/index.less";
	#app{
		.tab{
			display: flex;
			width: 100%;
			height: 40px;
			line-height: 40px;
			.border-1px-after(rgba(7,17,27,0.1));
			.border-1px-before(rgba(7,17,27,0.1));
			.tab-item{
				flex: 1;
				text-align: center;
				font-size: 14px;
				color: rgb(77,85,93);
				a{
					display: block;
				}
				.active{
					color: red;
				}
			}
		}
	}
</style>
