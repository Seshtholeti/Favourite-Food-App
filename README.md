Response
{
  "statusCode": 200,
  "body": "{\"message\":\"Process completed successfully\",\"details\":[{\"phoneNumber\":\"+919949921498\",\"agentId\":\"Error\",\"disposition\":\"Error Fetching Record\"},{\"phoneNumber\":\"+918639694701\",\"agentId\":\"Error\",\"disposition\":\"Error Fetching Record\"}]}"
}

Function Logs
START RequestId: aa800c20-a3d4-442b-8204-58be012ff069 Version: $LATEST
2024-11-29T02:01:15.594Z	aa800c20-a3d4-442b-8204-58be012ff069	ERROR	Error processing phone number +919949921498: BadRequestException: Invalid request body
    at throwDefaultError (/var/runtime/node_modules/@aws-sdk/node_modules/@smithy/smithy-client/dist-cjs/index.js:840:20)
    at /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/smithy-client/dist-cjs/index.js:849:5
    at de_CommandError (/var/runtime/node_modules/@aws-sdk/client-connect/dist-cjs/index.js:9806:14)
    at process.processTicksAndRejections (node:internal/process/task_queues:95:5)
    at async /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/middleware-serde/dist-cjs/index.js:35:20
    at async /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/core/dist-cjs/index.js:165:18
    at async /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/middleware-retry/dist-cjs/index.js:320:38
    at async /var/runtime/node_modules/@aws-sdk/middleware-logger/dist-cjs/index.js:34:22
    at async Runtime.handler (file:///var/task/index.mjs:249:33) {
  '$fault': 'client',
  '$metadata': {
    httpStatusCode: 400,
    requestId: 'cfbfaebb-b4e4-45cf-9976-9afad79a3868',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  }
}
2024-11-29T02:01:15.754Z	aa800c20-a3d4-442b-8204-58be012ff069	ERROR	Error processing phone number +918639694701: BadRequestException: Invalid request body
    at throwDefaultError (/var/runtime/node_modules/@aws-sdk/node_modules/@smithy/smithy-client/dist-cjs/index.js:840:20)
    at /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/smithy-client/dist-cjs/index.js:849:5
    at de_CommandError (/var/runtime/node_modules/@aws-sdk/client-connect/dist-cjs/index.js:9806:14)
    at process.processTicksAndRejections (node:internal/process/task_queues:95:5)
    at async /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/middleware-serde/dist-cjs/index.js:35:20
    at async /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/core/dist-cjs/index.js:165:18
    at async /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/middleware-retry/dist-cjs/index.js:320:38
    at async /var/runtime/node_modules/@aws-sdk/middleware-logger/dist-cjs/index.js:34:22
    at async Runtime.handler (file:///var/task/index.mjs:249:33) {
  '$fault': 'client',
  '$metadata': {
    httpStatusCode: 400,
    requestId: '06be69aa-6146-46be-bdee-3a52cd72c3ee',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  }
}
2024-11-29T02:01:15.754Z	aa800c20-a3d4-442b-8204-58be012ff069	INFO	Call Details: [
  {
    phoneNumber: '+919949921498',
    agentId: 'Error',
    disposition: 'Error Fetching Record'
  },
  {
    phoneNumber: '+918639694701',
    agentId: 'Error',
    disposition: 'Error Fetching Record'
  }
]
END RequestId: aa800c20-a3d4-442b-8204-58be012ff069
REPORT RequestId: aa800c20-a3d4-442b-8204-58be012ff069	Duration: 1568.12 ms	Billed Duration: 1569 ms	Memory Size: 128 MB	Max Memory Used: 99 MB	Init Duration: 666.96 ms

Request ID
