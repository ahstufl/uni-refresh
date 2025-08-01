<template>
	<view class="content">
		<custom-scroll-refresh class="floor-box" :show-scrollbar="false" :refresher-enabled="refresherEnabled"
			:refresh-duration="refreshDuration" @refresh="handleCustomRefresh"
			@scroll-change="handleScrollChange" id="scroll-list">
			<view class="container">
				<view v-for="item in list" :key="item" class="item" >{{ item }}</view>
			</view>
		</custom-scroll-refresh>
	</view>
</template>

<script setup lang="ts">
	import { ref } from 'vue';
	const list = ref<(number|string)[]>([1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20]);
	const refresherEnabled = ref(true);
	const refreshDuration = ref(1000);
	const handleCustomRefresh = async () => {
		list.value = ['1-loading', '2-loading', '3-loading', '4-loading', '5-loading', '6-loading', '7-loading', '8-loading', '9-loading', '10-loading', '11-loading', '12-loading', '13-loading', '14-loading', '15-loading', '16-loading', '17-loading', '18-loading', '19-loading', '20-loading'];
		await new Promise(resolve => {
			setTimeout(() => {
				list.value = [1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20];
				resolve(true);
			}, 1000);
		});
	}
	const handleScrollChange = () => {
		console.log('handleScrollChange');
	}
</script>

<style lang="scss" scoped>
.content {
	box-sizing: border-box;
	padding: 20rpx;
	text-align: center;
	height: 100vh;
	display: flex;
	flex-direction: column;
	.title {
		font-size: 36rpx;
		color: #94bece;
		font-weight: bold;
	}
	.title-en {
		font-size: 24rpx;
		margin-bottom: 20rpx;
	}
	.floor-box {
		border: 1px solid #8f8f94;
		border-radius: 10rpx;
		height: 0rpx;
		flex: 1;
		.container {
			// height: 100vh;
			overflow: auto;
			display: flex;
			flex-direction: column;
			gap: 40rpx;
			padding: 20rpx;
			.item {
				text-align: center;
				font-size: 36rpx;
				color: #8f8f94;
				background-color: #94bece;
				padding: 20rpx;
				border-radius: 10rpx;
				color: #fff;
			}
		}
	}
}
</style>
