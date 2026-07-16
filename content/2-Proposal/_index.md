---
title: "Proposal"
date: 2026-07-10
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Wakan - Personalised Travel Assistant Using AI
## A Serverless AWS Solution for Tailored and Verified Travel Itineraries

### 1. Executive Summary
**Wakan** is an AI-powered personalized travel assistant system designed specifically for travel within Vietnam. It allows users to automatically generate optimized itineraries based on their real-time location, budget, travel duration, preferred mode of transport, travel companions, and preferred type of experience. 

By leveraging a fully Serverless architecture on AWS (incorporating AWS WAF, Amazon CloudFront, Amazon S3, Amazon API Gateway, Amazon Cognito, AWS Lambda, Amazon DynamoDB, AWS Secrets Manager, and Amazon CloudWatch), Wakan ensures high performance, minimal operational overhead, robust security, and seamless scalability to handle user growth.

---

### 2. Problem Statement

#### What’s the Problem?
Planning a travel itinerary is currently a highly fragmented and time-consuming process. Travelers must manually research destinations across various social media platforms, blogs, and maps, and then stitch together routes, timings, and budgets. 

While general-purpose generative AI models (such as ChatGPT or Gemini) can assist in writing itineraries, they suffer from two major flaws:
1. **User Prompting Complexity**: Users must write detailed, complex prompts (Prompt Engineering) to obtain structured, relevant results.
2. **AI Hallucinations**: Standard LLMs often invent non-existent locations, output incorrect geographic distances, or suggest routes that do not match the local Vietnamese transport context.

#### The Solution
**Wakan** solves these issues by providing a structured, choice-based user interface rather than a free-form chat. Users simply select their parameters (e.g., solo travel, family trip, low/high budget, short/long distance, single or multi-day). 
- **Controlled Inputs**: Restricting inputs to UI selection boxes reduces the risk of *Prompt Injection* attacks and ensures reliable inputs.
- **Server-Side Validation**: Backend validation checks ensure requests submitted directly via the API remain valid.
- **Structured Outputs**: The backend forces the AI model to return structured data matching a strict JSON schema, which is parsed and mapped directly onto the user interface.
- **Verified Places**: For premium tiers, the AI is grounded using a database of pre-verified locations, eliminating hallucinated or inaccurate recommendations.

#### Benefits and Return on Investment (ROI)
- **Time Saving**: Reduces travel planning time by up to 90% by consolidating route planning, place selection, and scheduling into a single click.
- **Serverless Cost-Efficiency**: The entire architecture runs serverless, meaning near-zero costs when idle and pay-per-use scaling. It perfectly aligns with the team's AWS Credits.
- **API Cost Optimization**: An aggressive caching layer on DynamoDB hashes rounded user locations and preferences, delivering instant hits for similar requests and avoiding expensive LLM API invocations.

---

### 3. Solution Architecture
The platform is built using a highly secure, serverless multi-tier AWS architecture to manage user authentication, data caching, verified location grounding, and AI processing.

![Wakan Solution Architecture](/worlog/images/2-Proposal/kientruc.jpg)

#### AWS Services Used
- **AWS WAF**: Blocks malicious web traffic, SQL injection, and bots at the HTTP/HTTPS boundary.
- **Amazon CloudFront**: Acts as a Content Delivery Network (CDN) to host static frontend assets globally and route dynamic `/api/*` traffic to the API Gateway.
- **Amazon S3**: Hosts the static web application files (HTML, CSS, JS) securely.
- **Amazon API Gateway**: Serves as the secure REST API gateway, integrating with Amazon Cognito for authentication and managing technical rate limits.
- **Amazon Cognito**: Handles secure user registration, login, and token-based (JWT) session authorization.
- **AWS Lambda**: Divided into two dedicated functions for separation of concerns:
  1. **AWS Lambda (Orchestrator)**: Handles request validation, queries DynamoDB for quotas/subscriptions, computes cache keys, and handles caching logic.
  2. **AWS Lambda (AI Processor)**: Connects to external AI APIs, retrieves credentials from Secrets Manager, formats prompts, and validates JSON structure.
