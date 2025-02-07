<template lang="pug">
    .search-chat
        .message-history
            .message(v-for="message in messageHistory" :key="message.message" :style="{'justify-content': message.from === 'user' ? 'flex-end' : 'flex-start'}")
                .assistance-message(v-if="message.from === 'assistance'")
                    button.btn-icon.text-bold-14 BOT
                    .message-text.assistance-color 
                        p.text-regular-14(v-html="message.message")
                .user-message(v-if="message.from === 'user'")
                    .message-text.user-color 
                        p.text-regular-14 {{message.message}}
                    button.btn-icon.text-bold-14 USER
            .assistance-message(v-if="loading")
                button.btn-icon.text-bold-14 BOT
                .message-text.assistance-color 
                    p.text-regular-14 Searching... {{ loadingPercentage }}%
        .input-container(v-if="step === 'conversation'")
            input.simple-input-h30(type="text" placeholder="Type your message here..." @keyup.enter="sendMessage" v-model="userMessage") 
            button.btn-outline-rounded-30(@click="sendMessage") Send
    
    </template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { KaiStudio } from 'kaistudio-sdk-js';

// if you are using Saas
const kaiSearch = new KaiStudio({ organizationId: import.meta.env.VITE_ORGANIZATION_ID, instanceId: import.meta.env.VITE_INSTANCE_ID, apiKey: import.meta.env.VITE_API_KEY });
const multiDocuments: boolean = import.meta.env.VITE_MULTI_DOCUMENTS === 'true';

const messageHistory = ref<{ from: string; message: string }[]>([]);
const userMessage = ref('');
const userMessageTemp = ref('');
const conversationId = ref('');
const step = ref('conversation');
const loadingPercentage = ref(0);
const loading = ref(false);
let loadingInterval: number | null = null;

const sendMessage = async () => {
    try {
        const userMsg = { from: 'user', message: userMessage.value };
        messageHistory.value.push(userMsg);
        userMessageTemp.value = userMessage.value;
        userMessage.value = '';

        startLoading();

        const result = await conversation();

        stopLoading();

        if (result) {
            conversationId.value = result['id'];
        }

        if (result && result['message']) {
            processSearchResult(result['message']);
            init();
        }
    } catch (error) {
        console.log(error);
    }
};

async function conversation() {
    try {
        step.value = 'searching';
        return await kaiSearch.chatbot().conversation(conversationId.value, userMessageTemp.value, multiDocuments, 'user_id'); //you can change "user_id" with your user email
    } catch (error) {
        console.log(error);
    }
}

function startLoading() {
    loading.value = true;
    loadingPercentage.value = 0;

    loadingInterval = setInterval(() => {
        if (loadingPercentage.value < 90) {
            loadingPercentage.value += Math.floor(Math.random() * 3) + 3;
        }
    }, 1000);
}

function stopLoading() {
    if (loadingInterval) {
        clearInterval(loadingInterval);
        loadingInterval = null;
    }
    loadingPercentage.value = 100;
    setTimeout(() => {
        loading.value = false;
        loadingPercentage.value = 0;
    }, 500);
}

async function processSearchResult(result: any) {
    try {
        if (result['action'] == 'SEARCH' && result['datas']['isAnswered']) {
            messageHistory.value.push({ from: 'assistance', message: 'Answer:' });
            messageHistory.value.push({ from: 'assistance', message: result['content'] });

            if (result['datas']['documents'].length > 0) {
                let documents = '';
                for (let doc of result['datas']['documents']) {
                    documents += `name: ${doc.name}<br>url: <a href="${doc.url}" style="color: var(--primary-color)" target="_blank">${doc.url}</a><br><br>`;
                }
                messageHistory.value.push({ from: 'assistance', message: 'Source:' });
                messageHistory.value.push({ from: 'assistance', message: documents });
            }
        } else {
            messageHistory.value.push({ from: 'assistance', message: result['content'] });
            if ('reason' in result['datas'] && result['datas']['reason'] != '') {
                messageHistory.value.push({ from: 'assistance', message: result['datas']['reason'] });
            }
            step.value = 'conversation';
        }
    } catch (error) {
        console.log(error);
    }
}

function init() {
    userMessageTemp.value = '';
    userMessage.value = '';
    step.value = 'conversation';
}

onMounted(() => {
    messageHistory.value.push({ from: 'assistance', message: 'Hello, how can I help you today?' });
});
</script>

<style lang="scss" scoped>
.search-chat {
    width: 799px;
    height: 100%;
    background-color: var(--dark-grey-color);
    border: 1px solid var(--color-border);
    border-radius: 12px;
    padding: 11px 11px 30px 11px;
    display: flex;
    flex-direction: column;
    overflow: scroll;

    .message-history {
        width: 100%;
        height: 100%;
        border-radius: 10px;
        background-color: var(--grey-4-color);
        padding: 20px;
        overflow: scroll;
        button {
            height: 50px;
            width: 50px;
            justify-content: center;
        }

        .assistance-message {
            display: flex;
            flex-direction: row;
            width: fit-content;
            max-width: 80%;
            align-items: center;
            .assistance-color {
                background-color: var(--grey-5-color);
            }
        }

        .message-text {
            width: 100%;
            margin: 10px 20px;
            color: white;
            padding: 10px 10px 10px 10px;
            border-radius: 10px;
            word-break: break-word;
            white-space: pre-wrap;
        }

        .message {
            display: flex;
            flex-direction: row;
            width: 100%;

            .user-message {
                display: flex;
                flex-direction: row;
                width: fit-content;
                max-width: 80%;
                align-items: center;
                .user-color {
                    background-color: var(--color-border);
                }
            }
        }
    }
    .input-container {
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: space-between;
        margin-top: 20px;

        input {
            background-color: white;
            color: black;
        }
    }
}
</style>
