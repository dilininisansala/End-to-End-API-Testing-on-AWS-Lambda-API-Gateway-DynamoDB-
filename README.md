# End-to-End-API-Testing-on-AWS-Lambda-API-Gateway-DynamoDB-
I recently completed a hands-on project to strengthen my cloud testing skills by building and testing a serverless backend using AWS services.

⚙️**𝐓𝐞𝐜𝐡 𝐒𝐭𝐚𝐜𝐤 𝐔𝐬𝐞𝐝:**
* AWS Lambda → Backend logic (CRUD operations)
* AWS API Gateway → Exposes REST APIs
* AWS DynamoDB → Database (stores data)
* AWS IAM → Permissions
* Amazon CloudWatch → Logs & debugging
* Postman

⚙️**𝐈𝐦𝐩𝐥𝐞𝐦𝐞𝐧𝐭𝐚𝐭𝐢𝐨𝐧 𝐇𝐢𝐠𝐡𝐥𝐢𝐠𝐡𝐭𝐬**
*  Integrated AWS Lambda with API Gateway
*  Connected APIs to DynamoDB (real database)
*  Implemented full CRUD operations:

⚙️**𝐀𝐏𝐈 𝐆𝐚𝐭𝐞𝐰𝐚𝐲 𝐒𝐞𝐭𝐮𝐩**
* Created resource /𝐞𝐦𝐩𝐥𝐨𝐲𝐞𝐞𝐬
* Added methods
🔹GET
🔹POST
🔹PUT
🔹DELETE
* Verified each method:
  * Integration type = Lambda Function
  * Connected to appropriate Lambda functions
