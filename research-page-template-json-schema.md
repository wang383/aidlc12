# Research: Page Template Systems & JSON Schema Design for UI Intermediate Representation

## Executive Summary

This research examines how JSON schema-based systems serve as intermediate representations (IR) between product requirements and generated UI code. The landscape includes mature frameworks (Baidu AMIS, JSONForms, Formily, RJSF), emerging AI-driven approaches (Vercel JSON Render, dottxt), and the broader Server-Driven UI (SDUI) pattern used by Airbnb, Spotify, and Facebook. The core architectural insight across all systems is a **three-layer separation**: data schema (what), UI schema (how to render), and a component registry (mapping to actual implementations).

---

## 1. Major JSON Schema UI Frameworks

### 1.1 Baidu AMIS (16.4k GitHub stars)
- **Source**: [github.com/baidu/amis](https://github.com/baidu/amis)
- **Confidence**: HIGH (production-proven at Baidu scale, 16.4k stars)

AMIS is a front-end low-code framework that generates complete pages from JSON configuration. It is the most comprehensive JSON-to-UI system found in this research.

**Core Architecture:**
- Every UI element is a JSON node with a `type` field that maps to a registered renderer
- Pages are composed by nesting component descriptors
- Built-in component library includes: forms, CRUD tables, charts, dialogs, wizards, tabs, and 100+ component types
- Includes a visual drag-and-drop editor (amis-editor)

**Schema Pattern (reconstructed from docs and issues):**
```json
{
  "type": "page",
  "title": "User Management",
  "body": {
    "type": "crud",
    "api": "/api/users",
    "columns": [
      { "name": "id", "label": "ID" },
      { "name": "name", "label": "Name" },
      { "name": "email", "label": "Email" }
    ],
    "filter": {
      "type": "form",
      "body": [
        { "type": "input-text", "name": "name", "label": "Name" }
      ]
    }
  }
}
```

**Key Design Decisions:**
- `type` field is the universal discriminator for component resolution
- `body` is the standard slot for child content (can be object or array)
- API integration is declarative (`api` field on data-fetching components)
- Data binding uses template expressions: `${variable}`
- Conditional visibility via `visibleOn` expressions
- Actions/events are declarative: `{ "type": "action", "actionType": "ajax", "api": "..." }`

**Relevance**: AMIS proves that a single JSON descriptor can represent entire admin pages including layout, data fetching, forms, tables, and interactions. Its `type` + `body` pattern is the most battle-tested approach.

---

### 1.2 JSONForms (Eclipse Foundation)
- **Source**: [jsonforms.io](https://jsonforms.io/docs)
- **Confidence**: HIGH (well-documented, active maintenance, React/Angular/Vue support)

JSONForms uses a **dual-schema approach**: a JSON Schema for data structure and a separate UI Schema for rendering instructions.

**Data Schema (what data exists):**
```json
{
  "type": "object",
  "properties": {
    "name": { "type": "string" },
    "description": { "type": "string" },
    "done": { "type": "boolean" },
    "rating": { "type": "integer" }
  },
  "required": ["name"]
}
```

**UI Schema (how to render it):**
```json
{
  "type": "VerticalLayout",
  "elements": [
    { "type": "Control", "scope": "#/properties/name" },
    {
      "type": "Control",
      "scope": "#/properties/description",
      "options": { "multi": true }
    },
    { "type": "Control", "label": "Rating", "scope": "#/properties/rating" },
    { "type": "Control", "label": "Done?", "scope": "#/properties/done" }
  ]
}
```

**Key Design Decisions:**
- Separation of data schema from UI schema is the core principle
- `scope` uses JSON Pointer (`#/properties/name`) to bind UI controls to data fields
- Layout types: `VerticalLayout`, `HorizontalLayout`, `Group`, `Categorization`
- Custom renderers can override any default mapping
- Automatic type-to-widget inference: `string` -> text input, `boolean` -> checkbox, `integer` -> number input

**Relevance**: The dual-schema pattern is the cleanest separation of concerns. The `scope` mechanism for data binding via JSON Pointer is elegant and reusable.

---

### 1.3 React JSON Schema Form (RJSF)
- **Source**: [rjsf-team.github.io](https://rjsf-team.github.io/react-jsonschema-form/docs/api-reference/uiSchema/)
- **Confidence**: HIGH (widely adopted, mature ecosystem)

RJSF extends JSON Schema with a `uiSchema` overlay that controls rendering.

**uiSchema Pattern:**
```json
{
  "ui:title": "Title",
  "ui:description": "Description",
  "ui:classNames": "my-class",
  "fieldName": {
    "ui:widget": "textarea",
    "ui:placeholder": "Enter description...",
    "ui:options": { "rows": 5 }
  }
}
```

**Key Design Decisions:**
- `ui:widget` overrides the default widget for a field type
- `ui:field` replaces the entire field component
- `ui:options` passes arbitrary props to the widget
- `ui:order` controls field ordering
- `ui:definitions` enables reusable UI customizations for `$ref` schemas
- `ui:globalOptions` for site-wide settings
- Supports dynamic uiSchema via functions (data-dependent rendering)

**Field Type to Widget Mapping (default):**
| JSON Schema Type | Default Widget |
|---|---|
| `string` | `text` input |
| `string` + `format: "email"` | `email` input |
| `string` + `format: "uri"` | `url` input |
| `string` + `format: "date"` | date picker |
| `string` + `enum` | `select` dropdown |
| `number` / `integer` | `number` input |
| `boolean` | `checkbox` |
| `object` | nested fieldset |
| `array` | repeatable list |

**Relevance**: RJSF's type-to-widget mapping table is the canonical reference for how to automatically infer UI components from data types. The `ui:widget` override mechanism is essential for customization.

---

### 1.4 Alibaba Formily (11.9k GitHub stars)
- **Source**: [github.com/alibaba/formily](https://github.com/alibaba/formily)
- **Confidence**: HIGH (production at Alibaba scale, supports React/Vue 2/Vue 3)

Formily is Alibaba's unified form solution with deep JSON Schema integration.

**Key Design Decisions:**
- Schema is a tree where each node can be a Field
- `x-component` specifies which UI component to render
- `x-decorator` wraps the component (e.g., with a FormItem label)
- `x-reactions` handles cross-field dependencies
- Supports both JSON Schema paradigm (backend-driven) and JSX paradigm (frontend-driven)
- Integrated with Ant Design and Alibaba Fusion component libraries
- Includes a visual form designer

**Schema Pattern:**
```json
{
  "type": "object",
  "properties": {
    "username": {
      "type": "string",
      "title": "Username",
      "x-component": "Input",
      "x-decorator": "FormItem",
      "x-validator": [{ "required": true }]
    },
    "role": {
      "type": "string",
      "title": "Role",
      "x-component": "Select",
      "x-decorator": "FormItem",
      "enum": ["admin", "user", "guest"]
    }
  }
}
```

**Relevance**: Formily's `x-component` / `x-decorator` pattern is the most explicit component mapping approach. The `x-reactions` system for cross-field dependencies is more powerful than simple visibility toggles.

---

## 2. Emerging AI-Driven Approaches

### 2.1 Vercel JSON Render
- **Source**: [blog.logrocket.com/vercel-json-render-dynamic-ui](https://blog.logrocket.com/vercel-json-render-dynamic-ui/)
- **Confidence**: MEDIUM-HIGH (new but from Vercel, well-documented)

JSON Render addresses the problem of AI-generated UI by constraining model output to structured JSON rather than raw code.

**Three Core Concepts:**
1. **Catalog**: Defines allowed components, their props, types, and events (the vocabulary)
2. **Registry**: Maps abstract component definitions to real React implementations (the bridge)
3. **Spec**: The JSON object describing the current interface (the output)

**Architecture Flow:**
```
AI Model -> generates Spec (JSON) -> validated against Catalog -> rendered via Registry -> React UI
```

**Key Insight**: Instead of letting AI generate arbitrary code, you define a strict component vocabulary. The model composes interfaces within boundaries you control. This provides schema validation, predictable rendering, and security.

**Relevance**: This catalog/registry/spec pattern is directly applicable to any system that generates UI from an intermediate representation. The separation ensures the IR is always valid and renderable.

---

### 2.2 dottxt UI Generation Schema
- **Source**: [docs.dottxt.ai/json-schema/ui-generation](https://docs.dottxt.ai/json-schema/ui-generation)
- **Confidence**: MEDIUM (focused on LLM-driven generation)

dottxt provides a JSON Schema specifically designed for LLM-generated form UIs.

**Schema Design:**
```json
{
  "type": "object",
  "properties": {
    "title": { "type": "string", "minLength": 3, "maxLength": 80 },
    "layout": { "type": "string", "enum": ["single_column", "two_column"] },
    "fields": {
      "type": "array",
      "minItems": 1,
      "maxItems": 20,
      "items": {
        "type": "object",
        "properties": {
          "name": { "type": "string", "pattern": "^[a-z][a-z0-9_]*$" },
          "label": { "type": "string" },
          "component": {
            "type": "string",
            "enum": ["text", "email", "number", "select", "textarea", "checkbox"]
          },
          "required": { "type": "boolean" },
          "placeholder": { "type": "string" },
          "options": { "type": "array", "items": { "type": "string" } }
        },
        "required": ["name", "label", "component", "required"],
        "additionalProperties": false,
        "allOf": [
          {
            "if": { "properties": { "component": { "const": "select" } } },
            "then": { "required": ["options"] }
          }
        ]
      }
    }
  },
  "required": ["title", "layout", "fields"],
  "additionalProperties": false
}
```

**Key Design Decisions:**
- Strict `additionalProperties: false` prevents hallucinated fields
- Conditional requirements (select requires options) via `if/then`
- Constrained enums for component types
- Name pattern enforcement for code-safe identifiers

**Relevance**: This is the best example of a schema designed specifically for AI generation. The constraints prevent common LLM failure modes.

---

### 2.3 Generative UI Component Schema
- **Source**: [ajing.github.io](https://ajing.github.io/posts/2025-05-03-ui-representation-action-execution/)
- **Confidence**: MEDIUM (research/blog post, but well-structured)

A compact JSON component tree designed for LLM generation:

```json
{
  "$id": "https://example.com/schemas/component.json",
  "type": "object",
  "required": ["type"],
  "properties": {
    "id": { "type": "string" },
    "type": { "enum": ["Container", "Text", "Button", "Image"] },
    "props": { "type": "object" },
    "children": { "type": "array", "items": { "$ref": "#" } }
  }
}
```

**Key Advantages:**
- Type safety via JSON Schema validation
- Framework agnostic (same representation compiles to different frameworks)
- Efficient parsing (compact structure reduces token usage)
- Self-referential `children` enables arbitrary nesting

**Relevance**: This is the minimal viable component tree schema. The recursive `children` + `type` + `props` pattern is the universal foundation.

---

## 3. Server-Driven UI (SDUI) Patterns

### 3.1 Overview
- **Sources**: [devcookies.medium.com](https://devcookies.medium.com/server-driven-ui-design-patterns-a-professional-guide-with-examples-a536c8f9965f), [sdui.design](https://www.sdui.design/)
- **Confidence**: HIGH (industry-proven by Airbnb, Spotify, Facebook)

SDUI is the architectural pattern where the server defines UI structure via JSON, and clients render it. This is the production-scale version of JSON-to-UI.

**Four Key Design Patterns:**

1. **Template-Based Rendering**: Server sends predefined template identifiers
```json
{
  "type": "template",
  "content": {
    "header": "Welcome",
    "button": { "label": "Submit", "action": "submit_form" }
  }
}
```

2. **Component-Based Architecture**: Server sends individual component descriptors
```json
{ "type": "button", "label": "Submit", "action": "submit_form" }
```

3. **Dynamic Schema-Based Rendering**: Server sends a schema describing components and relationships
```json
{
  "form": {
    "title": "User Registration",
    "fields": [
      { "type": "text", "label": "Username" },
      { "type": "password", "label": "Password" }
    ]
  }
}
```

4. **JSON/XML Layouts**: Full layout descriptors
```json
{
  "type": "vertical_stack",
  "components": [
    { "type": "image", "source": "https://example.com/image.png" },
    { "type": "text", "content": "Welcome to SDUI" }
  ]
}
```

### 3.2 sdui.design Toolkit
- **Source**: [sdui.design](https://www.sdui.design/)
- **Confidence**: MEDIUM (newer project, but well-designed)

A cross-platform SDUI toolkit with Kotlin DSL, TypeScript renderer, and React bindings.

```json
{
  "root": {
    "type": "column",
    "children": [
      { "type": "text", "props": { "fontSize": 24, "fontWeight": "bold" } }
    ]
  }
}
```

**Packages:**
- `sdui-web`: Vanilla JS renderer (JSON to DOM)
- `sdui-web-react`: React bindings with `<SduiRenderer json={schema} />`
- `sdui-compose`: Kotlin Compose Multiplatform (Android/iOS/Desktop)
- Visual builder with drag-drop canvas

---

## 4. Template Composition Patterns

### 4.1 Page-Level Layout Composition

Based on analysis across AMIS, SDUI patterns, and Shopify's JSON templates, page-level composition follows these patterns:

**Pattern A: Slot-Based Layout (AMIS style)**
```json
{
  "type": "page",
  "title": "Dashboard",
  "toolbar": [...],
  "aside": { "type": "nav", "links": [...] },
  "body": [
    { "type": "grid", "columns": [...] }
  ]
}
```

**Pattern B: Section-Based Layout (Shopify/CMS style)**
```json
{
  "sections": {
    "header": { "type": "header", "settings": { "logo": "..." } },
    "main": {
      "type": "content-area",
      "blocks": [
        { "type": "hero-banner", "settings": { "title": "..." } },
        { "type": "product-grid", "settings": { "columns": 3 } }
      ]
    },
    "footer": { "type": "footer", "settings": { "links": [...] } }
  },
  "order": ["header", "main", "footer"]
}
```

**Pattern C: Nested Container Layout (Generic)**
```json
{
  "type": "layout",
  "direction": "vertical",
  "children": [
    {
      "type": "layout",
      "direction": "horizontal",
      "children": [
        { "type": "sidebar", "width": "250px", "children": [...] },
        { "type": "main-content", "flex": 1, "children": [...] }
      ]
    }
  ]
}
```

### 4.2 Common Layout Templates

| Template Name | Structure | Use Case |
|---|---|---|
| Admin Dashboard | sidebar + (header + content) | Backend management |
| Landing Page | header + hero + sections + footer | Marketing pages |
| Form Page | header + form + actions | Data entry |
| List/Detail | header + (list \| detail panel) | CRUD interfaces |
| Wizard | header + step-indicator + step-content + nav | Multi-step flows |

---

## 5. Field Type to Component Mapping Strategies

### 5.1 Automatic Inference (RJSF / JSONForms approach)

The most common strategy maps JSON Schema types to default components:

```
string                    -> TextInput
string + format:"email"   -> EmailInput
string + format:"date"    -> DatePicker
string + format:"uri"     -> URLInput
string + enum:[...]       -> Select/Dropdown
string + maxLength > 200  -> Textarea
number / integer          -> NumberInput
boolean                   -> Checkbox / Toggle
object                    -> Fieldset / Card
array                     -> Repeatable List
array + items.enum        -> MultiSelect / CheckboxGroup
```

### 5.2 Explicit Mapping (Formily / AMIS approach)

Components are explicitly specified in the schema:

```json
{
  "username": {
    "type": "string",
    "x-component": "Input",
    "x-component-props": { "placeholder": "Enter username" }
  }
}
```

### 5.3 Hybrid Strategy (Recommended)

Combine automatic inference with explicit overrides:

1. **Default mapping**: Infer component from data type + constraints
2. **Override via uiSchema**: Allow explicit component specification
3. **Custom renderers**: Register custom components for specific patterns
4. **Priority**: explicit override > custom renderer > default mapping

---

## 6. Data Binding and Interaction Representation

### 6.1 Data Binding Patterns

**JSONForms (JSON Pointer):**
```json
{ "type": "Control", "scope": "#/properties/user/properties/name" }
```

**AMIS (Template Expressions):**
```json
{ "type": "tpl", "tpl": "Hello, ${name}!" }
```

**Formily (Path-based):**
```json
{ "x-reactions": { "dependencies": ["status"], "fulfill": { "state": { "visible": "{{$deps[0] === 'active'}}" } } } }
```

### 6.2 Interaction/Action Representation

**AMIS Actions:**
```json
{
  "type": "button",
  "label": "Save",
  "actionType": "ajax",
  "api": { "method": "post", "url": "/api/save" },
  "confirmText": "Are you sure?"
}
```

**SDUI Event Pattern:**
```json
{
  "type": "button",
  "label": "Submit",
  "events": {
    "onClick": {
      "type": "api_call",
      "url": "/api/submit",
      "method": "POST",
      "onSuccess": { "type": "navigate", "url": "/success" }
    }
  }
}
```

### 6.3 Conditional Visibility

**AMIS:**
```json
{ "type": "input-text", "name": "reason", "visibleOn": "${status === 'rejected'}" }
```

**JSONForms (Rule):**
```json
{
  "type": "Control",
  "scope": "#/properties/reason",
  "rule": {
    "effect": "SHOW",
    "condition": {
      "scope": "#/properties/status",
      "schema": { "const": "rejected" }
    }
  }
}
```

---

## 7. Recommended IR Design for Page Generation

Based on all research, here is a synthesized recommended schema design:

### 7.1 Core Component Node
```json
{
  "id": "unique-id",
  "type": "component-type",
  "props": {},
  "children": [],
  "dataBinding": "#/path/to/data",
  "visibility": { "condition": "expression" },
  "events": {}
}
```

### 7.2 Page-Level Schema
```json
{
  "$schema": "page-descriptor/v1",
  "meta": {
    "title": "Page Title",
    "description": "Page description",
    "route": "/path"
  },
  "dataSchema": {
    "type": "object",
    "properties": { ... }
  },
  "dataSources": [
    { "id": "users", "type": "api", "url": "/api/users", "method": "GET" }
  ],
  "layout": {
    "type": "admin-layout",
    "slots": {
      "sidebar": { ... },
      "header": { ... },
      "content": { ... }
    }
  },
  "components": [
    {
      "id": "user-table",
      "type": "data-table",
      "dataSource": "users",
      "columns": [
        { "field": "name", "label": "Name", "component": "text" },
        { "field": "email", "label": "Email", "component": "link" }
      ],
      "actions": [
        { "label": "Edit", "event": "openEditDialog", "params": { "id": "${row.id}" } }
      ]
    }
  ]
}
```

### 7.3 Component Registry Pattern
```typescript
// The bridge between JSON type and actual component
const registry: Record<string, ComponentDefinition> = {
  "text-input":    { component: ElInput,    defaultProps: {} },
  "select":        { component: ElSelect,   defaultProps: {} },
  "data-table":    { component: ElTable,    defaultProps: {} },
  "date-picker":   { component: ElDatePicker, defaultProps: {} },
  "form":          { component: FormWrapper, defaultProps: {} },
  "card":          { component: ElCard,     defaultProps: {} },
  // ... layout components
  "grid":          { component: ElRow,      defaultProps: {} },
  "admin-layout":  { component: AdminLayout, defaultProps: {} },
};
```

---

## 8. Key Takeaways and Recommendations

### Universal Patterns Across All Systems:
1. **`type` field is universal**: Every system uses a `type` discriminator to resolve components
2. **Recursive tree structure**: `children` or `body` enables arbitrary nesting
3. **Props passthrough**: Component-specific configuration via a `props` or inline properties
4. **Data binding via paths**: JSON Pointer, template expressions, or path strings
5. **Registry/mapping layer**: Always a lookup table from type string to actual component

### Recommended Architecture:
1. **Dual schema** (JSONForms pattern): Separate data schema from UI schema
2. **Explicit component mapping** with automatic fallback (Formily hybrid)
3. **Catalog/Registry/Spec** separation (Vercel JSON Render pattern)
4. **Slot-based layouts** for page composition (AMIS pattern)
5. **Constrained enums** for AI generation safety (dottxt pattern)

### For AI-Driven Generation:
- Use `additionalProperties: false` to prevent hallucinated fields
- Constrain component types to a known enum
- Use conditional schemas (`if/then`) for dependent fields
- Keep the schema flat where possible (reduces token usage)
- Validate output against schema before rendering

---

## Sources

| # | Source | URL | Type |
|---|---|---|---|
| 1 | Baidu AMIS | https://github.com/baidu/amis | Framework |
| 2 | JSONForms | https://jsonforms.io/docs | Framework |
| 3 | RJSF uiSchema | https://rjsf-team.github.io/react-jsonschema-form/docs/api-reference/uiSchema/ | Framework |
| 4 | Alibaba Formily | https://github.com/alibaba/formily | Framework |
| 5 | Vercel JSON Render | https://blog.logrocket.com/vercel-json-render-dynamic-ui/ | Article |
| 6 | dottxt UI Generation | https://docs.dottxt.ai/json-schema/ui-generation | Docs |
| 7 | Generative UI Schema | https://ajing.github.io/posts/2025-05-03-ui-representation-action-execution/ | Blog |
| 8 | SDUI Design Patterns | https://devcookies.medium.com/server-driven-ui-design-patterns-a-professional-guide-with-examples-a536c8f9965f | Article |
| 9 | sdui.design | https://www.sdui.design/ | Toolkit |
| 10 | react-from-json | https://www.npmjs.com/package/react-from-json | Library |
| 11 | AMIS + Liferay | https://liferay.dev/blogs/-/blogs/skip-the-heavy-coding-why-amis-beats-fragments-and-client-extensions | Article |
| 12 | Low-code Platform Core | https://github.com/paulgung/lowcode-platform-core | Demo |
| 13 | Sage Schema Composition | https://developer.sage.com/200/web/guides/general/schema-composition/ | Docs |
| 14 | Bagisto JSON Templates | https://visual.bagistoplus.com/core-concepts/templates/json-yaml | Docs |
| 15 | react-jsonr | https://www.npmjs.com/package/react-jsonr | Library |
