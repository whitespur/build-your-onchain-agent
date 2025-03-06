# Wallets Monitor - Technical Architecture

## Project Overview

Wallets Monitor is an on-chain analytics system designed to track Solana blockchain token transactions and identify potential investment opportunities. The system monitors transactions from multiple wallets (referred to as "smart money" wallets) and alerts users when multiple tracked wallets buy the same token within a specific timeframe (6 hours).

## System Architecture

The architecture consists of several core components working together:

1. **Webhook API**: Receives transaction data from Helius.xyz webhook service
2. **Database**: Stores transaction data, wallet information, and token analysis
3. **Transaction Processing**: Parses and processes transaction data 
4. **Token Analysis**: Analyzes tokens bought by multiple wallets
5. **Notification System**: Sends formatted alerts to Telegram
6. **AI Analysis**: Uses DeepSeek LLM to summarize Twitter content related to tokens

### Technical Flow Diagram

```
┌───────────────┐      ┌────────────────┐      ┌─────────────────┐
│ Helius.xyz    │──────▶ Webhook API    │──────▶ Transaction     │
│ (Blockchain   │      │ (Next.js API)  │      │ Processing      │
│  Events)      │      └────────────────┘      │ (txParser &     │
└───────────────┘                              │  swapProcessor) │
                                               └────────┬────────┘
                                                        │
┌───────────────┐      ┌────────────────┐      ┌───────▼────────┐
│ Telegram Bot  │◀─────┤ AI Summary     │◀─────┤ Supabase       │
│ (Notifications)│      │ (DeepSeek LLM) │      │ Database       │
└───────────────┘      └────────────────┘      └───────┬────────┘
                                                       │
┌───────────────┐      ┌────────────────┐      ┌──────▼─────────┐
│ DexScreener   │◀─────┤ Token Analysis │◀─────┤ Realtime       │
│ & Twitter API │      │ (Multi-wallet  │      │ Monitoring     │
└───────────────┘      │  detection)    │      │ (Supabase      │
                       └────────────────┘      │  Realtime)     │
                                               └─────────────────┘
```

## Core Components

### 1. Database Schema

The system uses Supabase with two main tables:
- `txs`: Stores transaction data (wallet addresses, token addresses, amounts, timestamps)
- `wallets`: Maps wallet addresses to human-readable names for better identification

### 2. Webhook API (route.js)

- Receives transaction data from Helius.xyz webhooks
- Validates requests and processes transaction data
- Stores processed data in the Supabase database

### 3. Transaction Processing

Two parsers for processing transaction data:
- `swapProcessor.js`: Processes Helius swap event data
- `txParser.js`: Uses Shyft API for additional transaction parsing capabilities

### 4. Token Monitoring (strategy/index.js)

- Subscribes to Supabase realtime events for new transactions
- Identifies when multiple wallets buy the same token within 6 hours
- Checks token's market cap and age to filter out noise
- Triggers analysis and notification for qualifying tokens

### 5. Token Analysis (txsAnalyzer.js)

- Analyzes token transactions to extract meaningful metrics
- Calculates investment amounts, buy times, holding percentages
- Associates wallets with their known names when available
- Retrieves token price, supply, and market data from DexScreener

### 6. Notification System

- `messageTemplate.js`: Creates formatted Telegram messages with token data
- `telegram.js`: Sends notifications to configured Telegram channels
- Formats data with appropriate emojis and formatting for readability

### 7. AI Integration (aiSummary.js)

- Retrieves token-related tweets via Twitter API
- Uses DeepSeek LLM to analyze and summarize tweet content
- Provides narrative summaries and risk assessments in follow-up messages

## Technology Stack

- **Backend Framework**: Next.js
- **Database**: Supabase (PostgreSQL with realtime capabilities)
- **Blockchain Data**: 
  - Helius.xyz (Webhook & RPC)
  - Shyft API (Transaction parsing)
- **External APIs**:
  - DexScreener (Token info)
  - Twitter API (Social sentiment)
  - DeepSeek API (AI analysis)
- **Notifications**: Telegram Bot API
- **Deployment**: Vercel

## Key Features

1. **Multi-Wallet Detection**: Identifies tokens purchased by multiple tracked wallets
2. **Token Filtering**: Applies market cap and age filters to reduce noise
3. **Real-time Monitoring**: Uses Supabase realtime subscriptions for instant notifications
4. **Rich Analytics**: Provides detailed metrics on wallets' positions and behaviors
5. **AI-Enhanced Insights**: Summarizes social media sentiment and narratives using AI
6. **User-Friendly Alerts**: Formats data in readable, actionable Telegram messages

## Configuration Parameters

The system is highly configurable with parameters including:
- `MAX_AGE_DAYS`: Maximum token age for consideration (default: 7 days)
- `MIN_MARKET_CAP`: Minimum market cap threshold (default: $100,000)
- Time window for multi-wallet detection (6 hours)

## Monitoring Approach

The system follows these steps:
1. Capture transactions via webhooks
2. Store in database with Supabase
3. Monitor for new transactions in realtime
4. Detect when multiple wallets buy the same token
5. Filter tokens based on criteria (age, market cap)
6. Analyze token and transaction data
7. Generate and send notifications
8. Provide AI-powered sentiment analysis as follow-up 