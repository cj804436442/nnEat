<template>
	<view class="container">
		<view class="order_wrap">
			<scroll-view class="order_left" :scroll-into-view="menuScrollIntoView" scroll-with-animation scroll-y>
				<view :class="['order_item', currentId == item.id ? 'item_choosed' : '']" v-for="(item, index) in goods" :key="index" @click="chooseMenu(item.id)">
					{{ item.name }}
				</view>
			</scroll-view>
			<scroll-view class="order_right" scroll-with-animation scroll-y :scroll-top="cateScrollTop" @scroll="handleGoodsScroll">
				<view class="right_content">
					<view class="right_wrap" ref="contentItem" v-for="(item, index) in goods" :key="index" :id="`cate-${item.id}`">
						<view class="order_type">{{ item.name }}</view>
						<view class="food_item" v-for="(food, key) in item.list" :key="key">
							<view class="list_left"><image class="food_pic" src="../../static/c1.png" mode=""></image></view>
							<view class="list_right">
								<view class="food_name">{{ food.name }}</view>
								<view class="describtion">有口味要求请备注一下~</view>
								<view class="price">¥ {{ food.price }}个亲亲</view>
								<view class="icon_wrap">
									<view class="icon_box">
										<image v-if="food.total > 0" class="minus" src="../../static/minus.png" mode="" @click="minus(food)"></image>
										<text v-if="food.total > 0" class="food_total">{{ food.total }}</text>
										<image class="add" src="../../static/add.png" mode="" @click="add(food)"></image>
									</view>
								</view>
							</view>
						</view>
						<view class="h20"></view>
					</view>
				</view>
			</scroll-view>
		</view>
		<view class="ordered">
			<view class="ordered_left">
				<view class="eat_wrap">
					<image class="eat_icon" src="../../static/eat_icon.png" mode=""></image>
					<text class="eat_num" v-if="totalNum > 0">{{ totalNum }}</text>
				</view>
				<view class="total_text">
					<view class="total">¥{{ money }}个亲亲</view>
					<view class="total_des">另需配送费¥4个亲亲</view>
				</view>
			</view>
			<view class="ordered_right" :class="['ordered_right', totalNum > 0 ? 'blue' : '']" @click="openPopup">
				<view class="confirm'">{{ confirmText }}</view>
			</view>
		</view>
	</view>
