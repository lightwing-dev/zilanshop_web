<template>
	<view>
		<view class="header" :style="{position:headerPosition,top:headerTop}">
			<view class="target" v-for="(target,index) in orderbyList" @tap="select(index)" :key="index" :class="[target.selected?'on':'']">
				{{target.text}}
				<view v-if="target.orderbyicon" class="icon" :class="target.orderbyicon[target.orderby]"></view>
			</view>
		</view>
		<!-- 占位 -->
		<view class="place"></view>
		<!-- 商品列表 -->
		<view class="goods-list">
			<view class="product-list">
				<view class="product" v-for="(goods) in goodsList" :key="goods.goods_id" @tap="toGoods(goods)">
					<image mode="widthFix" :src="CONSTANT.baseURL+goods.img"></image>
					<view class="name">{{goods.gname}}</view>
					<view class="info">
						<view class="price">{{goods.price}}</view>
						<view class="slogan">销量:{{goods.salesVolume}}</view>
					</view>
				</view>
			</view>
			<view class="loading-text" v-if="goodsList.length==0">{{loadingText}}</view>
		</view>
	</view>
</template>

<script>
	import CONSTANT from '@/common/constant.js'
	export default {
		data() {
			return {
				CONSTANT: CONSTANT,
				goodsList: [],
				pageBean: [],
				loadingText: "暂无此商品...",
				headerTop: "0px",
				headerPosition: "fixed",
				orderbyList: [
					{
						text: "全部",
						selected: true,
						orderbyicon: false,
						orderby: 0
					},{
						text: "销量",
						selected: false,
						orderbyicon: false,
						orderby: 0
					},
					{
						text: "价格",
						selected: false,
						orderbyicon: ['sheng', 'jiang'],
						orderby: 0
					}
				],
				orderby: "sheng",
				option: {
					type: "",
					gname: "",
					gtypeid: ""
				},
				price: "",
				salesVolume: "",
				orderByColunm: ',',
				obj: {
					gname: '',
					gtypeid: '',
					price: '',
					salesVolume: '',
					pageIndex: '',

				}
			};
		},
		onLoad: function(option) { //option为object类型，会序列化上个页面传递的参数
			// console.log(option.cid); //打印出上个页面传递的参数。
			uni.setNavigationBarTitle({
				title: option.gname
			});
			console.log(option);
			if (option.type == 1) {
				console.log(1)
				this.option.gname = option.gname;
				this.obj.gname = option.gname;
				this.selectByGname(this.obj);
			}
			if (option.type == 2) {
				console.log(2)
				this.option.gtypeid = option.gtypeid;
				this.obj.gtypeid = option.gtypeid;
				this.selectByGname(this.obj);
			}
			this.option.type = option.type
			if (option == null) {
				this.getList();
			}

			//兼容H5下排序栏位置
			// #ifdef H5
			//定时器方式循环获取高度为止，这么写的原因是onLoad中head未必已经渲染出来。
			let Timer = setInterval(() => {
				let uniHead = document.getElementsByTagName('uni-page-head');
				if (uniHead.length > 0) {
					this.headerTop = uniHead[0].offsetHeight + 'px';
					clearInterval(Timer); //清除定时器
				}
			}, 1);
			// #endif
		},
		onPageScroll(e) {
			//兼容iOS端下拉时顶部漂移
			if (e.scrollTop >= 0) {
				this.headerPosition = "fixed";
			} else {
				this.headerPosition = "absolute";
			}
		},
		//下拉刷新，需要自己在page.json文件中配置开启页面下拉刷新 "enablePullDownRefresh": true
		onPullDownRefresh() {


		},
		//上拉加载，需要自己在page.json文件中配置"onReachBottomDistance"
		onReachBottom() {
			//根据商品名称查询
			if (this.option.type == 1) {
				if ((this.pageBean.pageindex + 1) <= this.pageBean.pagecount) {
					this.obj.gname = this.option.gname;
					this.obj.pageIndex = this.pageBean.nextpage;
					this.getPageByGname(this.obj);
				} else {
					uni.showToast({
						title: '我是有底线的'
					});
				}
			}
			//根据商品类型查询
			if (this.option.type == 2) {
				if ((this.pageBean.pageindex + 1) <= this.pageBean.pagecount) {
					this.obj.gtypeid = this.option.gtypeid;
					this.obj.pageIndex = this.pageBean.nextpage;
					this.getPageByGname(this.obj);
				} else {
					uni.showToast({
						title: '我是有底线的'
					});
				}
			}
			// let len = this.goodsList.length;
			// if (len >= 40) {
			// 	this.loadingText = "到底了";
			// 	return false;
			// } else {
			// 	this.loadingText = "正在加载...";
			// }
			// let end_goods_id = this.goodsList[len - 1].goods_id;
			// for (let i = 1; i <= 10; i++) {
			// 	let goods_id = end_goods_id + i;
			// 	let p = {
			// 		goods_id: goods_id,
			// 		img: '../../static/img/goods/p' + (goods_id % 10 == 0 ? 10 : goods_id % 10) + '.jpg',
			// 		name: '商品名称商品名称商品名称商品名称商品名称',
			// 		price: '￥168',
			// 		slogan: '1235人付款'
			// 	};
			// 	this.goodsList.push(p);
			// }
		},
		methods: {
			// reload() {
			// 	console.log("reload");
			// 	let tmpArr = []
			// 	this.goodsList = [];
			// 	let end_goods_id = 0;
			// 	for (let i = 1; i <= 10; i++) {
			// 		let goods_id = end_goods_id + i;
			// 		let p = {
			// 			goods_id: goods_id,
			// 			img: '../../static/img/goods/p' + (goods_id % 10 == 0 ? 10 : goods_id % 10) + '.jpg',
			// 			name: '商品名称商品名称商品名称商品名称商品名称',
			// 			price: '￥168',
			// 			slogan: '1235人付款'
			// 		};
			// 		this.goodsList.push(p);
			// 	}
			// },
			//商品跳转
			toGoods(e) {
				uni.showToast({
					title: '商品' + e.gid,
					icon: 'none'
				});
				uni.navigateTo({
					url: '../goods/goods?gid=' + e.gid
				});
			},
			//排序类型
			select(index) {
				this.orderbyList[index].selected = true;
				let len = this.orderbyList.length;
				for (let i = 0; i < len; i++) {
					if (i != index) {
						this.orderbyList[i].selected = false;
					}
				}
				if (this.option.type == 1) {
					if (this.orderbyList[index].text == "销量") {
						this.obj.price="";
						if (!this.salesVolume) {
							this.salesVolume = "asc"
						}
						if (this.salesVolume == "asc") {
							this.obj.salesVolume = "asc"
							this.selectByGname(this.obj);
						}
						if (this.salesVolume == "desc") {
							this.obj.salesVolume = "desc"
							this.selectByGname(this.obj);
						}
					}
					if (this.orderbyList[index].text == "价格") {
						this.obj.salesVolume="";
						if (!this.price) {
							this.price = "asc"
						}
						if (this.price == "asc") {
							this.obj.price = "asc"
							this.selectByGname(this.obj);
						}
						if (this.price == "desc") {
							this.obj.price = "desc"
							this.selectByGname(this.obj);
						}
					}
					if (this.orderbyList[index].text == "好评") {

					}
				}
				if (this.option.type == 2) {
					if (this.orderbyList[index].text == "销量") {
						this.obj.price="";
						console.log(this.salesVolume);
						if (!this.salesVolume) {
							this.salesVolume = "asc"
						}
						if (this.salesVolume == "asc") {
							this.obj.salesVolume = "asc"
							this.selectByGname(this.obj);
						}
						if (this.salesVolume == "desc") {
							this.obj.salesVolume = "desc"
							this.selectByGname(this.obj);
						}
					}
					if (this.orderbyList[index].text == "价格") {
						this.obj.salesVolume="";
						if (!this.price) {
							this.price = "asc"
						}
						if (this.price == "asc") {
							this.obj.price = "asc"
							this.selectByGname(this.obj);
						}
						if (this.price == "desc") {
							this.obj.price = "desc"
							this.selectByGname(this.obj);
						}
					}
					if (this.orderbyList[index].text == "好评") {

					}
				}
				this.orderByColunm = this.orderbyList[index].text;
				console.log(this.orderbyList[index].text);

				// let tmpTis = this.orderbyList[index].text + "排序 "
				// if (this.orderbyList[index].orderbyicon) {
				// 	let type = this.orderbyList[index].orderby == 0 ? '升序' : '降序';
				// 	if (this.orderbyList[index].selected) {
				// 		type = this.orderbyList[index].orderby == 0 ? '降序' : '升序';
				// 		this.orderbyList[index].orderby = this.orderbyList[index].orderby == 0 ? 1 : 0;
				// 	}
				// 	tmpTis += type
				// }
				// this.orderbyList[index].selected = true;
				// let len = this.orderbyList.length;
				// for (let i = 0; i < len; i++) {
				// 	if (i != index) {
				// 		this.orderbyList[i].selected = false;
				// 	}
				// }
				// uni.showToast({
				// 	title: tmpTis,
				// 	icon: "none"
				// });
			},
			getList(obj) {
				uni.request({
					url: CONSTANT.baseURL + '/goods_web/getByName', //仅为示例，并非真实接口地址。
					headers: {
						'Content-Type': 'application/x-www-form-urlencoded'
					},
					method: 'POST',
					success: (json) => {

						this.goodsList = json.data.data;
						this.pageBean = json.data.pageBean;
					}
				});
			},
			selectByGname(obj) {
				console.log(obj)
				uni.request({
					url: CONSTANT.baseURL + '/goods_web/getByName', //仅为示例，并非真实接口地址。
					data: {
						gname: obj.gname,
						gtypeid: obj.gtypeid,
						price: obj.price,
						salesVolume: obj.salesVolume
					},
					// headers: {
					// 	'Content-Type': 'application/x-www-form-urlencoded'
					// },
					method: 'GET',
					dataType: "json",
					success: (json) => {
						this.goodsList = json.data.data;
						this.pageBean = json.data.pageBean;
						console.log(json.data)
						this.salesVolume = json.data.salesVolume;
						this.price = json.data.price;
					}
				});
			},
			getPageByGname(obj) {
				uni.request({
					url: CONSTANT.baseURL + '/goods_web/getByName', //仅为示例，并非真实接口地址。
					data: {
						gname: obj.gname,
						gtypeid: obj.gtypeid,
						price: obj.price,
						salesVolume: obj.salesVolume,
						pageIndex: obj.pageIndex
					},
					// headers: {
					// 	'Content-Type': 'application/x-www-form-urlencoded'
					// },
					method: 'GET',
					dataType: "json",
					success: (json) => {
						json.data.data.forEach((obj, index) => {
							this.goodsList.push(json.data.data[index]);
						})
						this.pageBean = json.data.pageBean;
						console.log(this.goodsList)
					}
				});
			},
		}

	}
