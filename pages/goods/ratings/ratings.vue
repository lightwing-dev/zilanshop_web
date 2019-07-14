<template>
	<view>
		<view class="content">
			<view class="label">
				<view v-for="(label,index) in labelList" :class="{'on':index==labelIndex}" @tap="loadRatings(index)" :key="label.type">
					{{label.name}}({{label.number}})
				</view>
			</view>
			<view class="list">
				<view class="row" v-for="(row,Rindex) in ratingsList" :key="Rindex">
					<view class="left">
						<view class="face">
							<image :src="CONSTANT.baseURL+row.zuser.headimg"></image>
						</view>
					</view>
					<view class="right">
						<view class="name-date">
							<view class="username">
								{{row.zuser.username}}
							</view>
							<view class="date">
								{{timestampToTime(row.createTime)}}
							</view>
						</view>
						<view class="first">
							<view class="rat">
								{{row.content}}
							</view>
							<view class="img-video">
								<view class="box" v-for="item in row.evaluationimages" @tap="showBigImg(item.imgpath,row.evaluationimages)"
								 :key="item">
									<image mode="aspectFill" :src="CONSTANT.baseURL+item.imgpath"></image>
								</view>
							</view>
						</view>
					</view>
				</view>
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
				labelList: [{
						name: '全部',
						number: 0,
						type: 'all'
					},
					{
						name: '1分',
						number: 0,
						type: 'one'
					},
					{
						name: '2分',
						number: 0,
						type: 'tow'
					},
					{
						name: '2分',
						number: 0,
						type: 'three'
					},
					{
						name: '4分',
						number: 0,
						type: 'four'
					},
					{
						name: '5分',
						number: 0,
						type: 'five'
					}
				],
				labelIndex: 0,
				pageBean: [],
				ratingsList: [],
			};
		},
		onLoad(option) {
			this.selectByGid(option.gid)
		},
		//下拉刷新，需要自己在page.json文件中配置开启页面下拉刷新 "enablePullDownRefresh": true
		onPullDownRefresh() {
			setTimeout(function() {
				uni.stopPullDownRefresh();
			}, 1000);
		},
		//上拉加载，需要自己在page.json文件中配置"onReachBottomDistance"
		onReachBottom() {
			uni.showToast({
				title: '触发上拉加载'
			});
		},
		methods: {
			loadRatings(index) {
				this.labelIndex = index;
				this.selectByStar(index);
			},
			showBigImg(src, srclist) {
				let img = [];
				srclist.forEach((obj, index) => {
					img.push(CONSTANT.baseURL + obj.imgpath);
				})
				uni.previewImage({
					current: CONSTANT.baseURL + src,
					urls: img
				});
			},
			selectByGid(gid) {
				uni.request({
					url: CONSTANT.baseURL + '/zEvaluation_web/selectByGid?gid=' + gid, //仅为示例，并非真实接口地址。
					headers: {
						'Content-Type': 'application/x-www-form-urlencoded'
					},
					method: 'GET',
					success: (json) => {
						this.ratingsList = json.data.data;
						this.labelList[0].number = parseInt(this.ratingsList.length);
						this.ratingsList.forEach((obj, index) => {
							if (obj.star == 1) {
								this.labelList[1].number = parseInt(this.labelList[1].number + 1);
							}
							if (obj.star == 2) {
								this.labelList[2].number = parseInt(this.labelList[2].number + 1);
							}
							if (obj.star == 3) {
								this.labelList[3].number = parseInt(this.labelList[3].number + 1);
							}
							if (obj.star == 4) {
								this.labelList[4].number = parseInt(this.labelList[4].number + 1);
							}
							if (obj.star == 5) {
								this.labelList[5].number = parseInt(this.labelList[5].number + 1);
							}
						})
						this.pageBean = json.data.pageBean;
					}
				});
			},
			selectByStar(star) {
				uni.request({
					url: CONSTANT.baseURL + '/zEvaluation_web/selectByGid?star=' + star, //仅为示例，并非真实接口地址。
					headers: {
						'Content-Type': 'application/x-www-form-urlencoded'
					},
					method: 'GET',
					success: (json) => {
						this.ratingsList = json.data.data;
						this.pageBean = json.data.pageBean;
					}
				});
			},
			/**
			 * @param {Object} timestamp时间转换
			 */
			timestampToTime(timestamp) {
				var date = new Date(timestamp); //时间戳为10位需*1000，时间戳为13位的话不需乘1000
				var Y = date.getFullYear() + '-';
				var M = (date.getMonth() + 1 < 10 ? '0' + (date.getMonth() + 1) : date.getMonth() + 1) + '-';
				var D = (date.getDate() < 10 ? '0' + date.getDate() : date.getDate()) + ' ';
				return Y + M + D;
			},
		},
	}
