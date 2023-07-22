<template>
	<view class="container"><button type="default" @click="goto">点餐</button></view>
</template>

<script>
export default {
	data() {
		return {
		};
	},
	mounted() {
	},
	methods: {
		goto() {
			uni.navigateTo({
				url: '/pages/order/index'
			});
		}
	}
};
</script>

<style lang="less">
.container {
	padding: 20px;
	font-size: 14px;
	line-height: 24px;
}
</style>
