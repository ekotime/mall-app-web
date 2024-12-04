<template>
	<view class="container">
		<view class="left-bottom-sign"></view>
		<view class="back-btn yticon icon-zuojiantou-up" @click="navBack"></view>
		<view class="right-top-sign"></view>
		<!-- 设置白色背景防止软键盘把下部绝对定位元素顶上来盖住输入框等 -->
		<view class="wrapper">
			<view class="empty">
				<view class="empty-tips">
					<t-form :data="formData" :rules="rules" ref="form" @reset="onReset" @submit="onSubmit">
						<t-form-item label="邮箱" name="email">
							<t-input v-model="formData.email" placeholder="请输入邮箱"></t-input>
							<t-button theme="default" variant="base" @click="getAuthCode">获取验证码</t-button>
						</t-form-item>
						<t-form-item label="验证码" name="authCode">
							<t-input v-model="formData.authCode" placeholder="请输入验证码"></t-input>
						</t-form-item>
						<t-form-item label="用户名" help="这里可以展示一段说明文字" name="username">
							<t-input v-model="formData.username" placeholder="请输入用户名"></t-input>
						</t-form-item>
						<t-form-item label="密码" name="password">
							<t-input type="password" v-model="formData.password" placeholder="请输入密码"></t-input>
						</t-form-item>
						<t-form-item label="确认密码" name="rePassword" help="自定义异步校验方法">
							<t-input type="password" v-model="formData.rePassword" placeholder="请再次输入密码"></t-input>
						</t-form-item>
						<t-form-item style="margin-left: 100px">
							<t-space size="10px">
								<t-button theme="primary" type="submit">提交</t-button>
								<t-button theme="default" variant="base" type="reset">重置</t-button>
								<t-button theme="default" variant="base" @click="handleClear">清空校验结果</t-button>
							</t-space>
						</t-form-item>
					</t-form>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import {
		memberGetAuthCode,
		memberRegister,
	} from '@/api/member.js';
	const INITIAL_DATA = {
		username: '',
		password: '',
		rePassword: '',
		email: '',
		authCode: '',
	};
	export default {
		data() {
			return {
				formData: {
					...INITIAL_DATA
				},
				// FormItem.rules 优先级大于 Form.rules
				rules: {
					username: [{
							required: true,
							message: '姓名必填',
							type: 'error',
							trigger: 'blur',
						},
						// trigger 默认为 'change'
						{
							required: true,
							message: '姓名必填',
							type: 'error'
						},
						{
							whitespace: true,
							message: '姓名不能为空'
						},
						{
							min: 2,
							message: '至少需要两个字符，一个中文等于两个字符',
							type: 'warning',
							trigger: 'blur',
						},
						{
							max: 10,
							message: '姓名字符长度超出',
							type: 'warning',
							trigger: 'blur',
						},
					],
					password: [{
							required: true,
							message: '密码必填',
							type: 'error'
						},
						// 自定义校验规则：不同的值可以有不同的校验结果，不同的校验类型
						{
							validator: this.passwordValidator
						},
					],
					rePassword: [{
							required: true,
							message: '密码必填',
							type: 'error'
						},
						// 自定义校验规则：自定义异步校验规则
						{
							validator: this.rePassword,
							message: '两次密码不一致'
						},
					],
					email: [{
							required: true,
							message: '邮箱必填'
						},
						{
							email: {
								ignore_max_length: true
							},
							message: '请输入正确的邮箱地址'
						},
					],
					authCode: [{
							required: true,
							message: '验证码必填'
						},
					],
				}
			}
		},
		onLoad() {},
		methods: {
			navBack() {
				uni.navigateBack();
			},
			onReset() {
				this.$message.success('重置成功');
			},
			async onSubmit({
				validateResult,
				firstError
			}) {
				if (validateResult === true) {
					this.$message.success('提交成功');
					memberRegister({
						username: this.formData.username,
						password: this.formData.password,
						telephone: this.formData.email,
						authCode: this.formData.authCode
					}).then(response => {
						// 注册成功后，跳转到登录页面并携带用户名和密码
						uni.navigateTo({
							url: '/pages/public/login?username=' + this.formData.username
						});
					}).catch(() => {
						this.logining = false;
					});
				} else {
					console.log('Errors: ', validateResult);
					this.$message.warning(firstError);
				}
			},
			handleClear() {
				this.$refs.form.clearValidate();
			},
			getAuthCode() {
				// 验证邮箱格式
				const emailRegex = /^[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,6}$/;
				if (!emailRegex.test(this.formData.email)) {
					this.$message.warning('请输入正确的邮箱地址');
					return;
				} else {
					//发送验证码
					memberGetAuthCode({
						email: this.formData.email,
					}).then(response => {
						this.$message.success('验证码发送成功');
					}).catch(() => {
						this.logining = false;
					});
				}
			},
			// 自定义异步校验器，使用 resolve 返回结果控制校验结果、校验信息、校验结果类型
			userNameValidator(val) {
				return new Promise((resolve) => {
					const timer = setTimeout(() => {
						if (['Zhang', 'Li', 'Wang'].includes(val)) {
							resolve({
								result: true
							});
						} else {
							resolve({
								result: false,
								message: '用户名不存在',
								type: 'error'
							});
						}
						clearTimeout(timer);
					}, 10);
				});
			},
			// 自定义校验器，不同的值输出不同的校验结果。支持异步校验（文案选自某密码重置站点，如有侵权，请联系我们删除）
			passwordValidator(val) {
				if (val.length > 0 && val.length <= 2) {
					return {
						result: false,
						message: '太简单了！再开动一下你的小脑筋吧！',
						type: 'error'
					};
				}
				if (val.length > 2 && val.length < 4) {
					return {
						result: false,
						message: '还差一点点，就是一个完美的密码了！',
						type: 'warning'
					};
				}
				return {
					result: true,
					message: '太强了，你确定自己记得住吗！',
					type: 'success'
				};
			},
			// 自定义异步校验器
			rePassword(val) {
				return new Promise((resolve) => {
					const timer = setTimeout(() => {
						resolve(this.formData.password === val);
						clearTimeout(timer);
					});
				});
			},
		},

	}
