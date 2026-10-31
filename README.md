# SmartPool AI

SmartPool AI is an AI-powered personal and shared-finance web application built for the **PayPal AI Hackathon**.

It helps individuals, couples, and families understand where their money goes, capture financial activity with minimal effort, and make shared-money decisions with clear rules and confirmations.

PayPal is the financial backbone of the application: transactions, payouts, recurring payments, payment requests, and pool operations are designed around PayPal Sandbox integrations. The AI agent interprets financial information and helps users act on it, while the backend remains responsible for authorization, validation, and money-movement rules.

> **Hackathon demo:** SmartPool AI is a sandbox and demonstration project. It is not a custodial financial product and does not provide financial advice.

## The problem

People often do not know where their money goes because manually tracking every expense is tedious. Shared finances create an additional problem: it is difficult to keep a transparent record of who contributed, who requested money, and how shared expenses were approved.

SmartPool AI addresses three connected needs:

1. **Effortless capture** — record expenses by voice or scan a receipt.
2. **Useful interpretation** — categorize movements, visualize spending, detect recurring charges, and generate recommendations.
3. **Controlled action** — prepare payments, shared expenses, and transfers with explicit confirmation and server-side rules.

## Modes

The user selects the context in which the application should operate:

- **Personal** — personal movements, budgets, goals, subscriptions, and investments.
- **Couple** — shared and individual expenses, transfers between partners, and common goals.
- **Family** — allowances, children's money requests, individual expenses, and household support.

## Main sections

The application starts with a dashboard containing a monthly overview, alerts, shortcuts, and access to each financial section:

- Dashboard
- Movements
- Charts
- Categories
- Budgets and goals
- Subscriptions
- Family
- Pools
- Investments
- Summaries

The floating AI agent is available throughout the application.

## AI agent

The agent supports chat and browser voice input. It can also process a receipt image and return structured data for the user to review.

Planned agent capabilities include:

- Register an income or expense from a natural-language request.
- Extract merchant, amount, date, and category from a receipt.
- Generate a summary for a selected date range.
- Detect recurring payments and review subscription usage.
- Recommend potential spending cuts using a four-part priority matrix.
- Retrieve market data and explain an asset analysis without presenting it as financial advice.
- Create and manage shared pools.
- Prepare payment requests, bill splits, and transfers.
- Require confirmation before sensitive financial actions.

The AI agent prepares actions; it does not bypass backend authorization, pool rules, payment limits, or confirmation requirements.

## PayPal as the financial backbone

Every financial flow is designed to connect back to PayPal or to an explicitly enabled simulation mode.

| Financial flow | Planned PayPal integration |
|---|---|
| Import and register movements | Transaction Search API and webhooks |
| Recurring payments and allowances | Subscriptions API and Payouts API |
| Payment requests and collections | Orders API or Invoicing API |
| Pool contributions and withdrawals | Orders API and Payouts API |
| Expired requests | Refund flow where applicable |

PayPal Sandbox accounts will be used for the hackathon demonstration. The application also includes a simulated-payment path so the team can develop and demo the product when a Sandbox capability is unavailable or when an external call is not appropriate.

## Shared pools

A SmartPool is a private, invitation-only shared fund. Each member's contribution remains attributable to that member, and shared expenses are approved through a majority vote.

Core pool concepts:

- Individual contribution history.
- Individual withdrawable balance.
- Shared expense requests with a destination, amount, requester, and description.
- Majority approval before an expense is paid.
- Equal expense allocation among pool members.
- Notifications when a request opens and reminders after 24 hours.
- Expiration after 48 hours without the required majority.
- Reserve rules that protect scheduled expenses before a withdrawal.
- Configurable thresholds and approval requirements governed by the pool.

For the Sandbox demo, a business test account can represent a pool. The backend is the only component that should order payments from that account. If separate Sandbox business accounts are not available, a shared test account and an application-level ledger can represent the pools for demonstration purposes.

Public crowdfunding pools are outside the scope of this project.

## Security and payment controls

Financial actions must not be executed from a single unverified AI instruction.

The intended transfer flow is:

1. The agent explains the destination, amount, and reason.
2. The user confirms the proposed action.
3. The agent asks for a second confirmation.
4. The backend validates authorization, limits, expiration, and business rules.
5. The user provides a PIN.
6. The backend executes the PayPal payment or the simulated payment.
7. The result is recorded as a movement.

PINs must be stored as secure hashes, not plaintext. Children and adults use their own confirmation flow, and adult-controlled limits apply to family payments. These rules are enforced by the server, not by the model.

## Architecture

```text
                         Vercel
                 React + Vite + TypeScript
                              |
                         HTTPS / JSON
                              |
                              v
                         Render
                     FastAPI + Python
                              |
             +----------------+----------------+
             |                |                |
             v                v                v
        Supabase          PayPal Sandbox     Gemini API
        PostgreSQL        APIs + webhooks    AI services
```

### Frontend

