# COMPLETE APPLICATION DOCUMENTATION
## Gestion Parc Informatique - IT Asset Management System

**Generated:** June 2026  
**Application Version:** 1.0 (Production Ready)  
**Architecture:** React Frontend + Flask Backend + SQLite Database

---

## 1. PROJECT OVERVIEW

### Application Name
**Gestion Parc Informatique** (IT Asset Management System)

### Main Purpose
A comprehensive web-based application for managing IT assets, inventory, equipment lifecycle, and operational workflow within an organization. The system tracks:
- IT equipment inventory in stock (Stock)
- IT equipment deployed in the field (Parc)
- Equipment movements (entries/exits)
- Equipment disposal tracking (Déchets)
- IT infrastructure locations and network equipment (Locaux IT)
- User access control and audit logging

### Target Users
1. **Inventory Managers** - Manage stock levels, imports/exports
2. **IT Technicians** - Track equipment movements and deployments
3. **System Administrators** - User management, audit logs, permissions
4. **Facility Managers** - Manage IT locations and infrastructure
5. **End Users** - View-only access to equipment information
6. **Supervisors/Managers** - Dashboard analytics and reporting

### Technology Stack
- **Frontend:** React 18 with JSX, Tailwind CSS, Recharts for visualizations
- **Backend:** Flask 2.x with SQLAlchemy ORM, Flask-JWT for authentication
- **Database:** SQLite (primary) / MySQL (optional)
- **APIs:** RESTful JSON APIs with JWT bearer token authentication
- **Deployment:** XAMPP (PHP/Apache) for local development

---

## 2. NAVIGATION STRUCTURE

### Main Navigation Items (Sidebar)
The application has a persistent sidebar with 6 main sections:

```
┌─────────────────────────────┐
│   Tableau de Bord (/)       │  [Dashboard]
│   Parc (/parc)              │  [Equipment deployed in field]
│   Stock (/stock)            │  [Equipment in inventory]
│   Locaux IT (/locaux-it)    │  [IT locations & infrastructure]
│   Mouvements (/mouvements)  │  [Equipment entry/exit logs]
│   Déchets (/dechet)         │  [Disposal & waste tracking]
└─────────────────────────────┘
```

### Top Navigation Bar
- **Logo:** Hutchinson company branding
- **Theme Toggle:** Light/Dark mode switch with Sun/Moon icons
- **User Menu:** Profile, password change, admin controls
- **Logout Button:** Sign out and redirect to login

### Admin Controls (Accessible via user menu)
1. **User Management**
   - Create new users
   - Reset user passwords
   - Manage permissions (export, import, edit)
   - View audit activity logs with pagination

2. **Activity History**
   - Search audit logs by action/target
   - Pagination (6 items per page)
   - Timestamp tracking for all operations

3. **Password Management**
   - Change own password
   - Reset other users' passwords (admin only)
   - Create visitor accounts with temporary access

### User Roles & Permissions
Three-tier permission model:
- **Admin Role**
  - Full system access
  - User management
  - Activity audit logs
  - Password management
  - All CRUD operations

- **View-only Role**
  - Read access to all pages
  - Cannot modify, export, or import data

- **User Role** (Customizable)
  - Custom permission_export: Boolean
  - Custom permission_import: Boolean
  - Custom permission_edit: Boolean

### User Workflow Examples

**New Equipment Flow:**
```
Admin creates user → User logs in → User can view dashboard → 
User can import stock (if permission_import=true) → 
User can export reports (if permission_export=true) →
User can create/edit items (if permission_edit=true)
```

**Equipment Lifecycle:**
```
New item added to Stock → 
Item moved via Mouvements (Entrée/Sortie) → 
Item deployed to Parc → 
Item exits service → 
Item recorded in Déchets (Waste) → 
Can be restored to Stock if needed
```

---

## 3. PAGE-BY-PAGE ANALYSIS

### PAGE 1: TABLEAU DE BORD (DASHBOARD)
**Route:** `/`  
**Purpose:** Executive overview and key performance indicators

#### Components
- **Header Section**
  - Welcome message with user's role
  - Theme toggle button
  - Background animation effects

- **Statistics Cards**
  - Total stock quantity (sum of all stock items)
  - Total parc equipment count (deployed items)
  - Number of IT locations
  - Critical stock count (quantity < 5)

- **Bar Charts** (Using Recharts library)
  - **Stock by Activity Type** - Shows distribution across FSS, IMS, C2S, Commun
  - **Parc by Activity Type** - Equipment distribution by department/activity
  - **Stock by Equipment Type** - Count of PC Portables, PC Fixes, IPOs in stock
  - **Parc by Equipment Type** - Count of deployed equipment types

- **Critical Stock Section**
  - Red/Green badge system
  - RED: Quantity < 5 (critical)
  - GREEN: Quantity >= 5 (adequate)
  - Item name, type, quantity displayed

- **Recent Movements Table**
  - Last 5 entries/exits
  - Columns: Date, Type, Equipment Name, Quantity
  - Reverse chronological order

- **IT Assistant Chatbot** (Floating widget)
  - Natural language question processing
  - 12 preset quick questions
  - Custom question input field
  - Maintains conversation history

#### Forms Present
- None (Dashboard is read-only)

#### Tables Present
- Recent Movements (5-row preview table)

#### Statistics/Cards Displayed
- Stock quantity total
- Parc equipment total  
- IT locations count
- Critical items count (with threshold indicator)
- Equipment type distribution cards

#### Buttons and Actions
- Export Dashboard as CSV/Excel (if user has permission)
- Theme toggle (light/dark mode)
- Chatbot open/close toggle
- Preset question buttons in chatbot
- Custom question input in chatbot

#### Filters and Search Features
- Dashboard data filters by activity type (auto-hides OILJK, POMLI7 labels)
- Chatbot offers filtered question views

#### Data Displayed
- All equipment across all activities
- Filtered movements (only active, is_active=True)
- Critical threshold: < 5 units

#### User Interactions
- View charts dynamically
- Ask chatbot questions about system state
- Export dashboard statistics
- Monitor equipment levels at a glance

#### Business Logic
- Aggregates stock quantities across all types
- Tracks equipment by type and activity
- Identifies critical stock items automatically
- Maintains equipment movement history
- Calculates statistics in real-time

#### Related API Endpoints
- `GET /api/stock/stats` - Stock statistics
- `GET /api/stock/critical?threshold=5` - Critical items
- `GET /api/mouvements/recent?limit=5` - Recent movements
- `GET /api/parc/stats` - Parc statistics
- `GET /api/locaux-it/` - IT locations count
- `POST /api/dashboard/ask` - Natural language question processing

---

### PAGE 2: PARC (EQUIPMENT IN FIELD)
**Route:** `/parc`  
**Purpose:** Manage deployed IT equipment across the organization

#### Components
- **Header Section**
  - Page title: "Parc Informatique"
  - Description: "Gestion des équipements déployés en parc"
  - Export buttons (CSV/Excel)
  - Import template download
  - Bulk import functionality

- **Search & Filter Bar**
  - Text search across all equipment
  - Filter by Type (dropdown)
  - Filter by Service (dropdown)
  - Filter by Emplacement/Location (dropdown)
  - Filter by Activité (FSS, IMS, C2S, Commun)

- **Equipment List**
  - Paginated table (5 items per page, customizable)
  - Selectable rows (checkbox for bulk delete)
  - Edit/Delete buttons per row
  - Serial number duplicate detection

#### Forms Present
- **Add Equipment Form**
  - Fields: Name, Alternate Username, OS Name, OS Version, Type
  - Fields: Model, Manufacturer, Processor, Serial Number
  - Fields: RAM, Disk Drive, Location, Service, Activity
  - Required fields validation
  - Serial number uniqueness check (warns of duplicates)

- **Edit Equipment Form**
  - Same fields as Add form
  - Pre-filled with existing data
  - In-place editing

#### Tables Present
- **Parc Equipment Table**
  - Columns: Name, Alt Username, OS Name, OS Version, Type, Model
  - Columns: Manufacturer, Processor, RAM, Disk, Location, Service, Activity
  - Columns: Actions (Edit, Delete)
  - Sortable by column headers
  - Hover effects showing actions
  - Pagination controls

#### Statistics/Cards Displayed
- Total parc count (in header)
- Equipment type counts

#### Buttons and Actions
- **"Ajouter un Équipement"** - Opens add form
- **"Exporter CSV"** / **"Exporter Excel"** - Export all parc data
- **"Importer"** - Upload CSV/Excel file
- **"Télécharger Template"** - Download import template
- **Edit button** (pencil icon) - Edit equipment
- **Delete button** (trash icon) - Delete equipment (soft delete)
- **"Supprimer sélections"** - Bulk delete selected items

#### Filters and Search Features
- Search box across all fields (name, model, serial number)
- Type dropdown filter
- Service dropdown filter
- Location dropdown filter
- Activity dropdown (FSS/IMS/C2S/Commun)

