<template>
	<div class="top"></div>
	<div class="content">
		<div class="row" :class="{'empty-row': scoreText === '请选择'}">{{ scoreText }}</div>
		<div v-show="calculateText" class="row calculate-row">
			<span>{{ calculateText }}</span>
		</div>
		<div v-show="scoreAfterText" class="row calculate-row-second">
			<span>{{ scoreAfterText }}</span>
			<span v-show="totalCaculateText">{{ totalCaculateText }}</span>
		</div>
	</div>
	<div class="bottom">
		<slot></slot>
		<div class="column">
			<div class="item" v-for="item in winning_pattern"
				:class="{'active': activeWinningsText.includes(item[0]), 'lose': item[0] === '点炮'}" :key="item[0]"
				@click="clickWinning(item)">
				<span>{{ item[0] }}</span>
				<span>{{ item[1] }}</span>
			</div>
		</div>
		<div class="column">
			<div class="item" v-for="item in card_pattern" :class="activePatternText.includes(item[0]) ? 'active':''"
				:key="item[0]" @click="clickPattern(item)">
				<span>{{ item[0] }}</span>
				<span>{{ item[1] }}</span>
			</div>
		</div>

		<div class="footer">
			<div class="left">
				<div class="num-wrap">
					<div v-for="i in 9" :key="i" class="opt-item operator number" @click="clickNumber(i)">
						{{i}}
					</div>
				</div>
			</div>
			<div class="right">
				<div class="operator-wrap">
					<div v-for="i in operator_list" :key="i" class="opt-item operator" @click="clickOperator(i)">
						{{i}}
					</div>
				</div>
				<div class="clear-wrap">
					<div class="clear-left">
						<div class="opt-item operator number" @click="clickNumber(0)">0</div>
						<div class="opt-item delete" @click="clickDelete">
							<image class="img" mode="heightFix" :src="deleteIcon" />
						</div>
					</div>
					<div class="clear-right">
						<div class="opt-item clear" @click="clickClear">
							<span>清 空</span>
						</div>
						<div class="opt-item caculate">
							<span>=</span>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script setup>
	import {
		ref,
		computed,
		onMounted,
		watchEffect,
		watch
	} from 'vue'
	import deleteIcon from '@/static/imgs/delete.png'

	// 1. 定义常量（保持原有业务逻辑）
	// 牌型（长春麻将番数）
	const card_pattern = [
		['夹胡', 2],
		['飘胡', 4],
		['飘顶', 8],
		['七小对', 16],
		['豪七', 32],
		['双豪七', 64],
	]
	// 胡牌方式
	const winning_pattern = [
		['自摸', 2],
		['摸宝', 4],
		['对宝', 8],
		['庄', 2],
		['站立', 2],
		['点炮', 2]
	]
	// 运算符
	const operator_list = ['+', '-', '×']

	const activeWinnings = ref([]) // 选中的胡牌方式
	const activePattern = ref([]) // 选中的牌型
	const activeOperators = ref([]) // 数字操作
	const historyQueue = ref([]) // 备份上一次的选项

	// 选中的胡牌方式名称
	const activeWinningsText = computed(() => {
		return activeWinnings.value.map(item => item[0])
	})

	// 选中的牌型名称
	const activePatternText = computed(() => {
		return activePattern.value.map(item => item[0])
	})

	// 倍数计算过程（如：自摸×2 × 夹胡×2）
	const scoreText = computed(() => {
		const allSelected = [...activeWinnings.value, ...activePattern.value]
		return allSelected.length ? allSelected.map(i => `${i[0]}×${i[1]}`).join(' × ') : '请选择'
	})

	// 基础倍数计算结果（所有选中项的乘积）
	const calculateText = computed(() => {
		const allSelected = [...activeWinnings.value, ...activePattern.value]
		if (!allSelected.length) return ''
		return allSelected.reduce((acc, cur) => acc * cur[1], 1)
	})

	// 倍数乘积和加蛋计算过程（如：x 2 + 4）
	const scoreAfterText = computed(() => {
		if (!activeOperators.value.length) {
			return ''
		}
		return `${calculateText.value || 1}${activeOperators.value.join('')}`
	})

	// 优化后的总分计算结果，遵循先乘后加减的运算规则
	const totalCaculateText = computed(() => {
		if (!activeOperators.value.length) {
			return ''
		}

		// 获取基础倍数，如果没有选择牌型则默认为1
		const baseMultiplier = calculateText.value || 1

		// 处理activeOperators，确保它是一个有效的计算序列
		const operators = [...activeOperators.value]

		// 如果最后一个元素是运算符，则移除它（末尾运算符不参与计算）
		const lastElement = operators[operators.length - 1]
		if (lastElement && operator_list.includes(lastElement)) {
			operators.pop()
		}

		// 如果没有有效操作符，直接返回基础倍数
		if (operators.length === 0) {
			return ''
		}

		// 优化计算逻辑：先处理乘法，再处理加减
		return evaluateExpression(baseMultiplier, operators)
	})

	// 优化后的表达式求值函数，遵循先乘后加减规则
	const evaluateExpression = (baseValue, operators) => {
		try {
			// 1. 将操作序列转换为表达式数组
			const expression = convertToExpressionArray(baseValue, operators)

			// 2. 先计算所有乘法
			const afterMultiplication = processMultiplication(expression)

			// 3. 再计算加减法
			return processAdditionSubtraction(afterMultiplication)
		} catch (error) {
			console.error('计算错误:', error)
			return '计算错误'
		}
	}

	// 将操作序列转换为表达式数组
	const convertToExpressionArray = (baseValue, operators) => {
		const expression = []

		// 添加基础值
		expression.push({
			type: 'number',
			value: baseValue
		})

		// 处理操作序列
		let currentNumber = ''
		let lastWasOperator = false

		for (let i = 0; i < operators.length; i++) {
			const element = operators[i]

			if (['+', '-', '×'].includes(element)) {
				// 如果是运算符，处理之前的数字
				if (currentNumber !== '') {
					expression.push({
						type: 'number',
						value: parseInt(currentNumber, 10)
					})
					currentNumber = ''
				}

				// 添加运算符
				expression.push({
					type: 'operator',
					value: element
				})
				lastWasOperator = true
			} else {
				// 如果是数字
				if (lastWasOperator) {
					currentNumber = element.toString()
				} else {
					// 如果上一个也是数字，合并成多位数
					currentNumber += element.toString()
				}
				lastWasOperator = false
			}
		}

		// 处理最后一个数字（如果存在）
		if (currentNumber !== '') {
			expression.push({
				type: 'number',
				value: parseInt(currentNumber, 10)
			})
		}

		return expression
	}

	// 处理乘法：先计算所有乘法运算
	const processMultiplication = (expression) => {
		const result = []
		let i = 0

		while (i < expression.length) {
			const current = expression[i]

			if (current.type === 'operator' && current.value === '×') {
				// 找到乘法运算符，取出前一个数字和下一个数字
				const prevNum = result.length > 0 ? result[result.length - 1] : expression[i - 1]
				const nextNum = expression[i + 1]

				// 确保都有值
				if (prevNum && nextNum && prevNum.type === 'number' && nextNum.type === 'number') {
					// 计算乘法
					const product = prevNum.value * nextNum.value

					// 从结果中移除前一个数字（如果已添加）
					if (result.length > 0 && result[result.length - 1] === prevNum) {
						result.pop()
					}

					// 添加乘积结果
					result.push({
						type: 'number',
						value: product
					})

					// 跳过已处理的数字
					i += 2
				} else {
					// 如果格式错误，直接添加当前元素
					result.push(current)
					i++
				}
			} else {
				// 非乘法运算符或数字，直接添加
				result.push(current)
				i++
			}
		}

		return result
	}

	// 处理加减法：从左到右计算
	const processAdditionSubtraction = (expression) => {
		let result = 0
		let currentOperator = '+'

		for (let i = 0; i < expression.length; i++) {
			const element = expression[i]

			if (element.type === 'number') {
				// 应用当前运算符
				if (currentOperator === '+') {
					result += element.value
				} else if (currentOperator === '-') {
					result -= element.value
				}
			} else if (element.type === 'operator') {
				// 更新当前运算符
				currentOperator = element.value
			}
		}

		return result
	}

	// 辅助函数：应用单个运算操作
	const applyOperation = (currentValue, number, operator) => {
		switch (operator) {
			case '+':
				return currentValue + number
			case '-':
				return currentValue - number
			case '×':
				return currentValue * number
			default:
				return currentValue
		}
	}

	const clickNumber = (i) => {
		// 检查是否可以输入数字：需要至少一个运算符开头
		if (activeOperators.value.length === 0 ||
			(activeOperators.value.length > 0 &&
				!['+', '-', '×'].includes(activeOperators.value[0]))) {
			// 如果第一个字符不是运算符，则不能输入数字
			return;
		}

		// 检查最后一个字符是不是运算符
		const lastElement = activeOperators.value[activeOperators.value.length - 1]
		if (lastElement && ['+', '-', '×'].includes(lastElement)) {
			// 最后一个字符是运算符，可以添加数字
			activeOperators.value.push(i)
		} else if (typeof lastElement === 'number' ||
			(typeof lastElement === 'string' && !isNaN(lastElement))) {
			// 最后一个字符是数字，继续追加数字（形成多位数）
			const lastIndex = activeOperators.value.length - 1
			const newValue = parseInt(activeOperators.value[lastIndex].toString() + i.toString(), 10)
			activeOperators.value[lastIndex] = newValue
		}
	}

	const clickOperator = (i) => {
		if (activeOperators.value.length === 0) {
			// 第一个字符必须是运算符
			activeOperators.value.push(i)
		} else {
			const lastElement = activeOperators.value[activeOperators.value.length - 1]

			if (['+', '-', '×'].includes(lastElement)) {
				// 如果最后一个是运算符，替换它
				activeOperators.value[activeOperators.value.length - 1] = i
			} else if (typeof lastElement === 'number' ||
				(typeof lastElement === 'string' && !isNaN(lastElement))) {
				// 如果最后一个是数字，添加运算符
				activeOperators.value.push(i)
			}
		}
	}

	const clickDelete = () => {
		if (activeOperators.value.length === 0) {
			return;
		}

		const lastElement = activeOperators.value[activeOperators.value.length - 1]

		if (typeof lastElement === 'number' && lastElement > 9) {
			// 如果是多位数，删除最后一位
			const newValue = Math.floor(lastElement / 10)
			if (newValue === 0) {
				// 如果删除后变为0，直接移除这个元素
				activeOperators.value.pop()
			} else {
				activeOperators.value[activeOperators.value.length - 1] = newValue
			}
		} else {
			// 如果是单数字或运算符，直接删除
			activeOperators.value.pop()
		}
	}

	// 选择胡牌方式/其他规则
	const clickWinning = (item) => {
		let newActiveWinnings = [...activeWinnings.value]
		const isExist = newActiveWinnings.some(i => i[0] === item[0])

		// 切换选中状态
		if (isExist) {
			newActiveWinnings = newActiveWinnings.filter(i => i[0] !== item[0])
		} else {
			newActiveWinnings.push(item)
			// 自摸/摸宝/对宝/点炮 互斥
			if (['自摸', '摸宝', '对宝', '点炮'].includes(item[0])) {
				newActiveWinnings = newActiveWinnings.filter(i => i[0] === item[0] || !['自摸', '摸宝', '对宝', '点炮']
					.includes(i[0]))
			}
			if (['站立'].includes(item[0])) {
				activePattern.value = activePattern.value.filter(i =>
					!['七小对', '豪七', '双豪七'].includes(i[0])
				)
			}

		}
		activeWinnings.value = newActiveWinnings
	}

	// 选择牌型 全部互斥
	const clickPattern = (item) => {
		// 添加一条历史记录
		activePattern.value = activePattern.value.some(i => i[0] === item[0]) ? [] : [item]
		if (['七小对', '豪七', '双豪七'].includes(item[0])) {
			activeWinnings.value = activeWinnings.value.filter(i =>
				!['站立'].includes(i[0])
			)
		}
	}

	// 清空所有选择
	const clickClear = () => {
		activeWinnings.value = []
		activePattern.value = []
		activeOperators.value = []
		historyQueue.value = []
	}


	onMounted(() => {})
