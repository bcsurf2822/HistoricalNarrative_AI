# Historical Site AI Agent - Deployment Guide

## 📋 Prerequisites

### AWS Account Setup
1. **AWS Account** with appropriate permissions
2. **AWS CLI** installed and configured
3. **Node.js 18+** and npm installed
4. **AWS CDK** installed globally: `npm install -g aws-cdk`

### Required AWS Services Access
- Amazon Bedrock (Claude/Nova models)
- Amazon Bedrock AgentCore
- AWS Lambda
- Amazon API Gateway
- Amazon S3
- Amazon DynamoDB
- Amazon Textract
- Amazon Rekognition (optional)

## 🚀 Quick Start Deployment

### Step 1: Clone and Setup
```bash
# Clone the project
git clone <your-repo>
cd historical-agent

# Install Lambda dependencies
cd lambda
npm install

# Install Frontend dependencies
cd ../frontend
npm install
```

### Step 2: AWS Services Setup

#### Enable Bedrock Models
```bash
# Enable Claude model access in AWS Console
aws bedrock list-foundation-models --region us-east-1

# Request model access for:
# - anthropic.claude-3-sonnet-20240229-v1:0
# - anthropic.claude-3-haiku-20240307-v1:0
```

#### Deploy Infrastructure
```bash
cd infrastructure

# Bootstrap CDK (first time only)
cdk bootstrap

# Deploy the stack
cdk deploy HistoricalAgentStack
```

### Step 3: Bedrock Agent Setup

#### Create Bedrock Agent
1. Go to Amazon Bedrock Console → Agents
2. Create new agent: "HistoricalSiteAgent"
3. Configure agent settings:
   - **Model**: Claude 3 Sonnet
   - **Instructions**: See agent-instructions.md
   - **Action Groups**: Configure web search and knowledge base actions

#### Create Knowledge Base
1. Go to Bedrock Console → Knowledge bases
2. Create new knowledge base: "HistoricalKnowledgeBase"
3. Data source: S3 bucket (historical-knowledge-base)
4. Embedding model: Amazon Titan Text Embeddings
5. Vector store: Amazon OpenSearch Serverless

### Step 4: Environment Configuration

Create `.env` files:

**Lambda Environment Variables** (set in CDK):
```bash
BEDROCK_AGENT_ID=your-agent-id
BEDROCK_AGENT_ALIAS_ID=TSTALIASID
IMAGE_BUCKET=historical-agent-images
CONVERSATION_TABLE=conversation-memory
USER_PROFILE_TABLE=user-profiles
KNOWLEDGE_BASE_BUCKET=historical-knowledge-base
```

**Frontend Environment** (`.env`):
```bash
REACT_APP_API_URL=https://your-api-gateway-url.amazonaws.com
REACT_APP_ENVIRONMENT=production
```

### Step 5: Deploy Application

#### Deploy Lambda Functions
```bash
cd lambda
# Functions are deployed via CDK
cdk deploy
```

#### Deploy Frontend
```bash
cd frontend

# Build for production
npm run build

# Create S3 bucket for hosting
aws s3 mb s3://historical-agent-frontend

# Configure S3 for static website hosting
aws s3 website s3://historical-agent-frontend --index-document index.html

# Deploy to S3
npm run deploy:s3
```

## 🔧 Configuration Details

### Bedrock Agent Instructions
```
You are a knowledgeable and culturally sensitive historical guide AI agent. Your role is to:

1. **Interpret Historical Content**: Analyze text from historical markers and provide rich, contextual information
2. **Provide Multiple Perspectives**: Present different historical viewpoints while maintaining accuracy
3. **Personalize Experiences**: Adapt responses based on user interests and background
4. **Generate Visual Descriptions**: Create detailed descriptions for historical visualization
5. **Maintain Engagement**: Ask thoughtful follow-up questions to deepen understanding

**Core Capabilities**:
- Historical research and fact verification
- Cultural sensitivity in presenting difficult topics
- Storytelling that brings history to life
- Educational content adaptation for different audiences

**Guidelines**:
- Always verify information through multiple sources
- Present multiple perspectives when discussing controversial topics
- Adapt language and complexity based on user profile
- Encourage curiosity and deeper exploration
- Respect cultural sensitivities and historical trauma

**Available Tools**:
- Web search for current historical research
- Knowledge base with historical documents and interpretations
- Image generation descriptions for historical visualization
```

### Action Groups Configuration

#### Web Search Action Group
```json
{
  "actionGroupName": "web_search",
  "description": "Search the web for historical information and current research",
  "functionSchema": {
    "functions": [
      {
        "name": "search_historical_information",
        "description": "Search for historical information about locations, events, or people",
        "parameters": {
          "query": {
            "type": "string",
            "description": "Search query for historical information"
          }
        }
      }
    ]
  }
}
```

#### Knowledge Base Action Group
```json
{
  "actionGroupName": "knowledge_retrieval",
  "description": "Retrieve information from the historical knowledge base",
  "functionSchema": {
    "functions": [
      {
        "name": "query_knowledge_base",
        "description": "Query the knowledge base for specific historical information",
        "parameters": {
          "query": {
            "type": "string",
            "description": "Query for knowledge base search"
          },
          "filters": {
            "type": "object",
            "description": "Filters for narrowing search results"
          }
        }
      }
    ]
  }
}
```