#### Data Displayed
- All active parc equipment (is_active=True)
- Equipment specifications and location details
- Service assignment information
- Activity/department assignment

#### User Interactions
- Add new equipment to parc
- Edit equipment details
- Delete/retire equipment
- Bulk delete multiple items
- Import from CSV/Excel file
- Export parc inventory
- Search for specific equipment
- Filter by various criteria
- Check for serial number duplicates

#### Business Logic
- Soft delete (is_active = False, not permanently deleted)
- Serial number uniqueness validation
- Activity/Service/Type normalization
- Import validation with template checking
- Bulk operations support
- Timestamp tracking (creation/modification)

#### Related API Endpoints
- `GET /api/parc/?page=1&per_page=10&search=term` - List equipment
- `GET /api/parc/:id` - Get single equipment
- `POST /api/parc/` - Create equipment
- `PUT /api/parc/:id` - Update equipment
- `DELETE /api/parc/:id` - Delete equipment (soft)
- `POST /api/parc/bulk-delete` - Bulk delete
- `GET /api/parc/stats` - Statistics
- `POST /api/parc/import` - Import from file
- `GET /api/parc/import-template` - Download template
- `GET /api/parc/export` - Export data (CSV/Excel)

#### Equipment Types Supported
```
PC Portable, PC Fixe, IPO, Imprimante A4, Imprimante Location,
Imprimante Traceur, Imprimante Étiquette, Écran, Souris Fil,
Clavier Fil, Souris Sans Fil, Clavier et Souris, Casque,
Douchette, Cable, Autre
```

#### Activity Types
```
FSS, IMS, C2S, Commun
```

---

### PAGE 3: STOCK (EQUIPMENT INVENTORY)
**Route:** `/stock`  
**Purpose:** Manage equipment inventory awaiting deployment

#### Components
- **Header Section**
  - Page title: "Stock"
  - Description: "Gestion du stock d'équipements"
  - Export buttons (CSV/Excel)
  - Import functionality
  - Add New Item button

- **Stock Critique Section** (TOP PRIORITY DISPLAY)
  - Title: "Stock Critique"
  - Badge system with colors:
    - RED badge: Quantity < 5 (rgba(220,38,38,0.08))
    - GREEN badge: Quantity >= 5 (rgba(34,197,94,0.08))
  - Item name, type, quantity shown
  - Quick visual alert system

- **Search & Filter Bar**
  - Text search across equipment name
  - Filter by Type Stock (FSS, IMS, C2S, Commun)
  - Pagination controls (10 items per page default)

- **Stock Items Table**
  - Paginated table with all active stock
  - Serial number duplicate detection
  - Edit/Delete buttons per row
  - Display all equipment details

#### Forms Present
- **Add Stock Item Form**
  - Fields: Equipment Name, Type, Quantity, Type Stock
  - Fields: State (nouveau, bon état, mauvais état, en panne)
  - Fields: RAM, Storage, Processor, Serial Number
  - Fields: Activity, System, OS Version, Manufacturer
  - Fields: Disk Drive, and 8+ additional equipment specs
  - Form visibility toggle with scrollable container
  - Sticky header when scrolling
  - Serial number uniqueness check with warning

- **Edit Stock Item Form**
  - Same fields as Add form
  - Pre-filled data
  - Validation and duplicate checks

#### Tables Present
- **Stock Items Table**
  - Columns: Name, Type, Quantity, Type Stock, State, Equipment Details
  - Sortable columns
  - Hover effects for actions
  - Status indicators (NEW, USED, DAMAGED)
  - Pagination with page controls

#### Statistics/Cards Displayed
- Total stock items count
- Critical items count (< threshold)
- Items by type stock (FSS, IMS, C2S, Commun)
- Stock status breakdown

#### Buttons and Actions
- **"Ajouter un Matériel"** - Show/hide form
- **"Exporter CSV"** - Export all stock
- **"Exporter Excel"** - Export to XLSX
- **"Importer"** - Upload file
- **"Télécharger Template"** - Download template
- **Edit button** (pencil) - Inline edit
- **Delete button** (trash) - Delete stock item
  - Creates automatic Déchet record when deleted
  - All fields copied to Déchet
  - Can be restored from Déchet page

#### Filters and Search Features
- Equipment name search (real-time)
- Type Stock dropdown filter
- Pagination with custom items-per-page
- Pagination controls (First, Previous, Next, Last)

#### Data Displayed
- All active stock items (is_active=True)
- Equipment specifications (RAM, Storage, Processor, etc.)
- Stock classification (FSS/IMS/C2S/Commun)
- Equipment state/condition
- Quantity per item
- Creation and modification dates

#### User Interactions
- Add new stock items
- Edit existing items
- Delete items (with automatic Déchet creation)
- Search for items by name
- Filter by type stock
- Export data in CSV/Excel format
- Import stock from template file
- View critical items at top
- Check for serial number duplicates

#### Business Logic
- Critical threshold: < 5 units
- Soft delete with Déchet auto-creation
- All stock fields copied to Déchet on deletion:
  - nom_equipement, type_equipement, quantite, type_stock, etat
  - ram, stockage, processeur, numero_serie, activite
  - systeme, manufacturer, disque_dur
- Description auto-generated: "Supprimé du stock le [DATE]"
- Serial number uniqueness validation per type
- Quantity and state tracking
- Import validation and error handling

#### Related API Endpoints
- `GET /api/stock/?page=1&per_page=10&type_stock=FSS&search=term` - List items
- `GET /api/stock/:id` - Get single item
- `POST /api/stock/` - Create item
- `PUT /api/stock/:id` - Update item
- `DELETE /api/stock/:id` - Delete item (creates Déchet)
- `GET /api/stock/critical?threshold=5` - Critical items
- `GET /api/stock/stats` - Statistics
- `POST /api/stock/import` - Import from file
- `GET /api/stock/import-template` - Download template
- `GET /api/stock/export?format=csv&type_stock=FSS` - Export data

#### Equipment States
```
Nouveau (New), Occasion Bon État (Used Good), 
Occasion Mauvaise État (Used Poor), En Panne (Broken)
```

---

### PAGE 4: MOUVEMENTS (EQUIPMENT MOVEMENTS)
**Route:** `/mouvements`  
**Purpose:** Track equipment entries, exits, and movements between locations

#### Components
- **Header Section**
  - Page title: "Mouvements"
  - Description: "Gestion des entrées/sorties et transferts"
  - Export buttons (CSV/Excel)
  - Add New Movement button

- **Movements Table**
  - Paginated list (10 items per page)
  - All movement records
  - Search functionality
  - Edit/Delete buttons

- **Movement History Section**
  - Deleted movements can be restored
  - Timeline of deleted movements
  - Restore/Delete permanently buttons

- **Add Movement Form**
  - Dropdown: Movement Type (Entrée, Sortie, Entrée depuis Parc)
  - Conditional fields based on movement type:
    - **Entrée:** Source selection (Achat/Stock), Equipment type, details
    - **Sortie:** Destination selection, Equipment from stock, details
    - **Entrée depuis Parc:** Select parc item to retrieve

#### Forms Present
- **Create Movement Form**
  - Field: Type Mouvement (Entrée/Sortie)
  - Field: Source Type (Entrée: Achat; Sortie: vers Parc/vers Poubelle)
  - Field: Equipment Type (required for dropdowns)
  - Field: Equipment Selection (Stock or Parc items based on type)
  - Field: Quantity (default 1)
  - Field: Type Stock (FSS, IMS, C2S, Commun)
  - Field: State (nouveau, bon état, mauvais état, en panne)
  - Conditional Fields: RAM, Processor, System, Alternate Username, OS Version, etc.
  - Field: Description (optional notes)
  - Field: Date (default today)
  - Helpful error messages:
    - "Aucun stock disponible" if no stock for type
    - "Aucun équipement [type] disponible en stock" if type selected but empty

#### Tables Present
- **Movements Table**
  - Columns: Date, Type, Equipment Name, Type, Quantity, Type Stock
  - Columns: Destination, Description, Actions
  - Sortable, filterable
  - Pagination

- **Movement History Table**
  - Columns: Equipment, Type, Quantity, Date Deleted, Actions
  - Restore/Permanently Delete buttons
  - Shows all soft-deleted movements

#### Statistics/Cards Displayed
- Total movements count
- Movements by type (Entrées vs Sorties)
- Stock availability indicators

#### Buttons and Actions
- **"Nouveau mouvement"** - Open add form (gradient button with theme colors)
- **"Exporter CSV"** / **"Exporter Excel"** - Export all movements
- **"Dupliquer"** - Copy mouvement to create similar one
- **Edit button** - Edit movement details
- **Delete button** - Soft delete (moves to history)
- **"Restaurer"** - Restore from history (undo soft delete)
- **"Supprimer définitivement"** - Permanent delete

#### Filters and Search Features
- Search by equipment name/type
- Filter by movement type (Entrée/Sortie)
- Type equipment filter (auto-loads stock/parc options)
- Pagination controls