</script>

<style lang='scss'>
	page {
		background: #fff;
	}

	.empty {
		position: fixed;
		left: 0;
		top: 0;
		width: 100%;
		height: 100vh;
		padding-bottom: 100upx;
		display: flex;
		justify-content: center;
		flex-direction: column;
		align-items: center;
		background: #fff;

		image {
			width: 420upx;
			height: 420upx;
			margin-bottom: 30upx;
		}

		.empty-tips {
			display: flex;
			font-size: $font-sm+16upx;
			color: $font-color-disabled;

			.navigator {
				color: $uni-color-primary;
				margin-left: 0upx;
			}
		}
	}

	.container {
		padding-top: 115px;
		position: relative;
		width: 100vw;
		height: 100vh;
		overflow: hidden;
		background: #fff;
	}

	.wrapper {
		position: relative;
		z-index: 90;
		background: #fff;
		padding-bottom: 40upx;
	}

	.back-btn {
		position: absolute;
		left: 40upx;
		z-index: 9999;
		padding-top: var(--status-bar-height);
		top: 40upx;
		font-size: 40upx;
		color: $font-color-dark;
	}

	.left-top-sign {
		font-size: 120upx;
		color: $page-color-base;
		position: relative;
		left: -16upx;
	}

	.right-top-sign {
		position: absolute;
		top: 80upx;
		right: -30upx;
		z-index: 95;

		&:before,
		&:after {
			display: block;
			content: "";
			width: 400upx;
			height: 80upx;
			background: #b4f3e2;
		}

		&:before {
			transform: rotate(50deg);
			border-radius: 0 50px 0 0;
		}

		&:after {
			position: absolute;
			right: -198upx;
			top: 0;
			transform: rotate(-50deg);
			border-radius: 50px 0 0 0;
			/* background: pink; */
		}
	}

	.left-bottom-sign {
		position: absolute;
		left: -270upx;
		bottom: -320upx;
		border: 100upx solid #d0d1fd;
		border-radius: 50%;
		padding: 180upx;
	}
</style>