</script>

<style lang="scss">
	@font-face {
		font-family: "HMfont-home";
		src: url('data:application/x-font-woff2;charset=utf-8;base64,d09GMgABAAAAAAMMAAsAAAAAB3gAAALAAAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAHEIGVgCCfgqCXII4ATYCJAMMCwgABCAFhG0HQxuDBsiemjxBRCmUAtQ2VRCBG4ig2u+zZ3c/ortTgBIoQOVPBYSJYkUggWRSPqyjY2yEQyPe5FzmkfXucrOArBAKSdt/TjNmNTZyfrOTKAwgNQXmqbkEBgj2eS6nNz3A+YFyXHs9P2raURxQgHtR7yiyEknMW4bXLsaDeEygbVEkxYXiyhqgV+BVgbgmCQzQ55xKS2doCvWag0W8o9FMz9M24G34/fgvPvQktQxvPL5axIPc38m/U/JI9VSLIKjm80M7RsYWoBAXa4PHVHFxi0rbKaotAfuaFfxOqSp+Jwvlbv/wCImow+1dsAaGdEcqZnRrKhAggQzqaOI24Bmw02JRZmZ88bS/fq2vH6Y1yywb576F7tn3l1/5e7zm6Ze2+cW37DBXjFEftC+6U7vil0/zOvXIl3lf7cx/3DkeysBDV/tDQ5N7tli9AzsiHDgcO55136FS8LLTzBygm4Q9u6bCp1zAy0lh7v/L+PnQ0P71YAZeG0pE3eWwRIIOANX5dz4JQCX8hQLB+5z/hXZb5ofeVMCvjc0fKFEp+rs1bzkU/H5Vz67iarArF1vJlH4oO8g0SGi7EVU4OnY43jzr3U5omqVIGhaQNS2Rhd1CTcc26ppOo21TyfGOCUIlSgsbJhHC0DqSvp/IhnbIwh6gZu4BdcNQoe1iuC7sWAnJWEAQQxEP2V4ocQE/ph5qjDqrkNDpZUhWE4rrEJGlDIyLji1WSpAfkTm2yF1CPKUYYhLwwWL0HPJ6AzBIAm7E0WiR0mB6TAyue1M0F/ABo/MIhEEhPIjVC5JwAvywGE8ZS5+vggg6eTFIC6ejWAchZNL0UJxosT3IEqO/F+dehMi6COJRFAZhRIAPVIz8iJcUA6Bg/Tw3hENFE0fkg9LFSP1wX2P0+mbf952ANvxUjhQ5is4tRurpjHJLjF9IRbKIHAAA') format('woff2');
	}

	.icon {
		font-family: "HMfont-home" !important;
		font-size: 26upx;
		font-style: normal;

		&.sheng {
			&:before {
				content: "\e737";
			}
		}

		&.jiang {
			&:before {
				content: "\e736";
			}
		}

	}

	.header {
		width: 92%;
		padding: 0 4%;
		height: 79upx;
		display: flex;
		justify-content: space-around;
		align-items: flex-end;
		position: fixed;
		top: 0;
		z-index: 10;
		background-color: #fff;
		border-bottom: solid 1upx #eee;

		.target {
			width: 20%;
			height: 60upx;
			display: flex;
			justify-content: center;
			align-items: center;
			font-size: 28upx;
			margin-bottom: -2upx;
			color: #aaa;

			&.on {
				color: #555;
				border-bottom: 4upx solid #f06c7a;
				font-weight: 600;
				font-size: 30upx;
			}


		}
	}

	.place {

		background-color: #ffffff;
		height: 100upx;

	}

	.goods-list {
		.loading-text {
			width: 100%;
			display: flex;
			justify-content: center;
			align-items: center;
			height: 60upx;
			color: #979797;
			font-size: 44upx;
		}

		.product-list {
			width: 92%;
			padding: 0 4% 3vw 4%;
			display: flex;
			justify-content: space-between;
			flex-wrap: wrap;

			.product {
				width: 48%;
				border-radius: 20upx;
				background-color: #fff;
				margin: 0 0 15upx 0;
				box-shadow: 0upx 5upx 25upx rgba(0, 0, 0, 0.1);

				image {
					width: 100%;
					border-radius: 20upx 20upx 0 0;
				}

				.name {
					width: 92%;
					padding: 10upx 4%;
					display: -webkit-box;
					-webkit-box-orient: vertical;
					-webkit-line-clamp: 2;
					text-align: justify;
					overflow: hidden;
					font-size: 30upx;
				}

				.info {
					display: flex;
					justify-content: space-between;
					align-items: flex-end;
					width: 92%;
					padding: 10upx 4% 10upx 4%;

					.price {
						color: #e65339;
						font-size: 30upx;
						font-weight: 600;
					}

					.slogan {
						color: #807c87;
						font-size: 24upx;
					}
				}
			}

		}
	}
</style>
