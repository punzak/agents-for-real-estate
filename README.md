# Agents for Real Estate

AI-powered multi-agent system for real estate, built on Amazon Bedrock AgentCore. Specialized agents collaborate to help home buyers search for properties, analyze markets, evaluate neighborhoods, and get personalized recommendations.

## Live Demo

🏠 **[Try the demo](https://donpf1n10on18.cloudfront.net)**

## What It Does

This system deploys 8 specialized real estate AI agents that work together:

| Agent | Role |
|-------|------|
| **RealtorOrchestratorAgent** | Routes queries to the right specialist agent |
| **BuyerMatchingAgent** | Matches buyers with properties based on lifestyle, budget, and preferences |
| **HomeValuationAgent** | AI-powered home valuations with market comps |
| **MarketIntelligenceAgent** | Market trends, pricing analysis, investment ROI |
| **NeighborhoodExpertAgent** | Schools, safety, walkability, amenities, community insights |
| **ListingOptimizerAgent** | Listing performance optimization for sellers/agents |
| **DealPipelineAgent** | Transaction tracking and deal pipeline management |
| **MarketTrendsAgent** | Interest rates, economic factors, and market timing |

## Example Scenarios

**Home Buyer Tab:**
- 🏠 "Find me a 3-4 bed home in Austin with good schools, under $650K"
- 🎓 "I'm a first-time buyer with $80K saved — where should I look?"
- 🏘️ "Compare Mueller vs South Austin for a family with young kids"
- 📊 "Is 78704 a buyer's or seller's market right now?"
- 💰 "Is this $725K listing fairly priced?"

**Real Estate Agent Tab:**
- 📋 "My listing at 1234 Oak St has been sitting for 30 days — what should I do?"
- 📈 "Give me a CMA for a 3-bed in Mueller"
- 🏗️ "What's the pipeline status for my Q2 deals?"
- 💵 "Price my new listing at 5678 Elm Drive"

## Architecture

```
┌─────────────────────────────────────────────────────┐
│  Angular Frontend (CloudFront + S3)                 │
│  - Chat interface with voice (Nova Sonic)           │
│  - Scenario cards for quick demos                   │
│  - Auto-generated visualizations                    │
└─────────────────────┬───────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────┐
│  Amazon Bedrock AgentCore Runtime                   │
│  ┌─────────────────────────────────────────────┐    │
│  │  RealtorOrchestratorAgent (Claude Sonnet)   │    │
│  │  Routes to specialist agents via A2A        │    │
│  └──────┬──────┬──────┬──────┬──────┬──────┬───┘    │
│         │      │      │      │      │      │        │
│  ┌──────▼┐ ┌──▼───┐ ┌▼────┐ ┌▼────┐ ┌▼───┐ ┌▼───┐ │
│  │Buyer  │ │Home  │ │Mkt  │ │Nbhd │ │List│ │Deal│  │
│  │Match  │ │Value │ │Intel│ │Exprt│ │Opt │ │Pipe│  │
│  └───────┘ └──────┘ └─────┘ └─────┘ └────┘ └────┘  │
└─────────────────────┬───────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────┐
│  Bedrock Knowledge Base (RAG)                       │
│  - Austin market data, listings, neighborhoods      │
│  - First-time buyer guides, valuation guides        │
│  - Investment analysis, recent sales comps          │
└─────────────────────────────────────────────────────┘
```

## Tech Stack

- **AI Runtime**: Amazon Bedrock AgentCore (container-based)
- **Foundation Model**: Claude Sonnet 4 (via Bedrock)
- **Knowledge Base**: Bedrock Knowledge Bases with RAG
- **Frontend**: Angular + AWS Amplify Auth (Cognito)
- **Voice**: Amazon Nova Sonic (real-time voice interface)
- **Infrastructure**: CloudFormation, S3, CloudFront, DynamoDB, Lambda
- **Agent Protocol**: A2A (Agent-to-Agent) for multi-agent coordination

## Project Structure

```
├── frontend/                    # Angular app (UI)
│   └── src/app/
│       ├── components/          # Chat, visualizations, agent management
│       └── services/            # Bedrock, Cognito, DynamoDB integrations
├── agentcore/deployment/
│   └── agent/
│       ├── handler.py           # Main agent handler
│       ├── agent-instructions-library/  # Agent prompts (8 real estate agents)
│       ├── agent_cards/         # A2A agent card definitions
│       └── shared/              # Utilities, memory, knowledge base helpers
├── configs/
│   └── tab-configurations.json  # UI tab/scenario definitions
├── sample-data/
│   └── knowledge-base-docs/     # Austin real estate RAG content
├── cloudformation/              # AWS infrastructure templates
├── lambda/                      # Supporting Lambda functions
└── scripts/                     # Deploy & config management scripts
```

## Prerequisites

- AWS Account with Bedrock access (Claude Sonnet, Nova Sonic)
- Node.js 18+
- Python 3.11+
- AWS CLI v2

## Deployment

### 1. Deploy infrastructure
```bash
aws cloudformation deploy \
  --template-file cloudformation/infrastructure-core.yml \
  --stack-name realtor-agents-core \
  --capabilities CAPABILITY_NAMED_IAM
```

### 2. Deploy the agent runtime
```bash
cd agentcore/deployment
./build_and_deploy.sh
```

### 3. Upload knowledge base docs
```bash
aws s3 sync sample-data/knowledge-base-docs/ s3://YOUR-KB-BUCKET/
```

### 4. Build and deploy frontend
```bash
cd frontend
npm install
ng build --configuration production
# Deploy build/ to S3 + CloudFront
```

### 5. Upload configs to DynamoDB
```bash
python scripts/upload_agent_configs_to_dynamodb.py
python scripts/upload_tab_configs_to_dynamodb.py
```

## Customizing

**Add a new agent:**
1. Write the instruction prompt in `agentcore/deployment/agent/agent-instructions-library/YourAgent.txt`
2. Create an agent card in `agent_cards/YourAgent.agent.card.json`
3. Add it to the orchestrator's `tool_agent_names` in DynamoDB config
4. Add scenarios in `configs/tab-configurations.json`
5. Redeploy

**Change the knowledge base content:**
Upload new markdown docs to the S3 bucket backing the Bedrock Knowledge Base. The agents use RAG to pull relevant context at query time.

## Cost Estimate

| Service | Usage | ~Cost/month |
|---------|-------|-------------|
| Bedrock (Claude Sonnet) | ~1000 queries | ~$15 |
| AgentCore Runtime | Always-on container | ~$50 |
| DynamoDB | On-demand | ~$1 |
| CloudFront + S3 | Static hosting | ~$1 |
| Cognito | Auth | Free tier |
| **Total** | | **~$65-70/month** |

## License

Apache 2.0 — See [LICENSE.txt](LICENSE.txt)
