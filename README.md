## AI-Powered API Tester 🚀

**Tired of writing tedious API tests?** Let AI do the heavy lifting! 🤖 This Vue.js application harnesses the power of Google's Gemini AI to generate and execute comprehensive API tests, making your testing process smarter and more efficient.

**Imagine:**

* **Effortless Test Generation:**  Describe your API and let the AI generate a wide range of test cases, covering various scenarios and edge cases.
* **Automated Execution:**  Run tests with a single click, receiving detailed results, including status codes, responses, and descriptions.
* **Powerful Authentication:** Support for API Keys, Bearer Tokens, and JWT tokens ensures secure testing of your APIs.

**Let's get started!**

### Getting Started

1. **Clone the repository:**

   ```bash
   git clone https://github.com/2105789/ApiTesto.git
   ```

2. **Install dependencies:**

   ```bash
   cd ApiTesto
   npm install
   ```

3. **Obtain a Gemini API Key:**

   * Sign up for a Google Cloud Platform account: [https://cloud.google.com/](https://cloud.google.com/)
   * Enable the Google Generative AI API: [https://cloud.google.com/generative-ai/docs/quickstart](https://cloud.google.com/generative-ai/docs/quickstart)
   * Create a new API key in your project: [https://cloud.google.com/iam/docs/creating-managing-keys](https://cloud.google.com/iam/docs/creating-managing-keys)

4. **Set up the API Key:**

   * Run the application: `npm run serve`
   * Click the "Set API Key" button.
   * Enter your Gemini API Key and click "Save Key".

5. **Start testing!**

   * **API Endpoint:** Enter the full URL of your API endpoint.
   * **HTTP Method:** Select the HTTP method (GET or POST) for your API call.
   * **Payload Example:**  Enter a JSON representation of the payload for POST requests. Leave this blank for GET requests.
   * **Authentication:** Select an authentication method and provide the necessary credentials.
   * **Test API Connection:** Verify your API endpoint configuration.
   * **Generate and Run Tests:**  Let the AI generate and execute your tests.

###  Sample Code Snippets

**Generating Test Cases:**

```javascript
const prompt = `
  Given the following API endpoint, HTTP method, and payload example, generate all possible test cases:
  API Endpoint: ${apiEndpoint.value}
  HTTP Method: ${httpMethod.value}
  Payload Example: ${payloadExample.value}

  For each test case, provide:
  1. A description of the test case
  2. The modified payload for the test (for POST requests) or query parameters (for GET requests)

  Format your response as a JSON array of objects, each with 'description' and 'payload' fields.
  Ensure the response is valid JSON. Do not include any explanatory text outside the JSON structure.
`;

const model = genAI.getGenerativeModel({ model: "gemini-1.5-pro-latest" });
const result = await model.generateContent(prompt);
const generatedTestCases = JSON.parse(cleanJsonResponse(result.response.text()));
```

**Running Test Cases:**

```javascript
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
    // Handle error cases
  }
};
```

### Technologies

* **Vue.js:** Frontend framework for building the user interface.
* **Google Generative AI:**  Used to generate test cases using the Gemini AI model.
* **Axios:**  HTTP client library for making API requests.

### Contributing

Contributions are welcome! Feel free to open issues, submit pull requests, or propose new features.

### License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