#### Data Displayed
- Equipment name and type
- Quantity and unit type
- Source/destination information
- Movement date and timestamp
- Related details (specs for PC types)
- User who created movement
- Current status (active/deleted)

#### User Interactions
- Record equipment entry to stock
- Record equipment exit from stock
- Transfer equipment between stock and parc
- Send equipment to waste (Déchet)
- Edit movement details
- Delete movements with undo capability
- Restore deleted movements
- View movement history
- Export movement logs
- Search and filter movements

#### Business Logic
- Three movement types:
  1. **Entrée:** Item added to stock (from purchase)
  2. **Sortie:** Item removed from stock
     - Can go to Parc (deploy)
     - Can go to Waste (Déchet)
  3. **Entrée depuis Parc:** Item returned from parc to stock

- Soft delete with restoration capability
- Type-dependent source/destination validation
- Automatic stock/parc filtering based on type selection
- Movement history tracking (MouvementHistorique)
- Timestamp on all operations

#### Related API Endpoints
- `GET /api/mouvements/?page=1&per_page=10&type_mouvement=Entrée&search=term` - List
- `GET /api/mouvements/:id` - Get single
- `POST /api/mouvements/` - Create movement
- `PUT /api/mouvements/:id` - Update movement
- `DELETE /api/mouvements/:id` - Delete (soft)
- `GET /api/mouvements/sources/stock?type_equipement=pc%20portable` - Stock items
- `GET /api/mouvements/sources/parc?type_equipement=pc%20portable` - Parc items
- `GET /api/mouvements/recent?limit=5` - Last movements
- `GET /api/mouvements/historique` - Deleted movements
- `PUT /api/mouvements/historique/:id/restore` - Restore from history
- `DELETE /api/mouvements/historique/:id` - Permanent delete
- `GET /api/mouvements/export` - Export movements
- `GET /api/mouvements/stats` - Statistics

---

### PAGE 5: DÉCHETS (WASTE/DISPOSAL)
**Route:** `/dechet`  
**Purpose:** Track equipment removed from service and disposal records

#### Components
- **Header Section**
  - Page title: "Déchets"
  - Description: "Historique des sorties vers déchets depuis le stock"
  - Export buttons (CSV/Excel)
  - Search bar

- **Waste Items Table**
  - Paginated list (10 per page)
  - All disposal records
  - Equipment details
  - Restore/Delete buttons

#### Forms Present
- None (Read-only view, creation happens via Stock deletion)

#### Tables Present
- **Waste Items Table**
  - Columns: Date, Equipment Name, Type, Quantity, Type Stock, State
  - Columns: Serial Number, Details, Actions
  - Pagination controls

#### Statistics/Cards Displayed
- Total waste items count
- Waste by type
- Disposal timeline

#### Buttons and Actions
- **"Exporter CSV"** / **"Exporter Excel"** - Export waste log
- **"Restaurer"** button - Restore item back to stock
  - Creates new Stock entry with all fields preserved
  - Soft-deletes the Déchet record (is_active=False)
  - Auto-logged in audit trail
- **Delete button** - Permanently remove from waste (hard delete or soft)

#### Filters and Search Features
- Search by equipment name/type/serial number
- Pagination controls

#### Data Displayed
- Equipment specifications (all fields from Stock preserved)
- Disposal date (date_dechet)
- Reason for disposal (description)
- Original stock type and activity
- Equipment condition at time of disposal

#### User Interactions
- View all disposed equipment
- Search waste records
- Restore equipment back to stock (reversible)
- Delete waste records
- Export waste logs for reporting
- Track equipment lifecycle

#### Business Logic
- Automatic creation when Stock item deleted
- All Stock fields copied to Déchet:
  - Equipment specs, serial numbers, state, activity
  - Description: "Supprimé du stock le [DATE]"
- Restore capability:
  - Creates new Stock entry from Déchet data
  - Soft-deletes Déchet record (is_active=False)
  - Fully reversible operation
- Soft delete pattern (is_active=False) for all deletions
- Audit logging on all operations
- Timestamp tracking (date_dechet, date_modification)

#### Related API Endpoints
- `GET /api/dechets/?page=1&per_page=10&search=term` - List waste
- `GET /api/dechets/:id` - Get single
- `DELETE /api/dechets/:id` - Delete waste item
- `POST /api/dechets/:id/restore` - Restore to stock
- `GET /api/mouvements/export?scope=dechets` - Export waste

#### Workflow Example
```
1. User deletes stock item "Dell Laptop"
2. Stock.delete() is called
3. Automatically creates Déchet record with all fields
4. Stock item marked is_active=False
5. User can see item in Déchet page
6. User clicks "Restaurer"
7. New Stock entry created with same data
8. Déchet marked is_active=False
9. Item returns to Stock page
10. Operation logged in audit trail
```

---

### PAGE 6: LOCAUX IT (IT LOCATIONS & INFRASTRUCTURE)
**Route:** `/locaux-it`  
**Purpose:** Manage IT infrastructure, server rooms, network equipment

#### Components
- **Header Section**
  - Page title: "Gestion des Locaux IT"
  - Description: "Naviguez dans vos locaux, baies et matériels réseau"
  - Export buttons for Locaux and Baies (CSV/Excel)
  - Import template for network equipment

- **Add Local Form**
  - Field: Local Name (unique)
  - Field: Location/Localisation (optional)
  - Add button

- **Locaux List** (Expandable sections)
  - Each local is a collapsible accordion
  - Displays: Name, Description, Creation Date
  - Expand to see associated Baies
  - Actions: Add Baie, Edit Local, Delete Local

- **Baies IT List** (Within each Local)
  - List of server racks within location
  - Each baie shows equipment count
  - Actions: Add Equipment, Edit Baie, Delete Baie
  - Material list under each baie

- **Network Equipment (Matériels IT)**
  - Type-based organization
  - Specifications table with columns:
    - Type, Name, Manufacturer, Model, Version, OS/Firmware
    - Serial Number, Stack Role, Stack IP, Actions

#### Forms Present
- **Add Local Form**
  - Field: Nom (required, unique)
  - Field: Localisation (optional)
  - Submit: "Créer Local"

- **Add Baie Form**
  - Field: Nom (required)
  - Field: Description (optional)
  - Pre-filled in local context
  - Submit: "Créer Baie"

- **Add Equipment Form (Matériel IT)**
  - Field: Type Matériel (Sw-core, Sw-management, Sw-acces, etc.)
  - Field: Name (required)
  - Field: Manufacturer, Model, Version
  - Field: OS/Firmware
  - Field: Serial Number, Processor, RAM, Storage
  - Field: Stack Role, Stack IP
  - Field: Description
  - Submit: "Ajouter Équipement"

- **Transfer Equipment Form**
  - Select destination Local IT
  - Select destination Baie
  - Move equipment between locations
  - Submit: "Transférer"

#### Tables Present
- **Materiels IT Table** (Per Baie)
  - Columns: Type, Name, Manufacturer, Model, Version
  - Columns: OS/Firmware, Serial Number, Stack Role, Stack IP
  - Row actions: Transfer, Delete
  - Hover effects for visibility

#### Statistics/Cards Displayed
- Count of IT locations
- Count of baies per location
- Equipment count per baie
- Equipment type distribution

#### Buttons and Actions
- **"+ Ajouter un Local"** - Open local creation form
- **"+ Créer Baie"** - Add server rack to local
- **"+ Ajouter Matériel"** - Add network equipment
- **"Transférer"** - Move equipment between locations
- **"Supprimer"** - Delete equipment (trash icon)
- **Edit icon** - Modify equipment details
- **Chevron icons** - Expand/collapse sections
- **Export Locaux** - Download locations (CSV/Excel)
- **Export Baies** - Download racks (CSV/Excel)
- **Import Matériel** - Upload network equipment

#### Filters and Search Features
- Accordion expansion/collapse (visual filtering)
- Search within local by baie name
- Equipment type filter (implicit through type display)

#### Data Displayed
- Local IT name and location
- Baie information and equipment count
- Network equipment specifications
- Equipment assignment information
- Stack configuration (role and IP)
- Equipment status and condition

#### User Interactions
- Create IT locations (CIM2, CIM6, CIM7, CIM4H1, CIM4H2)
- Manage server racks (baies) within locations
- Add network equipment (switches, routers, firewalls, servers)
- Transfer equipment between locations
- Edit equipment details
- Delete equipment
- Export infrastructure inventory
- Import bulk network equipment

#### Business Logic
- Hierarchical structure: LocalIT > BaieIT > MaterielIT
- Unique local names (nom UNIQUE in DB)
- Soft delete for all items (is_active=False)
- Cascade delete: Baie delete also soft-deletes related equipment
- Timestamp tracking (creation/modification dates)
- Auto-populated default locations (optional init endpoint)
- Default baie configuration per location

