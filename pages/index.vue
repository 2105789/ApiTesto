<template>
    <div
      class="min-h-screen bg-gradient-to-br from-gray-900 to-gray-800 text-white"
    >
    <div class="container mx-auto p-8">
    <div class="flex justify-between items-center mb-4">
        <h1 class="text-4xl md:text-5xl font-extrabold">
        AI-Powered API Tester
        </h1>
        <button class="btn btn-primary" @click="openApiKeyModal">
        <Icon name="heroicons:key" class="w-6 h-6" /> Set API Key
        </button>
    </div>

    <div class="text-center mb-12">
        <p class="text-lg md:text-xl text-gray-300">
        Revolutionize your API testing with AI-generated test cases
        </p>
    </div>

    <div class="grid grid-cols-1 md:grid-cols-2 gap-8 mb-12">
        <div class="card bg-gray-800 shadow-xl">
        <div class="card-body">
            <h2 class="card-title flex items-center">
            <Icon name="material-symbols:add-link-rounded" class="w-6 h-6" />
            API Endpoint
            </h2>
            <div class="flex flex-col gap-2">
            <select
                v-model="httpMethod"
                class="select select-bordered bg-gray-700"
            >
                <option value="GET">GET</option>
                <option value="POST">POST</option>
            </select>
            <input
                v-model="apiEndpoint"
                type="text"
                placeholder="Enter API endpoint"
                class="input input-bordered bg-gray-700"
            />
            </div>
        </div>
        </div>

        <div class="card bg-gray-800 shadow-xl">
        <div class="card-body">
            <h2 class="card-title flex items-center">
            <Icon
                name="material-symbols-light:package-outline-rounded"
                class="w-6 h-6"
            />
            Payload Example
            </h2>
            <textarea
            v-model="payloadExample"
            class="textarea textarea-bordered h-24 w-full bg-gray-700"
            placeholder="Enter payload example (JSON)"
            :disabled="httpMethod === 'GET'"
            ></textarea>
        </div>
        </div>
    </div>

    <div class="card bg-gray-800 shadow-xl mb-8">
        <div class="card-body">
        <h2 class="card-title flex items-center">
            <Icon name="material-symbols:security" class="w-6 h-6" />
            Authentication
        </h2>
        <div class="flex flex-col gap-4">
            <select
            v-model="authMethod"
            class="select select-bordered w-full bg-gray-700"
            >
            <option value="noAuth">No Auth</option>
            <option value="apiKey">API Key</option>
            <option value="bearerToken">Bearer Token</option>
            <option value="jwtToken">JWT Token</option>
            </select>
            <div v-if="authMethod !== 'noAuth'" class="flex flex-col gap-2">
            <input
                v-if="authMethod === 'apiKey'"
                v-model="authKey"
                type="text"
                placeholder="API Key Name"
                class="input input-bordered bg-gray-700"
            />
            <input
                v-model="authValue"
                type="text"
                :placeholder="getAuthValuePlaceholder()"
                class="input input-bordered bg-gray-700"
            />
            </div>
        </div>
        </div>
    </div>

    <div class="text-center mb-12 space-y-4">
        <button
        @click="testAPI"
        class="btn btn-primary btn-lg mr-2"
        :disabled="isTestingAPI"
        >
        <template v-if="isTestingAPI">
            <span class="loading loading-spinner"></span>
            Testing API...
        </template>
        <template v-else>
            <Icon name="charm:circle-tick" class="w-6 h-6" />
            Test API Connection
        </template>
        </button>
        <button
        @click="generateAndRunTests"
        class="btn btn-primary btn-lg"
        :disabled="isLoading"
        >
        <template v-if="isLoading">
            <span class="loading loading-spinner"></span>
            Processing...
        </template>
        <template v-else>
            <Icon name="mingcute:send-plane-fill" class="w-6 h-6" />
            Generate and Run Tests
        </template>
        </button>
    </div>

    <div
        v-if="apiTestResult"
        :class="[
        'alert',
        apiTestResult.success ? 'alert-success' : 'alert-error',
        'shadow-lg',
        'mb-8',
        ]"
        @click="openModal(apiTestResult)"
    >
        <Icon
        :name="
            apiTestResult.success
            ? 'material-symbols:check-circle-outline'
            : 'material-symbols:error-outline'
        "
        class="w-6 h-6"
        />
        <span>{{ apiTestResult.message }}</span>
        <button
        class="btn btn-sm btn-ghost ml-auto"
        @click.stop="apiTestResult = null"
        >
        <Icon name="material-symbols:close" class="w-4 h-4" />
        </button>
    </div>

    <div v-if="errorMessage" class="alert alert-error shadow-lg mb-8">
        <Icon name="material-symbols:release-alert" class="w-6 h-6" />
        <span>{{ errorMessage }}</span>
    </div>

    <div class="card bg-gray-800 shadow-xl overflow-hidden mb-8">
        <div class="card-body">
        <h2 class="card-title text-2xl mb-4 flex items-center">
            <Icon name="carbon:result-new" class="w-6 h-6" />
            Test Results
        </h2>
        <div class="overflow-x-auto max-h-64">
            <table class="table w-full">
            <thead>
                <tr class="bg-gray-700 sticky top-0">
                <th>Test Case</th>
                <th>Description</th>
                <th>Status Code</th>
                <th>Status</th>
                <th>Actions</th>
                </tr>
            </thead>
            <tbody>
                <tr
                v-for="(testCase, index) in testCases"
                :key="index"
                class="hover:bg-gray-700"
                >
                <td>Test Case {{ index + 1 }}</td>
                <td>{{ testCase.description }}</td>
                <td>{{ testCase.statusCode }}</td>
                <td>
                    <span
                    class="badge"
                    :class="{
                        'badge-success':
                        testCase.statusCode >= 200 &&
                        testCase.statusCode < 300,
                        'badge-error':
                        testCase.statusCode < 200 ||
                        testCase.statusCode >= 300,
                    }"
                    >
                    {{
                        testCase.statusCode >= 200 && testCase.statusCode < 300
                        ? "Passed"
                        : "Failed"
                    }}
                    </span>
                </td>
                <td>
                    <button
                    @click="openModal(testCase)"
                    class="btn btn-sm btn-outline btn-accent"
                    >
                    View Details
                    </button>
                </td>
                </tr>
            </tbody>
            </table>
        </div>
        </div>
    </div>

    <!-- Modal -->
    <div class="modal" :class="{ 'modal-open': showModal }">
        <div class="modal-box w-11/12 max-w-5xl bg-gray-800 rounded-lg">
        <h3 class="font-bold text-lg mb-4">Test Case Details</h3>
        <div v-if="modalData" class="grid grid-cols-1 gap-4">
            <div
            v-if="
                modalData.success !== undefined || modalData.error !== undefined
            "
            >
            <h4 class="font-semibold mb-2">API Test Result:</h4>
            <p>{{ modalData.success ? "Success" : "Error" }}</p>
            <p>{{ modalData.message }}</p>
            </div>
            <div v-if="modalData.description">
            <h4 class="font-semibold mb-2">Description:</h4>
            <p>{{ modalData.description }}</p>
            </div>
            <div v-if="modalData.payload">
            <h4 class="font-semibold mb-2">Payload:</h4>
            <pre class="bg-gray-700 p-4 rounded-lg overflow-x-auto">
                <code>{{ JSON.stringify(modalData.payload, null, 2) }}</code>
                </pre>
            </div>
            <div v-if="modalData.statusCode">
            <h4 class="font-semibold mb-2">Status Code:</h4>
            <p>{{ modalData.statusCode }}</p>
            </div>
            <div v-if="modalData.response">
            <h4 class="font-semibold mb-2">Response:</h4>
            <pre class="bg-gray-700 h-96 p-4 rounded-lg overflow-x-auto">
                <code>{{ JSON.stringify(modalData.response, null, 2) }}</code>
                </pre>
            </div>
        </div>
        <div class="modal-action">
            <button class="btn btn-primary" @click="closeModal">
            <Icon
                name="material-symbols:cancel-outline-rounded"
                class="w-6 h-6"
            />
            Close
            </button>
        </div>
        </div>
    </div>

    <!-- API Key Modal -->
    <div class="modal" :class="{ 'modal-open': showApiKeyModal }">
        <div class="modal-box w-11/12 max-w-md bg-gray-800 rounded-lg">
        <h3 class="font-bold text-lg mb-4">Set Gemini API Key</h3>
        <input
            v-model="geminiApiKey"
            type="text"
            placeholder="Enter your Gemini API Key"
            class="input input-bordered w-full bg-gray-700 mb-4"
        />
        <div class="modal-action">
            <button class="btn btn-primary" @click="saveApiKey">
            Save Key
            </button>
            <button class="btn btn-ghost" @click="closeApiKeyModal">
            Cancel
            </button>
        </div>
        </div>
    </div>
    </div>
