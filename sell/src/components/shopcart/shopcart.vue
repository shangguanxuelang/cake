<template>
	<div> <!-- 由于vue2.0不支持template下main挂载多个根元素，所以在此创建div将shopcart和mark包裹进来 -->
	<div class="shopcart">
		<div class="content" @click="togglelist">
			<div class="content-left">
				<div class="logo-wrapper">
					<div class="logo" :class="{'highlight':totalCount>0}">
						<i class="icon-shopping_cart" :class="{'highlight':totalCount>0}"></i>
					</div>
					<div class="num" v-show='totalCount>0'>{{totalCount}}</div>
				</div>
				<div class="price" :class="{'highlight':totalPrice>0}">￥{{totalPrice}}</div>
				<div class="desc">另需配送费￥{{deliveryPrice}}元</div>
			</div>
			<div class="content-right" @click.stop.prevent="pay">
				<div class="pay" :class="payClass">
					{{payDesc}}
				</div>
			</div>
		</div>
		<div class="ball-container">
			<transition-group name="drop" v-on:before-enter="beforeEnter"
                          v-on:enter="enter"
                          v-on:after-enter="afterEnter">
				<div v-for="(ball,index) in balls" v-show="ball.show" class="ball" :key="index">
					<div class="inner inner-hook" :key="index"></div>
				</div>
			</transition-group>
		</div>
		<transition name="fold">
			<div class="shopcart-list" v-show="listShow">
				<div class="list-header">
					<h1 class="title">购物车</h1>
					<span class="empty" @click="empty">清空</span>
				</div>
				<div class="list-content" ref="listContent">
					<ul>
						<li class="food" v-for="food in selectFoods">
							<span class="name">{{food.name}}</span>
							<div class="price">
								<span>￥{{food.price*food.count}}</span>
							</div>
							<div class="cartcontrol-wrapper">
								<cartcontrol :food="food"></cartcontrol>
							</div>
						</li>
					</ul>
				</div>
			</div>
		</transition>
	</div>
	<transition name="fade">
		<div class="list-mask" v-show="listShow" @click="hideList"></div>
	</transition>
</div>	
</template>

<script>
	import BScroll from 'better-scroll';
	import cartcontrol from '@/components/cartcontrol/cartcontrol.vue';
	export default{
		props:{
			selectFoods:{
				type:Array,
				default(){
					return [];
				}
			},
			deliveryPrice:{
				type:Number,
				default:0
			},
			minPrice:{
				type:Number,
				default:0
			}
		},
		components:{
			cartcontrol,
		},
		data(){
			return{
				fold:true,
				balls:[
					{
						show:false
					},
					{
						show:false
					},
					{
						show:false
					},
					{
						show:false
					},
					{
						show:false
					}
				],
				dropBalls:[]
			};
		},
		computed:{
			listShow(){
				if(!this.totalCount){
					this.fold=true;
					return false;
				}
				var show=!this.fold;
				if(show){
					this.$nextTick(()=>{
						if(!this.scroll){
							this.scroll=new BScroll(this.$refs.listContent,{
								click:true
							});
						}else{
							this.scroll.refresh();
						}
						
					});
				}
				return show;
			},
			totalPrice(){
				var total=0;
				this.selectFoods.forEach(function(food){
					total+=food.price*food.count
				});
				return total;
			},
			totalCount(){
				var count=0;
				this.selectFoods.forEach(function(food){
					count+=food.count;
				});
				return count;
			},
			payDesc(){
				if(this.totalPrice===0){
					return `￥${this.minPrice}元起送`;
				}else if(this.totalPrice<this.minPrice){
					var diff=this.minPrice-this.totalPrice;
					return `还差￥${diff}元起送`;
				}else{
					return '去结算';
				}
			},
			payClass(){
				if(this.totalPrice<this.minPrice){
					return 'not-enough';
				}else{
					return 'enough';
				}
			}
		},
		methods:{
			pay:function(){
				if(this.totalPrice<this.minPrice){
					return;
				}
				window.alert(`支付${this.totalPrice}元`);
			},
			hideList:function(){
				this.fold=true;
			},
			empty:function(){
				this.selectFoods.forEach(function(food){
					food.count=0;
				})
			},
			togglelist:function(){
				if(!this.totalCount){
					return;
				}
				this.fold=!this.fold;
			},
			drop:function(el){
				for(var i = 0; i < this.balls.length; i++){
					var ball = this.balls[i];
					if(!ball.show){
						ball.show=true;
						ball.el=el;
						this.dropBalls.push(ball);
						return;
					}
				}
			},
			beforeEnter(el) {
//				console.log(this.balls);
		        //el元素表示小球的div，ball.el表示加按钮
		        var count = this.balls.length;
		        while (count--) {
		          var ball = this.balls[count];
		          if (ball.show) {
		            var rect = ball.el.getBoundingClientRect();
		            var x = rect.left - 32;
		            var y = -(window.innerHeight - rect.top - 22);
//		            console.log(x);
//		            console.log(y);
		            el.style.display = '';
		            el.style.webkitTransform = `translate3d(0,${y}px,0)`;
		            el.style.transform = `translate3d(0,${y}px,0)`;
		            var inner = el.getElementsByClassName('inner-hook')[0];
		            inner.style.webkitTransform = `translate3d(${x}px,0,0)`;
		            inner.style.transform = `translate3d(${x}px,0,0)`;
		          }
		        }
		      },
		      enter(el) {
		        /* eslint-disable no-unused-vars */
		        var rf = el.offsetHeight;
		        this.$nextTick(() => {
		          el.style.webkitTransform = 'translate3d(0,0,0)';
		          el.style.transform = 'translate3d(0,0,0)';
		          var inner = el.getElementsByClassName('inner-hook')[0];
		          inner.style.webkitTransform = 'translate3d(0,0,0)';
		          inner.style.transform = 'translate3d(0,0,0)';
		        });
		      },
		      afterEnter(el) {
		        var ball = this.dropBalls.shift();
		        if (ball) {
		          ball.show = false;
		          el.style.display = 'none';
		        }
		      }
		}
	}