## 📊 AWS Services Integration

### Core Services Used
- ✅ **Amazon Bedrock AgentCore**: Primary agent orchestration
- ✅ **Amazon Bedrock/Claude**: LLM for reasoning and generation  
- ✅ **AWS Lambda**: Serverless compute for all functions
- ✅ **Amazon API Gateway**: RESTful API endpoints
- ✅ **Amazon DynamoDB**: Conversation memory and user profiles
- ✅ **Amazon S3**: Image storage and knowledge base documents
- ✅ **Amazon Textract**: OCR for historical marker text extraction

### Enhanced Services (Recommended)
- 🔄 **Amazon Bedrock Knowledge Bases**: RAG implementation
- 🔄 **Amazon Rekognition**: Enhanced image analysis
- 🔄 **AWS Step Functions**: Complex workflow orchestration
- 🔄 **Amazon OpenSearch**: Advanced search capabilities
- 🔄 **Amazon CloudFront**: Global content delivery
- 🔄 **AWS Amplify**: Frontend hosting with CI/CD

### Optional Services
- **Amazon SageMaker**: Custom model fine-tuning
- **Amazon Translate**: Multi-language support
- **Amazon Polly**: Text-to-speech for audio tours
- **Amazon Transcribe**: Voice input processing

## 🎯 AWS AI Agent Qualification Checklist

### ✅ Reasoning LLMs for Decision-Making
- **Primary**: Amazon Bedrock Claude 3 Sonnet for complex reasoning
- **Fallback**: Direct model invocation for error handling
- **Implementation**: Multi-step reasoning in agent orchestration

### ✅ Autonomous Capabilities
- **Without Human Input**: Automatic text extraction, historical context analysis
- **With Human Input**: Interactive conversations, personalized responses
- **Implementation**: Agent-driven workflow with user preference adaptation

### ✅ External Integrations
- **APIs**: Web search for current historical research
- **Databases**: DynamoDB for memory, S3 for knowledge base
- **Tools**: Textract for OCR, Rekognition for image analysis
- **Other Agents**: Potential multi-agent collaboration for complex queries

## 🔒 Security & Best Practices

### IAM Permissions
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeAgent",
        "bedrock:Retrieve",
        "bedrock:RetrieveAndGenerate"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "textract:DetectDocumentText",
        "textract:AnalyzeDocument"
      ],
      "Resource": "*"
    }
  ]
}
```

### Environment-Specific Configurations
- **Development**: Reduced timeouts, verbose logging
- **Production**: Optimized performance, error handling
- **Testing**: Mock services, local development

## 🚦 Monitoring & Observability

### CloudWatch Metrics
- Lambda function duration and errors
- API Gateway request counts and latency
- DynamoDB read/write capacity
- Bedrock model invocation costs

### Logging Strategy
- Structured JSON logs for all Lambda functions
- User interaction tracking (privacy-compliant)
- Agent decision paths and reasoning logs
- Error tracking and alerting

## 💰 Cost Optimization

### Expected Costs (Monthly, 1000 users)
- **Bedrock Model Invocations**: ~$50-100
- **Lambda Executions**: ~$10-20
- **DynamoDB**: ~$5-15
- **S3 Storage**: ~$5-10
- **API Gateway**: ~$3-5
- **Textract**: ~$10-20
- **Total Estimated**: ~$85-170/month

### Cost Reduction Strategies
- Use Bedrock Haiku for simple responses
- Implement intelligent caching
- Optimize Lambda memory allocation
- Use DynamoDB on-demand billing

## 🐛 Troubleshooting

### Common Issues
1. **Bedrock Access Denied**: Ensure model access is granted
2. **CORS Errors**: Check API Gateway CORS configuration
3. **Lambda Timeouts**: Increase timeout for image processing
4. **Agent Not Responding**: Verify agent ID and alias configuration

### Debug Commands
```bash
# Check CloudWatch logs
aws logs describe-log-groups --log-group-name-prefix /aws/lambda/historical

# Test API endpoints
curl -X POST https://your-api-gateway-url.amazonaws.com/process-image

# Monitor DynamoDB items
aws dynamodb scan --table-name conversation-memory --limit 5
```

## 📈 Next Steps & Iterations

### Immediate Improvements (Week 1-2)
1. Add error handling and retry logic
2. Implement conversation memory compression
3. Add basic analytics dashboard
4. Create user feedback collection

### Enhanced Features (Month 1-2)
1. Multi-modal content generation (images, audio)
2. Advanced personalization algorithms
3. Social sharing capabilities
4. Offline mode for downloaded content

### Advanced Capabilities (Month 3-6)
1. Custom model fine-tuning with SageMaker
2. Real-time collaborative tours
3. AR/VR integration capabilities
4. Multi-language support
5. Integration with museum/park systems

## 📞 Support & Resources

- **AWS Documentation**: https://docs.aws.amazon.com/bedrock/
- **Bedrock AgentCore Guide**: https://docs.aws.amazon.com/bedrock/latest/userguide/agents.html
- **Troubleshooting Guide**: See troubleshooting.md
- **Community Forum**: [Your community link]

---

**🎉 Congratulations!** You now have a fully functional AI Agent that meets all AWS qualification requirements and provides an engaging historical site interpretation experience.
