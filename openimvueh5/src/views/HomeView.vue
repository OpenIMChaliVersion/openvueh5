<script setup lang="ts">

import { ref, onMounted, onUnmounted } from 'vue';
import { getSDK, CbEvents, type CallbackEvent, type MessageItem, Platform } from '@openim/client-sdk';
import axios from 'axios';
import { v4 as uuidv4 } from 'uuid'
import md5 from 'md5'
// 基础响应接口：所有 API 响应都应包含的状态字段
export interface ApiResponse<T = any> {
    errCode: number;
    errMsg: string;
    errDlt: string;
    data: T; // 泛型 T 用于指定实际业务数据的类型
}
// 1. 初始化 SDK 实例
const IMSDK = getSDK();

// 2. 聊天室状态
const isConnected = ref(false);
const connectStatus = ref('Connecting...');
const messageList = ref<MessageItem[]>([]); // 存储聊天消息
const inputMessage = ref(''); // 存储用户输入的消息
const targetUserID = 'recipient user ID'; // 👈 替换为你要聊天的对方的 UserID
const loading = ref(false);
const loadingStr = ref('初始化中...');
const baseUrl = 'kefu.acone.icu' // 👈 替换为你的服务器地址
const CHAT_URL = `https://${baseUrl}/chat`
const API_URL = `https://${baseUrl}/api`
const WS_URL = `wss://${baseUrl}/msg_gateway`
// 3. 连接状态处理函数
function handleConnecting() {
    connectStatus.value = 'Connecting... 🟡';
    console.log('Connecting...');
}

function handleConnectFailed({ errCode, errMsg }: CallbackEvent<any>) {
    connectStatus.value = `Connection failed ❌: ${errCode} - ${errMsg}`;
    isConnected.value = false;
    console.error('Connection failed:', errCode, errMsg);
}

function handleConnectSuccess() {
    connectStatus.value = 'Connection successful ✅';
    isConnected.value = true;
    console.log('Connection successful!');
    // 连接成功后可以获取历史消息
    // fetchHistoryMessages(); // 暂不实现历史消息，保持简单
}

// 4. 接收新消息的处理函数
function handleNewMessages({ data }: CallbackEvent<MessageItem[]>) {
    console.log('New messages received:', data);
    // 将新消息添加到消息列表的末尾
    messageList.value.push(...data);
    // 滚动到底部（在 onMounted/watch 中处理更好，但这里为简易起见，先省略）
}

// 5. 发送消息的函数
async function sendMessage() {
    if (!isConnected.value || !inputMessage.value.trim()) {
        console.warn('Not connected or message is empty.');
        return;
    }

    try {
        // 创建文本消息
        const message = (await IMSDK.createTextMessage('hello openim')).data;
        // 发送消息给目标用户
        await IMSDK.sendMessage({
            recvID: "", // 对方的 UserID
            groupID: '769535529', // 单聊群组ID为空
            message,
        });

        console.log('Message sent successfully:', message);

        // 清空输入框并手动将发送的消息添加到列表
        messageList.value.push(message);
        inputMessage.value = '';

    } catch (err) {
        console.error('Failed to send message:', err);
    }
}

async function sendSms(params: {}): Promise<ApiResponse> {

    const response = await axios.post<ApiResponse>(`${CHAT_URL}/account/code/send`, JSON.stringify({
        ...params,
        operationID: Date.now() + '',
    }), {
        headers: {
            'Content-Type': 'application/json', // 这是一个常见的header，通常用于POST请求
            'Operationid': uuidv4(),//;header
            'X-Custom-Header': 'some-value' // 示例：自定义header
        }
    })
    return response.data;
}
async function verifyCode(params: {}): Promise<ApiResponse> {

    const response = await axios.post<ApiResponse>(`${CHAT_URL}/account/code/verify`, JSON.stringify({
        ...params,
        operationID: Date.now() + '',
    }), {
        headers: {
            'Content-Type': 'application/json', // 这是一个常见的header，通常用于POST请求
            // 'Authorization': 'Bearer YOUR_TOKEN', // 示例：如果你需要认证
            'Operationid': uuidv4(),//;header
            'X-Custom-Header': 'some-value' // 示例：自定义header
        }
    })
    return response.data;
}
async function register(params: {}): Promise<ApiResponse> {

    const response = await axios.post<ApiResponse>(`${CHAT_URL}/account/register`, JSON.stringify({
        ...params,
        operationID: Date.now() + '',
    }), {
        headers: {
            'Content-Type': 'application/json', // 这是一个常见的header，通常用于POST请求
            // 'Authorization': 'Bearer YOUR_TOKEN', // 示例：如果你需要认证
            'Operationid': uuidv4(),//;header
            'X-Custom-Header': 'some-value' // 示例：自定义header
        }
    })
    return response.data;
}
async function login(params: {}): Promise<ApiResponse> {

    const response = await axios.post<ApiResponse>(`${CHAT_URL}/account/login`, JSON.stringify({
        ...params,
        operationID: Date.now() + '',
    }), {
        headers: {
            'Content-Type': 'application/json', // 这是一个常见的header，通常用于POST请求
            'Operationid': uuidv4(),//;header
            'X-Custom-Header': 'some-value' // 示例：自定义header
        }
    })
    return response.data;
}
const onchaliAuto = async () => {

    let vemail = "";
    vemail = localStorage.getItem('IMAccount') as string || '';
    const timestampInMillis: number = +new Date();
    const date = new Date();
    const day: number = date.getDate();
    const hours: number = date.getHours();
    const minutes: number = date.getMinutes();
    const seconds: number = date.getSeconds();
    const invitationCode = '';
    const vpass = 'chali22222'
    loading.value = true
    try {

        if (vemail === '') {
            loadingStr.value = '正在初始化 0%'
            const vname = `访客${day}${hours}${minutes}${seconds}`
            vemail = `wuchali${timestampInMillis}@163.com`

            loadingStr.value = '正在初始化 10%'
            const res_send = await sendSms({
                phoneNumber: '',
                areaCode: '86',
                email: vemail,
                invitationCode: invitationCode,
                usedFor: 1,
            })
            loadingStr.value = '正在初始化 11%'
            loadingStr.value = '正在初始化 12%'
            loadingStr.value = '正在初始化 30%'
            const res_verify = await verifyCode({
                phoneNumber: '',
                areaCode: '86',
                email: vemail,
                verifyCode: '666666',
                platform: 5,
                usedFor: 1,
            })
            loadingStr.value = '正在初始化 50%'

            const res = await register({
                verifyCode: '666666',
                deviceID: '',
                invitationCode: invitationCode,
                autoLogin: true,
                user: {
                    nickname: vname,
                    phoneNumber: '',
                    areaCode: '',
                    faceURL: 'https://kefu.acone.icu/api/object/9653164283/1761021086245.png',
                    email: vemail,
                    birth: 0,
                    gender: 0,
                    password: md5(vpass),
                    confirmPassword: vpass,
                },
                platform: 5
            })
            if (res.errCode === 0) {
                loadingStr.value = '正在初始化 70%'
                localStorage.setItem('IMAccount', vemail)
                return res
            }
        }
        else {
        
            const res = await login({
                    phoneNumber: '',
                    password: md5(vpass),
                    areaCode: "86",
                    verifyCode: "",
                    email: vemail,
                    platform: 5,
                    deviceID: "",
                    account:""
            })
           if (res.errCode === 0) {
                loadingStr.value = '正在初始化 90%'
                return res
           }
        }
    } catch (error) {
        //     feedbackToast({ message: t('messageTip.loginFailed'), error })
    }
}

