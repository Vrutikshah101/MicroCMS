# AGENTS.md

## Purpose

This file defines the engineering, architecture, security, and implementation standards for all coding agents working on the MicroCMS platform.

Agents must treat this document as the default design contract unless a task-specific instruction explicitly overrides a section.

---

## 1. Product Summary

MicroCMS is a **multi-tenant CMS platform** built on **ASP.NET Core** where a single CMS instance manages multiple sub-websites.

Each sub-website must behave as an isolated tenant with independently configurable:

- branding
- themes
- layouts
- menus
- pages
- components
- features
- workflows
- permissions
- documents

The platform must be:

- configuration-driven
- secure by design
- VAPT-ready
- auditable
- modular
- maintainable

---

## 2. Non-Negotiable Architecture Decisions

### 2.1 Application Architecture
Agents must build the system as a:

- **modular monolith**
- using **ASP.NET Core**
- with clear module boundaries
- with future option to extract services later if required

Do **not** start with microservices unless explicitly instructed.

### 2.2 Storage Architecture
Use the following storage split exactly:

#### MySQL
MySQL is the **primary system database** and must store all structured and behavioral data, including:

- tenants
- domains
- users
- roles
- permissions
- pages
- layouts
- regions
- menus
- themes
- feature toggles
- workflows
- approvals
- audit logs
- settings
- form submissions
- component definitions
- component instances
- component configuration
- structured CMS content
- metadata relationships

#### CouchDB
CouchDB is **only for PDF/document storage and related document metadata**.

CouchDB must **not** be used for:

- page layout configuration
- feature toggles
- theme logic
- permission logic
- menu logic
- rendering logic
- workflow logic
- security rules
- UI configuration

Allowed CouchDB usage:

- PDF storage
- document storage
- versioned uploaded documents
- document metadata
- file attachments linked to CMS entities

#### Redis
Redis is used for:

- configuration caching
- page output caching
- tenant-aware cache keys
- frequently accessed lookup caching
- session or short-lived performance optimization when needed

---

## 3. Core Design Principles

All agents must follow these principles:

### 3.1 Configuration First
The CMS must be configuration-driven, not hardcoded.

Examples:
- component visibility must come from database configuration
- layout composition must come from database configuration
- site branding must come from database configuration

### 3.2 Controlled Configuration
Configuration must be flexible but bounded.

Agents must **not** create uncontrolled dynamic systems that allow arbitrary behavior without validation.

Use:
- controlled tables
- constrained values
- typed settings
- approved component registry
- validated schemas or models

### 3.3 Multi-Tenant Isolation
Every relevant read/write must be tenant-aware.

No agent may implement logic that risks cross-tenant data access.

### 3.4 Secure by Design
Security is a core requirement, not a later enhancement.

### 3.5 Auditability
Important changes must be attributable to:
- who changed it
- what changed
- when it changed
- previous value if applicable

### 3.6 VAPT Readiness
All code and design choices must support VAPT outcomes.

Agents must prefer:
- explicit validation
- constrained behavior
- secure defaults
- least privilege
- reduced attack surface

---

## 4. Required Functional Model

### 4.1 Tenant Model
Each website is a tenant.

Each tenant can have its own:
- domain or subdomain
- branding
- theme
- menus
- page layouts
- page content
- enabled components
- feature toggles
- users and access scopes
- document library

### 4.2 Single CMS, Many Sites
A single admin system must manage all tenants, while enforcing tenant isolation and tenant-scoped permissions.

### 4.3 Single-Click Configurability
The platform must support simple admin operations such as:

- hide ministry pane
- disable quick links
- change header block
- switch theme setting
- toggle widget visibility

These actions must be possible without code changes.

Example requirement:
If an admin wants to remove the ministry pane, that must be achievable by a single configuration action from the admin UI.

---

## 5. Component-Driven UI Standard

### 5.1 Registered Component System
All renderable page units must be registered components.

Examples:
- ministry pane
- quick links
- hero banner
- latest news block
- footer links
- department profile block

Agents must not allow arbitrary unregistered components to be rendered.

### 5.2 Component Constraints
Each component must define:

- unique key
- display name
- allowed placement regions
- whether configurable
- configuration contract
- visibility flag
- renderer binding

### 5.3 Region Constraints
Each page layout must use defined regions such as:

- header
- top_nav
- left_sidebar
- main
- right_sidebar
- footer

Agents must not allow random or unsafe placement logic without validation.

### 5.4 Visibility Model
Component visibility must be configurable through structured storage in MySQL.

Example:
If `ministry_pane` is disabled for a tenant homepage, the renderer must skip it.

---

## 6. Configuration Hierarchy

Agents must implement layered configuration with this precedence:

1. Page-level configuration
2. Tenant-level configuration
3. Global/default configuration

Higher precedence overrides lower precedence.

This rule must be consistently applied in:
- page rendering
- theme settings
- feature toggles
- component visibility
- content presentation

---

## 7. Rendering Standard

