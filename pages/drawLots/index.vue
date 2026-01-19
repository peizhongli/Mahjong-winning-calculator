<template>
	<view>
		<div class="btn">
			<image class="img" mode="widthFix" :src="bgImg" @click="showModal = true" />
		</div>
		<template v-if="showModal"><draw-page @onClose="()=>handleClose()"></draw-page></template>
	</view>
</template>

<script setup>
	import bgImg from '@/static/imgs/back.png'
	import DrawPage from './drawPage/drawPage.vue'
	import {
		ref,
	} from 'vue'

	const showModal = ref(false) // 布尔值响应式变量

	const handleClose = () => {
		showModal.value = false
	}
</script>

<style lang="scss">
	.btn {
		display: flex;
		justify-content: flex-end;

		.img {
			width: 60px;
			display: block;
			cursor: pointer;
		}
	}
</style>