- React
- Vite
- TypeScript
- Tailwind CSS
- React Router
- AG Grid Community
- AG Charts Community
- Web Speech API for browser voice input

The frontend is a private dashboard application hosted on Vercel. It communicates with FastAPI through a single HTTP API and does not contain PayPal secrets or server-side payment rules.

### Backend

- Python 3.11
- FastAPI
- Pydantic
- SQLAlchemy 2
- Alembic
- `httpx` for PayPal REST calls
- Gemini through the Google GenAI SDK
- `yfinance` and fallback data for market analysis

The backend is hosted on Render. It owns authentication, data validation, database access, AI tool execution, PayPal communication, webhooks, pool rules, transfer confirmation, scheduled tasks, and notifications.

### Database

Supabase provides the PostgreSQL database. The planned data model includes:

- Users
- Groups and household relationships
- Movements
- Categories
- Budgets and goals
- Subscriptions
- Pools
- Pool contributions
- Expense requests
- Votes
- Pending transfers

### API testing

Postman, Bruno, or an equivalent API client can be used to test the FastAPI contract and PayPal Sandbox flows. A tunnel such as Cloudflare Tunnel or ngrok may be used during local development to receive PayPal webhooks.

## Repository structure

```text
.
├── backend/
│   ├── app/
│   │   ├── agents/          # AI agent and callable tools
│   │   ├── api/             # FastAPI routes and dependencies
│   │   ├── core/            # Configuration, security, and exceptions
│   │   ├── db/              # SQLAlchemy database and models
│   │   ├── schemas/         # Pydantic request and response schemas
│   │   ├── services/        # PayPal, AI, market, email, and simulation services
│   │   └── tasks/           # Scheduled jobs and expiration checks
│   ├── tests/               # Backend tests
│   ├── alembic/             # Database migrations
│   ├── alembic.ini
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── components/      # Reusable UI components
│   │   ├── hooks/           # Frontend data and state hooks
│   │   ├── lib/             # API client and shared constants
│   │   ├── pages/           # Application screens
│   │   ├── services/        # API-facing frontend services
│   │   ├── types/           # TypeScript API types
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── package.json
│   └── vite.config.ts
├── docs/                    # API, data model, PayPal, and demo documentation
├── infra/                   # Render and local container configuration
├── LICENSE
└── README.md
```

## Local development

The repository is currently being scaffolded. The commands below describe the intended development workflow once the application dependencies and entry points are implemented.

### Backend

```bash
cd backend
python -m venv .venv

# Windows
.venv\Scripts\activate

pip install -r requirements.txt
uvicorn app.main:app --reload
```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

### Environment variables

Secrets belong in local environment configuration or the deployment provider, never in the repository.

Expected backend variables include:

```text
DATABASE_URL=
PAYPAL_CLIENT_ID=
PAYPAL_CLIENT_SECRET=
PAYPAL_BASE_URL=
PAYPAL_WEBHOOK_ID=
GEMINI_API_KEY=
JWT_SECRET=
FRONTEND_URL=
SIMULATION_MODE=
```

## Demonstration flow

The intended hackathon demo is shorter than three minutes:

1. Show the problem of untracked personal and shared spending.
2. Record an expense with voice input and show it in the movement table and charts.
3. Scan a receipt and show the extracted movement.
4. Generate a monthly summary and review a subscription recommendation.
5. In family mode, show a child's money request, adult approval, second confirmation, and PIN-protected payment.
6. In pool mode, show contributions, an expense request, majority voting, and the resulting payment.
7. Briefly show an explained market quote and recommendation.
8. Close with the role of PayPal and AI in the product.

## Hackathon scope and limitations

SmartPool AI is intentionally presented as a sandbox demonstration:

- PayPal accounts and payments are test transactions.
- Some demo history may combine Sandbox transactions with clearly identified sample data.
- Pool balances are an application-level demonstration of shared accounting.
- The project does not claim to custody real customer funds.
- Market analysis is informational and is not financial advice.
- The agent cannot cancel external subscriptions that were not created by the application; it can explain how the user can cancel them.

If scope must be reduced for the hackathon deadline, market analysis is the first feature to defer, followed by receipt scanning. The core milestone is a PayPal movement appearing in the categorized movements and charts, because the rest of the financial experience depends on that foundation.

## Hackathon timeline

- **November 4, 2026:** Target milestone for a PayPal movement to appear categorized in the charts.
- **November 8, 2026:** Feature freeze.
- **November 9–11, 2026:** Demo video, README, license, and Devpost delivery preparation.
- **November 12, 2026:** Final submission target.

The official hackathon page should be checked again before submission to confirm the final deadline and requirements.

## Team workflow

- One repository.
- One branch per contributor.
- Short daily synchronization.
- Fast review before merging.
- Each contributor tests their work end to end.
- PayPal Sandbox capabilities are verified early, especially Transaction Search, Subscriptions, Orders, Payouts, webhooks, and business test accounts.

## License

This project is released under the license included in [`LICENSE`](LICENSE).