### 7.1 No Hardcoded Page Assembly
Razor pages, controllers, or APIs must not hardcode tenant-specific layouts.

### 7.2 Rendering Flow
Agents should implement rendering with the following logical flow:

1. Resolve tenant
2. Validate route and access
3. Load page definition
4. Load layout definition
5. Load component instances for the page
6. Apply feature and visibility rules
7. Resolve component settings
8. Render page through approved component renderers
9. Cache result where appropriate

### 7.3 Required Render Services
Agents should structure rendering around services such as:

- Tenant Resolver
- Page Resolver
- Layout Resolver
- Theme Resolver
- Component Registry Service
- Component Rendering Service
- Feature Toggle Service
- Cache Service

---

## 8. Data Ownership Rules

### 8.1 MySQL-Owned Domains
MySQL must own:
- business rules
- configuration logic
- security logic
- workflow state
- relational references
- audit records
- rendering metadata

### 8.2 CouchDB-Owned Domains
CouchDB must own only:
- document binaries or file storage references
- PDF records
- document metadata documents
- document versions

### 8.3 Document Attachment Model
Documents stored in CouchDB should be linked from MySQL entities by reference.

Example use cases:
- page attachments
- circular uploads
- reports
- downloadable scheme PDFs
- gallery documents
- notices

---

## 9. Security Standards

Agents must design and implement with the following minimum security baseline.

### 9.1 Authentication
Support secure authentication with:
- strong password policy
- optional MFA for admin users
- secure session or token handling
- account lockout as appropriate

### 9.2 Authorization
Use strict RBAC.

Authorization must be:
- role-based
- permission-based where needed
- tenant-scoped
- least-privilege oriented

### 9.3 Tenant Isolation
Every tenant-aware query must enforce tenant boundaries.

Never trust tenant identifiers directly from the client without server-side resolution and verification.

### 9.4 Input Validation
All user input must be validated server-side.

This includes:
- forms
- component settings
- uploaded file metadata
- admin settings
- API payloads
- route parameters

### 9.5 XSS Protection
Agents must:
- encode output appropriately
- sanitize rich text where needed
- disallow unsafe HTML/JS injection through admin settings

### 9.6 CSRF Protection
All authenticated state-changing web requests must be protected.

### 9.7 File Upload Security
For document uploads:
- validate extension
- validate MIME type
- validate size
- generate safe storage names
- block executable content
- scan if malware scanning is available

### 9.8 Secret Management
Secrets must not be hardcoded.

Use environment variables, secret stores, or secure configuration providers.

### 9.9 Headers and Hardening
Use security headers as appropriate, including:
- CSP
- HSTS
- X-Frame-Options or equivalent
- Referrer-Policy
- X-Content-Type-Options

### 9.10 Dependency Hygiene
Agents must prefer maintained libraries and avoid unnecessary dependencies.

---

## 10. VAPT-Oriented Rules

Agents must make choices that improve penetration testing outcomes.

### 10.1 Avoid
- arbitrary dynamic execution
- unrestricted admin HTML injection
- raw SQL concatenation
- direct database exposure to public clients
- hidden backdoor settings
- bypassable permission checks

### 10.2 Prefer
- explicit allowlists
- typed configuration
- parameterized queries
- foreign keys and constraints where useful
- auditable state transitions
- reduced attack surface

### 10.3 Required Test Areas
Code should be written to withstand checks for:
- SQL injection
- XSS
- CSRF
- IDOR
- broken access control
- privilege escalation
- insecure upload handling
- cross-tenant leakage
- verbose error exposure
- misconfiguration

---

## 11. Audit and Governance

### 11.1 Audit Requirements
Agents must ensure important actions are logged, including:
- configuration changes
- component enable/disable
- page publish/unpublish
- role and permission changes
- document upload/delete
- theme changes
- tenant setting changes

### 11.2 Workflow Model
Important changes should support:
- draft
- review
- approve
- publish
- rollback

Workflow state and approvals should be stored in MySQL.

### 11.3 Rollback
Where applicable, design for reversibility of changes.

---

## 12. Recommended Project Structure

Agents should prefer a clean modular structure similar to:

- `src/MicroCms.Web`
- `src/MicroCms.Application`
- `src/MicroCms.Domain`
- `src/MicroCms.Infrastructure`
- `src/MicroCms.Infrastructure.MySql`
- `src/MicroCms.Infrastructure.CouchDb`
- `src/MicroCms.Infrastructure.Redis`
- `src/MicroCms.Modules.Tenants`
- `src/MicroCms.Modules.Pages`
- `src/MicroCms.Modules.Components`
- `src/MicroCms.Modules.Themes`
- `src/MicroCms.Modules.Documents`
- `src/MicroCms.Modules.Security`
- `src/MicroCms.Modules.Workflows`
- `src/MicroCms.Modules.Audit`

Exact naming can vary, but module boundaries should remain clear.

---

## 13. Required Logical Agents / Services

Agents implementing code should preserve the following logical responsibilities.