#### Related API Endpoints
- `GET /api/locaux-it/` - List all locations
- `GET /api/locaux-it/:id` - Get single location
- `POST /api/locaux-it/` - Create location
- `PUT /api/locaux-it/:id` - Update location
- `DELETE /api/locaux-it/:id` - Delete location
- `GET /api/locaux-it/:localId/baies` - Get baies for location
- `GET /api/locaux-it/baies/:id` - Get single baie
- `POST /api/locaux-it/:localId/baies` - Create baie
- `PUT /api/locaux-it/baies/:id` - Update baie
- `DELETE /api/locaux-it/baies/:id` - Delete baie
- `POST /api/locaux-it/materiels` - Create network equipment
- `PUT /api/locaux-it/materiels/:id/transfer` - Transfer equipment
- `DELETE /api/locaux-it/materiels/:id` - Delete equipment
- `POST /api/locaux-it/materiels/import` - Import equipment
- `GET /api/locaux-it/materiels/import-template` - Download template
- `POST /api/locaux-it/default/init` - Initialize default locations
- `POST /api/locaux-it/default/init-baies` - Initialize default baies
- `GET /api/locaux-it/export` - Export locations
- `GET /api/locaux-it/baies/export` - Export baies

#### Default Locations (Optional Init)
```
CIM2 → [Baie 1]
CIM6 → [Baie 1, Baie 2, Baie 3, Baie 4]
CIM7 → [Baie 1, Baie 2]
CIM4H1 → [Baie 1]
CIM4H2 → [Baie 1]
```

#### Network Equipment Types (Baie)
```
Sw-core, Sw-management, Sw-acces, Sw-tor, Sw-hcs,
Controleur wifi, FirewallHCS, Firewall, Routeur, IPBX,
Serveur, Serveur HCS, Baie de STOCKAGE, NAS, Autres
```

---

### PAGE 7: LOGIN
**Route:** `/`  (Redirect to login if not authenticated)  
**Purpose:** User authentication and session initialization

#### Components
- **Login Form**
  - Company logo (Hutchinson)
  - Welcome message
  - Background gradient animation

#### Forms Present
- **Login Form**
  - Field: Username (required)
  - Field: Password (required)
  - Submit: "Se Connecter" button
  - Error messages for invalid credentials
  - Animated loading state on submit

#### Authentication Flow
1. User enters credentials
2. POST to `/api/auth/login` with {nom, password}
3. Backend validates against Utilisateur table
4. If valid: Returns JWT token + user object
5. Frontend stores token in localStorage
6. Token added to all subsequent API requests (Bearer header)
7. On 401 response: Auto-logout, clear token, redirect to login
8. On 401 auto-logout, user sees "Session expirée" message

#### Security Features
- JWT bearer token authentication
- Password hashing (werkzeug.security)
- Session expiration on logout
- 401 interceptor for stale sessions
- No credentials stored in frontend (localStorage only)

#### Related API Endpoints
- `POST /api/auth/login` - Authenticate user
- `PUT /api/auth/change-password` - Change own password
- `PUT /api/utilisateurs/:id/reset-password` - Reset password (admin)

---

## 4. DASHBOARD ANALYSIS (IT ASSISTANT CHATBOT)

### Overview
The IT Assistant is a lightweight, rule-based chatbot for natural language question processing. It does NOT use external AI APIs—all logic is deterministic and keyword-based.

### Components
- **Floating Widget** (Bottom right corner)
  - Open/Close button (toggle)
  - Message history display
  - Input field for questions
  - Preset quick-action buttons
  - Auto-scrolling to latest message

### Capabilities

#### Preset Questions (12 Total)
```
STOCK QUESTIONS:
1. "Combien au total en stock ?" 
   → "Il y a X articles en stock au total."

2. "Stock critique" 
   → "Y article(s) critique(s) : [list with counts]"

3. "Quel type contient le plus en stock ?" 
   → "[Type] est le type le plus courant : X article(s)."

PARC QUESTIONS:
4. "Total parc" 
   → "Total équipements en parc : Z article(s)."

5. "État du parc" 
   → Equipment distribution by state

MOVEMENT QUESTIONS:
6. "Derniers mouvements" 
   → List of last 5 movements with dates

7. "Dernier équipement sorti" 
   → "[Equipment] est sorti de [stock] le [DATE]."

8. "Dernier équipement entré" 
   → "[Equipment] est entré en [stock] le [DATE]."

IT LOCATION QUESTIONS:
9. "Nombre locaux IT ?" 
   → "Il y a X locaux IT."

SYSTEM QUESTIONS:
10. "Résumé système" 
    → Comprehensive system status overview

FLEXIBLE EQUIPMENT TYPE QUESTIONS:
11. Custom: "Nombre imprimantes ?" / "Combien imprimantes ?"
    → "[Type] en parc : X article(s)."

12. Custom: "PC Fixes en parc" / "PC Portables en parc" / "Combien serveurs ?"
    → "[Type] en parc : X article(s)."
```

### Natural Language Processing

#### Question Classification (Rule-Based)
1. **Critical Stock Detection**
   - Keywords: "critical", "critique", "stock", "below", "moins"
   - Returns: Critical items < threshold

2. **Total Equipment Query**
   - Keywords: "equipment", "equipement", "parc total", "nombre", "combien"
   - Returns: Total parc count

3. **Stock Availability Query**
   - Keywords: "stock", "available", "dispon"
   - Returns: Total stock quantity

4. **Movement Query**
   - Keywords: "latest", "derniers", "mouvement", "entree", "sortie"
   - Returns: Recent movements

5. **Equipment Type Query** (Three-tier matching)
   - Tier 1: Exact substring match
     - "imprimantes" matches "imprimantes" type
   - Tier 2: All words match (any order)
     - "Nombre PC Portables" matches "PC Portables"
   - Tier 3: Single-word type matching
     - "serveurs" matches "serveur" type
   - Returns: Count of equipment type in parc

### Supported Equipment Types
```
PARC: PC Portable, PC Fixe, IPO, Imprimante, Imprimante A4,
      Imprimante Location, Imprimante Traceur, Imprimante Étiquette,
      Écran, Souris, Clavier, Casque, Douchettes, Câbles, Autre

STOCK: PC Portable, PC Fixe, IPO, Imprimante, and others
```

### Response Format
- **Professional, Plain Text** (No markdown, no emojis)
- **Examples:**
  - "Il y a 45 articles en stock au total."
  - "PC Portables en parc : 12 article(s)."
  - "3 article(s) critique(s) : Laptop 1 (2 u.), Laptop 2 (3 u.)"
  - "Aucune donnée disponible pour cette question."

### User Interactions
1. **Click Preset Button** → Question sent automatically
2. **Type Custom Question** → Enter and press Send
3. **View Message History** → Scroll through conversation
4. **Switch Topics** → Ask any supported question
5. **Widget Toggle** → Open/close chatbot

### API Integration
- All questions POST to `/api/dashboard/ask`
- Request payload: `{question: "user question text"}`
- Response: `{answer: "response text"}`

### Related API Endpoints
- `GET /api/dashboard/assistant-stats` - Get all stats for processing
- `POST /api/dashboard/ask` - Process natural language question

---

## 5. INVENTORY/STOCK MANAGEMENT FEATURES

### Stock Module Overview
The Stock module is the core inventory management system.

### Key Features

#### 1. Stock CRUD Operations
- **Create:** Add new items with full specifications
- **Read:** View all items with pagination
- **Update:** Modify item details anytime
- **Delete:** Remove items (creates Déchet record automatically)

#### 2. Import/Export
- **Export Formats:** CSV, Excel (XLSX)
- **Import:** CSV/Excel with template validation
- **Template Download:** Get standardized import template
- **Validation:** Required fields check, data type validation

#### 3. Critical Stock Tracking
- **Threshold:** Default 5 units
- **Color System:**
  - RED (< 5): Critical, needs reorder
  - GREEN (≥ 5): Adequate stock
- **Dashboard Alert:** Shows critical items prominently

#### 4. Stock Types (Activities)
```
FSS - Fireworks Safety System
IMS - Information Management System
C2S - Common Command System
COMMUN - Shared/Common
```

#### 5. Equipment States
```
Nouveau (New) - Fresh from supplier
Occasion Bon État (Used Good) - Still functional, good condition
Occasion Mauvaise État (Used Poor) - Functional but degraded
En Panne (Broken) - Non-functional, needs repair/disposal
```

#### 6. Serial Number Management
- Uniqueness validation (no duplicates within type)
- Warning on duplicate detection during add/edit
- Search by serial number capability
- Tracking across stock/parc/déchet

#### 7. Equipment Specifications Tracked
```
RAM, Storage, Processor, Disk Drive,
OS/System, Manufacturer, Model,
Accessories, Alternate Username,
Service, Activity/Department,
Location, Equipment Type
```

### Workflow Example: New Laptop Stock Item