</template>
<script>
import { getCurrentTime } from "@/common/util.js"
export default {
	data() {
		return {
			menuScrollIntoView: '',
			currentId: 'A01',
			cateScrollTop: 1,
			sizeCalcState: false,
			confirmText: '¥1个亲亲起',
			totalNum: 0,
			money: 0,
			openid: '',
			sessionKey: '',
			unionid: '',
			accessToken: '',
			foodSelectList: [],
			foodList: [],
			goods: [
				{
					name: '红烧菜',
					id: 'A01',
					list: [
						{
							name: '红烧肉',
							price: '1',
							total: ''
						},
						{
							name: '红烧鱼',
							price: '2',
							total: ''
						},
						{
							name: '红烧鸭',
							price: '1',
							total: ''
						},
						{
							name: '红烧猪蹄',
							price: '3',
							total: ''
						}
					]
				},
				{
					name: '蔬菜类',
					id: 'A02',
					list: [
						{
							name: '青菜炒香菇',
							price: '1',
							total: ''
						},
						{
							name: '酸辣土豆丝',
							price: '1',
							total: ''
						},
						{
							name: '炒茄子',
							price: '1',
							total: ''
						},
						{
							name: '炒莴苣',
							price: '1',
							total: ''
						},
						{
							name: '地三鲜',
							price: '2',
							total: ''
						}
					]
				},
				{
					name: '汤类',
					id: 'A03',
					list: [
						{
							name: '西红柿蛋汤',
							price: '1',
							total: ''
						},
						{
							name: '排骨汤',
							price: '3',
							total: ''
						}
					]
				},
				{
					name: '泡面',
					id: 'A04',
					list: [
						{
							name: '泡面1',
							price: '1',
							total: ''
						},
						{
							name: '泡面2',
							price: '1',
							total: ''
						},
						{
							name: '泡面3',
							price: '1',
							total: ''
						},
						{
							name: '泡面4',
							price: '1',
							total: ''
						}
					]
				}
			]
		};
	},
	watch: {
		money: {
			handler(newVal, oldVal) {
				if (newVal > 0) {
					this.confirmText = '去下单';
				} else {
					this.confirmText = '¥1个亲亲起';
				}
			},
			immediate: true
		}
	},
	mounted() {
		this.getUnionid();
		this.getAccessToken();
	},
	methods: {
		//发送消息推送
		messagePush() {
			let food = ""
			this.foodList.forEach((item,index)=> {
				food = food + item.name + '*' + item.num + '\n'
			})
			wx.request({
				url: `https://api.weixin.qq.com/cgi-bin/message/subscribe/send?access_token=${this.accessToken}`,
				method: 'POST',
				data: {
					template_id: '7QRjzGha0QmNF53BHi2EEPhKkfN56lVN8N86JMrcdtE',
					touser: 'ow5-y5R4k9Icf2GHOgTxgFB8bVxY',
					data: {
						thing2: { value: food }, //点餐内容
						time4: { value: getCurrentTime() }, //点餐时间
						thing9: { value: '暂无' }, //备注
						amount10: { value: this.money } //消费金额
					},
					miniprogram_state: 'trial',
					lang: 'zh_CN'
				},
				success: response => {
					console.log('response：' + JSON.stringify(response));
					uni.showToast({
						title: '下单成功',
						//将值设置为 success 或者直接不用写icon这个参数
						icon: 'success',
						//显示持续时间为 2秒
						duration: 2000
					})  
				},
				fail: error => {
					console.log('请求失败：', error);
				}
			});
		},
		// 是否设置过授权
		openPopup() {
			const _this = this;
			// 获取用户的当前设置，判断是否点击了“总是保持以上，不在询问”
			wx.getSetting({
				withSubscriptions: true, // 是否获取用户订阅消息的订阅状态，默认false不返回
				success(res) {
					console.log('res.authSetting', res.authSetting);
					if (res.authSetting['scope.subscribeMessage']) {
						console.log('用户点击了“总是保持以上，不再询问”');
					} else {
						console.log('用户没有点击“总是保持以上，不再询问”则每次都会调起订阅消息');
						//因为没有选择总是保持，所以需要调起授权弹窗再次授权
						_this.authorizationBtn();
					}
				}
			});
		},
		// 授权
		authorizationBtn() {
			const _this = this;
			wx.requestSubscribeMessage({
				tmplIds: ['7QRjzGha0QmNF53BHi2EEPhKkfN56lVN8N86JMrcdtE'],
				success(res) {
					console.log('授权成功');
					_this.goOrder()
				}
			});
		},
		goOrder() {
			if (this.money > 0) {
				// wx.request({
				// 	url: 'http://localhost:5526/api/order',
				// 	method: 'POST',
				// 	data: {
				// 		food: '红烧肉11',
				// 		orderTime: '2023-06-10 17:13:06',
				// 		remark: '22',
				// 		price: '1'
				// 	},
				// 	success: function(res) {
				// 		// 请求成功的回调函数
				// 		console.log('订单提交成功');
				// 		// 在这里可以处理接口返回的结果，或执行其他操作
				// 	},
				// 	fail: function(error) {
				// 		// 请求失败的回调函数
				// 		console.log('订单提交失败', error);
				// 		// 在这里可以处理请求失败的情况，如显示错误提示等
				// 	}
				// });
				this.messagePush();
			} else {
				return;
			}
		},
		add(food) {
			food.total++;
			this.money += Number(food.price);
			this.totalNum++;
			this.foodSelectList.push(food.name);
			var occurrences = {};
			for (var i = 0; i < this.foodSelectList.length; i++) {
				var element = this.foodSelectList[i];
				if (occurrences[element]) {
					occurrences[element]++;
				} else {
					occurrences[element] = 1;
				}
			}
			this.foodList = Object.entries(occurrences).map((item)=>{
				return {
					name: item[0],
					num: item[1]
				}
			})
		},
		minus(food) {
			food.total--;
			this.totalNum--;
			this.money -= Number(food.price);
			let index = this.foodSelectList.indexOf(food.name)
			if(index !== -1) {
				this.foodSelectList.splice(index,1)
			}
			var occurrences = {};
			for (var i = 0; i < this.foodSelectList.length; i++) {
				var element = this.foodSelectList[i];
				if (occurrences[element]) {
					occurrences[element]++;
				} else {
					occurrences[element] = 1;
				}
			}
			this.foodList = Object.entries(occurrences).map((item)=>{
				return {
					name: item[0],
					num: item[1]
				}
			})
		},
		chooseMenu(id) {
			//点击菜单项事件
			if (!this.sizeCalcState) {
				this.calcSize();
			}
			this.currentId = id;
			this.$nextTick(() => (this.cateScrollTop = this.goods.find(item => item.id == id).top));
		},
		handleGoodsScroll({ detail }) {
			//商品列表滚动事件
			if (!this.sizeCalcState) {
				this.calcSize();
			}
			const { scrollTop } = detail;
			let tabs = this.goods.filter(item => item.top <= scrollTop).reverse();
			if (tabs.length > 0) {
				// this.goodsName = tabs[0].name;
				this.currentId = tabs[0].id;
			}
		},
		calcSize() {
			let h = 0;
			this.goods.forEach(item => {
				let view = uni.createSelectorQuery().select(`#cate-${item.id}`);
				view.fields(
					{
						size: true
					},
					data => {
						item.top = h;
						h += data.height;
						item.bottom = h;
					}
				).exec();
			});
			this.sizeCalcState = true;
		},
		getUnionid() {
			wx.login({
				success: function(res) {
					if (res.code) {
						// 将 code 发送给开发者服务器
						wx.request({
							url: 'https://api.weixin.qq.com/sns/jscode2session',
							data: {
								appid: 'wx71fab1cea420c751',
								secret: 'f56a372069b972044fb00beba1422f7e',
								js_code: res.code,
								grant_type: 'authorization_code'
							},
							success: response => {
								this.openid = response.data.openid;
								this.sessionKey = response.data.session_key;
								this.unionid = response.data.unionid;
								console.log('openid：' + this.openid);
								// 处理返回的用户信息
								// 可以将 openid、sessionKey、unionid 存储到本地或发送到服务器进行进一步处理
							},
							fail: error => {
								console.log('请求失败：', error);
							}
						});
					} else {
						console.log('登录失败：', res.errMsg);
					}
				},
				fail: function(error) {
					console.log('登录失败：', error);
				}
			});
		},
		getAccessToken() {
			wx.request({
				url: 'https://api.weixin.qq.com/cgi-bin/token',
				data: {
					grant_type: 'client_credential',
					appid: 'wx71fab1cea420c751',
					secret: 'f56a372069b972044fb00beba1422f7e'
				},
				success: response => {
					this.accessToken = response.data.access_token;
					console.log('accessToken：' + this.accessToken);
				},
				fail: error => {
					console.log('请求失败：', error);
				}
			});
		}
	}
};
</script>