</script>

<style lang="less">
	@import "../../common/stylus/mixin.less";
	.shopcart{
		position: fixed;
		left: 0;
		bottom: 0;
		z-index: 50;
		width: 100%;
		height: 48px;
		background: #000;
		.content{
			display: flex;
			background: #141d27;
			font-size: 0;
			color: rgba(255,255,255,0.4);
			.content-left{
				flex: 1;
				.logo-wrapper{
					display: inline-block;
					vertical-align: top;
					position: relative;
					top: -10px;
					margin: 0 12px;
					padding: 6px;
					width: 56px;
					height: 56px;
					box-sizing: border-box;
					border-radius: 50%;
					background: #141d27;
					.num{
						position: absolute;
						top: 0;
						right: 0;
						width: 24px;
						height: 16px;
						line-height: 16px;
						text-align: center;
						border-radius: 16px;
						font-size: 9px;
						font-weight: 700;
						color: #fff;
						background: rgb(240,20,20);
						box-shadow: 0 4px 8px 0 rgba(0,0,0,0.4);
					}
					.logo{
						width: 100%;
						height: 100%;
						border-radius: 50%;
						background: #2d343c;
						text-align: center;
						&.highlight{
							background: rgb(0,160,220);
						}
						.icon-shopping_cart{
							font-size: 24px;
							color: #80858a;
							line-height: 44px;
							&.highlight{
								color: #fff;
							}
						}
						
					}
				}
				.price{
					display: inline-block;
					vertical-align: top;
					margin-top: 12px;
					line-height: 24px;
					padding-right: 12px;
					box-sizing: border-box;
					border-right: 1px solid rgba(255,255,255,0.1);
					font-size: 16px;
					font-weight: 700;
					color: rgba(255,255,255,0.4);
					&.highlight{
						color: #fff;
					}
				}
				.desc{
					display: inline-block;
					vertical-align: top;
					margin: 12px 0 0 12px;
					line-height: 24px;
					font-size: 10px;
				}
			}
			.content-right{
				flex:0 0 105px;
				width: 105px;
				.pay{
					height: 48px;
					line-height: 48px;
					text-align: center;
					font-size: 12px;
					font-weight: 700;
					background: #2b333b;
					&.not-enough{
						background: #2b333b;
					}
					&.enough{
						background: #00b43c;
						color: #fff;
					}
				}
			}
		}
		.ball-container{
			.ball{
				position: fixed;
				left: 32px;
				bottom: 22px;
				z-index: 200;
				&.drop-enter-active, &.drop-leave-active{
					transition: all 0.5s cubic-bezier(0.49,-0.29,0.75,0.41);
			        .inner{
			        	width: 16px;
				        height: 16px;
				        border-radius: 50%;
				        background: rgb(0, 160, 220);
				        transition: all 0.5s linear;
			        }
				}
		        
				/*&.drop-enter-to,&.drop-leave{
					transition: all 0.4s;
					.inner{
						transition: all 0.4s;
					}
				}*/
			}
		}
		.shopcart-list{
			position: absolute;
			left: 0;
			top: 0;
			z-index: -1;
			width: 100%;
			transform: translate3d(0, -100%, 0);
		    &.fold-enter-active, &.fold-leave-active{
		    	transition: all 0.5s;
		    }
		    &.fold-enter, &.fold-leave-to{
		    	transform: translate3d(0, 0, 0);
		    }
		    .list-header{
		    	height: 40px;
		    	line-height: 40px;
		    	padding: 0 18px;
		    	background: #f3f5f7;
		    	border-bottom: 1px solid rgba(7,17,27,0.1);
		    	.title{
		    		float: left;
		    		font-size: 14px;
		    		color: rgb(7,17,27);
		    	}
		    	.empty{
		    		float: right;
		    		font-size: 12px;
		    		color: rgb(0,160,220);
		    	}
		    }
		    .list-content{
		    	padding: 0 18px;
		    	max-height: 217px;
		    	overflow: hidden;
		    	background: #fff;
		    	ul{
		    		.food{
		    			position: relative;
		    			padding: 12px 0;
		    			box-sizing: border-box;
		    			.border-1px-after(rgba(7,17,27,0.1));
		    			.name{
		    				line-height: 24px;
		    				font-size: 14px;
		    				color: rgb(7,17,27);
		    			}
		    			.price{
		    				position: absolute;
		    				right: 90px;
		    				bottom: 12px;
		    				line-height: 24px;
		    				font-size: 14px;
		    				font-weight: 700;
		    				color: rgb(240,20,20);
		    			}
		    			.cartcontrol-wrapper{
		    				position: absolute;
		    				right: 0;
		    				bottom: 6px;
		    			}
		    		}
		    	}
		    }
		}
	}
	.list-mask{
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		z-index: 40;
		backdrop-filter: blur(10px);
		background: rgba(7, 17, 27, 0.6);
		transition: all 0.5s;
		&.fade-enter-to,&.fade-leave{
		    opacity: 1;
		}
		&.fade-enter,&.fade-leave-to{
			opacity: 0;
		}
		  
	}
</style>