```
1. Inventory manager receives 10 Dell Laptops
2. Goes to Stock page, clicks "Ajouter un Matériel"
3. Fills form:
   - Nom: "Dell Latitude 5530"
   - Type: "PC Portable"
   - Quantité: 10
   - Type Stock: "IMS"
   - État: "Nouveau"
   - RAM: "16GB DDR4"
   - Stockage: "512GB SSD"
   - Processeur: "Intel i7-12700H"
   - N° Série: "1AB2CD3EF4GH"
   - Manufacturer: "Dell"
4. Clicks "Sauvegarder"
5. Item appears in Stock table with GREEN badge
6. On Dashboard: Critical count unchanged (10 >= 5)
7. If laptop used, one can be moved to Parc via Mouvements
8. If laptop fails, can be deleted → creates Déchet record
9. From Déchet: Can restore to Stock if repair planned
```

---

## 6. EQUIPMENT MANAGEMENT FEATURES

### Parc Module Overview
The Parc module tracks equipment deployed in the field across the organization.

### Key Features

#### 1. Equipment Registration
- Record all deployed IT equipment
- Detailed specifications (OS, processor, RAM, disk, etc.)
- Assignment information (service, location, user)
- Serial number tracking

#### 2. Equipment Lifecycle States
```
Deployed → In Use → Maintenance → Retired → Waste
```

#### 3. Location & Service Tracking
- **Emplacement:** Physical location (office, server room, etc.)
- **Service:** Department/division assignment (FSS, IMS, C2S, Commun)
- **Activité:** Activity/project code
- **Alternate Username:** User if different from primary account

#### 4. Search & Filter
- Find equipment by name, model, serial number
- Filter by type (PC Portable, PC Fixe, IPO, etc.)
- Filter by service assignment
- Filter by location/emplacement
- Filter by activity

#### 5. Bulk Operations
- Select multiple items with checkboxes
- Delete selected items in batch
- Export selected or all items

#### 6. Serial Number Tracking
- Unique identifier per equipment
- Duplicate detection on add/edit
- Search capability across deployments

### Equipment Types Supported
```
PC Portable (Laptop)
PC Fixe (Desktop)
IPO (Thin Client / Special Equipment)
Imprimante A4 (4-color laser/inkjet)
Imprimante Location (Leased printer)
Imprimante Traceur (Plotter)
Imprimante Étiquette (Label printer)
Écran (Monitor)
Souris Fil (Wired Mouse)
Clavier Fil (Wired Keyboard)
Souris Sans Fil (Wireless Mouse)
Clavier et Souris (Keyboard + Mouse combo)
Casque (Headset)
Douchette (Barcode Scanner)
Cable (Network/Power cable)
Autre (Other)
```

### Workflow Example: Deploy Equipment to Field

```
1. Technician receives new laptop for employee
2. Goes to Parc page, clicks "Ajouter un Équipement"
3. Fills form:
   - Name: "Laptop_User_123"
   - Type: "PC Portable"
   - OS: "Windows 11"
   - Model: "Dell Latitude"
   - Serial: "XYZ123ABC"
   - Service: "IMS"
   - Emplacement: "Bureau 4.2"
4. Clicks "Sauvegarder"
5. Equipment now appears in Parc list
6. Can be searched, filtered, exported
7. On Dashboard: Parc total increments by 1
8. Equipment tracked throughout its service life
9. Can be transferred between locations
10. When retired: Record in Déchet, retire from Parc
```

---

## 7. USER MANAGEMENT FEATURES

### Admin-Only Functions

#### 1. User CRUD Operations
- **Create Users:** Add new accounts with role/permissions
- **Read Users:** View all users, their roles, permissions
- **Reset Password:** Admin can reset any user's password
- **Delete Users:** Soft-delete user accounts
- **Permissions:** Assign export, import, edit permissions

#### 2. User Roles
```
ADMIN - Full access, user management
USER - Configurable permissions per user
VIEW_ONLY - Read-only access to all pages
MANAGER - Supervise subordinates (optional)
```

#### 3. Permission Model
- **permission_export:** Can export data to CSV/Excel
- **permission_import:** Can upload/import files
- **permission_edit:** Can create/modify/delete items
- **Default:** All false except export=true for new users

#### 4. Password Management
- Change own password (requires current password verification)
- Admin can reset other users' passwords
- Minimum 8 characters for new passwords
- Hashed storage (werkzeug.security.generate_password_hash)

#### 5. Activity Audit Log
- Tracks all user actions (create, update, delete, login)
- Searchable by action, target, date
- Pagination (6 items per page)
- Shows user name, timestamp, action details
- Example entries:
  - "connexion - Connexion réussie pour admin"
  - "ajout stock - Dell Laptop (PC Portable)"
  - "modification parc - Equipment_123"
  - "suppression déchet - Old Equipment"

#### 6. Visitor Account
- Optional temporary account creation
- Limited permissions
- Can be reset/deleted anytime

### Workflow Example: Create New User

```
1. Admin logs in
2. Clicks user menu → "Gestion Utilisateurs"
3. Clicks "Créer Nouvel Utilisateur"
4. Fills form:
   - Nom: "john.doe"
   - Mot de passe: "SecurePass123"
   - Rôle: "user"
   - Permission Export: TRUE
   - Permission Import: FALSE
   - Permission Edit: TRUE
5. Clicks "Créer"
6. User created, password hashed
7. John can now login with "john.doe"
8. John can export/edit but not import
9. Actions logged in audit trail
```

---

## 8. DATABASE STRUCTURE

### Entity Relationship Diagram

```
UTILISATEURS (Users)
├─ id (PK)
├─ nom (unique)
├─ password (hashed)
├─ role (admin, user, view_only)
├─ permission_export, permission_import, permission_edit
├─ is_active (soft delete flag)
└─ date_creation, date_modification

STOCK (Inventory Items)
├─ id (PK)
├─ nom_equipement
├─ type_equipement
├─ quantite
├─ type_stock (FSS, IMS, C2S, Commun)
├─ etat (novo, bon, mauvais, panne)
├─ Specs: ram, stockage, processeur, numero_serie
├─ activite, systeme, manufacturer, disque_dur
├─ is_active
└─ date_creation, date_modification

PARC (Deployed Equipment)
├─ id (PK)
├─ name
├─ alternate_username
├─ os_name, os_version
├─ type, model, version
├─ manufacturer
├─ Specs: numero_serie, processeur, ram, disque_dur
├─ emplacement, service, activite
├─ is_active
└─ date_creation, date_modification

MOUVEMENT (Equipment Movements)
├─ id (PK)
├─ type_mouvement (entrée, sortie)
├─ nom_equipement, type_equipement
├─ quantite, type_stock
├─ local_it_destination, baie_destination
├─ Specs: ram, stockage, processeur, numero_serie, systeme
├─ utilisateur_id (FK → UTILISATEURS)
├─ is_active
└─ date_mouvement, (date_creation implied)

DECHET (Waste/Disposal)
├─ id (PK)
├─ nom_equipement, type_equipement
├─ quantite, type_stock, etat
├─ Specs: ram, stockage, processeur, numero_serie
├─ activite, systeme, manufacturer, disque_dur
├─ description ("Supprimé du stock le...")
├─ is_active
└─ date_dechet

MOUVEMENTS_HISTORIQUE (Deleted Movements Archive)
├─ id (PK)
├─ mouvement_id_original (reference)
├─ nom_equipement, type_mouvement, quantite
├─ type_stock, local_it_destination
├─ Specs & description
├─ date_mouvement_originale, date_suppression
├─ is_active
└─ (audit trail)

AUDIT_LOG (Activity Tracking)
├─ id (PK)
├─ utilisateur_id (FK, nullable)
├─ utilisateur_nom
├─ action (connexion, ajout, modification, suppression)
├─ cible (entity reference)
├─ details (human-readable description)
└─ created_at (auto timestamp)

LOCAUX_IT (IT Locations)
├─ id (PK)
├─ nom (unique)
├─ localisation
├─ is_active
├─ baies (relationship)
└─ date_creation, date_modification

BAIES_IT (Server Racks)
├─ id (PK)
├─ nom
├─ numero
├─ local_it_id (FK → LOCAUX_IT)
├─ description
├─ equipements (relationship)
├─ is_active
└─ date_creation, date_modification

MATERIEL_IT (Network Equipment)
├─ id (PK)
├─ type_materiel (Sw-core, Firewall, etc.)
├─ nom
├─ marque, modele, version
├─ os_firmware, numero_serie
├─ processeur, ram, stockage
├─ stack_role, stack_ip
├─ baie_id (FK → BAIES_IT, implicit)
└─ (various tracking fields)
```

### Key Design Patterns

#### 1. Soft Delete (is_active = False)
- All deletions are logical, not physical
- Records preserved for audit/recovery
- Queries filter WHERE is_active=True
- Enables undo/restore capabilities

#### 2. Relationships & Cascades
- Utilisateur → AuditLog (1:M)
- Utilisateur → Mouvement (1:M)
- LocalIT → BaieIT (1:M, cascade delete soft)
- BaieIT → MaterielIT (1:M, cascade delete soft)