</div>
</template>
  
<script setup>
    import { ref } from "vue";
    import { GoogleGenerativeAI } from "@google/generative-ai";
    import axios from "axios";
    
    const apiEndpoint = ref("");
    const httpMethod = ref("GET");
    const payloadExample = ref("");
    const authMethod = ref("noAuth");
    const authKey = ref("");
    const authValue = ref("");
    const testCases = ref([]);
    const modalData = ref(null);
    const showModal = ref(false);
    const isLoading = ref(false);
    const isTestingAPI = ref(false);
    const errorMessage = ref("");
    const apiTestResult = ref(null);
    const geminiApiKey = ref("");
    const showApiKeyModal = ref(false);
    
    // Function to get the API Key from local storage
    const getApiKey = () => {
        if (typeof window !== "undefined" && window.localStorage) {
        return window.localStorage.getItem("geminiApiKey");
        } else {
        return ""; // Return an empty string if not in the browser
        }
    };
    
    // Function to save the API Key to local storage
    const saveApiKey = () => {
        if (typeof window !== "undefined" && window.localStorage) {
        window.localStorage.setItem("geminiApiKey", geminiApiKey.value);
        }
        closeApiKeyModal();
    };
    
    // Initialize the AI client using the API Key
    let genAI;
    const initAIClient = () => {
        const apiKey = getApiKey();
        geminiApiKey.value = getApiKey();
        if (apiKey) {
        genAI = new GoogleGenerativeAI(apiKey);
        }
    };
    
    // Call initAIClient after getting the API key from local storage
    const apiKey = getApiKey(); // Get the key first
    initAIClient(); // Initialize genAI
    
    const openApiKeyModal = () => {
        showApiKeyModal.value = true;
    };
    
    const closeApiKeyModal = () => {
        showApiKeyModal.value = false;
    };
    
    const getAuthValuePlaceholder = () => {
        switch (authMethod.value) {
        case "apiKey":
            return "API Key Value";
        case "bearerToken":
            return "Bearer Token";
        case "jwtToken":
            return "JWT Token";
        default:
            return "";
        }
    };
    
    const testAPI = async () => {
        isTestingAPI.value = true;
        apiTestResult.value = null;
        errorMessage.value = "";
    
        try {
        const config = getAxiosConfig();
        const response = await axios(config);
        apiTestResult.value = {
            success: true,
            message: `API test successful. Status code: ${response.status}`,
            response: response.data,
        };
        } catch (error) {
        console.error("API test error:", error);
        if (error.response) {
            // The request was made and the server responded with a status code
            // that falls out of the range of 2xx
            apiTestResult.value = {
            success: false,
            message: `API test failed: Server responded with status ${error.response.status}`,
            response: error.response.data,
            };
        } else if (error.request) {
            // The request was made but no response was received
            apiTestResult.value = {
            success: false,
            message:
                "API test failed: No response received from server. This might be due to a CORS issue.",
            };
        } else {
            // Something happened in setting up the request that triggered an Error
            apiTestResult.value = {
            success: false,
            message: `API test failed: ${error.message}`,
            };
        }
        } finally {
        isTestingAPI.value = false;
        }
    };
    
    const generateAndRunTests = async () => {
        isLoading.value = true;
        errorMessage.value = "";
        testCases.value = [];
    
        try {
        const generatedTestCases = await generateTestCases();
        if (generatedTestCases.length === 0) {
            throw new Error("No valid test cases were generated. Please try again.");
        }
        testCases.value = await Promise.all(generatedTestCases.map(runTestCase));
        } catch (error) {
        console.error("Error generating or running tests:", error);
        errorMessage.value = `Error: ${error.message}. Please try again or adjust your input.`;
        } finally {
        isLoading.value = false;
        }
    };
    
    const generateTestCases = async () => {
        const model = genAI.getGenerativeModel({ model: "gemini-1.5-pro-latest" });
    
        const prompt = `
                Given the following API endpoint, HTTP method, and payload example, generate all possible test cases as many you can think the more the merrier:
                API Endpoint: ${apiEndpoint.value}
                HTTP Method: ${httpMethod.value}
                Payload Example: ${
                httpMethod.value === "POST"
                    ? payloadExample.value
                    : "N/A (GET request)"
                }
        
                For each test case, provide:
                1. A description of the test case
                2. The modified payload for the test (for POST requests) or query parameters (for GET requests)
        
                Format your response as a JSON array of objects, each with 'description' and 'payload' fields.
                Ensure the response is valid JSON. Do not include any explanatory text outside the JSON structure.
            `;
    
        const result = await model.generateContent(prompt);
        const response = await result.response;
        const cleanedResponse = cleanJsonResponse(response.text());
    
        try {
        return JSON.parse(cleanedResponse);
        } catch (error) {
        console.error("Failed to parse AI response:", cleanedResponse);
        throw new Error(
            "Failed to generate valid test cases. AI response was not in the expected format.",
        );
        }
    };
    
    const cleanJsonResponse = (response) => {
        const jsonStart = response.indexOf("[");
        const jsonEnd = response.lastIndexOf("]") + 1;
        if (jsonStart === -1 || jsonEnd === 0) {
        throw new Error("AI response does not contain a valid JSON array");
        }
        return response.slice(jsonStart, jsonEnd);
    };
    
    const getAxiosConfig = (payload = null) => {
        const config = {
        method: httpMethod.value.toLowerCase(),
        url: apiEndpoint.value,
        validateStatus: () => true,
        headers: {},
        };
    
        if (httpMethod.value === "POST" && payload) {
        config.data = payload;
        } else if (httpMethod.value === "GET" && payload) {
        config.params = payload;
        }
    
        switch (authMethod.value) {
        case "apiKey":
            config.headers[authKey.value] = authValue.value;
            break;
        case "bearerToken":
            config.headers["Authorization"] = `Bearer ${authValue.value}`;
            break;
        case "jwtToken":
            config.headers["Authorization"] = `JWT ${authValue.value}`;
            break;
        }
    
        return config;
    };
    
    const runTestCase = async (testCase) => {
        try {
        const config = getAxiosConfig(testCase.payload);
        const response = await axios(config);
    
        return {
            ...testCase,
            statusCode: response.status,
            response: response.data,
        };
        } catch (error) {
        console.error("Test case error:", error);
        return {
            ...testCase,
            statusCode: error.response ? error.response.status : "Error",
            response: { error: error.message },
        };
        }
    };
    
    const openModal = (data) => {
        modalData.value = data;
        showModal.value = true;
    };
    
    const closeModal = () => {
        modalData.value = null;
        showModal.value = false;
    };
</script>
  