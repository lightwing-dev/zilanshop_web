<template>
	<view>
		<view class="content">
			<view class="list">
				<view class="row" v-for="(row,index) in addressList" :key="index" @tap="select(row)">
					<view class="left">
						<view class="head">
							{{row.name.substring(0,1)}}
						</view>
					</view>
					<view class="center">
						<view class="name-tel">
							<view class="name">{{row.name}}</view>
							<view class="tel">{{row.phone}}</view>
							<view class="default" v-if="row.isdefault==0">
								默认
							</view>
						</view>
						<view class="address">
							{{row.province}}-{{row.city}}-{{row.district}} {{row.address}}
						</view>
					</view>
					<view class="right">
						<view class="icon bianji" @tap.stop="edit(row)">

						</view>
					</view>
				</view>
			</view>
		</view>
		<view class="add">
			<view class="btn" @tap="add">
				<view class="icon tianjia"></view>新增地址
			</view>
		</view>
	</view>
</template>
<script>
	import CONSTANT from '@/common/constant.js'
	export default {
		data() {
			return {
				CONSTANT: CONSTANT,
				isSelect: false,
				addressList: [],
			};
		},
		onShow() {
			this.getById();
			uni.getStorage({
				key: 'delAddress',
				success: (e) => {
					let len = this.addressList.length;
					if (e.data.hasOwnProperty('id')) {
						for (let i = 0; i < len; i++) {
							if (this.addressList[i].id == e.data.id) {
								this.addressList.splice(i, 1);
								break;
							}
						}
					}
					uni.removeStorage({
						key: 'delAddress'
					})
				}
			})
			uni.getStorage({
				key: 'saveAddress',
				success: (e) => {
					let len = this.addressList.length;
					if (e.data.hasOwnProperty('id')) {
						for (let i = 0; i < len; i++) {
							if (this.addressList[i].id == e.data.id) {
								this.addressList.splice(i, 1, e.data);
								break;
							}
						}
					} else {
						let lastid = this.addressList[len - 1];
						lastid++;
						e.data.id = lastid;
						this.addressList.push(e.data);
					}
					uni.removeStorage({
						key: 'saveAddress'
					})
				}
			})
		},
		onLoad(e) {
			if (e.type == 'select') {
				this.isSelect = true;
			}
		},
		methods: {
			edit(row) {
				uni.setStorage({
					key: 'address',
					data: row,
					success() {
						uni.navigateTo({
							url: "edit/edit?type=edit"
						})
					}
				});

			},
			add() {
				uni.navigateTo({
					url: "edit/edit?type=add"
				})
			},
			select(row) {
				//是否需要返回地址(从订单确认页跳过来选收货地址)
				if (!this.isSelect) {
					return;
				}
				uni.setStorage({
					key: 'selectAddress',
					data: row,
					success() {
						uni.navigateBack();
					}
				})
			},
			getById(uid) {
				let that = this;
				uni.request({
					url: CONSTANT.baseURL + '/address_web/getList?uid=' + 1,
					headers: {
						'Content-Type': 'application/x-www-form-urlencoded'
					},
					method: 'POST',
					success: (json) => {
						that.addressList = json.data.data;
					}
				});
			}
		}
	}
</script>