#### 3. Field Replication (Stock → Déchet)
- All Stock fields copied to Déchet on deletion
- Maintains complete record of equipment state
- Supports restoration with full data recovery

#### 4. Timestamp Tracking
- date_creation: Set once on creation (immutable)
- date_modification: Updated on every change
- Used for audit trails and sorting

#### 5. Audit Trail
- AuditLog captures every action
- Stores user, timestamp, action type, target, details
- Searchable and filterable
- Immutable record for compliance

### Database Storage
- **Default:** SQLite (local file: `backend/gestion_parc.db`)
- **Optional:** MySQL (configure via `.env` file)
- **Migrations:** Schema auto-created on app startup (Flask-SQLAlchemy)

### Field Sizes & Types
- **Strings:** VARCHAR(255) typical, some smaller (100, 50)
- **Text:** TEXT for descriptions, accessories
- **Integers:** INT for quantities, IDs
- **DateTime:** DATETIME for timestamps
- **Booleans:** BOOLEAN (0/1 in SQLite)

---

## 9. DESIGN ANALYSIS

### Current Design System

#### Color Palette

**Light Theme:**
```
Primary Background: #f8fbff (very light blue)
Surface Colors:
  --surface-0: transparent
  --surface-1: rgba(255, 255, 255, 0.7) (semi-transparent white)
  --surface-2: #ffffff (pure white)
  --surface-3: #f8fafc (light blue-gray)

Text Colors:
  --text-0: #0f172a (very dark, near black)
  --text-1: #334155 (dark gray-blue)
  --text-2: #475569 (medium gray)
  --text-3: #64748b (light gray)

Borders:
  --border-0: rgba(15, 23, 42, 0.08) (very light)
  --border-1: rgba(15, 23, 42, 0.12) (light)
  --border-2: rgba(148, 163, 184, 0.35) (medium)

Accents:
  --card-bg: #ffffff
  --table-head-bg: #f1f5f9
  --table-row-hover: #f8fafc
  --chart-grid: rgba(148, 163, 184, 0.35)
  --tooltip-bg: rgba(255, 255, 255, 0.96)
```

**Dark Theme:**
```
Primary Background: #0f172a (very dark blue-black)
Surface Colors:
  --surface-0: transparent
  --surface-1: rgba(30, 41, 59, 0.5) (semi-transparent dark)
  --surface-2: #0b1220 (dark panel)
  --surface-3: #0f172a (darkest)

Text Colors:
  --text-0: #f8fafc (near white)
  --text-1: #cbd5e1 (light gray)
  --text-2: #94a3b8 (medium gray)
  --text-3: #64748b (darker gray)

Input Styling:
  background-color: #0f172a !important
  color: #f8fafc
  placeholder: var(--text-3)

Borders:
  --border-0: rgba(148, 163, 184, 0.12)
  --border-1: rgba(148, 163, 184, 0.2)
  --border-2: rgba(148, 163, 184, 0.35)
```

#### Design Tokens
- **Font Family:** Manrope (weights: 400, 500, 600, 700, 800)
- **Font Sizes:** Tailwind defaults (sm: 12px, base: 16px, lg: 18px, xl: 20px, etc.)
- **Border Radius:** 
  - Buttons: rounded-full (50px)
  - Cards: rounded-[28px] (28px)
  - Forms: rounded-lg (8px)
- **Shadows:**
  - Cards: `0 10px 30px rgba(2, 6, 23, 0.08)` (light theme)
  - Buttons: `0 10px 24px rgba(2, 132, 199, 0.2)` (gradient blue)
- **Spacing:** Tailwind scale (4px base unit: 4, 8, 12, 16, 24, 32, etc.)

#### Component Design

**Buttons:**
- Style 1: Rounded-full with gradient background
- Style 2: Outline with border and transparent background
- Style 3: Icon buttons (small, no background)
- Hover: Color shift or shadow enhancement
- Disabled: Reduced opacity (50%), no cursor
- Icons: Lucide React (pencil, trash, download, upload, plus, etc.)

**Tables:**
- Striped rows with hover effects
- Header background (--surface-1 or --table-head-bg)
- Pagination controls below
- Sortable columns (implied via headers)
- Row actions on right side

**Forms:**
- Rounded containers (28px radius)
- Input fields with borders
- Labels above inputs
- Validation messages below fields
- Submit button at bottom
- Form appears in modal/drawer or inline

**Cards/Panels:**
- White background (light) or dark (dark mode)
- Rounded corners (28px)
- Subtle shadows
- Padding 24-32px typical
- Icons in top-left or top-right

**Modals:**
- Centered overlay
- Backdrop blur effect
- Rounded corners
- Close button (X icon)
- Keyboard-dismissible (ESC key)

### Current Strengths

1. **Dark Mode Support:** Complete theme system with CSS variables
2. **Responsive Design:** Mobile-friendly layout with Tailwind CSS
3. **Accessibility:** Semantic HTML, keyboard navigation, ARIA labels
4. **Consistent Design:** Unified color palette and component patterns
5. **User Feedback:** Loading states, error messages, confirmation dialogs
6. **Professional Appearance:** Clean, modern UI with smooth transitions
7. **Localized Content:** French language throughout
8. **Efficient State Management:** React Context API for auth and theme

### Current Weaknesses

1. **Form UX Issues:**
   - Form overlays can hide beneath top navbar (partially fixed with scrollable containers)
   - No inline validation feedback (submit-time validation only)
   - Error messages not always visible
   - Long forms require extensive scrolling

2. **Table Usability:**
   - Small text in dense tables (hard to read)
   - No horizontal scroll on mobile
   - Sorting not always obvious
   - Row actions sometimes hidden until hover

3. **Search & Filter:**
   - Limited to keyword search (no advanced filters)
   - Filter results not always instant
   - No saved filters
   - Filter state lost on page refresh

4. **Performance:**
   - Dashboard charts render all data (could paginate)
   - No virtualization for large tables
   - All API calls synchronous (could batch)

5. **Visual Hierarchy:**
   - Too many similar colors/weights
   - Critical alerts not distinctive enough (subtle red/green)
   - Important actions not always prominent

6. **Mobile Responsiveness:**
   - Sidebar collapses but layout not optimized
   - Touch targets sometimes too small
   - Form fields cramped on mobile

7. **Data Visualization:**
   - Only bar charts used
   - No pie/donut charts for proportions
   - No trend analysis or time-series
   - No heatmaps or advanced analytics

8. **Error Handling:**
   - Generic error messages
   - Network errors not well-handled
   - Retry mechanisms missing
   - User guidance limited

### UX Problems

1. **Discoverability:**
   - Features hidden in menus
   - No inline help or tooltips
   - First-time user guidance missing

2. **Data Management:**
   - No bulk edit operations
   - Undo/redo not available
   - History view limited to audit logs only
   - Deleted items hard to find in Déchet page

3. **Workflow Interruptions:**
   - Multi-step processes require page navigation
   - No progress indicators
   - Context lost between pages

4. **Reporting:**
   - Export-only analytics
   - No custom report builder
   - No scheduled exports
   - No dashboarding beyond home page

5. **Navigation:**
   - Sidebar always takes space
   - Breadcrumbs missing
   - No "back" button between related pages
   - Deep linking not always possible

---

## 10. AI DESIGN BRIEF

### OBJECTIVE
Complete redesign of the Gestion Parc Informatique application to modernize the user experience, improve visual hierarchy, enhance data visualization, and optimize workflows for efficiency and user satisfaction.

### DESIGN VISION
Create a **contemporary, intelligent, and intuitive IT asset management platform** that:
- Streamlines equipment lifecycle management from procurement to disposal
- Provides real-time visibility into inventory and asset deployment
- Enables data-driven decision-making through advanced analytics
- Supports rapid workflow execution with minimal friction
- Maintains professional appearance suitable for enterprise environments
- Scales to handle thousands of assets and users
- Provides role-based personalization

### DESIGN PRINCIPLES

1. **Progressive Disclosure**
   - Show only essential information by default
   - Reveal advanced features on demand
   - Reduce cognitive load on initial view

2. **Contextual Guidance**
   - Inline help and tooltips for complex features
   - Smart defaults based on user role and history
   - Error prevention through validation and warnings

3. **Visual Clarity**
   - Strong visual hierarchy with size, color, weight
   - Distinctive states for critical vs. normal operations
   - Consistent iconography and patterns

4. **Efficient Workflows**
   - Multi-step processes on single page or modal
   - Shortcuts for power users (keyboard commands, bulk operations)
   - Undo capability for destructive actions

5. **Data Transparency**
   - Clear labeling and field explanations
   - Visual indicators of data quality (recent, outdated, missing)
   - Audit trail visible where needed

### COLOR SYSTEM (NEW)

**Primary Colors:**
```
Primary Blue:     #1e40af (action buttons, primary text)
Secondary Teal:   #0d9488 (success, confirmations)
Warning Amber:    #b45309 (caution, warnings)
Danger Red:       #dc2626 (errors, destructive actions)
Info Sky:         #0284c7 (informational messages)

Accent Gradients:
- Blue to Purple: #1e40af → #7c3aed (premium features)
- Green to Teal:  #22c55e → #0d9488 (positive actions)
```

