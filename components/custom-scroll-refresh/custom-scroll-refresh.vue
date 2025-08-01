<template>
    <view class="custom-scroll-refresh" @touchstart="onTouchStart" @touchmove="onTouchMove" @touchend="onTouchEnd"
        @touchcancel="onTouchEnd">
        <view class="custom-scroll-refresh-top">
            <slot name="top"></slot>
        </view>
        <view class="custom-scroll-refresh-content">
            <view class="custom-refresher" :style="refresherStyle">
                <view class="refresher-content">
                    <!-- idle状态 -->
                    <slot name="refresher-idle" v-if="refreshStatus === 'idle'"></slot>
                    
                    <!-- pulling状态 -->
                    <slot name="refresher-pulling" v-if="refreshStatus === 'pulling'">
                        <view class="refresh-state">
                            <view class="refresh-icon" />
                            <text class="refresh-text">{{ pullText }}</text>
                        </view>
                    </slot>
                    
                    <!-- release状态 -->
                    <slot name="refresher-release" v-if="refreshStatus === 'release'">
                        <view class="refresh-state">
                            <view class="refresh-icon" />
                            <text class="refresh-text">{{ releaseText }}</text>
                        </view>
                    </slot>
                    
                    <!-- refreshing状态 -->
                    <slot name="refresher-refreshing" v-if="refreshStatus === 'refreshing'">
                        <view class="refresh-state">
                            <view class="refresh-icon refresh-loading" />
                            <text class="refresh-text">{{ refreshingText }}</text>
                        </view>
                    </slot>
                    
                    <!-- success状态 -->
                    <slot name="refresher-success" v-if="refreshStatus === 'success'">
                        <view class="refresh-state">
                            <view class="refresh-icon" />
                            <text class="refresh-text">{{ successText }}</text>
                        </view>
                    </slot>
                </view>
            </view>

            <!-- scroll-view内容 -->
            <scroll-view :class="`${scrollClass}`" :show-scrollbar="showScrollbar" :refresher-enabled="false"
                :scroll-y="!isPulling" @scroll="onScroll" :style="computedScrollStyle">
                <slot></slot>
            </scroll-view>
        </view>
    </view>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';

// Props定义
interface Props {
    showScrollbar?: boolean; // 是否显示滚动条
    refresherThreshold?: number; // 下拉刷新阈值
    refresherEnabled?: boolean; // 是否启用下拉刷新
    refreshDuration?: number; // 刷新动画时长
    refresherOutRate?: number; // 超出阈值后的衰减比例
    refresherPullRate?: number; // 下拉时的位移比例
    pullText?: string; // 下拉刷新文本
    releaseText?: string; // 释放刷新文本
    refreshingText?: string; // 正在刷新文本
    successText?: string; // 刷新成功文本
    scrollStyle?: Record<string, any>; // 滚动视图样式
    scrollClass?: string; // 滚动视图类名
    refreshable?: boolean; // 是否可刷新
}

const props = withDefaults(defineProps<Props>(), {
    showScrollbar: false,
    refresherThreshold: 100,
    refresherEnabled: true,
    refreshDuration: 60,
    refresherOutRate: 0.65,
    refresherPullRate: 0.75,
    pullText: '下拉刷新',
    releaseText: '释放刷新',
    refreshingText: '正在刷新',
    successText: '刷新成功',
    refreshable: true,
});

// Events定义
const emit = defineEmits<{
    refresh: [];
    pulling: [detail: any];
    restore: [detail: any];
    abort: [detail: any];
    scrollChange: [isScrollDisabled: boolean];
}>();

// 响应式数据
const refresherTriggered = ref(false);
const refreshStatus = ref<'idle' | 'pulling' | 'release' | 'refreshing' | 'success'>('idle');
const refreshProgress = ref(0);
const isRefresherVisible = ref(false);
const currentPullDistance = ref(0);

// 自定义touch控制
const isPulling = ref(false);
const startY = ref(0);
const currentY = ref(0);
const isAtTop = ref(true);

// 计算属性 - 参考z-paging的做法，使用负边距隐藏+opacity控制显示
const refresherStyle = computed(() => {
    if(!props.refreshable) return {
        height: 0,
        opacity: 0,
        transform: 'translateY(0px)'
    };
    return {
    'margin-top': `-${props.refresherThreshold}px`,
    height: `${props.refresherThreshold}px`,
    opacity: isRefresherVisible.value ? 1 : 0,
    transform: `translateY(${Math.min(currentPullDistance.value, props.refresherThreshold + 50)}px)`,
    };
});

const computedScrollStyle = computed(() => {
    if(!props.refreshable) return props.scrollStyle || {};
    return {
        ...props.scrollStyle,
        marginTop: `${Math.max(0, Math.min(currentPullDistance.value, props.refresherThreshold + 50))}px`
    };
});

// 自定义touch事件处理
const onTouchStart = (event: TouchEvent) => {
    if (!props.refresherEnabled || refreshStatus.value === 'refreshing') return;

    startY.value = event.touches[0].clientY;
    currentY.value = startY.value;
};

