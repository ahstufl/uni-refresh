<template>
    <view class="content">
        <custom-scroll-refresh class="floor-box" :show-scrollbar="false" :refresher-enabled="refresherEnabled"
            :refresh-duration="refreshDuration" @refresh="handleCustomRefresh" @scroll-change="handleScrollChange"
            id="scroll-list">
            <template #top>
                <view class="title">uni自定义下拉刷新</view>
                <view class="title-en">uni-custom-scroll-refresh</view>
            </template>
            <template #refresher-pulling>
                <image class="simple-loading" src="/static/refresh-static.png" mode="widthFix" />
                <text class="refresh-text">下拉刷新</text>
            </template>
            <template #refresher-release>
                <image class="simple-loading" src="/static/refresh-static.png" mode="widthFix" />
                <text class="refresh-text">释放刷新</text>
            </template>
            <template #refresher-refreshing>
                <image class="simple-loading" src="/static/refresh.gif" mode="widthFix" />
                <text class="refresh-text">正在刷新</text>
            </template>
            <template #refresher-success>
                <image class="simple-loading" src="/static/refresh-static.png" mode="widthFix" />
                <text class="refresh-text">刷新成功</text>
            </template>
            <view class="container">
                <view v-for="item in list" :key="item" class="item">{{ item }}</view>
            </view>
        </custom-scroll-refresh>
    </view>
</template>

<script setup lang="ts">
import { ref } from 'vue';
const list = ref<number[]>([1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20]);
const refresherEnabled = ref(true);
const refreshDuration = ref(1000);
const handleCustomRefresh = () => {
    console.log('handleCustomRefresh');
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
        background-color: #ECF3FF;

        .simple-loading {
            width: 90rpx;
            height: 90rpx;
        }

        .refresh-text {
            font-size: 24rpx;
            color: #8999B5;
        }

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