</script>

<style lang="scss">
	page {
		background-color: #fff;
	}

	@font-face {
		font-family: 'HMfont-home';
		src: url('data:application/x-font-woff2;charset=utf-8;base64,d09GMgABAAAAAANUAAsAAAAAB3QAAAMFAAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAHEIGVgCDBgqCWII/ATYCJAMMCwgABCAFhG0HORuIBsgekiRnhujwFwBXCiIAxcPz9/tv7XP2vT/DsvBb5Fg1ZYC1RgEPyACODfAANQ+//V/TFNCoPF+WVYQK7WH+Aedybff8hCMj5yv87Nwm3H7YXx7MinQJatc8OpHvH9gGsC8x2XUObekfXI5/ZAnMsyyXOaYtDDDeGtDe2CYrkIC8YeyClnieQK9pHaUTq8MZmpI5LRBPPceAZsYjJYt3C23F2hSfMUTd5Q3uAvgUfD/+GQWFoqnMi87dX9ow+Sn7GsMDCoorYGA6E9gmKjaBJK5XWq6pRJFNFb2qpNqc3kqhTOVHUQKOJms3+scrRMvM7QerMFr4KXPoBD/HNiigghZ03wK+TFBMVYeHlQy897Ofp+zDpYdo+PEx93z1OVb88JB9uvIUi/s+5x4vP0Zrx7KCUMR9wAYIw6E32MyJ88LvuW1EFMn7RHofPkKPn+Hn77ynry5bFAlZaWbJXqWR58UsbvF0O+8AsS79nfQ9fUeY6RvfZydgRSb5sNd5sOzdgUxsjGfrYQVOFmtpaRcCVkpawpWkIVKlpIRXIEpTfTBY4aXRzBJfsGpW4wqSIubw9XcCpTuh8IiyoAD3W57Yato9TEzz9T3ctOjHTAA6kNj6+sf8xuMPQ8Yytu1/Nz3Bj/9nRcDrZsPoPm1r8BtKEkMHzeBySJoKx+EYtmr3lkwFyOJ2HYzpGPU3XDbROZfQbSKBossUVN0WkSRsQqPPFrS67UOvDevNfcawisgerHsFEIZ9QTHoG1TDvpEk/EJj0j+0hgMPva4Ef88+y9HQ9jIZQjaa3ehVJbGy9dIO84/kFKHBaX5AuhLnnh+HJks+vqaYeIoFeYczkjaFqiWJcAUuozBMMG1JfLKk57ZJOu6ba6ve1LMaI2g7xohBEBsyvUAeSyKmvPqiXfj8EXEUQgZuqKoyXhGW83pHQz2DDshaEnequpVrcjeOESEUpLBEBK2AiYRWJIHS6lE+YhE97oBQaqyPaqmukt7ysuj9NkAv8+gaJWqk9kFhxKZXr0yUETsAAAAA') format('woff2');
	}

	.icon {
		font-family: 'HMfont-home' !important;
		font-size: 26upx;
		font-style: normal;

		&.bofang {
			&:before {
				content: '\e696';
			}
		}

		&.guanbi {
			&:before {
				content: '\e61a';
			}
		}

	}

	.myVideo {
		position: fixed;
		top: 50%;
		right: 100%;
	}

	.content {
		view {
			display: flex;
		}

		width: 94%;
		padding: 0 3%;

		.label {
			width: 100%;
			flex-wrap: wrap;

			view {
				padding: 0 20upx;
				height: 50upx;
				border-radius: 40upx;
				border: solid 1upx #ddd;
				align-items: center;
				color: #555;
				font-size: 26upx;
				background-color: #f9f9f9;
				margin: 10upx 20upx 10upx 0;

				&.on {
					border: solid 1upx #f06c7a;
					color: #f06c7a;
				}
			}
		}

		.list {
			width: 100%;
			flex-wrap: wrap;
			padding: 20upx 0;

			.row {
				width: 100%;
				margin-top: 30upx;

				.left {
					width: 10vw;
					flex-shrink: 0;
					margin-right: 10upx;

					.face {
						width: 100%;

						image {
							width: 10vw;
							height: 10vw;
							border-radius: 100%;
						}

					}
				}

				.right {
					width: 100%;
					flex-wrap: wrap;

					.name-date {
						width: 100%;
						justify-content: space-between;
						align-items: baseline;

						.username {
							font-size: 32upx;
							color: #555;
						}

						.date {
							font-size: 28upx;
							color: #aaa;
						}
					}

					.spec {
						width: 100%;
						color: #aaa;
						font-size: 26upx;
					}

					.first {
						width: 100%;
						flex-wrap: wrap;

						.rat {
							font-size: 30upx;
						}

						.img-video {
							width: 100%;
							flex-wrap: wrap;

							.box {
								width: calc((84.6vw - 50upx)/4);
								height: calc((84.6vw - 50upx)/4);
								margin: 5upx 5upx;
								position: relative;
								justify-content: center;
								align-items: center;

								image {
									position: absolute;
									width: 100%;
									height: 100%;
									border-radius: 10upx;
								}

								.playbtn {
									position: absolute;

									.icon {
										font-size: 80upx;
										color: rgba(255, 255, 255, .9)
									}
								}
							}
						}
					}

					.append {
						width: 100%;
						flex-wrap: wrap;

						.date {
							width: 100%;
							height: 40upx;
							border-left: 10upx solid #f06c7a;
							padding-left: 10upx;
							align-items: center;
							font-size: 32upx;
							margin: 20upx 0;
						}

						.rat {
							font-size: 30upx;
						}

						.img-video {
							width: 100%;
							flex-wrap: wrap;

							.box {
								width: calc((84.6vw - 10upx - (10upx * 4))/4);
								height: calc((84.6vw - 10upx - (10upx * 4))/4);
								margin: 5upx 5upx;
								position: relative;
								justify-content: center;
								align-items: center;

								image {
									position: absolute;
									width: 100%;
									height: 100%;
									border-radius: 10upx;
								}

								.playbtn {
									position: absolute;

									.icon {
										font-size: 80upx;
										color: rgba(255, 255, 255, .9);
									}
								}
							}
						}
					}
				}
			}
		}
	}

	/*
* <view class="list">
				<view class="row">

					<view class="right">
						
						<view class="spec">
							规格：XL
						</view>
						<view class="first">
							<view class="rat">
								好看，可以，不愧是专业的，才拿到手上就研究了半天才装上
							</view>
							<view class="img-video">
								<view class="box">
									<image src="https://ae01.alicdn.com/kf/HTB1wREwTXzqK1RjSZFvq6AB7VXaT.jpg"></image>
									<view class="playbtn">
										<view class="icon bofang"></view>
									</view>
								</view>
								<view class="box">
									<image src="https://ae01.alicdn.com/kf/HTB1wREwTXzqK1RjSZFvq6AB7VXaT.jpg"></image>
								</view>
								<view class="box">
									<image src="https://ae01.alicdn.com/kf/HTB1wREwTXzqK1RjSZFvq6AB7VXaT.jpg"></image>
								</view>
							</view>
						</view>
						<view class="append">
							<view class="date">
								65天后追加
							</view>
							<view class="rat">
								好看，可以，不愧是专业的，才拿到手上就研究了半天才装上
							</view>
							<view class="img-video">
								<view class="box">
									<image src="https://ae01.alicdn.com/kf/HTB1wREwTXzqK1RjSZFvq6AB7VXaT.jpg"></image>
								</view>
								<view class="box">
									<image src="https://ae01.alicdn.com/kf/HTB1wREwTXzqK1RjSZFvq6AB7VXaT.jpg"></image>
								</view>
							</view>
						</view>
					</view>
				</view>
			</view>
			* 
			* */
</style>
