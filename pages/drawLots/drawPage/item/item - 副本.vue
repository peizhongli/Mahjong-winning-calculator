<template>
	<view>
		<div ref="boxRef" class="item" :style="itemStyle" @click="handleClickItem">
			<image mode="widthFix" class="img" :src="back" alt="背景图" />
		</div>
	</view>
</template>

<script setup>
	import back from '@/static/imgs/back_2.png'
	import {
		computed,
		onMounted,
		onUnmounted,
		ref,
		watchEffect
	} from 'vue';

	const getRandomFloat = (min, max) => Math.random() * (max - min) + min

	const emit = defineEmits(['clickItem'])
	const props = defineProps(['inAnimation', 'parentRef', 'clickItem'])

	const boxRef = ref(null);
	const left = ref(0)
	const animationFrameId = ref(null)
	const top = ref(0)
	const xSpeed = ref(getRandomFloat(5, 8));
	const ySpeed = ref(getRandomFloat(5, 8));
	const boxLeft = ref(null);
	const boxTop = ref(null);

	const itemStyle = computed(() => {
		return {
			left: `${left.value}px`,
			top: `${top.value}px`,
		}
	})

	const handleClickItem = () => {
		emit('clickItem')
	}

	const stop = () => {
		const stopSpeed = 5;
		const newX = Math.max(left.value - 2, 0);
		const newY = Math.max(top.value - 2, 0);
		left.value = newX;
		top.value = newY;

		if (newX === 0 && newY === 0) {
			window.cancelAnimationFrame(animationFrameId.value);
		} else {
			// 持续执行动画
			animationFrameId.value = window.requestAnimationFrame(stop);
		}
	}
	// 动画运行函数
	const run = () => {
		const parent = props.parentRef
		if (!boxRef.value || !parent) return;
		// 获取父容器（边界）和盒子的DOM对象
		const wrap = parent;
		const box = boxRef.value;

		if (boxLeft.value === null || boxTop.value === null) {
			boxLeft.value = box.offsetLeft;
			boxTop.value = box.offsetTop;
		}

		// 获取父容器的实际宽高（移动边界）
		const wrapWidth = wrap.clientWidth;
		const wrapHeight = wrap.clientHeight;
		// 获取盒子自身尺寸
		const boxWidth = box.clientWidth;
		const boxHeight = box.clientHeight;

		let currentLeft = left.value;
		let currentTop = top.value;

		// 计算下一帧位置
		let newX = currentLeft + xSpeed.value;
		let newY = currentTop + ySpeed.value;

		// 边界碰撞判断（反弹逻辑）
		if (newX + boxWidth + boxLeft.value >= wrapWidth) {
			// console.log('👉到达右边界')
			newX = wrapWidth - boxWidth - boxLeft.value;
			xSpeed.value = -getRandomFloat(5, 10); // 反转X方向
		} else if (newX <= -boxLeft.value) {
			newX = -boxLeft.value;
			xSpeed.value = getRandomFloat(5, 10);
		}

		// 下边界
		if (newY + boxHeight + boxTop.value >= wrapHeight) {
			newY = wrapHeight - boxHeight - boxTop.value;
			ySpeed.value = -getRandomFloat(5, 10); // 反转Y方向
			// console.log('👇到达下边界')
		} else if (newY <= -boxTop.value) {
			newY = -boxTop.value;
			ySpeed.value = getRandomFloat(5, 10);
			// console.log('👆到达上边界')
		}

		left.value = newX;
		top.value = newY;

		// 持续执行动画
		animationFrameId.value = window.requestAnimationFrame(run);
	}

	watchEffect(() => {
		if (!props.inAnimation) {
			if (animationFrameId.value) {
				window.cancelAnimationFrame(animationFrameId.value);
			}
			animationFrameId.value = null
			stop()

		}
	})

	defineExpose({
		start: run
	})

	onUnmounted(() => {
		if (animationFrameId.value) {
			window.cancelAnimationFrame(animationFrameId.value);
		}
	});
</script>

<style lang="scss" scoped>
	.item {
		position: relative;

		.img {
			display: block;
			width: 100%;
		}
	}
</style>