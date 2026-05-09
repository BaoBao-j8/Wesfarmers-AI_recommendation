# Monitor Dashboard Component

## Overview

The **Monitor Dashboard** is a comprehensive React-based executive control panel for the OneIntelligence AI Platform. It provides real-time monitoring, AI orchestration, demand forecasting, product design capabilities, and cross-brand personalization across all Wesfarmers brands (Bunnings, Kmart, Target, Officeworks, Catch, Priceline, and API Health).

## Features

### 1. Command Centre
- Real-time KPIs: Group Revenue, Transactions, Inventory Health, AI Model Calls
- Cross-brand revenue trends visualization
- Brand performance cards with live metrics
- AI platform activity monitoring across all brands

### 2. Demand AI & Inventory
- AI-powered demand forecasting with confidence scores
- Brand-specific 30/60/90-day forecast horizons
- Inventory alerts with AI-generated reorder recommendations
- Approval workflow for stock management

### 3. Model Orchestration Hub
- Multi-vendor AI routing (Anthropic Claude, OpenAI GPT-4o, Google Vertex, Internal Model)
- Cost vs. Performance analysis
- Active routing rules per use case
- Live AI usage logs with latency tracking

### 4. AI Product Design Studio
- Generative product concept creation using Claude AI
- Brand-specific design briefs
- Real-time concept generation with 3 distinct variants
- Save-to-pipeline workflow for products

### 5. Shopping Agent
- Conversational AI assistant with cross-brand history awareness
- Real-time product recommendations
- Multi-brand context understanding
- OnePass loyalty integration

### 6. Cross-Brand Personalisation
- AI-generated customer segments (personas)
- Personalisation lift tracking by segment
- Active rules management
- AI Ethics governance audit trail

## Architecture

### Tech Stack
- **Frontend**: React 18
- **UI Components**: Recharts (data visualization)
- **Styling**: CSS-in-JS with glass-morphism design system
- **AI Integration**: Anthropic Claude API
- **Design System**: Wesfarmers navy/blue/green color palette

### Key Components

#### Layout Components
- `Sidebar`: Navigation with role-based access (Executive, Operator, Customer)
- `TopBar`: Page header with search and notifications

#### Page Components
- `CommandCentre()`: Executive dashboard
- `DemandAI()`: Inventory & forecasting
- `ModelOrchestration()`: AI vendor management
- `ProductDesignAI()`: Generative design studio
- `ShoppingAgent()`: Conversational commerce
- `Personalisation()`: Segment management

#### UI Utilities
- `KpiTile()`: Metric display cards
- `glassCard`: Reusable glass-morphism styling
- Helper functions: `fmtM()` (millions), `fmtK()` (thousands), `rand()` (random numbers)

### Design System

```javascript
const G = {
  navy: "#0A2240",           // Primary
  blue: "#005DAA",           // Secondary
  green: "#00A651",          // Accent
  glass: "rgba(255,255,255,0.72)",
  navBg: "rgba(10,34,64,0.96)",
  text: "#0A2240",
  textMid: "#3A5275",
  textLight: "#7A94B0",
  border: "rgba(0,93,170,0.12)",
  shadow: "0 8px 32px rgba(10,34,64,0.10), ...",
};
```

## Role-Based Access

| Role | Pages |
|------|-------|
| Executive / C-Suite | Command Centre, Demand AI, Model Orchestration, Personalisation |
| Brand Operator | Demand AI, Model Orchestration, Product Design AI, Personalisation |
| Customer | Shopping Agent |

## Data Flow

1. **Real-time Updates**: KPIs update every 2.5 seconds
2. **AI Integration**: Claude API for product generation and shopping assistance
3. **State Management**: React hooks (useState, useEffect, useRef, useCallback)
4. **Mock Data**: Generated functions for revenue, forecasts, vendor data, and inventory

## Usage

### Basic Integration

```jsx
import App from './Monitor-dashboard';

export default function Root() {
  return <App />;
}
```

### Required Dependencies

```json
{
  "react": "^18.0.0",
  "react-dom": "^18.0.0",
  "recharts": "^2.10.0",
  "lucide-react": "^latest"
}
```

### Environment Variables

```
REACT_APP_ANTHROPIC_API_KEY=your_api_key_here
```

## Customization

### Add a New Brand

Edit the `BRANDS` array:
```javascript
const BRANDS = [
  { id: "newbrand", name: "New Brand", color: "#FF0000", bg: "#FFF0F0", short: "NBR" },
  // ...
];
```

### Modify Color Scheme

Update the `G` (globals) object with your brand colors.

### Add New Routing Rules

Edit the routing rules in `ModelOrchestration()` component.

## Performance Considerations

- **Memoization**: Consider using `React.memo()` for expensive chart components
- **Data Limits**: Forecast data capped at 90 days
- **API Calls**: Product generation calls Anthropic API (be mindful of rate limits)
- **Chart Rendering**: Recharts ResponsiveContainer handles responsive sizing

## Known Limitations

1. Mock data generation uses random values (replace with real APIs)
2. Product generation requires valid Anthropic API key
3. Chat messages stored in local state (no persistence)
4. No backend integration for persistent storage

## Future Enhancements

- [ ] Real-time data streaming (WebSocket)
- [ ] Persistent database integration
- [ ] Advanced filtering and exports
- [ ] Multi-user collaboration
- [ ] Mobile responsive optimization
- [ ] Dark mode toggle
- [ ] Analytics export/reporting

## License

Part of the OneIntelligence Platform for Wesfarmers Group.
