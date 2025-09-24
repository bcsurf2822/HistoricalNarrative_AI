# Historical Site AI Agent - AWS Architecture

## High-Level Architecture

```
User Upload (Image) → API Gateway → Lambda → Textract (OCR) → Bedrock Agent → Knowledge Base/Web Search → Response Generation → DynamoDB (Memory) → Frontend
```

## AWS Services Architecture

### Core Agent Services
- **Amazon Bedrock AgentCore**: Primary agent orchestration and reasoning
- **Amazon Bedrock (Claude/Nova)**: LLM for natural language understanding and generation
- **Amazon Bedrock Knowledge Bases**: RAG implementation for historical data
- **Strands Agents**: Advanced agent capabilities within Bedrock

### Supporting Services
- **AWS Lambda**: Serverless compute for all functions
- **Amazon API Gateway**: RESTful API endpoints
- **Amazon Textract**: OCR for extracting text from historical markers
- **Amazon S3**: Storage for images, knowledge base documents, generated content
- **Amazon DynamoDB**: Conversation memory and user profiles
- **Amazon Rekognition**: Enhanced image analysis and scene understanding
- **AWS Step Functions**: Orchestrate complex multi-step agent workflows

### Optional Enhancement Services
- **Amazon SageMaker**: Custom model fine-tuning for historical context
- **Amazon Q**: Development assistance and code generation
- **AWS AppSync**: Real-time updates for chat interface
- **Amazon CloudFront**: CDN for fast image and content delivery

## Agent Workflow

### 1. Image Processing Pipeline
```
Image Upload → Textract (OCR) → Rekognition (Scene Analysis) → Context Extraction
```

### 2. Knowledge Retrieval
```
Extracted Text → Bedrock Knowledge Base (RAG) → Web Search Fallback → Historical Context
```

### 3. Agent Reasoning
```
User Context + Historical Data + Conversation Memory → Bedrock Agent → Personalized Response
```

### 4. Response Generation
```
Agent Decision → Content Type (Text/Visual/Interactive) → Memory Update → User Response
```

## Key Features Implemented

### Agent Qualifications Met
✅ **Reasoning LLMs**: Bedrock AgentCore with Claude/Nova for complex decision-making  
✅ **Autonomous Capabilities**: Independent historical research and content generation  
✅ **External Integrations**: Web search, knowledge bases, image processing, memory systems  

### Core Primitives Used
- **Function Calling**: Tool integration for web search and knowledge retrieval
- **Memory Management**: Persistent conversation and user profile storage
- **Multi-modal Processing**: Text, image, and generated content handling
- **Dynamic Prompting**: Context-aware prompt generation based on user state

## Serverless Benefits
- **Cost Efficiency**: Pay only for actual usage
- **Scalability**: Automatic scaling for varying loads
- **Maintenance**: Minimal infrastructure management
- **Integration**: Seamless AWS service integration

## Data Flow
1. **Input**: Historical marker image upload
2. **Processing**: OCR text extraction and scene analysis
3. **Retrieval**: Knowledge base search and web research
4. **Reasoning**: Agent analyzes context and user preferences
5. **Generation**: Creates personalized historical narrative
6. **Interaction**: Offers dialogue options and visual content
7. **Memory**: Stores conversation and learns user preferences
