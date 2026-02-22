# ISO Middleware - UI Components Guide

**Last Updated**: February 15, 2026  
**Framework**: Next.js 14 with React 18 & TypeScript  
**Styling**: Tailwind CSS  
**Icons**: Lucide React  

## Table of Contents
1. [Architecture Overview](#architecture-overview)
2. [Global Styles](#global-styles)
3. [Layout Components](#layout-components)
4. [Project Management Components](#project-management-components)
5. [API & Authentication Components](#api--authentication-components)
6. [AI Agent Components](#ai-agent-components)
7. [Assistant Components](#assistant-components)
8. [Common Patterns](#common-patterns)

---

## Architecture Overview

### Technology Stack
```json
{
  "framework": "Next.js 14.2.5",
  "ui": "React 18.3.1",
  "styling": "Tailwind CSS 3.4.3",
  "icons": "Lucide React 0.441.0",
  "blockchain": "Ethers.js 6.16.0",
  "language": "TypeScript 5.4.5"
}
```

### Color Palette
```css
/* Base Colors (Slate-based) */
--bg-start: #f8fafc;     /* slate-50 */
--bg-end: #ffffff;       /* white */
--text: #0f172a;         /* slate-900 */

/* Primary Actions */
bg-slate-900             /* Dark buttons, active states */
bg-blue-600              /* Primary actions */
bg-green-600             /* Success states */
bg-red-600               /* Danger/Delete actions */

/* Status Colors */
bg-emerald-50            /* Success backgrounds */
bg-blue-50               /* Info backgrounds */
bg-purple-50             /* Special features */
```

---

## Global Styles

### Location
`web-alt/app/globals.css`

### Custom Classes

#### `.card`
```css
@apply bg-white rounded-xl shadow-sm border border-slate-200;
```
**Usage**: Main container for panels and sections
**Example**:
```tsx
<div className="card p-4 space-y-3">
  {/* Content */}
</div>
```

#### `.badge`
```css
@apply inline-flex items-center rounded-full bg-slate-100 px-2 py-0.5 text-[11px] text-slate-600;
```
**Usage**: Small status indicators and labels
**Example**:
```tsx
<span className="badge">Active</span>
```

### Layout Gradient
```css
body {
  background: linear-gradient(to bottom, var(--bg-start), var(--bg-end));
}
```

---

## Layout Components

### 1. RootLayout
**File**: `web-alt/app/layout.tsx`

**Purpose**: Main application wrapper with navigation and gradient background

**Structure**:
```tsx
<html lang="en" className="h-full">
  <body className="min-h-screen bg-gradient-to-b from-slate-50 to-white">
    <TopNavigation />
    <main className="max-w-7xl mx-auto px-4 py-6">
      {children}
    </main>
  </body>
</html>
```

**Metadata**:
- Title: "ISO Middleware — Alternative UI"
- Description: "Next.js UI with persistent side AI assistant for ISO 20022 Middleware"

**Configuration**:
```typescript
export const dynamic = "force-dynamic";
export const revalidate = false;
```

---

### 2. TopNavigation
**File**: `web-alt/components/TopNavigation.tsx`

**Purpose**: Sticky top navigation bar with tabbed interface

**Features**:
- 4 main navigation tabs
- Responsive design (desktop + mobile)
- Active state highlighting
- API docs link

**Navigation Tabs**:
```typescript
const tabs = [
  { path: "/", label: "Receipts & Data", icon: Receipt },
  { path: "/operations", label: "Operations", icon: Workflow },
  { path: "/settings", label: "Settings", icon: Settings },
  { path: "/agents", label: "AI Agents", icon: Bot },
];
```

**Styling**:
```tsx
// Desktop Navigation
className={`
  inline-flex items-center gap-2 px-4 py-2 rounded-lg text-sm font-medium
  ${active 
    ? "bg-slate-900 text-white" 
    : "text-slate-600 hover:bg-slate-100 hover:text-slate-900"
  }
`}

// Mobile Navigation (hidden md:flex)
className={`
  flex-1 flex flex-col items-center gap-1 px-2 py-2 rounded-lg text-xs
  ${active ? "bg-slate-900 text-white" : "text-slate-600 hover:bg-slate-100"}
`}
```

**Example**:
```tsx
<TopNavigation />
// Renders sticky navigation at top with gradient white background
```

---

## Project Management Components

### 3. ProjectsPanel
**File**: `web-alt/components/ProjectsPanel.tsx`

**Purpose**: Wallet connection, SIWE authentication, and project management

**Key Features**:
1. **Wallet Connection** (MetaMask/EIP-1193)
2. **SIWE (Sign-In with Ethereum)** authentication
3. **Project Registration** with API key generation
4. **Project Switching** between multiple projects
5. **Session Management** (logout)

**State Management**:
```typescript
const [store, setStore] = useState<StorePublic>({ projects: [] });
const [projectName, setProjectName] = useState("My Project");
const [lastKey, setLastKey] = useState<string | null>(null);
const [loading, setLoading] = useState(false);
const [error, setError] = useState<string | null>(null);
```

**SIWE Message Format** (EIP-4361):
```typescript
function buildSiweMessage(args: {
  domain: string;
  address: string;
  nonce: string;
  chainId: number;
  uri: string;
}): string {
  return (
    `${args.domain} wants you to sign in with your Ethereum account:\n` +
    `${args.address}\n\n` +
    `URI: ${args.uri}\n` +
    `Version: 1\n` +
    `Chain ID: ${args.chainId}\n` +
    `Nonce: ${args.nonce}\n` +
    `Issued At: ${new Date().toISOString()}\n`
  );
}
```

**UI Elements**:

#### Active Project Display
```tsx
<div className="rounded border p-3 bg-white/60">
  <div className="text-xs text-slate-600">Active</div>
  <div className="text-sm font-medium">{active.name}</div>
  <div className="text-xs text-slate-600">{active.owner_wallet}</div>
</div>
```

#### Connect Button
```tsx
<button className="inline-flex items-center bg-slate-900 text-white rounded px-3 py-2">
  <Wallet className="h-4 w-4 mr-2" />
  Connect + Register
</button>
```

#### API Key Display (Success State)
```tsx
<div className="rounded border p-3 bg-emerald-50">
  <div className="text-xs text-emerald-800 font-medium flex items-center gap-2">
    <CheckCircle2 className="h-4 w-4" />
    API key returned (copy now; stored server-side)
  </div>
  <div className="mt-2 flex gap-2">
    <input className="flex-1 font-mono text-xs" value={lastKey} readOnly />
    <button onClick={copyToClipboard}>Copy</button>
  </div>
</div>
```

#### Project List
```tsx
<div className="space-y-2">
  {store.projects.map((p) => (
    <button
      className={`flex-1 rounded px-2 py-2 border 
        ${isActive ? "bg-slate-900 text-white" : "bg-white hover:bg-slate-50"}`}
      onClick={() => setActive(p.id)}
    >
      <div className="text-sm font-medium">{p.name}</div>
      <div className="text-xs">{p.owner_wallet}</div>
    </button>
  ))}
</div>
```

**API Routes**:
- `GET /api/store/projects` - List projects
- `POST /api/store/register` - Register new project with SIWE
- `POST /api/store/projects/active` - Set active project
- `DELETE /api/store/projects/:id` - Remove project
- `POST /api/store/logout` - Clear session

---

### 4. ProjectsOverviewPanel
**File**: `web-alt/components/ProjectsOverviewPanel.tsx`

**Purpose**: Display project details and statistics (implementation varies)

**Common Patterns**:
- Project metrics display
- Activity timeline
- Quick actions

---

### 5. ProjectConfigPanel
**File**: `web-alt/components/ProjectConfigPanel.tsx`

**Purpose**: Project configuration and settings

**Features**:
- Project metadata editing
- Configuration options
- Integration settings

---

## API & Authentication Components

### 6. APIKeysPanel
**File**: `web-alt/components/APIKeysPanel.tsx`

**Purpose**: Create, list, and revoke API keys for the active project

**Dependencies**:
```typescript
import { useProjectStore } from "../lib/client/useProjectStore";
```

**State**:
```typescript
const [keys, setKeys] = useState<APIKeyInfo[]>([]);
const [label, setLabel] = useState("dev-key");
const [lastCreatedKey, setLastCreatedKey] = useState<string | null>(null);
```

**Key Type**:
```typescript
type APIKeyInfo = {
  id: string;
  label: string;
  role: string;
  project_id?: string | null;
  created_at: string;
  revoked_at?: string | null;
};
```

**UI Components**:

#### Create Key Form
```tsx
<div className="flex gap-2">
  <input 
    className="flex-1 border rounded px-3 py-2" 
    value={label} 
    onChange={(e) => setLabel(e.target.value)} 
  />
  <button className="bg-slate-900 text-white rounded px-3 py-2">
    <PlusCircle className="h-4 w-4 mr-2" />
    Create
  </button>
</div>
```

#### New Key Display (One-time view)
```tsx
<div className="rounded border p-3 bg-emerald-50">
  <div className="text-xs text-emerald-800 font-medium">
    New API key (copy now; it will not be shown again)
  </div>
  <div className="mt-2 flex gap-2">
    <input className="flex-1 font-mono text-xs" value={lastCreatedKey} readOnly />
    <button onClick={copyToClipboard}>
      <Copy className="h-4 w-4" />
    </button>
  </div>
</div>
```

#### Key List Item
```tsx
<div className="flex items-center justify-between gap-2 rounded border p-2 bg-white">
  <div className="min-w-0">
    <div className="text-sm font-medium truncate">{k.label}</div>
    <div className="text-xs text-slate-600 truncate">
      role={k.role} • created={k.created_at}
      {revoked ? ` • revoked=${k.revoked_at}` : ""}
    </div>
  </div>
  <button 
    onClick={() => revokeKey(k.id)} 
    disabled={revoked}
  >
    <Trash2 className="h-4 w-4" />
  </button>
</div>
```

**API Integration**:
- Keys returned via `X-API-Key` header (one-time only)
- All calls proxied through `/api/proxy` to protect secrets
- Automatic integration with `ConnectionStringGenerator`

**Security Note**:
```tsx
<div className="text-xs text-slate-500">
  Keys are created by calling /v1/auth/api-keys through /api/proxy 
  so the active project's cookie-auth key is used.
  The raw key secret is returned only once in the X-API-Key response header.
</div>
```

---

### 7. ConnectionStringGenerator
**File**: `web-alt/components/ConnectionStringGenerator.tsx`

**Purpose**: Generate connection strings in multiple formats for SDK integration

**Props**:
```typescript
interface Props {
  apiKey: string;
  apiUrl?: string;
  projectName?: string;
}
```

**Supported Formats**:
```typescript
type Format = "env" | "json" | "uri" | "shell" | "python" | "typescript";
```

**Output Examples**:

#### ENV Format
```bash
# .env file for ISO Middleware integration
ISO_MIDDLEWARE_URL=http://127.0.0.1:8000
ISO_MIDDLEWARE_API_KEY=iso_key_abc123...
ISO_PROJECT_NAME=My Project
```

#### JSON Format
```json
{
  "url": "http://127.0.0.1:8000",
  "apiKey": "iso_key_abc123...",
  "project": "My Project"
}
```

#### Connection URI
```
iso-mw://iso_key_abc123...@127.0.0.1:8000?project=My%20Project
```

#### Shell Export
```bash
export ISO_MIDDLEWARE_URL="http://127.0.0.1:8000"
export ISO_MIDDLEWARE_API_KEY="iso_key_abc123..."
export ISO_PROJECT_NAME="My Project"
```

#### Python SDK Example
```python
from iso_middleware_sdk import ISOClient

client = ISOClient(
    base_url="http://127.0.0.1:8000",
    api_key="iso_key_abc123..."
)

# Example: List receipts
receipts = client.list_receipts(scope="mine")
print(f"Found {receipts['total']} receipts")
```

#### TypeScript SDK Example
```typescript
import IsoMiddlewareClient from "iso-middleware-sdk";

const client = new IsoMiddlewareClient({
  baseUrl: "http://127.0.0.1:8000",
  apiKey: "iso_key_abc123..."
});

// Example: List receipts
const receipts = await client.listReceipts({ scope: "mine" });
console.log(`Found ${receipts.total} receipts`);
```

**UI Styling**:
```tsx
<div className="rounded border p-3 bg-emerald-50 space-y-3">
  {/* Format selector */}
  <select className="w-full border border-emerald-200 rounded px-3 py-2">
    {formats.map(f => <option key={f.value}>{f.label}</option>)}
  </select>
  
  {/* Code display */}
  <pre className="text-xs bg-slate-950 text-slate-100 p-3 rounded-lg font-mono">
    {generateString()}
  </pre>
  
  {/* Copy button (absolute positioned) */}
  <button className="absolute top-2 right-2 bg-slate-800/80 text-white">
    <Copy className="h-3 w-3" />
    {copied ? "Copied!" : "Copy"}
  </button>
</div>
```

**Features**:
- ✅ One-click copy to clipboard
- ✅ 6 format options
- ✅ QR Code placeholder (coming soon)
- ✅ Syntax-highlighted code display
- ✅ Real SDK examples

---

## AI Agent Components

### 8. AgentsList
**File**: `web-alt/components/agents/AgentsList.tsx`

**Purpose**: Sidebar list of AI agents with creation interface

**Agent Type**:
```typescript
interface Agent {
  id: string;
  name: string;
  wallet_address: string;
  xmtp_address?: string;
  status: string;
  created_at: string;
}
```

**UI Sections**:

#### Header with Actions
```tsx
<div className="flex items-center justify-between mb-4">
  <h2 className="text-lg font-bold">AI Agents</h2>
  <div className="flex gap-2">
    <button className="p-2 bg-blue-600 text-white rounded-lg">
      <Zap className="h-4 w-4" /> {/* Template button */}
    </button>
    <button className="p-2 bg-slate-900 text-white rounded-lg">
      <Plus className="h-4 w-4" /> {/* Create button */}
    </button>
  </div>
</div>
```

#### Quick Guide (Dismissible)
```tsx
<div className="p-3 bg-blue-50 border border-blue-200 rounded-lg text-sm">
  <div className="flex justify-between items-start mb-2">
    <div className="font-semibold text-blue-900">Quick Guide</div>
    <button onClick={onHideGuide}>
      <X className="h-3 w-3" />
    </button>
  </div>
  <div className="text-blue-800 space-y-1">
    <div>1️⃣ Create agent (+ button)</div>
    <div>2️⃣ Configure AI (optional)</div>
    <div>3️⃣ Download agent package</div>
    <div>4️⃣ Run: npm install && npm start</div>
  </div>
</div>
```

#### New Agent Form
```tsx
<form onSubmit={handleSubmit}>
  <input name="name" placeholder="Agent Name (e.g., My ISO Agent)" required />
  <input name="wallet" placeholder="Wallet Address (0x...)" required />
  <input name="xmtp" placeholder="XMTP Address (optional)" />
  
  <div className="flex gap-2">
    <button type="submit" className="flex-1 bg-slate-900 text-white">
      Create
    </button>
    <button type="button" className="bg-slate-200">
      Cancel
    </button>
  </div>
</form>
```

#### Agent List Item
```tsx
<div 
  onClick={() => onSelectAgent(agent)}
  className={`p-3 rounded-lg cursor-pointer ${
    selectedAgent?.id === agent.id
      ? "bg-slate-900 text-white"
      : "bg-white hover:bg-slate-100"
  }`}
>
  <div className="flex items-start gap-2">
    <Bot className="h-5 w-5 mt-0.5" />
    <div className="flex-1 min-w-0">
      <div className="font-semibold truncate">{agent.name}</div>
      <div className="text-xs opacity-70 truncate">
        {agent.wallet_address.slice(0, 8)}...{agent.wallet_address.slice(-6)}
      </div>
      <div className="text-xs opacity-60 mt-1">
        {agent.status === "active" ? (
          <>
            <span className="w-2 h-2 bg-green-500 rounded-full animate-pulse"></span>
            Online
          </>
        ) : (
          <>
            <span className="w-2 h-2 bg-slate-400 rounded-full"></span>
            Offline
          </>
        )}
      </div>
    </div>
  </div>
</div>
```

**Layout**:
- Width: `w-80` (fixed sidebar)
- Background: `bg-slate-50`
- Overflow: `overflow-y-auto`

---

### 9. AgentDetails
**File**: `web-alt/components/agents/AgentDetails.tsx`

**Purpose**: Detailed view and management of individual agent

**Props**:
```typescript
interface AgentDetailsProps {
  agent: Agent;
  copiedAddress: string;
  onCopy: (text: string, label: string) => void;
  onEdit: () => void;
  onTest: () => void;
  onShowQR: () => void;
  onClone: () => void;
  onDelete: () => void;
  onDownload: () => void;
  API_BASE: string;
}
```

**UI Sections**:

#### Deployment Instructions
```tsx
<div className="bg-blue-50 border border-blue-200 rounded-lg p-4">
  <div className="flex items-start gap-2">
    <HelpCircle className="h-5 w-5 text-blue-600" />
    <div className="text-sm text-blue-900">
      <div className="font-semibold mb-1">How to Deploy Your Agent</div>
      <ol className="space-y-1 ml-4 list-decimal">
        <li>Click "Download Agent" to get your personalized agent package</li>
        <li>Extract the ZIP and edit .env with your wallet private key</li>
        <li>Run: <code>npm install && npm start</code></li>
        <li>Your agent will listen for XMTP messages and handle payments automatically</li>
      </ol>
    </div>
  </div>
</div>
```

#### Quick Action Cards (3-column grid)
```tsx
<div className="grid grid-cols-3 gap-3">
  <button className="p-4 bg-white border hover:border-blue-500 hover:bg-blue-50">
    <Send className="h-5 w-5 text-blue-600 mb-2" />
    <div className="font-semibold text-sm">Test Agent</div>
    <div className="text-xs text-slate-600">Send test message</div>
  </button>
  
  <button className="p-4 bg-white border hover:border-purple-500 hover:bg-purple-50">
    <QrCode className="h-5 w-5 text-purple-600 mb-2" />
    <div className="font-semibold text-sm">QR Code</div>
    <div className="text-xs text-slate-600">Share agent info</div>
  </button>
  
  <button className="p-4 bg-white border hover:border-green-500 hover:bg-green-50">
    <Copy className="h-5 w-5 text-green-600 mb-2" />
    <div className="font-semibold text-sm">Clone Agent</div>
    <div className="text-xs text-slate-600">Duplicate config</div>
  </button>
</div>
```

#### Agent Information Card
```tsx
<div className="bg-white border rounded-lg p-6">
  <div className="flex justify-between items-start mb-4">
    <h3 className="text-lg font-bold">Agent Information</h3>
    <button onClick={onEdit}>
      <Edit2 className="h-4 w-4" />
    </button>
  </div>
  
  <div className="space-y-3">
    {/* Name */}
    <div>
      <div className="text-sm text-slate-600">Name</div>
      <div className="font-semibold">{agent.name}</div>
    </div>
    
    {/* Wallet Address with Copy */}
    <div>
      <div className="text-sm text-slate-600 mb-1">Wallet Address</div>
      <div className="flex items-center gap-2">
        <span className="font-mono text-sm">{agent.wallet_address}</span>
        <button onClick={() => onCopy(agent.wallet_address, "wallet")}>
          {copiedAddress === "wallet" ? (
            <Check className="h-4 w-4 text-green-600" />
          ) : (
            <Copy className="h-4 w-4 text-slate-400" />
          )}
        </button>
      </div>
      <div className="text-xs text-slate-500 mt-1">
        This wallet will make x402 micropayments
      </div>
    </div>
    
    {/* Status with Indicator */}
    <div>
      <div className="text-sm text-slate-600">Status</div>
      <div className="flex items-center gap-2 mt-1">
        <span className={`w-2 h-2 rounded-full ${
          agent.status === "active" 
            ? "bg-green-500 animate-pulse" 
            : "bg-slate-400"
        }`}></span>
        <span className="font-semibold">
          {agent.status === "active" ? "Online" : "Offline"}
        </span>
      </div>
    </div>
  </div>
</div>
```

#### Quick Deploy Buttons
```tsx
<div className="grid grid-cols-2 gap-2">
  <a 
    href="https://railway.app/new"
    className="px-4 py-2 bg-purple-600 text-white rounded flex items-center gap-2"
  >
    <Rocket className="h-4 w-4" />
    Deploy to Railway
    <ExternalLink className="h-3 w-3" />
  </a>
  
  <a 
    href="https://heroku.com/deploy"
    className="px-4 py-2 bg-indigo-600 text-white rounded flex items-center gap-2"
  >
    <Rocket className="h-4 w-4" />
    Deploy to Heroku
    <ExternalLink className="h-3 w-3" />
  </a>
</div>
```

#### Primary Actions
```tsx
<div className="mt-6 flex gap-2">
  <button 
    onClick={onDownload}
    className="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
  >
    <Download className="h-4 w-4" />
    Download Agent
  </button>
  
  <button 
    onClick={onDelete}
    className="px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700"
  >
    Delete Agent
  </button>
</div>
```

**Features**:
- ✅ Copy-to-clipboard for addresses
- ✅ Visual status indicators
- ✅ One-click deployment links
- ✅ Agent package download
- ✅ Test messaging interface

---

### 10. Additional Agent Components

#### AgentActivity.tsx
- Activity timeline
- Message history
- Transaction logs

#### AgentAnalytics.tsx
- Usage statistics
- Performance metrics
- Charts and graphs

#### AgentAISettings.tsx
- AI model configuration
- Provider selection
- Temperature/parameters

#### AgentPricing.tsx
- Payment configuration
- Pricing tiers
- Fee settings

#### AgentRevenue.tsx
- Revenue tracking
- Payment history
- Analytics

#### AgentAnchoring.tsx
- Blockchain anchoring settings
- Evidence anchoring
- Chain configuration

#### AgentModals.tsx
- QR code modal
- Test message modal
- Confirmation dialogs

#### AgentChat.tsx
- Live chat interface
- Message composition
- Real-time updates

---

## Assistant Components

### 11. AssistantPanel
**File**: `web-alt/components/AssistantPanel.tsx`

**Purpose**: Interactive AI assistant with scoped permissions for exploring receipts and SDKs

**Dependencies**:
```typescript
import { aiAssist, listReceipts, getAIStatus } from "../lib/api";
import { useProjectStore } from "../lib/client/useProjectStore";
```

**State Management**:
```typescript
const [sessionId, setSessionId] = useState<string>("");
const [history, setHistory] = useState<Message[]>([]);
const [msg, setMsg] = useState("");

// Safety toggles
const [allowReadReceipts, setAllowReadReceipts] = useState(false);
const [allowReadArtifacts, setAllowReadArtifacts] = useState(false);
const [allowConfigChanges, setAllowConfigChanges] = useState(false);
const [allowedIds, setAllowedIds] = useState<string[]>([]);

// Filters
const [statusFilter, setStatusFilter] = useState<string>("");
const [chainFilter, setChainFilter] = useState<string>("");
```

**AI Status Badge**:
```tsx
{aiStatus && (
  <span className={`text-xs px-2 py-0.5 rounded-full ${
    aiStatus.enabled 
      ? "bg-green-100 text-green-800" 
      : "bg-gray-100 text-gray-600"
  }`}>
    {aiStatus.enabled ? (
      <>
        <Sparkles className="h-3 w-3 inline mr-1" />
        {aiStatus.provider}
      </>
    ) : (
      "Basic"
    )}
  </span>
)}
```

**Safety & Scope Panel**:
```tsx
<div className="rounded-lg border p-3 bg-white/70 space-y-2">
  <div className="text-xs font-medium text-slate-600">Safety & Scope</div>
  
  {/* Permission Checkboxes */}
  <label className="flex items-center gap-2">
    <input 
      type="checkbox" 
      className="accent-slate-900" 
      checked={allowReadReceipts}
      onChange={(e) => setAllowReadReceipts(e.target.checked)}
    />
    Allow reading receipts
  </label>
  
  {/* Filter Controls (when receipts enabled) */}
  {allowReadReceipts && (
    <div className="grid grid-cols-2 gap-2 pl-5">
      <div>
        <label className="text-xs text-slate-600">Status</label>
        <select className="w-full border rounded px-2 py-1">
          <option value="">(any)</option>
          <option value="pending">pending</option>
          <option value="anchored">anchored</option>
          <option value="awaiting_anchor">awaiting_anchor</option>
          <option value="failed">failed</option>
        </select>
      </div>
      <div>
        <label className="text-xs text-slate-600">Chain</label>
        <input 
          className="w-full border rounded px-2 py-1" 
          placeholder="flare"
        />
      </div>
    </div>
  )}
  
  {/* Receipt ID Selection */}
  {allowReadReceipts && receipts?.items && (
    <div className="rounded border p-2 bg-white/60">
      <div className="text-xs text-slate-600 mb-1">
        Restrict to receipt IDs (optional)
      </div>
      <div className="max-h-32 overflow-auto space-y-1">
        {receipts.items.map((it) => (
          <label key={it.id} className="flex items-center gap-2 text-xs">
            <input 
              type="checkbox"
              checked={allowedIds.includes(it.id)}
              onChange={(e) => onToggleId(it.id, e.target.checked)}
            />
            <span className="truncate">{it.id} — {it.status}</span>
          </label>
        ))}
      </div>
    </div>
  )}
  
  <label className="flex items-center gap-2">
    <input 
      type="checkbox" 
      className="accent-slate-900"
      checked={allowReadArtifacts}
      onChange={(e) => setAllowReadArtifacts(e.target.checked)}
    />
    Allow reading artifacts (vc.json)
  </label>
  
  <label className="flex items-center gap-2">
    <input 
      type="checkbox" 
      className="accent-slate-900"
      checked={allowConfigChanges}
      onChange={(e) => setAllowConfigChanges(e.target.checked)}
    />
    Allow config changes (dangerous; off by default)
  </label>
</div>
```

**Chat Interface**:
```tsx
<div className="rounded-lg border p-3 bg-white/70">
  <div className="h-40 overflow-auto space-y-2 text-sm">
    {history.map((h, i) => (
      <div key={i} className={`flex ${
        h.role === "assistant" ? "justify-start" : "justify-end"
      }`}>
        <div className={`px-3 py-2 rounded-xl max-w-[85%] ${
          h.role === "assistant" 
            ? "bg-slate-100 text-slate-900" 
            : "bg-slate-900 text-white"
        }`}>
          {h.text}
        </div>
      </div>
    ))}
  </div>
  
  <div className="flex gap-2 mt-2">
    <input
      className="flex-1 border rounded px-3 py-2"
      value={msg}
      onChange={(e) => setMsg(e.target.value)}
      placeholder="Ask: List receipts, Receipt <id>, Verify <bundle_url>, SDK help (ts)…"
      onKeyDown={(e) => {
        if (e.key === "Enter" && !e.shiftKey) {
          e.preventDefault();
          send();
        }
      }}
    />
    <button 
      className="bg-slate-900 text-white rounded px-3 py-2"
      onClick={send}
      disabled={loading}
    >
      <MessageSquare className="h-4 w-4 mr-2" />
      {loading ? "Sending…" : "Ask"}
    </button>
  </div>
</div>
```

**Session Management**:
```tsx
<div className="rounded-lg border p-3 bg-white/70 space-y-2">
  <div className="font-medium">Session</div>
  <div className="text-slate-700 text-xs break-all">
    Session ID: {sessionId || "(initializing…)"}
  </div>
  
  <div className="mt-2 flex gap-2">
    <a 
      href={logUrl}
      target="_blank"
      className="inline-flex items-center rounded border px-3 py-2"
    >
      <Save className="h-4 w-4 mr-2" />
      Session Log
    </a>
    
    <button 
      onClick={() => setHistory([...])}
      className="inline-flex items-center rounded border px-3 py-2"
    >
      <Rocket className="h-4 w-4 mr-2" />
      Clear
    </button>
  </div>
</div>
```

**Key Features**:
- ✅ **Scoped Permissions**: Granular control over what AI can access
- ✅ **Receipt Filtering**: Status and chain filters
- ✅ **ID Whitelisting**: Restrict access to specific receipts
- ✅ **Session Persistence**: Session logs stored server-side
- ✅ **Multi-turn Conversations**: Full chat history
- ✅ **SDK Integration Help**: Built-in code examples

**Example Prompts**:
- `"List receipts"`
- `"Receipt <receipt-id>"`
- `"Verify <bundle_url>"`
- `"SDK help (ts)"` or `"SDK help (python)"`
- `"Show me pending receipts on flare"`

**API Request Format**:
```typescript
const req: AIAssistRequest = {
  messages: [
    { role: "user", content: "List receipts" },
    { role: "assistant", content: "Found 5 receipts..." }
  ],
  scope: {
    allow_read_receipts: true,
    allowed_receipt_ids: ["rec_123", "rec_456"],
    allow_read_artifacts: false,
    allow_config_changes: false
  },
  session_id: "ui-abc123",
  params: {
    filters: { status: "pending", chain: "flare" }
  }
};
```

---

## Common Patterns

### 12. Card Pattern
**Usage**: Consistent container styling across the application

```tsx
<div className="card p-4 space-y-3">
  <div className="flex items-center justify-between">
    <div>
      <div className="text-base font-semibold">Card Title</div>
      <div className="text-xs text-slate-600">Subtitle or description</div>
    </div>
    <a className="text-xs text-slate-600 underline" href="#">Link</a>
  </div>
  
  {/* Card content */}
</div>
```

### 13. Button Styles

#### Primary Button
```tsx
<button className="inline-flex items-center bg-slate-900 text-white rounded px-3 py-2 text-sm disabled:opacity-60">
  <Icon className="h-4 w-4 mr-2" />
  Button Text
</button>
```

#### Secondary Button
```tsx
<button className="inline-flex items-center rounded border px-3 py-2 text-sm hover:bg-slate-50">
  <Icon className="h-4 w-4 mr-2" />
  Button Text
</button>
```

#### Danger Button
```tsx
<button className="px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700">
  Delete
</button>
```

#### Success Button
```tsx
<button className="px-4 py-2 bg-green-600 text-white rounded hover:bg-green-700">
  Confirm
</button>
```

### 14. Status Indicators

#### Pulsing Active Status
```tsx
<div className="flex items-center gap-2">
  <span className="w-2 h-2 bg-green-500 rounded-full animate-pulse"></span>
  <span>Online</span>
</div>
```

#### Inactive Status
```tsx
<div className="flex items-center gap-2">
  <span className="w-2 h-2 bg-slate-400 rounded-full"></span>
  <span>Offline</span>
</div>
```

### 15. Copy-to-Clipboard Pattern
```tsx
const [copied, setCopied] = useState(false);

async function copyToClipboard(text: string) {
  await navigator.clipboard.writeText(text);
  setCopied(true);
  setTimeout(() => setCopied(false), 2000);
}

// UI
<button onClick={() => copyToClipboard(value)}>
  {copied ? (
    <Check className="h-4 w-4 text-green-600" />
  ) : (
    <Copy className="h-4 w-4 text-slate-400" />
  )}
</button>
```

### 16. Form Input Pattern
```tsx
<div className="space-y-2">
  <label className="text-sm text-slate-600">Field Label</label>
  <input
    className="w-full border rounded px-3 py-2"
    placeholder="Enter value..."
    required
  />
</div>
```

### 17. Success/Error Messages

#### Success Message
```tsx
<div className="rounded border p-3 bg-emerald-50">
  <div className="text-xs text-emerald-800 font-medium flex items-center gap-2">
    <CheckCircle2 className="h-4 w-4" />
    Success message here
  </div>
</div>
```

#### Error Message
```tsx
<div className="text-sm text-red-600">
  Error: {errorMessage}
</div>
```

### 18. Loading States
```tsx
{loading ? (
  <div className="text-center py-8 text-slate-500">Loading...</div>
) : (
  <div>{/* Content */}</div>
)}
```

### 19. Empty States
```tsx
<div className="text-center py-8 text-slate-500">
  No items found. Create one to get started.
</div>
```

### 20. Modal/Info Box Pattern
```tsx
<div className="bg-blue-50 border border-blue-200 rounded-lg p-4">
  <div className="flex items-start gap-2">
    <HelpCircle className="h-5 w-5 text-blue-600 flex-shrink-0" />
    <div className="text-sm text-blue-900">
      <div className="font-semibold mb-1">Title</div>
      <p>Information content here...</p>
    </div>
  </div>
</div>
```

---

## Responsive Design

### Breakpoints
- `sm`: 640px
- `md`: 768px
- `lg`: 1024px
- `xl`: 1280px
- `2xl`: 1536px

### Mobile-First Approach
```tsx
// Desktop: horizontal layout, Mobile: vertical stack
<div className="flex flex-col md:flex-row gap-4">
  <div className="flex-1">{/* Sidebar */}</div>
  <div className="flex-1">{/* Main content */}</div>
</div>

// Hide on mobile, show on desktop
<div className="hidden md:block">{/* Content */}</div>

// Show on mobile, hide on desktop
<div className="md:hidden">{/* Content */}</div>
```

---

## Performance Considerations

### Client-Side Rendering
All components use `"use client"` directive for Next.js App Router compatibility.

### State Management
- Local state with `useState` for component-specific data
- Custom hooks (`useProjectStore`) for shared state
- Session storage for persistence across refreshes

### API Proxying
All backend calls route through `/api/proxy/*` to:
- Keep API keys server-side only
- Avoid CORS issues
- Add authentication headers automatically

---

## Running the Application

### Development Mode
```bash
cd web-alt
npm install
npm run dev
```
Application runs at: `http://localhost:3000`

### Production Build
```bash
npm run build
npm start
```

### Environment Variables
Create `web-alt/.env.local`:
```bash
NEXT_PUBLIC_API_BASE=http://127.0.0.1:8000
```

---

## File Structure Summary

```
web-alt/
├── app/
│   ├── layout.tsx              # Root layout with TopNavigation
│   ├── page.tsx                # Home page (Receipts & Data)
│   ├── operations/page.tsx     # Operations page
│   ├── settings/page.tsx       # Settings page
│   ├── agents/page.tsx         # AI Agents page
│   └── globals.css             # Global styles
├── components/
│   ├── TopNavigation.tsx
│   ├── ProjectsPanel.tsx
│   ├── ProjectsOverviewPanel.tsx
│   ├── ProjectConfigPanel.tsx
│   ├── APIKeysPanel.tsx
│   ├── ConnectionStringGenerator.tsx
│   ├── AssistantPanel.tsx
│   ├── ActiveProjectBadge.tsx
│   └── agents/
│       ├── AgentsList.tsx
│       ├── AgentDetails.tsx
│       ├── AgentActivity.tsx
│       ├── AgentAnalytics.tsx
│       ├── AgentAISettings.tsx
│       ├── AgentPricing.tsx
│       ├── AgentRevenue.tsx
│       ├── AgentAnchoring.tsx
│       ├── AgentModals.tsx
│       └── AgentChat.tsx
└── lib/
    ├── api.ts                  # API client functions
    └── client/
        ├── ethereum.ts         # Ethereum provider detection
        └── useProjectStore.tsx # Project state hook
```

---

## Additional Resources

- **Next.js Documentation**: https://nextjs.org/docs
- **Tailwind CSS**: https://tailwindcss.com/docs
- **Lucide Icons**: https://lucide.dev/
- **Ethers.js**: https://docs.ethers.org/v6/
- **EIP-4361 (SIWE)**: https://eips.ethereum.org/EIPS/eip-4361

---

**End of UI Components Guide**
