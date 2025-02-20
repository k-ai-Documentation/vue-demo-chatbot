<template lang="pug">
    .search-chat
        .message-history
            .message(v-for="message in messageHistory" :key="message.message" :style="{'justify-content': message.from === 'user' ? 'flex-end' : 'flex-start'}")
                .assistance-message(v-if="message.from === 'assistance'")
                    button.btn-icon.text-bold-14.name-image BOT
                    .message-text.assistance-color 
                        p.text-regular-14(v-html="message.message")
                        p.text-regular-14(v-if="message.docList.length > 0") Source:
                        p.text-regular-14(v-for="doc in message.docList" :key="doc.name" v-if="message.docList.length > 0")
                            span(v-if="doc.url.includes('http')") 
                                a.link(:href="doc.url" target="_blank") {{ doc.name }}
                            span.link(@click="downloadFile(doc.name)" v-else) {{ doc.name }}
                .user-message(v-if="message.from === 'user'")
                    .message-text.user-color 
                        p.text-regular-14 {{message.message}}
                    button.btn-icon.text-bold-14.name-image USER
            .assistance-message(v-if="loading")
                button.btn-icon.text-bold-14.name-image BOT
                .message-text.assistance-color 
                    p.text-regular-14 Searching... {{ loadingPercentage }}%
        .input-container(v-if="step === 'conversation'")
            input.simple-input-h30(type="text" placeholder="Type your message here..." @keyup.enter="sendMessage" v-model="userMessage") 
            button.btn-outline-rounded-30(@click="sendMessage") Send
    
    </template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { KaiStudio } from 'kaistudio-sdk-js';
import Buffer from 'vue-buffer';

// if you are using Saas
const kaiSearch = new KaiStudio({ organizationId: import.meta.env.VITE_ORGANIZATION_ID, instanceId: import.meta.env.VITE_INSTANCE_ID, apiKey: import.meta.env.VITE_API_KEY });
const multiDocuments: boolean = import.meta.env.VITE_MULTI_DOCUMENTS === 'true';

const messageHistory = ref<{ from: string; message: string; docList: string[]; }[]>([]);
const userMessage = ref('');
const userMessageTemp = ref('');
const conversationId = ref('');
const step = ref('conversation');
const loadingPercentage = ref(0);
const loading = ref(false);
let loadingInterval: number | null = null;
const documentsList = ref<{ name: string; url: string }[]>([]);

const sendMessage = async () => {
    try {
        const userMsg = { from: 'user', message: userMessage.value, docList: [] };
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
        return await kaiSearch.chatbot().conversation(conversationId.value, userMessageTemp.value, multiDocuments, 'demo_vue'); //you can change "user_id" with your user email
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
            messageHistory.value.push({ from: 'assistance', message: "Answer:", docList: [] });
            let docList = [];
            if (result['datas']['documents'].length > 0) {
                docList = result['datas']['documents']
            }
            messageHistory.value.push({ from: 'assistance', message: result['content'], docList: docList });
        } else {
            messageHistory.value.push({ from: 'assistance', message: result['content'], docList: []});
            if ('datas' in result && 'reason' in result['datas'] && result['datas']['reason'] != '') {
                messageHistory.value.push({ from: 'assistance', message: result['datas']['reason'], docList: [] });
            }
            step.value = 'conversation'
        }
    } catch(error) {
        console.log(error);
    }
}

function init() {
    userMessageTemp.value = '';
    userMessage.value = '';
    step.value = 'conversation';
}

async function downloadFile(fileName: string) {
    let file = await kaiSearch.fileInstance().downloadFile(fileName)
    if (file) {
        const buffer = Buffer.from(file);
        const blob = new Blob([buffer]);
        const url = window.URL.createObjectURL(blob);
        let a = document.createElement('a');
        document.body.appendChild(a);
        a.setAttribute('style', 'display: none');
        a.href = url;
        a.download = fileName;
        a.click();
        window.URL.revokeObjectURL(url);
    }
}

onMounted(() => {
    messageHistory.value.push({ from: 'assistance', message: 'Hello, how can I help you today?', docList: []  });
});
</script>

    
<style lang="scss" scoped>
    .search-chat {
        width: 900px;
        height: 500px;
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
            background-color: var(--grey-5-color);
            padding: 20px;
            overflow: scroll;
            .name-image {
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
                    background-color: var(--color-border);
                }

                .link {
                    color: var(--primary-color);
                    cursor: pointer;
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
                line-height: 1.5;
                .button-download {
                    margin-left: 20px;
                }
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
            margin-top: 20px;
    
            input {
                background-color: white;
                color: black;
                max-width: none;
                flex-grow: 1;
                margin-right: 20px;
            }
        }
    }
</style>
    