**Neutral Grays (Light Theme):**
```
Surface 0 (Background):   #ffffff (pure white)
Surface 1 (Card):         #f8fafc (very light gray-blue)
Surface 2 (Hover):        #f1f5f9 (light gray-blue)
Text 0 (Primary):         #0f172a (very dark blue)
Text 1 (Secondary):       #334155 (dark gray-blue)
Text 2 (Tertiary):        #64748b (medium gray)
Text 3 (Disabled):        #94a3b8 (light gray)
Border:                   #e2e8f0 (light border)
```

**Neutral Grays (Dark Theme):**
```
Surface 0 (Background):   #0f172a (very dark blue)
Surface 1 (Card):         #1e293b (dark slate)
Surface 2 (Hover):        #334155 (lighter dark)
Text 0 (Primary):         #f1f5f9 (near white)
Text 1 (Secondary):       #cbd5e1 (light gray)
Text 2 (Tertiary):        #94a3b8 (medium gray)
Text 3 (Disabled):        #64748b (darker gray)
Border:                   #475569 (dark border)
```

**Status Colors:**
```
Success:   #22c55e (green, >= 5 units)
Warning:   #eab308 (yellow, 3-4 units)
Critical:  #dc2626 (red, < 3 units)
Inactive:  #94a3b8 (gray)
```

### TYPOGRAPHY SYSTEM

**Font Family:** Inter (system font fallback: -apple-system, BlinkMacSystemFont, 'Segoe UI')

**Scale:**
```
Display:      56px, weight 700 (page titles)
Heading 1:    40px, weight 700 (section headers)
Heading 2:    32px, weight 600 (subsection headers)
Heading 3:    24px, weight 600 (component headers)
Heading 4:    20px, weight 600 (label headers)
Body:         16px, weight 400 (default)
Small:        14px, weight 400 (secondary info)
Caption:      12px, weight 500 (labels, captions)
Code:         14px, weight 500, monospace (serial numbers, codes)
```

**Line Heights:**
```
Display:  1.1
Heading:  1.25
Body:     1.5
Small:    1.4
```

### LAYOUT SYSTEM

**Grid:**
- 12-column responsive grid
- Breakpoints: 480px (mobile), 768px (tablet), 1024px (desktop), 1440px (wide)
- Gutter: 16px (mobile), 24px (tablet+)

**Spacing Scale (4px base):**
```
xs: 4px     (2 units)
sm: 8px     (2 units)
md: 16px    (4 units)
lg: 24px    (6 units)
xl: 32px    (8 units)
2xl: 48px   (12 units)
3xl: 64px   (16 units)
```

**Container Sizes:**
```
Content:  800px (narrow, reading-focused)
Wide:     1200px (standard content)
Full:     100% (edge-to-edge)
Modal:    600px (dialog boxes)
Sidebar:  280px (navigation)
Drawer:   400px (side panels)
```

### COMPONENT DESIGN SPECIFICATIONS

#### Header/Navigation
- **Height:** 64px (compact), 80px (expanded)
- **Logo:** 48x48px, left-aligned with 24px padding
- **User Menu:** Right-aligned, avatar + name dropdown
- **Theme Toggle:** Icon button, 40x40px
- **Search Bar:** Optional, 300px wide, centered
- **Sticky Positioning:** Follows scroll, maintains context

#### Sidebar Navigation
- **Width:** 280px (expanded), 72px (collapsed)
- **Item Height:** 48px
- **Icon Size:** 24px
- **Font:** Body (16px)
- **Hover:** Highlight with subtle background color
- **Active:** Bold text + colored left border (4px)
- **Sections:** Collapsible groups with headers
- **Transition:** Smooth collapse/expand (200ms)

#### Cards & Panels
- **Border Radius:** 8px
- **Shadow (Light):** 0 1px 3px rgba(0,0,0,0.1)
- **Shadow (Hover):** 0 4px 12px rgba(0,0,0,0.15)
- **Padding:** 24px (default), 32px (generous)
- **Border:** 1px solid var(--border)
- **Background:** var(--surface-1)

#### Input Fields
- **Height:** 40px (default), 44px (large)
- **Border Radius:** 6px
- **Border:** 1px solid var(--border)
- **Padding:** 12px 16px
- **Font:** Body 16px
- **Focus:** 2px outline in primary color
- **Placeholder:** var(--text-3), opacity 60%
- **States:** Default, Focus, Disabled, Error
- **Label:** Caption, above input, 8px gap

#### Buttons
- **Height:** 40px (default), 44px (large), 32px (small)
- **Padding:** 12px 20px (default)
- **Border Radius:** 6px
- **Font:** Small 14px, weight 600
- **Icons:** 16x16px, 8px gap to text
- **States:** Default, Hover, Active, Disabled, Loading
- **Variants:**
  - Solid (primary action): Filled background + text
  - Outline (secondary): Border + transparent background
  - Ghost (tertiary): No border, low emphasis
  - Icon: Background-less, 40x40px square

**Color Combinations:**
```
Primary Button:     Blue background, white text
Secondary Button:   Gray background, dark text
Success Button:     Green background, white text
Danger Button:      Red background, white text
```

#### Tables
- **Header Height:** 44px
- **Row Height:** 52px (compact), 60px (generous)
- **Cell Padding:** 16px
- **Borders:** Bottom only (1px light border)
- **Striped:** Optional alternating row colors
- **Hover:** Highlight entire row with surface-2 color
- **Columns:** Sortable with icon (↑↓)
- **Actions:** Right-aligned in final column
- **Dense:** Reduced padding for data-heavy tables

**Pagination:**
```
Format: "Showing 1-10 of 247 items" | Page selector | Items per page
Height: 40px
Styling: Centered, subtle buttons
```

#### Forms
- **Layout:** Vertical (fields stacked)
- **Field Width:** 100% (mobile), 400px (desktop)
- **Gap Between Fields:** 24px
- **Label Style:** Required field * in red
- **Validation Message:** Small text below field, red color
- **Button Group:** Right-aligned at bottom
- **Multi-Step:** Progress indicator at top (visual breadcrumb)

#### Modals
- **Width:** 600px (standard), 400px (small), 900px (large)
- **Position:** Centered, vertically
- **Backdrop:** Overlay with 40% opacity
- **Border Radius:** 12px
- **Header:** Title + close button, bordered bottom
- **Body:** Padded, scrollable if tall
- **Footer:** Action buttons, right-aligned
- **Transitions:** Fade in/out (200ms)

#### Notifications/Toasts
- **Position:** Top-right corner, 16px from edge
- **Width:** 384px (max)
- **Height:** 56px
- **Duration:** 4 seconds (auto-dismiss)
- **Stack:** Up to 3 visible, queue additional
- **Types:** Success (green), Error (red), Warning (amber), Info (blue)
- **Icon + Message + Close button**

#### Badges/Tags
- **Height:** 24px
- **Padding:** 4px 12px
- **Border Radius:** 12px (pill-shaped)
- **Font:** Caption 12px, weight 600
- **Variants:** Solid, Outline, Subtle
- **Colors:** All primary colors available

#### Charts
- **Default Height:** 300px
- **Responsive:** Scale to container width
- **Padding:** 24px internal margins
- **Legend:** Below chart, horizontal layout
- **Tooltip:** Hover popup with formatting
- **Colors:** Use primary palette (blue, teal, amber, red)
- **Grid:** Light dotted lines
- **Axis Labels:** Rotated 45° if many items

#### Loading States
- **Spinner:** 32x32px animated circle
- **Skeleton:** Placeholder boxes matching content shape
- **Progress Bar:** Horizontal bar with % text
- **Duration Indication:** "Loading..." text below spinner
- **Cancellation:** Overlay with cancel button option

#### Empty States
- **Icon:** Large (80x80px) in gray color
- **Heading:** Descriptive title
- **Message:** Explanation text
- **Action Button:** "Create First Item" or "Import Data"
- **Illustration:** Optional background graphic

### LAYOUT SPECIFICATIONS

#### Page Structure
```
┌─────────────────────────────────────────┐
│  Header (64px) - Logo, Nav, Theme     │
├─────────┬───────────────────────────────┤
│         │                               │
│ Sidebar │      Main Content Area        │
│ (280px) │                               │
│         │  ┌─────────────────────────┐  │
│         │  │ Page Header Section     │  │
│         │  ├─────────────────────────┤  │
│         │  │ Filters/Search Bar      │  │
│         │  ├─────────────────────────┤  │
│         │  │ Data Table/Cards        │  │
│         │  │ (Paginated)             │  │
│         │  ├─────────────────────────┤  │
│         │  │ Pagination Controls     │  │
│         │  └─────────────────────────┘  │
│         │                               │
└─────────┴───────────────────────────────┘
```

