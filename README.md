# AWS Meteo Frontend

Real-time agricultural risk monitoring platform. Interactive maps, climate risk analysis, and AI-powered insights for farm management.

![React](https://img.shields.io/badge/React-18-blue)
![TypeScript](https://img.shields.io/badge/TypeScript-5-blue)
![Vite](https://img.shields.io/badge/Vite-5-purple)
![Tailwind](https://img.shields.io/badge/Tailwind-3-cyan)
![Leaflet](https://img.shields.io/badge/Leaflet-Maps-green)

## Features

- 🗺️ **Interactive Maps** - Leaflet-based visualization with drawing tools
- 🌡️ **Risk Heatmaps** - Drought, frost, flood, and custom risk layers
- 📊 **Analytics Dashboard** - Real-time metrics and risk forecasting
- 🤖 **AI Assistant** - Context-aware recommendations
- 👥 **User Management** - Role-based access, profiles, preferences
- 📱 **Responsive** - Mobile-first design
- 🔐 **Secure Auth** - Supabase or AWS Cognito integration

## Quick Start

### Prerequisites
- Node.js 18+
- npm/yarn
- Backend: Supabase or custom API

### Installation

```bash
git clone <repo-url>
cd aws_meteo_frontend

npm install
cp .env.example .env.local
# Fill in your API credentials
npm run dev
```

Open `http://localhost:5173`

## Configuration

### Environment Variables

```env
# API
VITE_API_URL=https://your-api.com
VITE_API_TIMEOUT=30000

# Supabase (Primary)
VITE_SUPABASE_URL=https://xxx.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key

# AWS Cognito (Alternative)
# VITE_AWS_REGION=us-east-1
# VITE_COGNITO_USER_POOL_ID=xxx
# VITE_COGNITO_CLIENT_ID=xxx

# Features
VITE_USE_MOCK_DATA=false  # true for dev without backend
VITE_ENABLE_ANALYTICS=true
```

### Database Setup (Supabase)

Run this SQL in Supabase dashboard:

```sql
-- User profiles
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  email TEXT UNIQUE NOT NULL,
  full_name TEXT,
  company TEXT,
  avatar_url TEXT,
  role TEXT DEFAULT 'viewer',
  preferences JSONB DEFAULT '{}',
  onboarding JSONB DEFAULT '{"completed": false, "currentStep": 0}',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Farm parcels
CREATE TABLE parcels (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  area DECIMAL,
  geojson JSONB NOT NULL,
  color TEXT DEFAULT '#3b82f6',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Sub-sections of parcels
CREATE TABLE paddocks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  parcel_id UUID REFERENCES parcels(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  area DECIMAL,
  geojson JSONB NOT NULL,
  crop_type TEXT,
  irrigation_type TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Enable RLS
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE parcels ENABLE ROW LEVEL SECURITY;
ALTER TABLE paddocks ENABLE ROW LEVEL SECURITY;

-- Access policies
CREATE POLICY "Users see own profile" ON profiles FOR SELECT USING (auth.uid() = id);
CREATE POLICY "Users manage own parcels" ON parcels FOR ALL USING (auth.uid() = user_id);
CREATE POLICY "Users manage own paddocks" ON paddocks FOR ALL 
  USING (parcel_id IN (SELECT id FROM parcels WHERE user_id = auth.uid()));
```

## Project Structure

```
src/
├── components/        # React components
│   ├── auth/         # Login, register, auth guards
│   ├── map/          # Map, layers, drawing tools
│   ├── sidebar/      # Dashboard tabs (farm, risks, chat)
│   ├── landing/      # Public landing page
│   └── ui/           # shadcn/ui components
├── pages/            # Page routes
│   ├── Dashboard.tsx
│   ├── Landing.tsx
│   ├── Login.tsx
│   ├── Onboarding.tsx
│   └── UserProfile.tsx
├── hooks/            # Custom React hooks
├── services/         # API calls, auth
├── store/            # Zustand state management
├── types/            # TypeScript interfaces
└── utils/            # Helpers
```

## Development

```bash
# Start dev server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Linting
npm run lint

# Run tests
npm run test

# Docker build
npm run docker:build
npm run docker:run
```

## API Integration

Backend API expected endpoints:

```
POST   /auth/login              - User login
POST   /auth/signup             - Register new user
GET    /profiles/:id            - Get user profile
PUT    /profiles/:id            - Update profile
GET    /parcels                 - List user's parcels
POST   /parcels                 - Create parcel
GET    /parcels/:id             - Get parcel details
PUT    /parcels/:id             - Update parcel
DELETE /parcels/:id             - Delete parcel
POST   /parcels/:id/paddocks    - Add paddock to parcel
PUT    /paddocks/:id            - Update paddock
DELETE /paddocks/:id            - Delete paddock
GET    /risks/heatmap           - Get risk layer data
GET    /risks/forecast          - Forecast risks for parcel
POST   /chat/message            - Send message to AI
GET    /chat/history            - Get chat history
```

## Deployment

### AWS S3 + CloudFront

```bash
npm run build
aws s3 sync dist/ s3://your-bucket --delete
aws cloudfront create-invalidation --distribution-id XXXXX --paths "/*"
```

### AWS Amplify

1. Connect GitHub repo to AWS Amplify Console
2. Set environment variables
3. Deploy on push

### Environment in Production

Set these in your hosting environment:

```
VITE_API_URL=https://api.yourdomain.com
VITE_SUPABASE_URL=https://xxx.supabase.co
VITE_SUPABASE_ANON_KEY=xxxxx
VITE_USE_MOCK_DATA=false
```

## Tech Stack

- **Frontend**: React 18, TypeScript, Vite
- **Styling**: Tailwind CSS, Framer Motion
- **UI Components**: shadcn/ui, Radix UI
- **Maps**: Leaflet, React Leaflet, Leaflet Draw
- **State**: Zustand
- **Forms**: React Hook Form, Zod validation
- **Charts**: Recharts
- **Tables**: TanStack React Table
- **HTTP**: TanStack React Query
- **Auth**: Supabase or AWS Cognito
- **Database**: PostgreSQL (via Supabase)
- **Testing**: Vitest, Testing Library
- **Linting**: ESLint

## Contributing

1. Create feature branch: `git checkout -b feature/name`
2. Make changes and test
3. Push to branch
4. Create pull request

## License

Proprietary - Confidential

## Support

For questions, contact the development team.
