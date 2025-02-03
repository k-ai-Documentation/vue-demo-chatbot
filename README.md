# Vue demo chatbot

## Introduction
This project is a demo for vue3, which contains 'chatbot' function with sdk-js. This demo will show you how to use sdk-js and chatbot in vue3.

## Before setup
clone [vue-demo-chabot](https://github.com/k-ai-Documentation/vue-demo-chatbot)

And your directory should like this:
```
|-your root directory
    |-vue-demo-chatbot
```
open terminal and run
```bash
cd vue-demo-chatbot
```
Add fill your keys in .env

If you are using SaaS version, you need 3 keys(organizationId, instanceId, apiKey).

If you are using Premise version, you need host and api key(optional).

See More about SaaS and Premise version in [here](https://github.com/k-ai-Documentation/sdk-js#usage-guide).

```bash
# if you are using saas 
VITE_ORGANIZATION_ID = ''
VITE_INSTANCE_ID = ''
VITE_API_KEY = ''

# if you are using version premise. You must need host, but api key is optional, depends on your enterprise settings. 
# VITE_HOST = ''
# VITE_API_KEY = ''
```


## Installation
```bash
npm install
```

### Compiles and hot-reloads for development
```
npm run dev
```
You can change paramater in .env file to change the search result.
```
# if you want to search result have multiple documents sources, set it to true.
VITE_MULTI_DOCUMENTS = true 
```


# How to use [sdk-js](https://github.com/k-ai-Documentation/sdk-js/tree/version2.0)

+ make sure you have installed sdk-js and vue-demo.
```bash
npm install
```
+ check in your package.json and node_modules, kaistudio-sdk-js should be installed.

+ check .env.development has VITE_ORGANIZATION_ID, VITE_INSTANCE_ID, VITE_API_KEY.

+ in your vue file, import kaistudio-sdk-js.
```js
import { KaiStudio } from "kaistudio-sdk-js";
```
if your are using typescript, you need to define searchResult.
```js
import type { chatbot } from 'kaistudio-sdk-js/modules/Search';
```
+ Create your sdk instance
````js
let sdk = new KaiStudio({ organizationId: process.env.VUE_APP_ORGANIZATION_ID , instanceId: process.env.VUE_APP_INSTANCE_ID, apiKey: process.env.VUE_APP_API_KEY });
````

+ use it in your methods
```js
async function conversation() {
        try {
            const result =  await kaiSearch.chatbot().conversation(conversationId.value, userMessageTemp.value, multiDocuments, "your_user_id")
            console.log(result);
        } catch (error) {
            console.log(error);
        }
    }
```
