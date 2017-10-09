<template>
	<div class="goods">
		<div class="menu-wrapper" ref="menuWrapper">
			<ul>
				<li v-for="(item,index) in goods" class="menu-item" :class="{'current':currentIndex===index}" @click="selectMenu(index,$event)">
					<span class="text border-1px-after"><span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>{{item.name}}</span>
				</li>
			</ul>
		</div>
		<div class="foods-wrapper" ref="foodsWrapper">
			<ul>
				<li v-for="item in goods" class="food-list food-list-hook">
					<h1 class="title">{{item.name}}</h1>
					<ul>
						<li @click="selectFood(food,$event)" v-for="food in item.foods" class="food-item border-1px-after">
							<div class="icon">
								<img width="57px" height="57px" :src="food.icon" />
							</div>
							<div class="content">
								<h2 class="name">{{food.name}}</h2>
								<p class="desc">{{food.description}}</p>
								<div class="extra">
									<span class="count">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
								</div>
								<div class="price">
									<span class="now">￥{{food.price}}</span>
									<span v-show="food.oldPrice" class="old">￥{{food.oldPrice}}</span>
								</div>
								<div class="cartcontrol-wrapper">
									<cartcontrol @event="getevent" :food="food"></cartcontrol>
								</div>
							</div>
						</li>
					</ul>
				</li>
			</ul>
		</div>
		<shopcart ref="shopcart" :select-foods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice" v-if="seller"></shopcart>
		
		<food :food="selectedFood" ref="food" @event="getevent" @foodEv="getevent" ></food>
	</div>
	
</template>

<script>
	import BScroll from 'better-scroll';
	import shopcart from '@/components/shopcart/shopcart.vue';
	import cartcontrol from '@/components/cartcontrol/cartcontrol.vue';
	import food from '@/components/food/food.vue';
	const ERR_OK=0;
	export default{
		props:{
			seller:{
				type:Object
			}
		},
		components:{
			shopcart,
			cartcontrol,
			food
		},
		data:function(){
			return{
				goods:[],
				listHeight:[],
				scrollY:0,
				selectedFood:{}
			}
		},
		computed:{
			currentIndex:function(){
				for(var i=0;i<this.listHeight.length;i++){
					var height1=this.listHeight[i];
					var height2=this.listHeight[i+1];
//					console.log(this.scrollY);
					if(!height2||(this.scrollY>=height1&&this.scrollY<height2)){
						return i;
					}
				}
				return 0;
			},
			selectFoods:function(){
				var foods=[];
				this.goods.forEach(function(good){
					good.foods.forEach(function(food){
						if(food.count){
							foods.push(food);
						}
					})
				})
				return foods;
			}
		},
		created(){
			this.classMap=['decrease','discount','guarantee','special','invoice'],
	  
			this.$http.get('./api/goods').then(function(res){
				res=res.body;
		  		if(res.errno===ERR_OK){
		  			this.goods=res.data;
//		  			console.log(this.goods);
					this.$nextTick(function(){
						this._initScroll();
						this._calculateHeight();
					});
		  		}
			})
		},
		methods:{
//			getFood(el) {
//		        this.$nextTick(() => {
//		          this.$refs.shopcart.drop(el);
//		        });
//		    },
			selectFood:function(food,event){
				if(!event._constructed){
					return;
				}
				this.selectedFood=food;
				this.$refs.food.show();
			},
			selectMenu:function(index,event){
				if(!event._constructed){
					return;
				}
				var foodList=this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
				var el=foodList[index];
				this.foodsScroll.scrollToElement(el,300);
//				console.log(index);
			},
			_initScroll:function(){
				var _this = this;
				this.menuScroll=new BScroll(this.$refs.menuWrapper,{
					click:true
				});
				this.foodsScroll=new BScroll(this.$refs.foodsWrapper,{click:true,probeType:3});
				this.foodsScroll.on('scroll',function(pos){
					_this.scrollY=Math.abs(Math.round(pos.y));
//					console.log(this)
				});
//				console.log(_this.scrollY)
			},
			_calculateHeight:function(){
				var foodList=this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
				var height=0;
				this.listHeight.push(height);
				for(var i=0;i<foodList.length;i++){
					var item=foodList[i];
					height+=item.clientHeight;
					this.listHeight.push(height);
				}
			},
			getevent(el) {
//				console.log(1231);
		        // 体验优化,异步执行下落动画
		        this.$nextTick(() =>{
		          this.$refs.shopcart.drop(el);
		        });
		      }
		}
	}
</script>

<style lang="less">
	@import "../../common/stylus/mixin.less";
	.goods{
		display: flex;
		position: absolute;
		width: 100%;
		top: 174px;
		bottom: 46px;
		overflow: hidden;
		.menu-wrapper{
			flex:0 0 80px;
			width: 80px;
			background: #f3f5f7;
			.menu-item{
				display: table;
				height: 54px;
				width: 56px;
				padding:0 12px;
				line-height: 14px;
				&.current{
					position: relative;
					z-index: 10;
					margin-top: -1px;
					background: #fff;
					font-weight: 700;
					.text{
						.border-none();
					}
				}
				.icon{
					display: inline-block;
					vertical-align: top;
					width: 12px;
					height: 12px;
					margin-right: 2px;
					background-size: 12px 12px;
					background-repeat: no-repeat;
					&.decrease{
						.bg-image('decrease_3')
					};
					&.discount{
						.bg-image('discount_3')
					};
					&.guarantee{
						.bg-image('guarantee_3')
					};
					&.invoice{
						.bg-image('invoice_3')
					};
					&.special{
						.bg-image('special_3')
					}
				}
				.text{
					display: table-cell;
					width: 56px;
					vertical-align: middle;
					.border-1px-after(rgba(7,17,27,0.1));
					font-size: 12px;
				}
			}
		}
		.foods-wrapper{
			flex:1;
			background:#fff;
			ul{
				.food-list{
					.title{
						padding-left:14px;
						height:26px;
						line-height:26px;
						border-left:2px solid #d9dde1;
						font-size: 12px;
						color: rgb(147,153,159);
						background: #f3f5f7;
					}
					ul{
						.food-item{
							display: flex;
							margin: 18px;
							padding-bottom: 18px;
							.border-1px-after(rgba(7,17,27,0.1));
							&:last-child{
								.border-none();
								margin-bottom: 0;
							}
							.icon{
								flex: 0 0 57px;
								margin-right: 10px;
							}
							.content{
								flex: 1;
								.name{
									margin: 2px 0 8px 0;
									height: 14px;
									line-height: 14px;
									font-size: 14px;
									color: rbg(7,17,27);
								}
								.desc,.extra{
									line-height: 10px;
									font-size: 10px;
									color: rgb(147,153,159);
								}
								.desc{
									margin-bottom: 8px;
									line-height: 12px;
								}
								.extra{
									.count{
										margin-right: 12px;
									}
								}
								.price{
									font-weight: 700;
									line-height: 24px;
									.now{
										margin-right: 8px;
										font-size: 14px;
										color: rgb(240,20,20);
									}
									.old{
										text-decoration: line-through;
										font-feature-settings: 10px;
										color: rgb(147,153,159);
									}
								}
								.cartcontrol-wrapper{
									position: absolute;
									right: 0;
									bottom: 12px;
								}
							}
						}
					}
				}
			}
		}
	}
</style>