const onTouchMove = (event: TouchEvent) => {
    if (!props.refresherEnabled || refreshStatus.value === 'refreshing') return;
    currentY.value = event.touches[0].clientY;
    const deltaY = currentY.value - startY.value;

    // 只有在顶部且向下拉时才处理
    if (isAtTop.value && deltaY > 0) {
        event.preventDefault(); // 阻止默认滚动
        isPulling.value = true;

        // 参考z-paging的下拉距离控制逻辑
        let effectiveDy = deltaY * props.refresherPullRate; // 基础比例控制

        // 如果超出阈值，应用衰减效果
        if (effectiveDy > props.refresherThreshold) {
            const excess = effectiveDy - props.refresherThreshold;
            const dampedExcess = excess * props.refresherOutRate;
            effectiveDy = props.refresherThreshold + dampedExcess;

            // 严格限制最大下拉距离
            effectiveDy = Math.min(effectiveDy, props.refresherThreshold * 2);
        }

        const progress = Math.min(effectiveDy / props.refresherThreshold, 1);
        refreshProgress.value = progress;

        // 更新UI状态
        if (effectiveDy > 0) {
            isRefresherVisible.value = true;
            currentPullDistance.value = effectiveDy - 27;

            if (progress < 1) {
                refreshStatus.value = 'pulling';
            } else {
                refreshStatus.value = 'release';
            }
            emit('scrollChange', true);
        }

        // 传递事件
        emit('pulling', { dy: effectiveDy });
    } else {
        emit('scrollChange', false);
        isPulling.value = false;
    }
};

const onTouchEnd = () => {
    if (!isPulling.value) return;

    isPulling.value = false;

    if (refreshStatus.value === 'release') {
        // 触发刷新
        onRefresh();
    } else {
        // 恢复状态
        onRestore();
    }
};

const onScroll = (event: any) => {
    // 检测是否在顶部
    isAtTop.value = event.detail.scrollTop <= 5;
};

// 刷新事件处理
const onRefresh = async () => {
    emit('scrollChange', false);
    refresherTriggered.value = true;
    isRefresherVisible.value = true;
    refreshStatus.value = 'refreshing';
    refreshProgress.value = 1;
    currentPullDistance.value = props.refresherThreshold; // 刷新时保持阈值高度

    try {
        // 触发外部刷新事件
        emit('refresh');

        // 等待刷新完成
        await new Promise(resolve => {
            setTimeout(()=>{
                refreshStatus.value = 'success';
                setTimeout(resolve, 400);
            }, props.refreshDuration);
        });

        // 直接隐藏，不显示成功状态
        refresherTriggered.value = false;
        isRefresherVisible.value = false;
        refreshStatus.value = 'idle';
        refreshProgress.value = 0;
        currentPullDistance.value = 0;
        isPulling.value = false;
    } catch (error) {
        refresherTriggered.value = false;
        isRefresherVisible.value = false;
        refreshStatus.value = 'idle';
        refreshProgress.value = 0;
        currentPullDistance.value = 0;
        isPulling.value = false;
    }
};

// 恢复事件处理
const onRestore = () => {
    // 延迟隐藏，让动画更自然
    setTimeout(() => {
        isRefresherVisible.value = false;
        refreshStatus.value = 'idle';
        refreshProgress.value = 0;
        currentPullDistance.value = 0;
        emit('scrollChange', false);
    }, 100);

    emit('restore', {});
};

// 暴露方法
const stopRefresh = () => {
    refresherTriggered.value = false;
    isRefresherVisible.value = false;
    refreshStatus.value = 'idle';
    refreshProgress.value = 0;
    currentPullDistance.value = 0;
    isPulling.value = false;
};

defineExpose({
    stopRefresh
});
</script>

<style lang="scss" scoped>
.custom-scroll-refresh {
    position: relative;
    height: 100%;
    display: flex;
    flex-direction: column;
    overflow: hidden;

    .custom-scroll-refresh-top {
        position: relative;
        z-index: 1000;
        // background-color: #fff;
    }

    .custom-scroll-refresh-content {
        position: relative;
        overflow-y: auto;
        flex: 1;
        height: 0;
    }
}

.custom-refresher {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    z-index: 999;
    // background: #ECF3FF;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: opacity 0.2s ease;

    .refresher-content {
        width: 100%;
        height: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
        flex-direction: column;

        .refresh-state {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;

            .refresh-icon {
                width: 32rpx;
                height: 32rpx;
                border: 4rpx solid #E5E5E5;
                border-top: 4rpx solid #007AFF;
                border-radius: 50%;
                margin-bottom: 16rpx;

                &.refresh-loading {
                    animation: loading-rotate 1s linear infinite;
                }
            }

            .refresh-text {
                font-size: 24rpx;
                color: #8999B5;
            }
        }
    }

}

@keyframes loading-rotate {
    0% {
        transform: rotate(0deg);
    }
    100% {
        transform: rotate(360deg);
    }
}
</style>