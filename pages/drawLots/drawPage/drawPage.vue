<template>
	<view>
		<div class="container">
			<div class="content">
				<div>
					<div v-if="isBack" class="item-wrap back" ref="wrapRef" @click="handleBackWrapClick">
						<item ref="childRefs" v-for="(i,index) in 4" :uniqKey="index" :key="i"
							:inAnimation="isAnimation" @clickItem="()=>handleItemClick(index)" />
					</div>
					<div v-else class="item-wrap" @click="handleFrontWrapClick">
						<image mode="widthFix" class="img" v-for="(i,index) in 4"
							:src="showResult.has(index) ? numToImg[cardList[index]] : back"
							@click="()=>handleItemClick(index)" />
					</div>
				</div>
				<div class="btn-wrap" @click.stop>
					<div class="reset-btn" @click="()=>handleResetClick()">重来</div>
					<div class="close-btn" @click="()=>handleClose()"></div>
				</div>
			</div>
		</div>
	</view>
</template>

<script setup>
	import north from '@/static/imgs/north.png'
	import south from '@/static/imgs/south.png'
	import west from '@/static/imgs/west.png'
	import east from '@/static/imgs/east.png'
	import back from '@/static/imgs/back.png'
	import Item from './item/item.vue'
	import {
		ref,
		defineEmits
	} from 'vue'

	const numToImg = {
		1: east,
		2: south,
		3: west,
		4: north
	}

	const isBack = ref(true)
	const isAnimation = ref(false)
	const childRefs = ref([])
	const wrapRef = ref(null)
	const cardList = ref([])
	const waitResult = ref(false)
	const showResult = ref(new Set())

	const emit = defineEmits(['onClose'])

	const handleClose = () => {
		emit('onClose')
	}

	const shuffleArray = (arr) => {
		// 复制原数组，避免修改原始数据
		const newArr = [...arr];
		// 从最后一个元素开始向前遍历
		for (let i = newArr.length - 1; i > 0; i--) {
			// 生成 0 ~ i 之间的随机整数（包含i）
			const randomIndex = Math.floor(Math.random() * (i + 1));
			// 交换当前元素和随机位置的元素
			[newArr[i], newArr[randomIndex]] = [newArr[randomIndex], newArr[i]];
		}
		return newArr;
	}

	const handleBackWrapClick = () => {
		if (waitResult.value) {
			return;
		}
		if (!isAnimation.value) {
			isBack.value = true
			childRefs.value.forEach((i => {
				i.start()
			}))

		} else {
			waitResult.value = true
			cardList.value = shuffleArray([1, 2, 3, 4])
		}
		isAnimation.value = !isAnimation.value
	}

	const handleResetClick = () => {
		isBack.value = true
		showResult.value.clear()
		waitResult.value = false
	}

	const handleItemClick = (index) => {
		if (isAnimation.value) {
			return;
		}
		if (waitResult.value && showResult.value.size < 4) {
			showResult.value.add(index)
			isBack.value = false
		}
	}
</script>

<style lang="scss" scoped>
	.container {
		width: 100vw;
		height: 100vh;
		position: fixed;
		top: 0;
		left: 0;
		background-color: rgba(0, 0, 0, 0.7);
		display: flex;
		align-items: center;
		justify-content: center;
		z-index: 99;

		.content {
			width: 85vw;
			border-radius: 8px;
			background-color: #fff;
			position: relative;
		}

		.btn-wrap {
			margin: 0 auto;
			position: absolute;
			top: -46px;
			display: flex;
			align-items: center;
			justify-content: center;
			gap: 12px;
			width: 100%;
		}

		.reset-btn,
		.close-btn {
			background-color: #fff;
			border: 2px solid #1f1f1f;
			border-radius: 6px;
			cursor: pointer;
			padding: 2px 6px;
			width: 60px;
			text-align: center;
			height: 35px;
			box-sizing: border-box;
			cursor: pointer;
			position: relative;
		}

		.reset-btn {
			color: #1f1f1f;
			font-size: 20px;
		}

		.close-btn {

			&::before,
			&::after {
				content: '';
				display: block;
				background-color: #1f1f1f;
				width: 2px;
				height: 20px;
				transform-origin: center;
				position: absolute;
				border-radius: 1px;
				top: 6px;
				left: 27px;
			}

			&::before {
				transform: rotate(45deg);
			}

			&::after {
				transform: rotate(-45deg);
			}
		}
	}

	.item-wrap {
		display: grid;
		grid-template-columns: repeat(2, 1fr);
		padding: 30px;
		position: relative;
		box-sizing: border-box;
		height: 44vh;

		.img {
			display: block;
			width: 100%;
		}

		&.back {
			gap: 30px;
		}
	}
</style>