<style lang="less">
.container {
	font-size: 14px;
	line-height: 24px;
}
.order_wrap {
	display: flex;
	flex-direction: row;
}
.order_left {
	display: flex;
	flex-direction: column;
	align-items: center;
	width: 100px;
	height: calc(100vh - 88px);
	overflow: scroll;
	background: #f1f1f1;
	.order_item {
		display: flex;
		justify-content: center;
		align-items: center;
		background: #f1f1f1;
		min-height: 40px;
		width: 100%;
		color: #666;
	}
	.item_choosed {
		background: #ffffff;
		color: #333333;
	}
	.item_choosed + .order_item {
		border-radius: 0 16px 0 0;
	}
	.border_radius {
		border-radius: 0 0 16px 0;
	}
}
.order_right {
	width: calc(100% - 100px);
	height: calc(100vh - 88px);
	background: #ffffff;
	overflow: scroll;
	.right_content {
		padding: 10px;
		.right_wrap {
			.h20 {
				height: 20px;
			}
		}
	}
	.order_type {
		color: #333333;
		font-size: 14px;
		font-weight: 600;
	}
	.food_item {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		align-items: center;
		width: 100%;
		height: 100px;
		margin-top: 20px;
	}
	.list_left {
		background-color: pink;
		height: 100px;
		.food_pic {
			width: 100px;
			height: 100px;
		}
	}
	.list_right {
		position: relative;
		display: flex;
		flex-direction: column;
		width: calc(100% - 60px);
		height: 100%;
		padding-left: 10px;
		.food_name {
			font-size: 14px;
			color: #333333;
			font-weight: 600;
		}
		.describtion {
			color: #999999;
			font-size: 10px;
		}
		.price {
			font-size: 14px;
			color: #e45034;
			font-weight: 600;
			position: absolute;
			bottom: 0;
		}
		.icon_wrap {
			position: absolute;
			height: 20px;
			bottom: 4px;
			right: 8px;
			width: 66px;
			text-align: right;
			.icon_box {
				position: relative;
				.food_total {
					position: absolute;
					width: 26px;
					right: 21px;
					text-align: center;
					top: -2px;
				}
				.add {
					position: absolute;
					width: 20px;
					height: 20px;
					right: 0;
				}
				.minus {
					position: absolute;
					width: 20px;
					height: 20px;
					left: 0;
				}
			}
		}
	}
}
.ordered {
	display: flex;
	flex-direction: row;
	justify-content: space-between;
	align-items: flex-start;
	width: 100%;
	height: 80px;
	position: fixed;
	bottom: 0;
	padding-top: 8px;
	background: #ffffff;
	.ordered_left {
		display: flex;
		flex-direction: row;
		align-items: center;
		width: calc(100% - 120px);
		margin-left: 16px;
		.eat_wrap {
			position: relative;
			width: 60px;
			.eat_icon {
				width: 48px;
				height: 48px;
			}
			.eat_num {
				position: absolute;
				display: flex;
				justify-content: center;
				align-items: center;
				width: 20px;
				height: 20px;
				background: #ef5c2f;
				color: #ffffff;
				border-radius: 50%;
				top: 0;
				right: 0;
			}
		}
		.total_text {
			margin-left: 12px;
			.total {
				font-size: 14px;
				font-weight: 700;
				color: #333333;
			}
			.total_des {
				font-size: 10px;
				color: #999999;
			}
		}
	}
	.ordered_right {
		display: flex;
		justify-content: center;
		align-items: center;
		border-radius: 20px;
		width: 100px;
		height: 40px;
		background: #999999;
		color: #ffffff;
		margin-right: 10px;
	}
	.blue {
		background: #46b7fb;
	}
}
</style>
