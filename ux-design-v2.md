# Product V2 UX Design Document

## Table of Contents
1. [System Architecture & Hierarchy](#system-architecture--hierarchy)
2. [AOP Draft Creation Flow](#aop-draft-creation-flow)
3. [Document Editor Experience](#document-editor-experience)
4. [Process Execution & Monitoring](#process-execution--monitoring)
5. [Exception Handling](#exception-handling)
6. [Guidance Dashboard](#guidance-dashboard)
7. [Triggers & Automation](#triggers--automation)
8. [User Personas & Workflows](#user-personas--workflows)
9. [Visual Design Patterns](#visual-design-patterns)
10. [Future Considerations](#future-considerations)

---

## System Architecture & Hierarchy

### Organizational Structure
```
Organization (Pepsi)
├── Workspace (Finance Dept)
│   ├── Agent (Finance AI Assistant)
│   │   ├── AOP Draft 1
│   │   ├── AOP Draft 2
│   │   └── Published Automations
│   └── Agent (Compliance AI Assistant)
└── Workspace (Logistics Dept)
    └── Agent (Supply Chain AI Assistant)
```

### Navigation Pattern
- **Top Level**: Organization switcher (for users with multi-org access)
- **Second Level**: Workspace tabs/sidebar navigation
- **Third Level**: Agent-specific dashboard with automation lists

---

## AOP Draft Creation Flow

### Entry Points
1. **Upload Existing SOP**
   - Drag-and-drop file upload interface
   - Support for PDF, Word, Excel, and plain text
   - AI extracts structure and converts to AOP format
   
2. **Template Selection**
   - Gallery view of pre-built SOP templates
   - Categories: Finance, HR, Operations, IT, etc.
   - Preview mode before selection
   
3. **Chat-Based Creation**
   - Conversational interface: "Describe the task you want to automate"
   - Progressive disclosure through clarifying questions
   - Real-time AOP generation as context builds

4. **Sample Data Upload**
   - Optional data upload to improve AI understanding
   - Automatic schema detection and field mapping

### Integration Detection Flow
```
User creates SOP → AI detects required books → Integration prompt appears
├── Organization-level credentials exist
│   ├── "Use existing Finance system credentials?" [Use Existing] [Add New]
│   └── Show which systems are already connected
└── No org credentials
    └── "Connect to [Salesforce] to enable this automation" [Connect Now]
```

---

## Document Editor Experience

### AOP Document Structure
- **Header**: Automation title, description, status badge
- **Process Visualization**: Interactive flowchart section
- **Inputs Section**: Auto-detected fields with validation
- **Steps Section**: Rich text with embedded media support
- **Outputs Section**: Expected results and data formats
- **Troubleshooting Guide**: Auto-populated from exception resolutions

### Rich Text Editor Features
- **Media Support**: Drag-and-drop images, screenshots, diagrams
- **Formatting**: Standard rich text (bold, italic, lists, headers)
- **Step Annotations**: Add clarifying notes to specific steps
- **Version History**: Side panel showing document evolution

### Editing Modes

#### 1. Direct Document Editing
- Click-to-edit any section
- Real-time collaboration indicators
- Auto-save every 15 seconds

#### 2. Grammarly-Style Clarification
- Underlined content indicates need for clarification
- Click underlined text → Opens contextual chat bubble
- In-situ clarification without leaving document context

#### 3. Chat-Based Editing
- Persistent chat panel alongside document
- "Make the email subject line more specific"
- "Add a step to validate the data before processing"
- Changes preview in document before applying

### Clarification Flow
```
User makes ambiguous edit → System underlines unclear content → 
User clicks → Contextual chat opens → System asks clarifying questions →
User responds → System updates document → Clarification resolved
```

---

## Process Execution & Monitoring

### Dashboard Overview
- **Metrics Cards**: Success rate, avg execution time, exceptions count
- **Recent Activity**: Timeline of automation runs
- **Quick Actions**: Create new automation, view exceptions, system health

### Published Automation Landing Page
**Priority Order:**
1. **Elevator Pitch Summary** (top banner)
2. **Runs & Outputs** (main content area)
3. **AOP Document** (secondary tab)

### Runs Visualization
```
┌─ Successful Runs ────────────────────────────────┐
│ ✅ Run #1234 - 2 hours ago                       │
│    📊 Processed 45 invoices                      │
│    💰 Total value: $123,456                      │
│    📁 Output: finance_report_2024.xlsx           │
└──────────────────────────────────────────────────┘

┌─ Exception Runs ─────────────────────────────────┐
│ ⚠️  Run #1233 - 4 hours ago                      │
│    🚫 Failed at step 3: Email validation         │
│    📋 3 records processed before failure         │
│    🔧 [Resolve Exception]                        │
└──────────────────────────────────────────────────┘
```

### Live Execution View
- **Progress Indicator**: Document with live cursor showing current step
- **Progress Bar**: Percentage completion estimate
- **No Raw Output**: Only business-relevant information displayed
- **Step Highlighting**: Current step highlighted in AOP document

---

## Exception Handling

### Exception Types & Routing
1. **System Admin Issues**: Infrastructure, connectivity, permissions
2. **Business Logic Issues**: Data validation, business rule violations
3. **Integration Issues**: API failures, authentication problems

### In-Document Exception Experience
- **Exception Marker**: Red comment thread icon on relevant AOP step
- **Business-Friendly Explanation**: "The email address format wasn't recognized"
- **Context Panel**: Shows relevant data, documents, and system state
- **Action Buttons**: Skip, Retry, Manual Input, Get Help

### Exception Chat Interface
```
🤖 System: "The customer email 'john@company' doesn't have a valid domain extension."

📁 [View Email Data] 📊 [Show Validation Rules] 

👤 User: "Use 'john@company.com' instead"

🤖 System: "I've updated the email to john@company.com. Should I:
   1. Continue with this correction
   2. Add this as a validation rule for future runs
   3. Both"

[Continue] [Add Rule] [Both]
```

### Guidance Preservation
- **Failed Attempts**: Store all attempted solutions
- **Successful Patches**: Auto-suggest for similar future exceptions
- **Learning Loop**: Multiple uses → Prompt to add to troubleshooting guide

---

## Guidance Dashboard

### Exception Management Interface
- **Priority Queue**: Highest impact exceptions first
- **Auto-Classification**: System admin, business user, technical issues
- **Grouping Options**: By symptom (default) or by AOP
- **Bulk Actions**: Apply fix to all similar exceptions

### Exception Handler Workflow
1. **Issue Summary**: Business-friendly problem description
2. **Context Investigation**: Access to relevant files, data, logs
3. **Guided Resolution**: Step-by-step fix suggestions
4. **Impact Assessment**: How many other runs affected
5. **Solution Application**: Apply fix to current run and/or future runs

### Self-Healing Pipeline (Future)
```
Issue Detected → Check Knowledge Base → Apply Known Fix → 
├── Success: Log auto-resolution
└── Failure: Create exception with attempted solutions
```

---

## Triggers & Automation

### Trigger Configuration UI

#### Email Trigger
```
┌─ Email Trigger Setup ────────────────────────────┐
│ From: [sales@company.com        ] [Any Address]  │
│ Subject Contains: [invoice]                       │
│ Attachment Type: [PDF] [Excel] [Any]             │
│ Folder: [Inbox/Automation] [Browse...]           │
│                                      [Test] [Save] │
└───────────────────────────────────────────────────┘
```

#### Schedule Trigger
- **Google Calendar-style Interface**: Visual time picker
- **Recurrence Options**: Daily, weekly, monthly, custom cron
- **Timezone Handling**: User's local timezone with UTC conversion
- **Holiday Exclusions**: Skip business holidays option

#### API/Webhook Trigger
```
┌─ API Integration ─────────────────────────────────┐
│ Endpoint: https://api.company.com/webhook/abc123  │
│ [📋 Copy URL] [🔑 Regenerate Key]                │
│                                                   │
│ Authentication: [Bearer Token] [API Key] [None]   │
│ Expected Payload: [JSON Schema Builder]           │
│                                      [Test] [Save] │
└───────────────────────────────────────────────────┘
```

---

## User Personas & Workflows

### Business User (Primary)
**Goals**: Create and monitor automations, resolve business exceptions
**Key Screens**: Dashboard, AOP Editor, Exception Resolution
**Pain Points**: Technical complexity, unclear error messages

### IT Administrator  
**Goals**: Manage integrations, system health, security
**Key Screens**: Organization settings, Integration management, System monitoring
**Pain Points**: Lack of technical details, limited control

### Exception Handler
**Goals**: Quickly resolve automation failures, prevent future issues
**Key Screens**: Guidance Dashboard, Exception details, Bulk resolution tools
**Pain Points**: Context switching, repetitive issues

---

## Visual Design Patterns

### Color System
- **Success**: Green (#10B981) - Completed runs, successful validations
- **Warning**: Amber (#F59E0B) - Pending actions, clarifications needed
- **Error**: Red (#EF4444) - Exceptions, failures, critical issues
- **Info**: Blue (#3B82F6) - System messages, helpful tips
- **Neutral**: Gray (#6B7280) - Secondary text, borders, backgrounds

### Typography Hierarchy
- **H1**: Page titles (32px, Bold)
- **H2**: Section headers (24px, Semi-bold)  
- **H3**: Subsection headers (20px, Semi-bold)
- **Body**: Regular text (16px, Regular)
- **Caption**: Metadata, timestamps (14px, Regular)

### Component Library
- **Status Badges**: Pill-shaped indicators for run status
- **Progress Indicators**: Linear progress bars with percentage
- **Chat Bubbles**: Distinct styling for user vs system messages
- **Exception Cards**: Expandable cards with action buttons
- **Data Tables**: Sortable, filterable tables for runs/exceptions

### Layout Patterns
- **Dashboard**: Card-based grid layout
- **Document Editor**: Two-column layout (document + chat/context)
- **Exception Handling**: Three-panel layout (list + details + actions)
- **Monitoring**: Timeline-based vertical layout

---

## Future Considerations

### Advanced Features (Post-V2)
1. **AOP Section Header Editing**: Allow customization of document structure
2. **Code Visibility for Engineers**: Optional technical view for debugging
3. **Advanced Analytics**: Trend analysis, performance optimization suggestions
4. **Mobile Experience**: Responsive design for exception handling on mobile
5. **Collaboration Features**: Multi-user editing, comments, approvals

### Technical Debt Items
1. **Version History Navigation**: Git-like diff view for AOP changes
2. **Subprocess Management**: Nested automation workflows
3. **Anomaly Detection**: AI-powered issue prediction
4. **Testing Framework**: Comprehensive test suite with sample data
5. **A2A Protocol**: Automation-to-automation communication

### Scalability Considerations
- **Performance**: Lazy loading for large automation lists
- **Search**: Full-text search across AOPs and run histories  
- **Permissions**: Role-based access control at organization/workspace level
- **Audit Trail**: Complete activity logging for compliance
- **Data Retention**: Configurable retention policies for run data

---

## Implementation Priority

### Phase 1 (MVP)
- Basic AOP creation and editing
- Simple exception handling
- Core dashboard and monitoring
- Email and schedule triggers

### Phase 2 (Enhanced)
- Advanced exception guidance
- Self-healing capabilities
- Improved analytics and reporting
- API/webhook triggers

### Phase 3 (Advanced)
- Multi-user collaboration
- Advanced automation features
- Mobile optimization
- Enterprise integrations

---

*This document serves as the foundational UX specification for Product V2. All design decisions should reference back to the user experience goals outlined here, with particular emphasis on reducing technical complexity while maintaining powerful automation capabilities.*