- **Amazon DynamoDB**: A fully managed NoSQL database hosting three key tables:
  1. `Users/Subscriptions`: Stores user accounts, active subscription tiers, and remaining daily quotas.
  2. `Cache`: Stores previously generated itineraries, indexed by rounded GPS coordinates and selections, using a Time-To-Live (TTL) to expire stale routes.
  3. `Verified Places`: Contains curated, verified local places used to ground AI responses for high-tier packages.
- **AWS Secrets Manager**: Securely stores API keys for external LLM endpoints.
- **Amazon CloudWatch**: Aggregates logs and monitors metrics (invocation errors, cache hit rates) with CloudWatch Alarms.

#### Component Design
1. **Edge and Content Delivery**: AWS WAF and CloudFront secure the ingress. CloudFront splits traffic: static requests pull from S3, while API calls are forwarded to API Gateway.
2. **Authentication & Authorization**: Cognito issues JSON Web Tokens (JWT) upon successful login. API Gateway uses a Cognito Authorizer to validate these tokens before executing Lambda functions.
3. **Business Logic & AI Orchestration**: 
   - **Lambda Orchestrator** checks the user's daily quota in DynamoDB. It hashes the input parameters (including rounded latitude/longitude to maximize cache hits) to search the `Cache` table. 
   - If a cache miss occurs, it invokes **Lambda AI Processor**.
   - **Lambda AI Processor** fetches the API key from Secrets Manager, queries the LLM, parses the response, and ensures compliance with the target JSON schema before returning it.
4. **Subscription Tiers**:
   - **Free**: Basic features, restricted daily quota, local/short itineraries.
   - **Plus**: Extended daily quota, support for multi-day and long-distance itineraries.
   - **Pro**: Premium experience utilizing the verified location database to completely prevent hallucinated spots.

---

### 4. Technical Implementation

#### Implementation Phases
The project is scheduled across 6 weeks (from Week 7 to Week 12) during the internship:
- **Phase 1: Design & Planning (Week 7)**: Write detailed use-cases, define the DynamoDB schemas, and create the UI wireframes using Google Drawings/Jamboard.
- **Phase 2: Infrastructure Setup (Week 8)**: Set up S3 web hosting, CloudFront, Cognito User Pools, and define IAM roles following the principle of least privilege.
- **Phase 3: Core Backend & Data Logic (Week 9)**: Write Lambda Orchestrator (cache & quota checks) and compile the `Verified Places` dataset. Secure credentials in Secrets Manager.
- **Phase 4: Frontend & Integration (Week 10)**: Build responsive web UI components, integrate maps, connect Cognito auth, and hook up the REST API.
- **Phase 5: Testing & Security Audit (Week 11)**: Implement CloudWatch logging, set up AWS Budgets, perform security scanning on API Gateway/Lambda, and optimize execution cold-starts.
- **Phase 6: Deployment & Delivery (Week 12)**: Run final integration testing, record a walkthrough demo video, and complete the project documentation.

#### Technical Requirements
- **Infrastructure as Code (IaC)**: Deploy resources programmatically using AWS CDK or Serverless Framework.
- **Backend Language**: Node.js/TypeScript for AWS Lambda to ensure rapid startup and execution speeds.
- **AI Structured Outputs**: Integration with AI models that support strict JSON output schema validation.
- **API Mapping Service**: Integration with map APIs (e.g., Amazon Location Service) for routing calculations.

---

### 5. Timeline & Milestones