</script>
<style lang="scss" scoped>
	@function grid-repeat($count) {
		@return repeat($count, 1fr);
	}

	@mixin grid-layout($count: 1, $gap: 6px) {
		display: grid;
		gap: $gap;
		grid-template-columns: grid-repeat($count);
	}

	.content {
		flex: 1;
		overflow-y: auto;
		padding: 8px 12px;
		color: #ff8615;
		font-size: 30px;

		.row {
			&.empty-row {
				color: #ccc;
			}
		}

		.calculate-row {
			display: flex;
			justify-content: flex-end;
			margin-top: 12px;
			font-size: 32px;
			// font-weight: bold;

			span {
				&:before {
					content: '=';
					margin-right: 4px;
				}
			}
		}

		.calculate-row-second {
			border-top: 2px solid #ff8615;
			padding-top: 6px;
			margin-top: 4px;
			text-align: right;

			span:nth-child(2) {
				&:before {
					content: '=';
					margin: 0 4px;
				}
			}
		}

		.calculate-total-row {
			margin-top: 20px;
			text-align: center;
			font-size: 36px;
			font-weight: bold;
			color: #f50;
		}
	}

	.bottom {
		flex-shrink: 0;
		padding: 6px;
	}

	// 牌型
	.column {
		@include grid-layout(6);

		&+.column {
			margin-top: 6px;
		}

		.item {
			aspect-ratio: 1 / 1.3;
		}
	}

	// 数字
	.num-wrap {
		@include grid-layout(3);

		.opt-item {
			width: 100%;
			font-size: 17px;
		}
	}

	.num-wrap,
	.operator-wrap,
	.clear-left {
		.opt-item {
			aspect-ratio: 1 / 1;
		}
	}

	.footer {
		@include grid-layout(2);
		margin-top: 6px;

		.right {
			display: flex;
			flex-direction: column;
			gap: 6px;

			.operator-wrap {
				@include grid-layout(3);
			}

			.clear-wrap {
				@include grid-layout(3);
				flex: 1;
			}

			.clear-left,
			.clear-right {
				display: flex;
				gap: 6px;
				flex-direction: column;
			}

			.clear-right {
				grid-column: span 2;

				.opt-item {
					flex: 1;
				}
			}

			.opt-item {
				font-size: 16px;
			}

			.operator:not(.number) {
				color: #ff8615;
				font-size: 25px;
				font-weight: bold;
			}

			.delete {
				.img {
					height: 20px;
					margin-top: 5px;
				}
			}

			.clear {
				grid-column: span 2;
				color: #ff8615;
				font-size: 16px;
			}

			.caculate {
				color: #fff;
				background-color: #ff8615;
				font-size: 17px;
				font-weight: bold;
			}
		}
	}


	.item,
	.opt-item {
		background-color: #fff;
		border-radius: 12px;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		font-size: 15px;
		user-select: none;
		-webkit-user-select: none;
		color: #1f1f1f;

		&:active {
			box-shadow: 0 1px 2px rgba(0, 0, 0, 0.04), inset 0 1px 3px rgba(0, 0, 0, 0.06) !important;
		}
	}

	.item {
		transition: border .2s;
		border: 1px solid transparent;

		span {
			&:last-child {
				color: rgb(255, 134, 21);
				font-size: 17px;
				font-weight: bold;

				&:before {
					content: '×'
				}
			}
		}

		&.active {
			border: 1px solid #87d068;
			box-shadow: 0 0 4px #87d068ff;
			color: #87d068;
			font-weight: bold;
			font-size: 16px;

			span {
				&:last-child {
					font-size: 18px;
				}
			}
		}

		&.lose {
			color: #f50;

			&.active {
				border-color: #f50;
				box-shadow: 0 0 4px #ff5500ff;
			}
		}
	}
</style>