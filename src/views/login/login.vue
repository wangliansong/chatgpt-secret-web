<template>
	<div class="login-container">
		<div class="form-container">
			<h2>Login</h2>
			<form @submit.prevent="login">
				<div class="form-group">
					<label for="username">Username</label>
					<input type="text" id="username" v-model="username" required @keypress="handlePress">
				</div>
				<div class="form-group">
					<label for="password">Password</label>
					<input type="password" id="password" v-model="password" required @keypress="handlePress">
				</div>
				<div class="form-group">
					<button type="submit" @click="handleVerify">Login</button>
				</div>
			</form>
			<div class="error-message">{{ errorMessage }}</div>
		</div>
	</div>
</template>

<script setup lang='ts'>
import {useAuthStore} from "@/store";
import {useMessage} from "naive-ui";
import {computed, ref} from "vue";
import {fetchLogin} from "@/api";
import {router} from '@/router'


interface Props {
	visible: boolean
}
interface LoginResponse {
	status:string
	code: string
	message: string
	token: string
}
defineProps<Props>()

const authStore = useAuthStore()

const ms = useMessage()

const loading = ref(false)
const username = ref('')
const password = ref('')


const disabled = computed(() => !username.value.trim() || !password.value.trim() || loading.value)
async function handleVerify() {
	// 取值
	const userName = username.value.trim()
	const pwd = password.value.trim()


	// 判断是否有输入
	if (!(userName || pwd))
		return

	try {
		// 开启加载状态
		loading.value = true
		// 发起请求
		const resp = await fetchLogin<LoginResponse>(userName, pwd);
		try {
			const data = resp.data
			if ( data.code === '200') {
				ms.success(data.message)

				// 登录成功，保存 token 到 localStorage
				localStorage.setItem('token', data.token)
				// 执行登录成功后的操作
				// ...
				await router.push({name: 'chat'});
			} else {
				// 登录失败，提示错误信息
				ms.success(data.message)
			}
		}
		catch(error: any) {
			// 错误弹框
			ms.error(error.message ?? 'error')
			// 清空token缓存
			authStore.removeToken()
			// 清空本文件缓存
			//token.value = ''
		}
	finally {
			// 关闭加载状态
			loading.value = false
		}

	}
	catch (error: any) {
		// 错误弹框
		ms.error(error.message ?? 'error')
		// 清空token缓存
		authStore.removeToken()
		// 清空本文件缓存
		//token.value = ''
	}
	finally {
		// 关闭加载状态
		loading.value = false
	}
}
/**
 * @description: 监听键盘Enter事件
 * @param {*} event
 */
function handlePress(event: KeyboardEvent) {
	if (event.key === 'Enter' && !event.shiftKey) {
		event.preventDefault()
		handleVerify()
	}
}
</script>

<style scoped>
.login-container {
	height: 100vh;
	display: flex;
	justify-content: center;
	align-items: center;
}

.form-container {
	width: 400px;
	padding: 20px;
	background-color: #fff;
	border-radius: 5px;
	box-shadow: 0 0 5px rgba(0, 0, 0, 0.3);
}

.form-group {
	margin: 10px 0;
}

label {
	display: block;
	margin-bottom: 5px;
}

input[type="text"], input[type="password"] {
	display: block;
	width: 100%;
	padding: 5px;
	border: 1px solid #ccc;
	border-radius: 3px;
	box-sizing: border-box;
}

button[type="submit"] {
	background-color: #007bff;
	color: #fff;
	padding: 5px 10px;
	border: none;
	border-radius: 3px;
	cursor: pointer;
	font-size: 16px;
}

.error-message {
	color: red;
	margin-top: 10px;
}
</style>