#### Dashboard Layout (NEW)
```
┌─────────────────────────────────────┐
│        Welcome Message              │
├────────────┬────────────┬───────────┤
│  Stat: X   │  Stat: Y   │  Stat: Z  │
│  Total     │  Critical  │  Active   │
├─────────────────────────────────────┤
│   Chart 1: Stock by Type            │
│   (60% of width, responsive)        │
├─────────────────────────────────────┤
│   Chart 2: Equipment Distribution   │
│   (60% of width, responsive)        │
├─────────────────────────────────────┤
│   Recent Movements (Table)          │
│   (Full width, horizontal scroll)   │
└─────────────────────────────────────┘
```

### INTERACTION PATTERNS

#### Delete Confirmation
```
1. User clicks delete button
2. Confirmation modal appears
3. Shows item name + "Are you sure?"
4. Buttons: Cancel | Delete (red)
5. On delete: Toast confirmation + auto-refresh
```

#### Form Validation
```
1. User fills field
2. On blur: Validate
3. Error message appears below field (red)
4. Field gets red border
5. Submit button disabled until valid
6. On correct input: Message disappears, field is valid
```

#### Search Experience
```
1. User types in search box
2. Debounced API call (300ms delay)
3. Loading spinner appears
4. Results filter in real-time
5. No results: Show empty state with suggestion
6. Clear button (X) resets search
```

#### Bulk Operations
```
1. Checkbox in table header = "select all"
2. Individual row checkboxes
3. Bulk action toolbar appears when items selected
4. Shows count: "3 items selected"
5. Action buttons (Delete, Export, etc.)
6. Select All checkbox shows partial state if not all selected
```

#### Workflow: Import Equipment
```
1. User clicks "Import" button
2. Modal opens with file upload area
3. Drag-drop file or click to browse
4. Preview: Show first 5 rows
5. Validation: Check headers, data types
6. If errors: Show error table with line numbers
7. If valid: Show "Import X items" button
8. On import: Progress bar + results summary
9. Close + auto-refresh table
```

### FEATURE ENHANCEMENTS

#### 1. Advanced Search & Filtering
- **Multi-field Search:**
  - Equipment type
  - Serial number
  - Date range (created/modified)
  - Activity/Department
  - Status (active/inactive)
  - Assigned user

- **Saved Filters:**
  - User can save filter combinations
  - Quick-access filter buttons
  - Shareable filter URLs

- **Smart Search:**
  - Autocomplete suggestions
  - Search history
  - Synonym matching (PC = Computer, Laptop = Portable)

#### 2. Enhanced Dashboard
- **Widget Customization:**
  - Drag-to-reorder widgets
  - Add/remove widgets
  - Resize charts
  - Save layout per user

- **Advanced Analytics:**
  - Trend analysis (equipment added/removed over time)
  - Heat maps (assets by location, service)
  - Pie charts for asset distribution
  - Comparison charts (planned vs. actual inventory)

- **Alerts & Notifications:**
  - Equipment reaching end-of-life
  - Low stock notifications
  - Maintenance reminders
  - Policy violations (e.g., unsupported OS versions)

#### 3. Improved Form UX
- **Multi-Step Forms:**
  - Progress indicator
  - Save draft functionality
  - Resume incomplete form
  - Field-level help text

- **Smart Defaults:**
  - Pre-fill from previous entries
  - Suggest similar equipment
  - Auto-complete based on history

- **Form Validation:**
  - Real-time field validation
  - Visual feedback (green checkmark)
  - Inline help for required fields

#### 4. Mobile Responsiveness
- **Responsive Sidebar:** Hamburger menu on mobile
- **Touch-Friendly:** 44px minimum touch targets
- **Mobile Tables:** Horizontal scroll with sticky first column
- **Full-Screen Forms:** Modal takes full width on mobile
- **Bottom Sheet:** Drawer-style for mobile navigation

#### 5. Accessibility
- **WCAG 2.1 AA Compliance:**
  - Keyboard navigation (Tab, Enter, Esc)
  - Screen reader support (ARIA labels)
  - Color contrast (4.5:1 for text)
  - Focus indicators
  - Alt text for images/icons

- **Keyboard Shortcuts:**
  - "/" = Search
  - "n" = New item
  - "?" = Help
  - "Esc" = Close modal

#### 6. Performance Optimization
- **Virtual Scrolling:** For large tables (1000+ rows)
- **Lazy Loading:** Images, charts load on demand
- **Code Splitting:** Separate bundles per page
- **Caching:** API responses cached locally
- **Pagination:** Default 20 items per page
- **Debouncing:** Search queries (300ms)

#### 7. Audit & Compliance
- **Change History:**
  - Show who changed what and when
  - Diff view for modifications
  - Ability to view previous versions
  - Retention policy (e.g., 2-year history)

- **Audit Trail:**
  - Searchable log
  - Export capability
  - Real-time updates
  - User attribution

#### 8. Collaboration Features
- **Comments/Notes:**
  - Add notes to equipment
  - @ mention other users
  - Notification system
  - Resolved/unresolved state

- **Equipment Assignment:**
  - Assign to user
  - Request equipment
  - Approval workflow
  - Hand-off tracking

### REDESIGNED WORKFLOWS

#### Workflow 1: Import Stock (Optimized)
```
Current (3 clicks):
1. Click "Importer"
2. Browse file
3. Submit

Redesigned (1-2 clicks):
1. Drag file onto table OR click import button
2. Auto-validate and show preview
3. One-click confirm import
4. Real-time progress with item count
5. Show results: X added, Y updated, Z errors
6. Export error report for failed items
7. Table auto-refreshes
```

#### Workflow 2: Find & Edit Equipment (Optimized)
```
Current (4 clicks):
1. Go to Parc page
2. Type search
3. Click edit
4. Change field
5. Save

Redesigned (2-3 clicks):
1. Global search (anywhere in app)
2. Click "Edit" from search result
3. Inline edit OR modal edit
4. Save or Cancel
5. Table updates instantly
```

#### Workflow 3: Track Equipment Disposal (Optimized)
```
Current:
1. Stock page → Find item → Delete
2. Déchet page → Find same item → Restore
3. Item returns to Stock

Redesigned:
1. Stock item → Right-click context menu
2. Option: "Delete & Log Disposal"
3. Quick form: Reason, date, notes
4. Item moves to Déchet with details
5. Later: Déchet item → Restore (one click)
6. Undo history available
```

#### Workflow 4: Create Equipment Movement (Optimized)
```
Current:
1. Click "Nouveau mouvement"
2. Select type (Entrée/Sortie)
3. Select source
4. Select equipment
5. Fill details
6. Save

Redesigned (with smart defaults):
1. Click "Nouveau mouvement"
2. Type equipment name (autocomplete)
3. System suggests type (Enter/Exit)
4. Fill quantity
5. System pre-fills specs from last similar item
6. Save
7. Toast confirmation with undo option
```

### MIGRATION STRATEGY

**Phase 1 (Weeks 1-2): Foundation**
- Implement new color system + typography
- Create new component library
- Redesign sidebar + header
- Modernize form components

**Phase 2 (Weeks 3-4): Pages**
- Redesign Dashboard with new analytics
- Redesign Stock & Parc pages
- Redesign Mouvements & Déchet pages
- Redesign Locaux IT page

**Phase 3 (Week 5): Features**
- Implement advanced search
- Add saved filters
- Add mobile responsiveness
- Add accessibility features

**Phase 4 (Week 6): Polish**
- Performance optimization
- Cross-browser testing
- User testing & feedback
- Bug fixes & refinements

**Phase 5 (Week 7): Launch**
- Documentation
- User training
- Deployment to production
- Post-launch support

### SUCCESS METRICS

1. **User Engagement:**
   - 30% increase in daily active users
   - 40% reduction in time-per-task
   - 25% increase in feature usage

2. **Quality Metrics:**
   - <1% error rate in data imports
   - 99.5% uptime
   - <2 second page load time
   - <200ms API response time

3. **User Satisfaction:**
   - NPS score >50
   - Task completion rate >95%
   - Support ticket reduction >20%
   - User feedback score >4.5/5

4. **Business Impact:**
   - 50% faster inventory audits
   - 30% reduction in asset loss
   - 20% improvement in stock accuracy
   - <1 day equipment deployment time

---

## CONCLUSION

The Gestion Parc Informatique application is a mature, production-ready system with solid functionality across all IT asset management domains. The proposed redesign builds on this foundation by:

1. **Modernizing the visual identity** with a contemporary design system
2. **Improving user workflows** through progressive disclosure and smart defaults
3. **Enhancing data visualization** with advanced charts and analytics
4. **Optimizing performance** for large datasets and many users
5. **Ensuring accessibility** for all user abilities
6. **Adding mobile support** for on-the-go asset management
7. **Strengthening compliance** with audit trails and change history

The redesign maintains all current functionality while significantly improving the user experience, making the platform more intuitive, efficient, and enjoyable to use.

---

**Document Version:** 1.0  
**Last Updated:** June 2026  
**Prepared for:** Development & Design Teams
