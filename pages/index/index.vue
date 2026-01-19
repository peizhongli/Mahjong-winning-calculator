<template>
	<view class="wrap">
		<Caculator><draw-lots /></Caculator>
	</view>
</template>

<script>
	import Caculator from '../caculator/index.vue'
	import DrawLots from '../drawLots/index.vue'
	export default {
		data() {
			return {

			}
		},
		components: {
			Caculator,
			DrawLots
		},
		onLoad() {},
		computed: {

		},
		methods: {

		}
	}
</script>

<style lang="scss">
	* {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
	}

	.wrap {
		height: 100%;
		padding-bottom: 0;
	}
</style>

<style lang="scss" scoped>
	.wrap {
		width: 100vw;
		// 适配顶部+底部安全区
		height: calc(100vh - 44px - env(safe-area-inset-top) - env(safe-area-inset-bottom));
		display: flex;
		flex-direction: column;
		background-color: rgb(247, 247, 247);
	}
</style>