// 6. 生命周期钩子
onMounted(async () => {

    const res = await onchaliAuto();
    console.log('Auto login result:', res);

    // 注册事件监听器
    IMSDK.on(CbEvents.OnConnecting, handleConnecting);
    IMSDK.on(CbEvents.OnConnectFailed, handleConnectFailed);
    IMSDK.on(CbEvents.OnConnectSuccess, handleConnectSuccess);
    IMSDK.on(CbEvents.OnRecvNewMessages, handleNewMessages);

    // 执行登录
    IMSDK.login({
        userID: res?.data.userID, // 👈 替换为你的 UserID
        token: res?.data.imToken, // 👈 替换为你的 Token
        platformID: 5,
        wsAddr: WS_URL, // 👈 替换为你的服务器地址
        apiAddr: API_URL, // 👈 替换为你的服务器地址
    });
    IMSDK.getLoginStatus().then(({ data }) => {
        console.log('Login status:', data);
    });

    IMSDK.on(CbEvents.OnRecvNewMessages, handleNewMessages);

});

onUnmounted(() => {
    // 组件卸载时，取消监听器（可选，但推荐）
    IMSDK.off(CbEvents.OnConnecting, handleConnecting);
    IMSDK.off(CbEvents.OnConnectFailed, handleConnectFailed);
    IMSDK.off(CbEvents.OnConnectSuccess, handleConnectSuccess);
    IMSDK.off(CbEvents.OnRecvNewMessages, handleNewMessages);
    // 登出或断开连接
    // IMSDK.logout(); // 如果需要彻底退出
});
</script>

<template>
    <main>

        <h2>OpenIM 简易聊天室 💬</h2>

        <p>Status: <span :style="{ color: isConnected ? 'green' : 'red' }">{{ connectStatus }}</span></p>
        <p>Chatting with: **{{ targetUserID }}**</p>

        <hr>

        <div class="message-container"
            style="height: 400px; overflow-y: auto; border: 1px solid #eee; padding: 10px; margin-bottom: 10px; background-color: #f9f9f9;">
            <div v-for="msg in messageList" :key="msg.clientMsgID"
                :class="{ 'my-message': msg.sendID === 'IM user ID', 'other-message': msg.sendID !== 'IM user ID' }"
                style="margin-bottom: 8px; padding: 5px; border-radius: 5px;">
                <strong>{{ msg.sendID }}:</strong> {{ msg.textElem?.content }}
            </div>
        </div>

        <div class="input-area" style="display: flex; gap: 10px;">
            <input v-model="inputMessage" @keyup.enter="sendMessage" :disabled="!isConnected"
                placeholder="Type a message..."
                style="flex-grow: 1; padding: 8px; border: 1px solid #ccc; border-radius: 4px;" />
            <button @click="sendMessage" :disabled="!isConnected || !inputMessage.trim()"
                style="padding: 8px 15px; border: none; border-radius: 4px; background-color: #007bff; color: white; cursor: pointer;">
                Send
            </button>
            <button @click="sendMessage" :disabled="!isConnected || !inputMessage.trim()"
                style="padding: 8px 15px; border: none; border-radius: 4px; background-color: #007bff; color: white; cursor: pointer;">
                Send
            </button>
        </div>
    </main>
</template>
<style scoped>
/* 简单的样式区分自己的消息和对方的消息 */
.my-message {
    text-align: right;
    background-color: #dcf8c6;
    /* 绿色系 */
    margin-left: 20%;
}

.other-message {
    text-align: left;
    background-color: #ffffff;
    /* 白色系 */
    margin-right: 20%;
}
</style>