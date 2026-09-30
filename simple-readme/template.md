# README template

Copy and fill. Delete optional sections not used.

```markdown
# Project Name

Brief description. Stack and critical external services or sibling repos.

## Requirements

- **Runtime**, version constraint
- **Service name**, what it is used for
- **Optional integration**, optional, when needed for specific features

## Quick Setup

### 1. Install dependencies

\`\`\`bash
npm install
# or: pip install -r requirements.txt
\`\`\`

### 2. Configure environment

Copy `.env.example` to `.env` and fill in the values:

\`\`\`bash
cp .env.example .env
\`\`\`

| Variable | Required | Description |
|----------|----------|-------------|
| `EXAMPLE_VAR` | Yes | What it does |
| `OPTIONAL_VAR` | No | Optional. Default `value` |

### 3. Database

Run [setup.sql](setup.sql) in the SQL editor. Create at least one auth user if applicable.

### 4. Run development server

\`\`\`bash
npm run dev
\`\`\`

Open [http://localhost:PORT](http://localhost:PORT).

### 5. Run production server

\`\`\`bash
npm run build
npm run start
\`\`\`

Or with Docker Compose:

\`\`\`bash
docker compose up --build -d
\`\`\`

### 6. Done

- API docs: `http://localhost:PORT/docs`
- Key note (build args, platform port, etc.)

<!-- Optional: Scripts -->

## Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Development server |
| `npm run build` | Production build |
| `npm run lint` | Lint |

<!-- Optional: Deployment -->

## Deployment

How and where this deploys (e.g. Render on push to `main`).

<!-- Optional: Security -->

## Security

### Authentication
- How users sign in
- Token/cookie model

### Other
- Rate limits, headers, bot protection as relevant

<!-- Optional: Integration -->

## Integration

How clients should call this service.

\`\`\`javascript
const response = await fetch(`${API_BASE_URL}/endpoint`, {
  method: 'GET',
  credentials: 'include',
});
\`\`\`

<!-- Optional: Logging -->

## Logging

Where to view logs and useful local flags.

<!-- Optional: Routes -->

## Routes

| Path | Description |
|------|-------------|
| `/home` | Landing page |

## Structure

\`\`\`
project-name/
├── src/
│   ├── components/       # UI components
│   ├── services/         # API clients
│   └── main entry file
├── .env.example
└── package.json
\`\`\`
```

## Section order (default)

1. Title + intro
2. Requirements
3. Quick Setup
4. Optional sections (Scripts, Deployment, Security, Integration, Logging, Routes, Features)
5. Structure