| Week | Team Member | Planned Tasks |
|---|---|---|
| **Week 7** | **All Members** | Meet to align on Free/Plus/Pro scopes, finalize architectural design, set up git repo, and initialize IAM accounts. |
| | Trần Hữu Khang | Define user flow use-cases & sketch DynamoDB tables. |
| | Lê Trần Anh Đức | Finalize system architecture documentation and draft security policies. |
| | Đinh Hoàng Vương | Design wireframes on Google Drawings/Jamboard for the choice-based questionnaire flow. |
| | Trần Phúc Đăng | Research external LLM APIs and draft the initial prompt templates. |
| **Week 8** | **All Members** | Define API request/response contracts between Frontend, Orchestrator, and AI Processor. |
| | Trần Hữu Khang | Create API Gateway structure and Lambda Orchestrator skeleton. |
| | Lê Trần Anh Đức | Provision AWS WAF, CloudFront, S3 buckets, and Cognito User Pool. |
| | Đinh Hoàng Vương | Implement static UI layouts and wire up Cognito authentication. |
| | Trần Phúc Đăng | Implement Lambda AI Processor skeleton and test LLM integrations. |
| **Week 9** | **All Members** | Perform peer code reviews to ensure quota and cache schemas align. |
| | Trần Hữu Khang | Implement full Orchestrator logic: quota calculations and coordinate rounding cache. |
| | Lê Trần Anh Đức | Configure Secrets Manager and set up API Gateway Usage Plans (rate limits). |
| | Đinh Hoàng Vương | Bind frontend components to the live REST API and handle loading/error states. |
| | Trần Phúc Đăng | Refine prompt templates and build the `Verified Places` DynamoDB dataset. |
| **Week 10** | **All Members** | Run initial end-to-end integration tests (UI -> API -> LLM) and fix bugs. |
| | Trần Hữu Khang | Debug cache hit/miss flows and configure custom TTL settings. |
| | Lê Trần Anh Đức | Set up CloudWatch Logs/Alarms and create AWS Budget alerts. |
| | Đinh Hoàng Vương | Render itinerary results on interactive maps and responsive timelines. |
| | Trần Phúc Đăng | Validate output quality for Free, Plus, and Pro packages. |
| **Week 11** | **All Members** | Review integration test results and identify system bottlenecks. |
| | Trần Hữu Khang | Conduct load testing and optimize Lambda memory/timeout configurations. |
| | Lê Trần Anh Đức | Conduct penetration testing (WAF bypass checks, IAM least-privilege audits). |
| | Đinh Hoàng Vương | Polish UI/UX, resolve mobile responsiveness issues, and prep demo. |
| | Trần Phúc Đăng | Write unit and integration tests; compile QA metrics. |
| **Week 12** | **All Members** | Execute final demo runs, record video walkthrough, compile report, and present. |

---

### 6. Budget Estimation
Since Wakan is built entirely on serverless services, it is highly cost-effective:
- **AWS Lambda**: $0.00/month (Free Tier covers up to 1 million requests).
- **Amazon DynamoDB**: ~$0.50/month (using On-Demand scaling for Free/Plus/Pro tables).
- **Amazon S3 & CloudFront**: ~$0.20/month for static web hosting and global distribution.
- **Amazon Cognito**: $0.00/month (Free Tier supports up to 50,000 MAUs).
- **AWS Secrets Manager**: $0.40/month (1 active secret for AI API Key).
- **AWS WAF**: ~$6.00/month (Base charge of $5.00 for Web ACL and $1.00 for custom rules).
- **Amazon CloudWatch**: ~$0.50/month for logs and basic alarms.
- **External AI API**: Pay-as-you-go (starts with free credits or minimal API token costs).

**Total Infrastructure Cost**: **~$7.60 USD / month** (excluding external LLM tokens), which fits comfortably within the student AWS Credits.

---

### 7. Risk Assessment

#### Risk Matrix
- **AI Hallucinations / Bad Data**: High Impact, Medium Probability.
- **AI API Cost Spikes**: Medium Impact, High Probability.
- **API Abuse / Spam**: Medium Impact, Medium Probability.
- **Credential Leakage**: Critical Impact, Low Probability.

#### Mitigation Strategies
- **Hallucinations**: Enforce grounding using `Verified Places` for the Pro package. Limit LLMs to selection and sequencing rather than creative writing.
- **AI API Costs**: Implement caching in DynamoDB (coordinates rounded to maximize cache hits). Cache responses up to 48 hours for multi-day trips.
- **Spam**: Implement Cognito token authentication, rate limit clients via API Gateway Usage Plans, and enforce daily quotas inside the Lambda Orchestrator.
- **Leakage**: Use AWS Secrets Manager. Do not commit credentials to Git. Use IAM Roles with least-privilege access.

---

### 8. Expected Outcomes
1. **Fully Functioning MVP**: A responsive web application where users can input travel preferences and receive structured, highly reliable itineraries.
2. **Optimized Serverless Architecture**: A secure cloud architecture on AWS that costs nearly nothing when idle and scales instantly during spikes.
3. **Comprehensive Knowledge Base**: Complete documentation detailing API contracts, security practices, and cost controls for serverless AI integrations.
