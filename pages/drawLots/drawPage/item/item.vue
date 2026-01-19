<template>
	<view class="wrap">
		<view ref="boxRef" class="item" :class="`item_${uniqKey}`" :style="itemStyle" @click="handleClickItem">
			<image mode="widthFix" class="img" :src="back" alt="背景图" />
		</view>
	</view>
</template>

<script setup>
	import back from '@/static/imgs/back_2.png'
	import {
		computed,
		onMounted,
		onUnmounted,
		ref,
		watchEffect,
		nextTick,
		watch
	} from 'vue';

	const getRandomFloat = (min, max) => Math.random() * (max - min) + min

	const emit = defineEmits(['clickItem'])
	const props = defineProps(['inAnimation', 'clickItem', 'uniqKey'])

	const boxRef = ref(null);
	const animationTimer = ref(null)
	const left = ref(0)
	const top = ref(0)
	const xSpeed = ref(getRandomFloat(5, 8));
	const ySpeed = ref(getRandomFloat(5, 8));
	const boxWidth = ref(0)
	const boxHeight = ref(0)
	const wrapWidth = ref(0)
	const wrapHeight = ref(0)
	const boxLeft = ref(null);
	const boxTop = ref(null);

	const itemStyle = computed(() => {
		return {
			left: `${left.value}px`,
			top: `${top.value}px`,
			position: 'absolute',
		}
	})

	const handleClickItem = () => {
		emit('clickItem')
	}

	const safeRequestAnimationFrame = (callback) => {

		return setTimeout(() => {
			callback();
		}, 16);
	}

	const updateNodeInfo = () => {
		return new Promise((resolve, reject) => {
			try {
				const query = uni.createSelectorQuery();

				// 使用 fields 方法同时获取多个节点信息
				query.select(`.back`).fields({
					size: true,
					rect: true
				}, (wrapData) => {
					let wrapLeft = 0;
					let wrapTop = 0;
					if (wrapData) {
						wrapLeft = wrapData.left
						wrapTop = wrapData.top
						wrapWidth.value = wrapData.width;
						wrapHeight.value = wrapData.height;
					}

					// 查询自身节点
					const query2 = uni.createSelectorQuery();
					query2.select(`.item_${props.uniqKey}`).fields({
						size: true,
						rect: true
					}, (itemData) => {
						if (itemData) {
							boxWidth.value = itemData.width;
							boxHeight.value = itemData.height;
							boxLeft.value = itemData.left - wrapLeft;
							boxTop.value = itemData.top - wrapTop;
							resolve()
						}
					}).exec();
				}).exec();
			} catch (error) {
				reject(error)
			}
		})
	}

	const stop = () => {
		const newX = Math.max(left.value - 2, 0);
		const newY = Math.max(top.value - 2, 0);
		left.value = newX;
		top.value = newY;
		if (newX !== 0 || newY !== 0) {
			console.log(props.uniqKey, 'newX', newX, 'newY', newY)
			animationTimer.value = safeRequestAnimationFrame(stop);
		}
	}

	// 动画运行函数
	const run = async () => {
		if (!boxRef.value) return;

		if (boxLeft.value === null) {
			await updateNodeInfo()
		}
		let currentLeft = left.value;
		let currentTop = top.value;

		// 计算下一帧位置
		let newX = currentLeft + xSpeed.value;
		let newY = currentTop + ySpeed.value;

		// 边界碰撞判断（反弹逻辑）
		if (newX + boxWidth.value + boxLeft.value >= wrapWidth.value) {
			newX = wrapWidth.value - boxWidth.value - boxLeft.value;
			xSpeed.value = -getRandomFloat(5, 10); // 反转X方向
		} else if (newX <= -boxLeft.value) {
			newX = -boxLeft.value;
			xSpeed.value = getRandomFloat(5, 10);
		}

		// 下边界
		if (newY + boxHeight.value + boxTop.value >= wrapHeight.value) {
			newY = wrapHeight.value - boxHeight.value - boxTop.value;
			ySpeed.value = -getRandomFloat(5, 10); // 反转Y方向
		} else if (newY <= -boxTop.value) {
			newY = -boxTop.value;
			ySpeed.value = getRandomFloat(5, 10);
		}

		left.value = newX;
		top.value = newY;

		// 持续执行动画
		if (props.inAnimation) {
			animationTimer.value = safeRequestAnimationFrame(run);
		}
	}

	// 使用 watch 而不是 watchEffect，提供更明确的清理逻辑
	watch(() => props.inAnimation, (newVal) => {
		if (!newVal) {
			if (animationTimer.value) {
				clearTimeout(animationTimer.value);
				animationTimer.value = null;
			}
			stop();
		}
	});

	// 暴露的方法
	const start = () => {
		run();
	}

	defineExpose({
		start
	})

	onUnmounted(() => {
		if (animationTimer.value) {
			clearTimeout(animationTimer.value);
			animationTimer.value = null;
		}
	});
</script>

<style lang="scss" scoped>
	.wrap {
		position: relative;
		width: 100%;
		height: 100%; // 添加高度以确保容器有尺寸
	}

	.item {
		position: absolute;
		width: 100%;

		.img {
			display: block;
			width: 100%;
		}
	}
</style>