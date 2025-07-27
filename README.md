## 🧩 Serverless Q&A Platform – Angular + AWS Lambda + API Gateway

This is a full-stack serverless Questions & Answers web application built using Angular (frontend) and AWS Lambda with API Gateway (backend). The backend is stateless, scalable, and integrated via REST APIs. Frontend is a Single Page Application (SPA) hosted on Amazon S3.

---

## 🧰 Prerequisites

- AWS Account with necessary permissions (API Gateway, Lambda, S3)
- AWS CLI configured (`aws configure`)
- Node.js (v18+)
- Angular CLI (`npm install -g @angular/cli`)
- An S3 bucket for frontend hosting

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/your-org/serverless-qna.git
cd serverless-qna
````

### 2. Install Dependencies

#### Angular (Frontend)

```bash
cd client
npm install
```

#### Lambda Functions

```bash
cd lambdas/UpsertQuestion
npm install
zip -r function.zip .
```

Repeat this for each Lambda function directory.

---

## ⚙️ Backend Setup – AWS API Gateway + Lambda

### Step 1: Create REST API in API Gateway

* Go to **API Gateway** > Create API > **REST API**
* Choose:

  * **New API**
  * API Name: `QuestionsAndAnswers`
  * Click **Create API**

---

### Step 2: Define Resources and Methods

#### `/Questions`

| Method | Lambda Function |
| ------ | --------------- |
| GET    | TableScan       |
| POST   | UpsertQuestion  |
| PUT    | UpsertQuestion  |

#### `/Questions/{id}`

| Method | Lambda Function |
| ------ | --------------- |
| GET    | GetSingleRecord |
| DELETE | DeleteRecord    |
| PUT    | UpsertQuestion  |

#### `/Questions/findOne`

| Method | Lambda Function |
| ------ | --------------- |
| GET    | FindOneQuestion |

#### `/Answers`

| Method | Lambda Function |
| ------ | --------------- |
| GET    | TableScan       |
| POST   | UpsertAnswer    |
| PUT    | UpsertAnswer    |

#### `/Answers/{id}`

| Method | Lambda Function |
| ------ | --------------- |
| GET    | GetSingleRecord |
| DELETE | DeleteRecord    |
| PUT    | UpsertAnswer    |

> For all methods:
>
> * Integration Type: **Lambda Function**
> * Use Lambda Proxy Integration: ✅
> * Region: `us-east-1`
> * Click **OK** when prompted to give API Gateway permission

---

### Step 3: Add CORS (OPTIONS method)

For each resource (`/Questions`, `/Answers`, etc.):

1. Create `OPTIONS` method
2. Integration Type: **Mock**
3. Method Response: Add headers:

   * `Access-Control-Allow-Origin`
   * `Access-Control-Allow-Headers`
   * `Access-Control-Allow-Methods`
   * `Access-Control-Allow-Credentials`
   * `X-Requested-With`
4. Integration Response > Header Mappings:

```text
Access-Control-Allow-Origin: '*'
Access-Control-Allow-Headers: 'Content-Type,X-Amz-Date,Authorization,X-Api-Key,x-requested-with'
Access-Control-Allow-Methods: 'OPTIONS,*'
Access-Control-Allow-Credentials: 'true'
X-Requested-With: '*'
```

---

### Step 4: Lambda CORS Headers (in code)

Each Lambda should use:

```js
const responseHeaders = (headers) => {
  const origin = headers.origin || headers.Origin;
  return {
    "Content-Type": "application/json",
    "X-Requested-With": "*",
    "Access-Control-Allow-Headers":
      "Content-Type,X-Amz-Date,Authorization,X-Api-Key,x-requested-with",
    "Access-Control-Allow-Origin": origin,
    "Access-Control-Allow-Methods": "OPTIONS,*",
    "Access-Control-Allow-Credentials": "true",
    Vary: "Origin",
  };
};

// Return response like:
return {
  statusCode: 200,
  headers: responseHeaders(event.headers),
  body: JSON.stringify(data),
};
```

---

### Step 5: Deploy API

1. In API Gateway, choose **Actions > Deploy API**
2. Stage Name: `api`
3. Note the **Invoke URL** (e.g., `https://xyz123.execute-api.us-east-1.amazonaws.com/api`)

---

## 🖼️ Frontend Configuration

### Update Environment File

In `client/src/environments/environment.prod.ts`:

```ts
export const environment = {
  production: true,
  api_url: 'https://xyz123.execute-api.us-east-1.amazonaws.com'
};
```

### Update main.ts

At the end of `main.ts`:

```ts
LoopBackConfig.filterOnUrl();
```

---

## 🏗️ Build and Deploy Frontend to S3

### 1. Build Angular App

```bash
ng build --configuration=production
```

### 2. Deploy to S3

```bash
aws s3 sync ./dist/client s3://your-s3-bucket-name --delete
```

Make sure:

* `index.html` is the default root document
* Public access and static website hosting are enabled

---

## ✅ Testing & Validation

* Use Postman or browser to test API endpoints
* Test frontend by visiting the S3 static site URL
* Check browser console for CORS or network issues

---

## 🧯 Troubleshooting

| Problem                | Solution                                       |
| ---------------------- | ---------------------------------------------- |
| 403 Error              | Check Lambda IAM role and permissions          |
| CORS Error             | Ensure OPTIONS method and headers are set      |
| No integration defined | Make sure method is saved after linking Lambda |
| Lambda not responding  | Check CloudWatch Logs                          |

---

## 📌 Summary

This project demonstrates:

* A scalable serverless backend using AWS Lambda + API Gateway
* A modern Angular SPA frontend hosted via S3
* Secure, CORS-compliant REST API integration
* Clean modular function structure and deployment

---