![11](https://github.com/user-attachments/assets/fe195df7-a2ff-4434-a49a-a1469c64627e)

* Created resource /𝐞𝐦𝐩𝐥𝐨𝐲𝐞𝐞𝐬/{𝐢𝐝}
* Attached methods:
🔹GET
🔹PUT
🔹DELETE
* Enabled Lambda proxy integration for all methods
![1](https://github.com/user-attachments/assets/9a07f33f-ae41-4aab-baf7-c819994b9007)
![10](https://github.com/user-attachments/assets/62e42fb8-ce55-4089-a8d0-9e54b285b17b)

After configuring endpoints, deployed API using 𝐀𝐦𝐚𝐳𝐨𝐧 𝐀𝐏𝐈 𝐆𝐚𝐭𝐞𝐰𝐚𝐲
Base URL obtained after deployment:
𝒉𝒕𝒕𝒑𝒔://𝒑1𝒔8𝒗𝒋𝒗𝒖8𝒄.𝒆𝒙𝒆𝒄𝒖𝒕𝒆-𝒂𝒑𝒊.𝒖𝒔-𝒆𝒂𝒔𝒕-1.𝒂𝒎𝒂𝒛𝒐𝒏𝒂𝒘𝒔.𝒄𝒐𝒎/𝑫𝒆𝒗
![6](https://github.com/user-attachments/assets/aa017c70-ff3d-4b21-95e6-06b72827f780)

📌 **𝐖𝐡𝐚𝐭 𝐈 𝐁𝐮𝐢𝐥𝐭**
A complete REST API for managing employee data:
/𝐞𝐦𝐩𝐥𝐨𝐲𝐞𝐞𝐬 
🔹GET (Get all employees)
🔹POST (Create employee)
/𝐞𝐦𝐩𝐥𝐨𝐲𝐞𝐞𝐬/{𝐢𝐝} 
🔹GET (Get employee by ID)
🔹PUT (Update employee)
🔹DELETE (Delete employee)

⚙️**AWS Lambda Setup**
* Create Lambda Function
* Go to AWS Lambda
* Click **Create Function** button
![12](https://github.com/user-attachments/assets/ef7536e9-83ad-49b9-b980-b2df3066ea63)
![13](https://github.com/user-attachments/assets/c9e41ade-163a-40e6-afbe-e01c67dc8b52)
```
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import {
  GetItemCommand,
  PutItemCommand,
  DeleteItemCommand,
  ScanCommand   // ✅ THIS is required for GET ALL
} from "@aws-sdk/client-dynamodb";

const client = new DynamoDBClient({ region: "us-east-1" });
const TABLE = "Employee";

export const handler = async (event) => {
  const method = event.httpMethod;
  const id = event.pathParameters?.id;

  try {

    // =========================
    // ✅ GET ALL EMPLOYEES
    // =========================
    if (method === "GET" && !id) {
      const result = await client.send(
        new ScanCommand({
          TableName: TABLE,
        })
      );

      return {
        statusCode: 200,
        body: JSON.stringify({
          message: "All employees fetched successfully",
          data: result.Items,
        }),
      };
    }

    // =========================
    // ✅ VALIDATION FOR ID METHODS
    // =========================
    if ((method === "GET" || method === "PUT" || method === "DELETE") && !id) {
      return {
        statusCode: 400,
        body: JSON.stringify({ message: "Employee ID is required" }),
      };
    }

    // =========================
    // ✅ GET BY ID
    // =========================
    if (method === "GET") {
      const result = await client.send(
        new GetItemCommand({
          TableName: TABLE,
          Key: {
            EmpId: { N: id },
          },
        })
      );

      if (!result.Item) {
        return {
          statusCode: 404,
          body: JSON.stringify({ message: "Employee not found" }),
        };
      }

      return {
        statusCode: 200,
        body: JSON.stringify({
          message: "Employee fetched successfully",
          data: result.Item,
        }),
      };
    }

    // =========================
    // ✅ POST (CREATE EMPLOYEE)
    // =========================
    if (method === "POST") {
      const body = JSON.parse(event.body);

      if (!body.EmpId || !body.FirstName) {
        return {
          statusCode: 400,
          body: JSON.stringify({
            message: "EmpId and FirstName are required",
          }),
        };
      }

      await client.send(
        new PutItemCommand({
          TableName: TABLE,
          Item: {
            EmpId: { N: body.EmpId.toString() },
            FirstName: { S: body.FirstName },
            LastName: { S: body.LastName || "" },
            Department: { S: body.Department || "" },
            JobTitle: { S: body.JobTitle || "" },
            Salary: { N: body.Salary ? body.Salary.toString() : "0" },
          },
        })
      );

      return {
        statusCode: 201,
        body: JSON.stringify({
          message: "Employee created successfully",
        }),
      };
    }

    // =========================
    // ✅ PUT (UPDATE EMPLOYEE)
    // =========================
    if (method === "PUT") {
      const body = JSON.parse(event.body);

      await client.send(
        new PutItemCommand({
          TableName: TABLE,
          Item: {
            EmpId: { N: id },
            FirstName: { S: body.FirstName || "" },
            LastName: { S: body.LastName || "" },
            Department: { S: body.Department || "" },
            JobTitle: { S: body.JobTitle || "" },
            Salary: { N: body.Salary ? body.Salary.toString() : "0" },
          },
        })
      );

      return {
        statusCode: 200,
        body: JSON.stringify({
          message: "Employee updated successfully",
        }),
      };
    }

    // =========================
    // ✅ DELETE
    // =========================
    if (method === "DELETE") {
      await client.send(
        new DeleteItemCommand({
          TableName: TABLE,
          Key: {
            EmpId: { N: id },
          },
        })
      );

      return {
        statusCode: 200,
        body: JSON.stringify({
          message: "Employee deleted successfully",
        }),
      };
    }

    return {
      statusCode: 405,
      body: JSON.stringify({ message: "Method not allowed" }),
    };

  } catch (error) {
    return {
      statusCode: 500,
      body: JSON.stringify({
        message: "Internal server error",
        error: error.message,
      }),
    };
  }
};
```

🔐**𝐈𝐀𝐌 𝐑𝐨𝐥𝐞 & 𝐏𝐞𝐫𝐦𝐢𝐬𝐬𝐢𝐨𝐧𝐬**
* AWS automatically created an execution role
Example: 𝒈𝒆𝒕𝑨𝒍𝒍𝑬𝒎𝒑𝒍𝒐𝒚𝒆𝒆𝒔-𝒓𝒐𝒍𝒆-𝒙𝒙𝒙𝒙
* Attached policy (for testing):
Example: 𝑨𝒎𝒂𝒛𝒐𝒏𝑫𝒚𝒏𝒂𝒎𝒐𝑫𝑩𝑭𝒖𝒍𝒍𝑨𝒄𝒄𝒆𝒔𝒔
![7](https://github.com/user-attachments/assets/fa324159-831a-4e17-a136-1dab44de291a)
![8](https://github.com/user-attachments/assets/c298454e-003e-4b2f-8d05-cd064450901a)

🧪 **𝐀𝐏𝐈 𝐓𝐞𝐬𝐭𝐢𝐧𝐠 (𝐏𝐨𝐬𝐭𝐦𝐚𝐧)**
*  Validated status codes (200, 201, 400, 404, 500)
*  Verified backend data consistency with DynamoDB
*  Debugged issues like:
   * Missing Authentication Token
   * Internal Server Error
   * Incorrect Lambda integration

🧠 **𝐊𝐞𝐲 𝐋𝐞𝐚𝐫𝐧𝐢𝐧𝐠𝐬**
* Difference between Lambda Proxy vs Non-Proxy integration
* Handling path parameters (/employees/{id})
* Importance of API deployment after changes
* Real-world debugging in cloud environments