<style lang="scss">
	view {
		display: flex;
	}

	@font-face {
		font-family: "HMfont-home";
		src: url('data:application/x-font-woff2;charset=utf-8;base64,d09GMgABAAAAAAMIAAsAAAAABvwAAAK8AAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAHEIGVgCDBgqBSIFAATYCJAMMCwgABCAFhG0HVBskBsiusClj9ljNiEIaBdizs389YOCARVDt99mzu+8dMMpzQEn5KEAZRez+yRDbSDIixkYo1tF/+vv5OxYJFk2VghXWbbzzPn8D/OMG3vvXGTU90ZFhOrNJROZwCRGjj/Iry36wAbfSxBsuNGggeG9sMbJKDd7xg8vpr4ACmWdZLnMtGxMwwUD3wCiywi3oDWMXuITzBNpNc4BP3j5/Q1thTQvE1SQiaOd8isKSrUJds7aIVyqt6XECAF6Gj49/sBcUSZVZc09duQng/CfPcXTVrIs+gj+fBWwTGZsghbhcGzurJhgZ1S6rt2fXipDmCv5PyNMltf2HRxJEzSrsBKtIk9wU32WS+E1w14UZ1HFiG+QkJg3ODWmyn5/20eOvTz5LnR6l8aWDT5Sn3wLtYlfNe7RIik/fN961C3Vftf6YZLr5ZMcjU/LExqD9u3LzvKE8KQtBGAp9ilm1XbAK2m83TdlozEvQ0Zbrh8HBMrKDB03MjRwHaJKP2f5jf+NfDvML4f+tHQX8+EJvkwL1z9Mqwfi/kd+zq+hCS5+LynN5piObGRlNaNedmrJc/R7jVUO3agmtOT7zJy32WkjWahGihbQJlQ5bklpT7ENotyG3ucOAjpoobVi3BxB6HSDp9h2yXne0kDSoDPtBrTdQaHc61D07LEezm1Im4wBLc2z6UoaO0bpR8SdHLifNCkPKL+s4CaLX5Skm77hknWNBdxLt9SzEmkqBWXAZ57lgSyVl37YaZqMzt7tWd6OtshTQdYJixLAAKplDTT5RCv3Bplu6/ycWcXJEW+pqrL+YGkuGR14unh7onazsVXcv13RNRPb0mBCqUaKAssDCcjsmUKt+VIr5zJbGiMjIGTfqV+sr21pfUXxALmvCmpMjRY5i9G5CZepynIyYZOr+sksyR2W0UHLiChIrRmXfA0E') format('woff2');
	}

	.icon {
		font-family: "HMfont-home" !important;
		font-size: 60upx;
		font-style: normal;
		color: #000000;

		&.bianji {
			&:before {
				content: "\e61b";
			}
		}

		&.tianjia {
			&:before {
				content: "\e81a";
			}
		}
	}

	.add {
		position: fixed;
		bottom: 0;
		width: 100%;
		height: 120upx;
		justify-content: center;
		align-items: center;

		.btn {
			box-shadow: 0upx 5upx 10upx rgba(0, 0, 0, 0.4);
			width: 70%;
			height: 80upx;
			border-radius: 80upx;
			background-color: #f06c7a;
			color: #fff;
			justify-content: center;
			align-items: center;

			.icon {
				height: 80upx;
				color: #fff;
				font-size: 30upx;
				justify-content: center;
				align-items: center;
			}

			font-size: 30upx;
		}
	}

	.list {
		flex-wrap: wrap;

		.row {
			width: 96%;
			padding: 20upx 2%;

			.left {
				width: 90upx;
				flex-shrink: 0;
				align-items: center;

				.head {
					width: 70upx;
					height: 70upx;
					background: linear-gradient(to right, #ccc, #aaa);
					color: #fff;
					justify-content: center;
					align-items: center;
					border-radius: 60upx;
					font-size: 35upx;
				}
			}

			.center {
				width: 100%;
				flex-wrap: wrap;

				.name-tel {
					width: 100%;
					align-items: baseline;

					.name {
						font-size: 34upx;
					}

					.tel {
						margin-left: 30upx;
						font-size: 24upx;
						color: #777;
					}

					.default {

						font-size: 22upx;

						background-color: #f06c7a;
						color: #fff;
						padding: 0 18upx;
						border-radius: 24upx;
						margin-left: 20upx;
					}
				}

				.address {
					width: 100%;
					font-size: 24upx;
					align-items: baseline;
					color: #777;
				}
			}

			.right {
				flex-shrink: 0;
				align-items: center;
				margin-left: 20upx;

				.icon {
					justify-content: center;
					align-items: center;
					width: 80upx;
					height: 60upx;
					border-left: solid 1upx #aaa;
					font-size: 40upx;
					color: #777;
				}
			}
		}
	}
</style>