### 13.1 Tenant Agent
Responsible for:
- resolving tenant from domain or route context
- loading tenant context
- enforcing tenant boundary rules

### 13.2 Configuration Agent
Responsible for:
- loading configuration from MySQL
- merging page, tenant, and global settings
- validating effective configuration

### 13.3 Page Agent
Responsible for:
- page retrieval
- page composition data
- page publishing state
- route-to-page mapping

### 13.4 Layout Agent
Responsible for:
- layout definitions
- region management
- layout selection rules

### 13.5 Component Registry Agent
Responsible for:
- registered component definitions
- allowed regions
- component contracts
- safe component resolution

### 13.6 Component Rendering Agent
Responsible for:
- rendering approved components
- enforcing visibility rules
- applying component settings

### 13.7 Theme Agent
Responsible for:
- tenant theme resolution
- branding settings
- theme asset mapping

### 13.8 Feature Toggle Agent
Responsible for:
- feature enable/disable decisions
- single-click toggle support
- scoped feature evaluation

### 13.9 Document Agent
Responsible for:
- storing and retrieving PDFs/documents via CouchDB
- validating uploads
- linking document references to CMS records
- document version handling where applicable

### 13.10 Security Agent
Responsible for:
- authorization
- authentication integration
- tenant-scoped permission checks

### 13.11 Workflow Agent
Responsible for:
- draft/review/approve/publish
- rollback support
- approval state transitions

### 13.12 Audit Agent
Responsible for:
- action logging
- change records
- compliance visibility

### 13.13 Cache Agent
Responsible for:
- cache key generation
- cache invalidation
- tenant-aware cache strategy

---

## 14. Database Design Expectations

### 14.1 MySQL Tables
Agents should expect or design tables around entities such as:

- Tenants
- TenantDomains
- Users
- Roles
- Permissions
- UserRoles
- RolePermissions
- Pages
- PageLayouts
- PageRegions
- Components
- ComponentInstances
- ComponentSettings
- Menus
- MenuItems
- Themes
- TenantSettings
- FeatureToggles
- Workflows
- WorkflowActions
- AuditLogs
- FormSubmissions
- DocumentReferences

### 14.2 Common Columns
Where relevant, include operational columns such as:
- Id
- TenantId
- CreatedBy
- CreatedOn
- ModifiedBy
- ModifiedOn
- IsActive
- IsDeleted
- VersionNo

### 14.3 Constraints
Prefer:
- foreign keys where appropriate
- unique constraints for natural uniqueness
- indexes on tenant-scoped frequently queried columns

---

## 15. Caching Standard

### 15.1 Tenant-Aware Cache Keys
Cache keys must be tenant-aware.

Examples:
- `tenant:{tenantId}:page:{pageKey}`
- `tenant:{tenantId}:theme`
- `tenant:{tenantId}:menu:{menuKey}`

### 15.2 Cache Invalidation
Any configuration change must invalidate only the relevant cache entries.

Examples:
- changing ministry pane visibility should invalidate affected layout/page cache
- updating theme should invalidate theme-related cache
- publishing page should invalidate page output cache

### 15.3 Avoid Dangerous Caching
Do not cache sensitive or permission-dependent results in a way that risks cross-user or cross-tenant leakage.

---

## 16. Coding Standards

Agents writing code should:

- prefer clear, readable, maintainable code
- use async patterns appropriately
- use dependency injection
- keep controllers thin
- keep business rules in application/domain services
- centralize validation
- centralize authorization checks where practical
- avoid duplicated logic
- add meaningful logging
- write tests for critical logic

---

## 17. API and UI Expectations

### 17.1 Admin UI
The admin UI should expose controlled settings for:
- tenant branding
- layout selection
- component visibility
- menu management
- page management
- theme changes
- document uploads
- workflow actions

### 17.2 Public UI
Public rendering must:
- be tenant-aware
- be configuration-driven
- be cache-friendly
- avoid exposing internal implementation details

### 17.3 Headless Capability
Where useful, design should allow API-based content delivery in addition to server-side rendering.

---

## 18. Explicit Prohibitions

Agents must not:

- use CouchDB for CMS behavior/configuration logic
- hardcode tenant-specific UI into shared views
- bypass authorization for convenience
- trust client-side tenant or role information
- allow arbitrary HTML/JS injection through settings without sanitization
- create cross-tenant shared mutable state without safeguards
- expose raw internal storage directly to the frontend

---

## 19. Decision Rule for Ambiguity

If a design or implementation choice is unclear, agents must prefer the option that is:

1. more secure
2. more tenant-safe
3. more auditable
4. more maintainable
5. more aligned with MySQL for structured behavior and CouchDB only for documents

---

## 20. Final Operating Standard

The target system is:

- a secure multi-tenant MicroCMS
- built on ASP.NET Core
- using MySQL as the primary configuration and business store
- using CouchDB only for PDFs/documents
- using Redis for caching
- using registered components and structured configuration
- designed for VAPT readiness
- designed for centralized control with tenant isolation

All coding agents must preserve this architecture unless explicitly instructed otherwise.
