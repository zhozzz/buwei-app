<template>
	<view class="flow-box">
		<view class="item" :class="item.left == 1 ? 'left' : ''"
			:style="'top:' + item.top + 'px'" v-for="(item, index) in list" :key="index"
			<!-- #ifdef H5 -->
			:data-item="JSON.stringify(item)"
			<!-- #endif -->
			<!-- #ifndef H5 -->
			:data-item="item"
			<!-- #endif -->
			@click="choose">
			<view class="pic">
				<image class="image" style="width: 100%; display: block;border-radius: 40upx;min-height:100pt;max-height: 240pt ;"
					:src="item.image ? item.image : error"></image>
			</view>
			<view class="content">
				<text>{{item.content}}</text>
				<view class="user">
					<image style="width: 40upx; height: 40upx; border-radius: 50%; margin-right: 10upx;" 
						:src="item.user.avatar ? item.user.avatar : error"></image>
					{{item.user.name}}
				</view>
			</view>
		</view>
		<view class="loading" v-show="loading" :style="'top: ' + loadingTop + 'px'" >
			<image src="/static/nairenk-waterfall-flow/loading.gif" style="width: 80upx; height: 80upx;"></image>
		</view>
	</view>
</template>

<script>
	export default {
		props: {
			// 数据列表
			list: {
				type: Array,
				default() {
					return []
				}
			},
			// 加载动画
			loading: {
				type: Boolean,
				default: false
			}
		},
		data() {
			return {
				error: '/static/nairenk-waterfall-flow/error.jpg', // 图片错误
				mark: 0, // 定位
				boxHeight: [], // 计算盒子 2 行的高度
				loadingTop: 0 // 加载动画滚动位置
			}
		},
		watch: {
			// 数据
			list: function (newVal, oldVal) {
				this.mark = oldVal.length;
				if (newVal != oldVal) {
					this.$nextTick(function () {
						setTimeout(()=>{
							this.waterFall();
						}, 100)
					})
				}
			}
		},
		methods: {
			// 瀑布流定位
			waterFall() {
				uni.createSelectorQuery().selectAll('.item').boundingClientRect().exec(res => {
					let item = res[0];
					let len = item.length;
					for (let i = this.mark; i < len-1; i++) {
						let height = parseInt(item[i].height);
						let boxHeight = height;
						if (i < 2) {
							this.$set(this.list[i], 'top', 0);
							this.$set(this.list[i], 'left', i);
							if (height > height) {
								height = height;
								boxHeight = height;
							}
							this.boxHeight.push(boxHeight);
						} else {
							let minHeight = this.boxHeight[0];
							let boxLen = this.boxHeight.length;
							let index = 0;
							for (var j = 0; j < boxLen; j++) {
								if (minHeight > this.boxHeight[j]) {
									minHeight = this.boxHeight[j];
									index = j;
								}
							}
							console.log(this.list[i])
							this.$set(this.list[i], 'top', this.boxHeight[index]);
							this.$set(this.list[i], 'left', index);
							this.boxHeight[index] = this.boxHeight[index] + boxHeight;
							this.loadingTop = this.boxHeight[~index + 2];
							setTimeout(()=> {
								let maxHeight = this.boxHeight[0];
								for (var j = 0; j < boxLen; j++) {
									if (maxHeight < this.boxHeight[j]) {
										maxHeight = this.boxHeight[j]
									}
								}
								this.loadingTop = maxHeight;
							})
						}
					}
				});
			},
			// 选中
			choose(e) {
				let item = e.currentTarget.dataset.item;
				/* #ifdef H5 */
				item = JSON.parse(e.currentTarget.dataset.item);
				/* #endif */
				this.$emit('click', item);
			}
		}
	}
</script>

<style scoped>
	.flow-box {
		position: relative;
		width: 100%;
		color: #1a1a1a;
	}
	.flow-box .item {
		position: absolute;
		left: 0;
		width: calc(50% - 20upx);
		border: 10upx solid #fff;
		background: #fff;
	}
	.flow-box .left {
		left: 370upx;
	}
	.flow-box .pic {
		background: #fff;
	}
	.flow-box .content {
		padding: 20upx;
		display: flex;
		justify-content: space-between;
		flex-wrap: wrap;
	}
	.flow-box .content text {
		width: 100%;
		font-size: 24upx;
		margin-bottom: 20upx;
	}
	.flow-box .user {
		display: flex;
		width: 220upx;
		overflow: hidden;
		font-size: 26upx;
		color: #666;
	}
	.loading {
		position: absolute;
		width: 100%;
		text-align: center;
		padding: 20upx 0;
	}
</style>
