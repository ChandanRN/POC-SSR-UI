# SSR UI - Server-Driven UI Architecture Demo

A modern **Server-Driven UI (SDUI)** proof-of-concept built with **Angular 20**, **Ionic 8**, and **Capacitor**. This project demonstrates how mobile/web application UIs can be dynamically rendered from JSON schemas served by a backend API or local configuration files — enabling instant UI updates without app store releases.

---

## Table of Contents

- [What is Server-Driven UI?](#what-is-server-driven-ui)
- [Architecture Overview](#architecture-overview)
- [Project Structure](#project-structure)
- [Key Implementation Details](#key-implementation-details)
  - [1. Schema Models](#1-schema-models)
  - [2. SDUI Service](#2-sdui-service)
  - [3. Dynamic Renderer Component](#3-dynamic-renderer-component)
  - [4. Generic Form Component](#4-generic-form-component)
  - [5. Data Source Switching](#5-data-source-switching)
  - [6. Backend API Server](#6-backend-api-server)
- [Supported UI Components](#supported-ui-components)
- [How It Works](#how-it-works)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Running the App](#running-the-app)
  - [Running the Backend](#running-the-backend)
- [Schema Examples](#schema-examples)
- [Customization & Extensibility](#customization--extensibility)
- [Benefits of This Approach](#benefits-of-this-approach)
- [License](#license)

---

## What is Server-Driven UI?

**Server-Driven UI (SDUI)** is an architectural pattern where the server defines the user interface structure and content via JSON (or similar) schemas, and the client application dynamically renders the UI based on that schema. Instead of hardcoding UI layouts in the app, the server sends declarative instructions that the client interprets and renders using pre-built component libraries.

### Key Advantages:
- **Instant UI Updates**: Change layouts, content, or styling without app store approval
- **Personalization**: Show/hide components based on user segments
- **Cross-Platform Consistency**: Same schema drives iOS, Android, and Web
- **Reduced Release Cycles**: No need to rebuild and redeploy the app for minor UI changes

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         SERVER-DRIVEN UI ARCHITECTURE                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌──────────────┐      HTTP/JSON      ┌──────────────────────────┐     │
│   │   Backend    │ ◄─────────────────► │     Angular Frontend     │     │
│   │  API Server  │   UI Schema JSON    │   (Ionic + Capacitor)    │     │
│   └──────────────┘                     └──────────────────────────┘     │
│          │                                          │                    │
│          ▼                                          ▼                    │
│   ┌──────────────┐                         ┌──────────────────┐         │
│   │  ui-schema   │                         │  SDUIService     │         │
│   │  data-schema │                         │  (Fetches JSON)  │         │
│   │  components  │                         └──────────────────┘         │
│   │   -schema    │                                  │                   │
│   └──────────────┘                                  ▼                   │
│                                              ┌──────────────────┐      │
│                                              │ DynamicRenderer  │      │
│                                              │   Component      │      │
│                                              └──────────────────┘      │
│                                                       │                  │
│                              ┌────────────────────────┼────────────────┐│
│                              ▼                        ▼                ││
│                    ┌─────────────────┐      ┌─────────────────┐        ││
│                    │  Banner, Grid,  │      │  GenericForm    │        ││
│                    │ Carousel, Card, │      │   Component     │        ││
│                    │ Button, Text... │      │ (Dynamic Forms) │        ││
│                    └─────────────────┘      └─────────────────┘        ││
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Project Structure

```
SSR-UI/
├── backend/                          # Node.js/Express mock API server
│   ├── server.js                     # API endpoints serving UI schemas
│   └── package.json
│
├── src/
│   ├── app/
│   │   ├── core/
│   │   │   └── services/
│   │   │       └── sdui.service.ts   # Core service: fetches & manages schemas
│   │   │
│   │   ├── features/
│   │   │   ├── home/
│   │   │   │   └── dynamic-renderer/ # Dynamic component renderer
│   │   │   │       ├── dynamic-renderer.component.ts
│   │   │   │       └── dynamic-renderer.component.html
│   │   │   │
│   │   │   ├── category/             # Category listing & detail pages
│   │   │   ├── add-form/             # Demo form page
│   │   │   └── form-builder/         # Form builder feature
│   │   │
│   │   ├── shared/
│   │   │   ├── components/
│   │   │   │   ├── generic-form/     # Dynamic form generator
│   │   │   │   ├── category-list/    # Category list component
│   │   │   │   ├── category-page/    # Category detail component
│   │   │   │   └── error-boundary/   # Error handling
│   │   │   │
│   │   │   ├── models/
│   │   │   │   └── schema.model.ts   # TypeScript interfaces for all schemas
│   │   │   │
│   │   │   ├── schemas/              # JSON schema definitions (optional)
│   │   │   ├── services/             # Shared services
│   │   │   └── themes/               # Theme configurations
│   │   │
│   │   ├── home/                     # Home page (entry point)
│   │   │   └── home.page.ts
│   │   │
│   │   ├── app.component.ts          # Root component
│   │   └── app.routes.ts             # Angular routing
│   │
│   ├── assets/                       # Static JSON schema files
│   │   ├── ui-config.json            # Page title & theme config
│   │   ├── data-schema.json          # Categories & product data
│   │   └── components-schema.json    # UI component definitions
│   │
│   ├── environments/                 # Environment configurations
│   └── theme/                        # Ionic theming & variables
│
├── angular.json                      # Angular CLI configuration
├── capacitor.config.ts               # Capacitor native app config
├── ionic.config.json                 # Ionic framework config
└── package.json
```

---

## Key Implementation Details

### 1. Schema Models

All UI elements are strictly typed using TypeScript interfaces defined in `src/app/shared/models/schema.model.ts`:

```typescript
// Base interface for all components
export interface BaseComponent {
  type: string;
  visible?: boolean;
  showFor?: string;        // Personalization flag
  personalization?: Record<string, unknown>;
}

// Union type of all supported components
export type SDUIComponent =
  | BannerComponent
  | GridComponent
  | CarouselComponent
  | CardComponent
  | ButtonComponent
  | TextBlockComponent
  | SpacerComponent
  | DividerComponent
  | FormComponent;

// Complete page schema
export interface PageSchema {
  pageTitle: string;
  theme?: ThemeConfig;
  components: SDUIComponent[];
  categories?: CategoryPage[];
  forms?: FormComponent[];
}
```

Each component type has its own interface extending `BaseComponent`, ensuring type-safe rendering and autocomplete support.

### 2. SDUI Service

The `SduiService` (`src/app/core/services/sdui.service.ts`) is the heart of the architecture:

**Key Responsibilities:**
- **Fetch Schemas**: Retrieves UI schemas from either local JSON files or a remote API
- **Data Source Management**: Allows runtime switching between `local` and `api` data sources
- **Action Execution**: Handles component actions like navigation, modal opening, etc.
- **State Management**: Maintains loading states, errors, and category state via RxJS BehaviorSubjects
- **Category Management**: Provides methods to fetch and filter category data

```typescript
// Dual data source support
fetchPageSchema(): Observable<PageSchema> {
  const schema$ = this.getDataSource() === 'api'
    ? this.fetchFromApi()           // GET http://localhost:3000/api/home-ui
    : this.fetchFromLocal();        // Merge ui-config + data-schema + components-schema

  return schema$.pipe(
    delay(800),                     // Simulate network latency
    catchError((err) => {
      this.handleError(err);
      return of(this.getDefaultSchema());
    })
  );
}
```

**Local Schema Composition:**
The local mode merges three separate JSON files into a unified `PageSchema`:
- `ui-config.json` — Page title and theme configuration
- `data-schema.json` — Categories and product catalog data
- `components-schema.json` — UI component definitions

### 3. Dynamic Renderer Component

The `DynamicRendererComponent` (`src/app/features/home/dynamic-renderer/`) renders any UI component based on its `type` field using Angular's `ngSwitch`:

```typescript
@Component({
  selector: 'app-dynamic-renderer',
  templateUrl: './dynamic-renderer.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class DynamicRendererComponent {
  @Input() component!: SDUIComponent;
  @Input() skeletonMode = false;

  // Type guards for each component type
  getBannerData(): BannerComponent { return this.component as BannerComponent; }
  getGridData(): GridComponent { return this.component as GridComponent; }
  // ... etc
}
```

**Template Strategy (ngSwitch):**
```html
<ng-container [ngSwitch]="component.type">
  <div *ngSwitchCase="'banner'">...</div>
  <div *ngSwitchCase="'grid'">...</div>
  <div *ngSwitchCase="'carousel'">...</div>
  <div *ngSwitchCase="'card'">...</div>
  <div *ngSwitchCase="'button'">...</div>
  <div *ngSwitchCase="'text'">...</div>
  <div *ngSwitchCase="'spacer'">...</div>
  <div *ngSwitchCase="'divider'">...</div>
  <div *ngSwitchCase="'form'">
    <app-generic-form [formConfig]="getFormData()"></app-generic-form>
  </div>
</ng-container>
```

**Features:**
- **Visibility Control**: Components can be hidden via `visible: false`
- **Personalization**: `showFor` property controls visibility per user segment (e.g., `premiumUser`)
- **Skeleton Loading**: Shows animated placeholders while data loads
- **Lazy Loading**: Images use `loading="lazy"` for performance
- **Action Handling**: Click events delegate to `SduiService.executeAction()`

### 4. Generic Form Component

The `GenericFormComponent` (`src/app/shared/components/generic-form/`) dynamically generates forms from JSON schema:

**Supported Field Types:**
- `text`, `email`, `tel`, `number`, `textarea`
- `select` (dropdown with options)
- `checkbox`, `radio`
- `rating` (star ratings)
- `file` (file uploads)

**Features:**
- **Dynamic Validation**: Supports `required`, `min`, `max`, `minLength`, `maxLength`, `pattern`
- **Conditional Visibility**: Fields can show/hide based on other field values
- **Multi-Step Forms**: Wizard-style forms with step navigation
- **Reactive Forms**: Built on Angular's `FormBuilder` and `FormGroup`

```typescript
// Conditional field visibility
isFieldVisible(field: FormField): boolean {
  if (!field.condition) return true;

  const dependentValue = this.form.get(field.condition.field)?.value;
  switch (field.condition.operator) {
    case '==': return dependentValue == field.condition.value;
    case '!=': return dependentValue != field.condition.value;
    case 'contains': return dependentValue.includes(field.condition.value);
    // ... etc
  }
}
```

### 5. Data Source Switching

The app supports toggling between local JSON files and a live API at runtime:

```typescript
// In HomePage
toggleDataSource(): void {
  const newSource = this.currentDataSource === 'local' ? 'api' : 'local';
  this.sduiService.setDataSource(newSource);
  this.loadSchema();  // Re-fetch with new source
}
```

**Use Cases:**
- **Development**: Use local JSON for rapid UI iteration
- **Testing**: Switch to API to test backend integration
- **Production**: API mode enables real-time content updates
- **Demos**: Show stakeholders how UI changes instantly without app updates

### 6. Backend API Server

A Node.js/Express server (`backend/server.js`) provides mock API endpoints:

**Endpoints:**
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/home-ui` | Returns complete UI schema |
| GET | `/api/health` | Health check with timestamp |
| POST | `/api/analytics` | Logs analytics events |
| GET | `/api/schema/variants` | Returns available schema variants |

The server simulates network latency (500ms) and returns a slightly different schema than local files to demonstrate the API-driven approach.

---

## Supported UI Components

| Component | Type | Description | Key Properties |
|-----------|------|-------------|----------------|
| **Banner** | `banner` | Full-width image banner | `image`, `action`, `altText` |
| **Grid** | `grid` | Icon grid (e.g., category navigation) | `title`, `columns`, `items[]` |
| **Carousel** | `carousel` | Horizontal scrolling product cards | `title`, `items[]` |
| **Card** | `card` | Content card with image/text/button | `title`, `subtitle`, `description`, `image` |
| **Button** | `button` | Action button | `label`, `color`, `action`, `icon` |
| **Text** | `text` | Text block | `content`, `size`, `align`, `color` |
| **Spacer** | `spacer` | Vertical spacing | `size` (pixels) |
| **Divider** | `divider` | Horizontal line separator | `color` |
| **Form** | `form` | Dynamic form | `fields[]`, `submitLabel`, `successMessage` |

---

## How It Works

### Step-by-Step Flow:

1. **App Initialization**
   - Angular app bootstraps with Ionic
   - `HomePage` initializes and calls `SduiService.fetchPageSchema()`

2. **Schema Fetching**
   - Service checks current data source (`local` or `api`)
   - Local: Merges 3 JSON files using `forkJoin`
   - API: Calls `GET http://localhost:3000/api/home-ui`

3. **Schema Processing**
   - Loading state is emitted via BehaviorSubject
   - Schema is returned to the HomePage component
   - Error handling falls back to a default schema

4. **Dynamic Rendering**
   - HomePage iterates over `schema.components` array
   - Each component is passed to `<app-dynamic-renderer>`
   - Renderer uses `ngSwitch` to render the appropriate template

5. **User Interaction**
   - Click events trigger `SduiService.executeAction(action)`
   - Navigation actions starting with `nav:` use Angular Router
   - Other actions log to console (extendable to real functionality)

6. **Category Pages**
   - Category routes use `:slug` parameter
   - `CategoryFeaturePage` fetches category data by slug
   - `CategoryPageComponent` renders category-specific layouts

---

## Getting Started

### Prerequisites

- Node.js 18+ and npm
- Angular CLI 20+
- Ionic CLI (optional but recommended)

### Installation

```bash
# Clone the repository
git clone <repository-url>
cd SSR-UI

# Install frontend dependencies
npm install

# Install backend dependencies
cd backend
npm install
cd ..
```

### Running the App

```bash
# Terminal 1: Start the Angular dev server
npm start
# Or: ng serve
# App will be available at http://localhost:4200

# Terminal 2: Start the backend API server
cd backend
npm start
# Or: node server.js
# API will be available at http://localhost:3000
```

### Switching Data Sources

Once the app is running:
1. Open the app in your browser
2. Click the **data source toggle button** in the header
3. Switch between **Local JSON** and **API** modes
4. Observe how the UI changes instantly without page reload

---

## Schema Examples

### Banner Component
```json
{
  "type": "banner",
  "image": "https://example.com/banner.jpg",
  "action": "openOffers",
  "altText": "Big Sale Banner",
  "visible": true
}
```

### Carousel with Products
```json
{
  "type": "carousel",
  "title": "Best Deals for You",
  "items": [
    {
      "name": "iPhone 15 Pro",
      "price": "₹1,19,999",
      "originalPrice": "₹1,29,999",
      "image": "https://example.com/iphone.jpg",
      "rating": 4.8,
      "discount": "8% off",
      "action": "viewProduct1"
    }
  ],
  "visible": true
}
```

### Personalized Text (Premium Users Only)
```json
{
  "type": "text",
  "content": "Premium Members get FREE delivery",
  "size": "small",
  "align": "center",
  "showFor": "premiumUser",
  "visible": true
}
```

### Dynamic Form
```json
{
  "type": "form",
  "title": "Demo New Product",
  "description": "Fill in the details below",
  "submitLabel": "Create Product",
  "successMessage": "Product added successfully!",
  "fields": [
    {
      "type": "text",
      "name": "productName",
      "label": "Product Name",
      "placeholder": "Enter product name",
      "required": true
    },
    {
      "type": "select",
      "name": "category",
      "label": "Category",
      "required": true,
      "options": [
        { "label": "Mobiles", "value": "mobiles" },
        { "label": "Fashion", "value": "fashion" }
      ]
    }
  ]
}
```

---

## Customization & Extensibility

### Adding a New Component Type

1. **Define the interface** in `schema.model.ts`:
   ```typescript
   export interface MyCustomComponent extends BaseComponent {
     type: 'myCustom';
     customProp: string;
   }
   ```

2. **Add to union type**:
   ```typescript
   export type SDUIComponent = ... | MyCustomComponent;
   ```

3. **Add case in template** (`dynamic-renderer.component.html`):
   ```html
   <div *ngSwitchCase="'myCustom'">...</div>
   ```

4. **Add type guard** in `dynamic-renderer.component.ts`:
   ```typescript
   getMyCustomData(): MyCustomComponent {
     return this.component as MyCustomComponent;
   }
   ```

### Adding a New Field Type to Forms

1. Add the type to `FormField.type` union in `schema.model.ts`
2. Add the corresponding Ionic form control in `generic-form.component.html`
3. Add validation logic in `generic-form.component.ts`

### Custom Themes

Each category/page can define its own theme:
```json
{
  "theme": {
    "primaryColor": "#007bff",
    "backgroundColor": "#f0f4f8",
    "layout": "grid"
  }
}
```

---

## Benefits of This Approach

| Benefit | Description |
|---------|-------------|
| **Instant Updates** | Change JSON on the server → UI updates immediately |
| **No App Store Delays** | Skip 1-2 day review cycles for UI changes |
| **A/B Testing** | Serve different schemas to different user groups |
| **Personalization** | Show/hide components per user segment |
| **Feature Flags** | Toggle features by including/excluding components |
| **Cross-Platform** | Same schema works on Web, iOS, and Android |
| **Reduced Bundle Size** | UI logic lives on server, not in app binary |
| **Analytics Integration** | Track which components are rendered and clicked |

---

## License

This project is for demonstration purposes. Feel free to use it as a reference for implementing Server-Driven UI in your own applications.

---

## Next Steps & Ideas

- **Remote Config Integration**: Connect to Firebase Remote Config or similar
- **Caching Strategy**: Implement schema caching with expiration
- **Offline Support**: Cache last-known schema for offline use
- **Animation Support**: Add `animation` property to component schema
- **Real Analytics**: Integrate with analytics providers
- **Auth Integration**: Personalize schemas based on authenticated user
- **CMS Integration**: Build a visual CMS for non-developers to edit schemas

