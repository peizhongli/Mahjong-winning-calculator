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
		nextTick
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
	const boxWidth = ref(null)
	const boxHeight = ref(null)
	const wrapWidth = ref(null)
	const wrapHeight = ref(null)

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

	const updateNodeInfo = () => {
		const query = uni.createSelectorQuery();

		query.select(`.back`).boundingClientRect((data) => {
				console.log('back', data)
				if (data) {
					wrapWidth.value = data.width;
					wrapHeight.value = data.height;
				}
			}).select(`.item_${props.uniqKey}`).boundingClientRect((data) => {
				console.log('item', data)
				if (data) {
					boxWidth.value = data.width;
					boxHeight.value = data.height;
				}
			})
			.exec();
	}

	const stop = () => {
		if (animationTimer.value) {
			clearTimeout(animationTimer.value);
			animationTimer.value = null;
		}
		const newX = Math.max(left.value - 2, 0);
		const newY = Math.max(top.value - 2, 0);
		left.value = newX;
		top.value = newY;

		if (newX !== 0 || newY !== 0) {
			animationTimer.value = setTimeout(stop, 16);
		}
	}

	// 动画运行函数
	const run = async () => {
		if (!boxRef.value) return;
		if (animationTimer.value) {
			clearTimeout(animationTimer.value);
			animationTimer.value = null;
		}
		let currentLeft = left.value;
		let currentTop = top.value;

		// 计算下一帧位置
		let newX = currentLeft + xSpeed.value;
		let newY = currentTop + ySpeed.value;

		// 边界碰撞判断（反弹逻辑）
		if (newX + boxWidth.value >= wrapWidth.value) {
			newX = wrapWidth.value - boxWidth.value;
			xSpeed.value = -getRandomFloat(5, 10); // 反转X方向
		} else if (newX <= 0) {
			newX = 0;
			xSpeed.value = getRandomFloat(5, 10);
		}

		// 下边界
		if (newY + boxHeight.value >= wrapHeight.value) {
			newY = wrapHeight.value - boxHeight.value;
			ySpeed.value = -getRandomFloat(5, 10); // 反转Y方向
		} else if (newY <= 0) {
			newY = 0;
			ySpeed.value = getRandomFloat(5, 10);
		}

		left.value = newX;
		top.value = newY;

		// 持续执行动画
		if (props.inAnimation) {
			animationTimer.value = setTimeout(run, 16);
		}
	}

	watchEffect(() => {
		if (!props.inAnimation) {
			stop();
		}
	});

	onMounted(() => {
		updateNodeInfo();
	});

	defineExpose({
		start: run
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