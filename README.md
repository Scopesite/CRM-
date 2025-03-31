# CRM-
Cloud Based CRM To Be Developed For Solopreneur's


# CRM MVP Technical Documentation

## Table of Contents
1. [Project Overview](#project-overview)
2. [System Architecture](#system-architecture)
3. [Database Schema Design](#database-schema-design)
4. [User Interface Design](#user-interface-design)
5. [Implementation Plan](#implementation-plan)
6. [Data Storage Strategy](#data-storage-strategy)
7. [Hosting Implementation](#hosting-implementation)
8. [Integration Details](#integration-details)
9. [Starter Code](#starter-code)
10. [Testing Strategy](#testing-strategy)
11. [Deployment Guide](#deployment-guide)
12. [Maintenance Considerations](#maintenance-considerations)

## Project Overview

This document provides comprehensive technical specifications for a Customer Relationship Management (CRM) Minimum Viable Product (MVP) designed specifically for a one-person website design agency. The system focuses on simplicity, efficiency, and integration capabilities, with a clean, intuitive interface inspired by Monday.com.

### Key Features
- Contact management with spreadsheet import
- Project management with Kanban-style interface
- Client portal for project status viewing
- Communication tools including email integration
- Sales pipeline for opportunity tracking
- Web scraping for lead generation
- Integration with third-party services (Zapier, PulseHD, Wix)
- Reporting and analytics
- Customization options for branding and workflows

### Target User
A solo entrepreneur running a website design agency who needs to manage clients, projects, and communications efficiently without the complexity of enterprise CRM solutions.

## System Architecture

The CRM MVP follows a modern web application architecture:

```
+-------------------+      +-------------------+      +-------------------+
|                   |      |                   |      |                   |
|  Client Browser   |<---->|  Express Server   |<---->|  MongoDB Database |
|  (React Frontend) |      |  (Node.js)        |      |                   |
|                   |      |                   |      |                   |
+-------------------+      +-------------------+      +-------------------+
         ^                         ^                          ^
         |                         |                          |
         v                         v                          v
+-------------------+      +-------------------+      +-------------------+
|                   |      |                   |      |                   |
|  Authentication   |      |  External APIs    |      |  File Storage     |
|  (JWT/Firebase)   |      |  (Zapier/PulseHD) |      |  (Firebase/S3)    |
|                   |      |                   |      |                   |
+-------------------+      +-------------------+      +-------------------+
```

### Tech Stack
- **Frontend**: React.js with Material-UI or Chakra UI
- **Backend**: Node.js with Express
- **Database**: MongoDB (document-based for flexibility)
- **Authentication**: JWT with option to use Firebase Authentication
- **File Storage**: Firebase Storage or Amazon S3
- **Hosting**: Railway, Firebase Hosting, or GitHub Pages with backend on Railway

## Database Schema Design

The MongoDB schema is designed to be flexible and scalable, with the following collections:

### Users Collection
```javascript
{
  _id: ObjectId,
  email: String,
  password: String, // Hashed
  firstName: String,
  lastName: String,
  role: String,
  avatar: String,
  companyName: String,
  companyLogo: String,
  brandColors: {
    primary: String,
    secondary: String,
    accent: String
  },
  settings: {
    emailNotifications: Boolean,
    defaultView: String,
    timezone: String
  },
  createdAt: Date,
  updatedAt: Date
}
```

### Contacts Collection
```javascript
{
  _id: ObjectId,
  userId: ObjectId, // Reference to user
  type: String, // "prospect", "client", "past_client"
  status: String, // "active", "inactive", "archived"
  firstName: String,
  lastName: String,
  company: String,
  position: String,
  email: String,
  phone: String,
  website: String,
  source: String, // How they were acquired
  tags: [String],
  notes: String,
  customFields: {}, // Flexible object for custom fields
  lastContacted: Date,
  leadScore: Number,
  socialProfiles: {
    linkedin: String,
    twitter: String,
    facebook: String,
    instagram: String
  },
  address: {
    street: String,
    city: String,
    state: String,
    zip: String,
    country: String
  },
  createdAt: Date,
  updatedAt: Date
}
```

### Projects Collection
```javascript
{
  _id: ObjectId,
  userId: ObjectId, // Reference to user
  contactId: ObjectId, // Reference to client
  name: String,
  description: String,
  type: String, // "website", "social_media", "graphic_design"
  status: String, // "proposal", "in_progress", "review", "completed"
  startDate: Date,
  dueDate: Date,
  completedDate: Date,
  budget: Number,
  invoiceStatus: String, // "not_invoiced", "invoiced", "paid"
  tasks: [
    {
      name: String,
      description: String,
      status: String, // "todo", "in_progress", "completed"
      dueDate: Date,
      completedDate: Date,
      timeTracked: Number // in minutes
    }
  ],
  files: [
    {
      name: String,
      url: String,
      type: String,
      size: Number,
      uploadedAt: Date
    }
  ],
  notes: String,
  createdAt: Date,
  updatedAt: Date
}
```

### Communications Collection
```javascript
{
  _id: ObjectId,
  userId: ObjectId, // Reference to user
  contactId: ObjectId, // Reference to contact
  projectId: ObjectId, // Optional reference to project
  type: String, // "email", "call", "meeting", "note"
  direction: String, // "inbound", "outbound"
  subject: String,
  content: String,
  attachments: [
    {
      name: String,
      url: String
    }
  ],
  status: String, // "draft", "sent", "opened", "replied"
  scheduledFor: Date,
  completedAt: Date,
  duration: Number, // For calls/meetings (in minutes)
  createdAt: Date,
  updatedAt: Date
}
```

### Deals Collection
```javascript
{
  _id: ObjectId,
  userId: ObjectId, // Reference to user
  contactId: ObjectId, // Reference to contact
  name: String,
  value: Number,
  currency: String,
  stage: String, // "lead", "qualified", "proposal", "negotiation", "closed_won", "closed_lost"
  probability: Number, // 0-100
  expectedCloseDate: Date,
  actualCloseDate: Date,
  products: [
    {
      productId: ObjectId, // Reference to product
      quantity: Number,
      price: Number,
      discount: Number
    }
  ],
  notes: String,
  createdAt: Date,
  updatedAt: Date
}
```

### Products Collection
```javascript
{
  _id: ObjectId,
  userId: ObjectId, // Reference to user
  name: String,
  description: String,
  category: String, // "website", "social_media", "graphic_design"
  price: Number,
  currency: String,
  unit: String, // "hour", "project", "month"
  active: Boolean,
  createdAt: Date,
  updatedAt: Date
}
```

### Activities Collection
```javascript
{
  _id: ObjectId,
  userId: ObjectId, // Reference to user
  entityType: String, // "contact", "project", "deal", etc.
  entityId: ObjectId, // Reference to the entity
  action: String, // "created", "updated", "deleted", "status_changed", etc.
  description: String,
  metadata: {}, // Additional context about the activity
  createdAt: Date
}
```

### Integrations Collection
```javascript
{
  _id: ObjectId,
  userId: ObjectId, // Reference to user
  service: String, // "zapier", "pulsehd", "wix", etc.
  status: String, // "active", "inactive", "error"
  credentials: {}, // Encrypted credentials
  settings: {}, // Service-specific settings
  lastSyncAt: Date,
  createdAt: Date,
  updatedAt: Date
}
```

## User Interface Design

The UI follows a clean, intuitive design inspired by Monday.com, with a focus on simplicity and efficiency.

### Dashboard Screen
```
+-------------------------------------------------------+
| LOGO   Search...                  Notifications Profile|
+-------+-----------------------------------------------+
|       |                                               |
| MENU  |  DASHBOARD                                    |
|       |                                               |
| Home  |  +------------+  +------------+  +-----------+|
|       |  | Active     |  | Projects   |  | Revenue   ||
|Contacts|  | Clients   |  | by Status  |  | This Month||
|       |  |            |  |            |  |           ||
|Projects|  | 12        |  | [PIE CHART]|  | $4,250    ||
|       |  +------------+  +------------+  +-----------+|
|Deals  |                                               |
|       |  RECENT ACTIVITY                              |
|Calendar|  +-------------------------------------------+|
|       |  | • New contact added: John Smith           ||
|Reports|  | • Project status changed: Website Redesign ||
|       |  | • Email sent to: Jane Doe                 ||
|Settings|  +-------------------------------------------+|
|       |                                               |
|       |  UPCOMING TASKS                               |
|       |  +-------------------------------------------+|
|       |  | □ Call John about proposal (Today)        ||
|       |  | □ Finish homepage design (Tomorrow)       ||
|       |  | □ Send invoice to ABC Corp (Apr 3)        ||
|       |  +-------------------------------------------+|
|       |                                               |
+-------+-----------------------------------------------+
```

### Contacts Screen
```
+-------------------------------------------------------+
| LOGO   Search...                  Notifications Profile|
+-------+-----------------------------------------------+
|       |                                               |
| MENU  |  CONTACTS                        + Add Contact|
|       |                                               |
| Home  |  Filter: All ▼  Tags: All ▼  Search: [      ]|
|       |                                               |
|Contacts|  +-------------------------------------------+|
|       |  | Name          | Company    | Status | Tags||
|Projects|  +-------------------------------------------+|
|       |  | John Smith    | ABC Corp   | Client | Web ||
|Deals  |  | Jane Doe      | XYZ Inc    |Prospect|Social||
|       |  | Bob Johnson   | 123 LLC    | Client |Design||
|Calendar|  | Alice Brown   | Best Co    |Prospect| Web ||
|       |  +-------------------------------------------+|
|Reports|                                               |
|       |  QUICK ACTIONS                                |
|Settings|  +------------+  +------------+  +-----------+|
|       |  | Call        |  | Email      |  | Add Task ||
|       |  +------------+  +------------+  +-----------+|
|       |                                               |
+-------+-----------------------------------------------+
```

### Project Management Screen
```
+-------------------------------------------------------+
| LOGO   Search...                  Notifications Profile|
+-------+-----------------------------------------------+
|       |                                               |
| MENU  |  PROJECTS                      + New Project  |
|       |                                               |
| Home  |  Filter: All ▼  Type: All ▼  Search: [      ]|
|       |                                               |
|Contacts|  +-------------------------------------------+|
|       |  | PROPOSAL | IN PROGRESS | REVIEW | COMPLETED|
|Projects|  +-------------------------------------------+|
|       |  |          |             |        |          |
|Deals  |  | ABC Corp |  XYZ Inc    | 123 LLC|          |
|       |  | Website  |  Social     | Logo   |          |
|Calendar|  | Redesign |  Campaign   | Design |          |
|       |  |          |             |        |          |
|Reports|  | Best Co  |             |        |          |
|       |  | E-commerce|             |        |          |
|Settings|  | Site     |             |        |          |
|       |  |          |             |        |          |
|       |  +-------------------------------------------+|
|       |                                               |
+-------+-----------------------------------------------+
```

### Client Portal Screen
```
+-------------------------------------------------------+
| YOUR LOGO                                      Log Out|
+-------------------------------------------------------+
|                                                       |
|  PROJECT: WEBSITE REDESIGN                            |
|                                                       |
|  Status: IN PROGRESS                                  |
|  Due Date: April 15, 2025                             |
|                                                       |
|  +---------------------------------------------------+|
|  | MILESTONES                                        ||
|  +---------------------------------------------------+|
|  | ✓ Project Kickoff                                 ||
|  | ✓ Wireframes Approved                             ||
|  | □ Homepage Design (In Review)                     ||
|  | □ Inner Pages Design                              ||
|  | □ Development                                     ||
|  | □ Testing                                         ||
|  | □ Launch                                          ||
|  +---------------------------------------------------+|
|                                                       |
|  FILES                                                |
|  +---------------------------------------------------+|
|  | • Wireframes.pdf                                  ||
|  | • Homepage_Draft_v1.jpg                           ||
|  | • Homepage_Draft_v2.jpg                           ||
|  +---------------------------------------------------+|
|                                                       |
|  MESSAGES                                  + New Message|
|  +---------------------------------------------------+|
|  | You: Here's the latest homepage design            ||
|  | Designer: What do you think of the new header?    ||
|  +---------------------------------------------------+|
|                                                       |
+-------------------------------------------------------+
```

## Implementation Plan

To avoid overwhelming Replit's capabilities, the implementation is divided into stages:

### Stage 1: Core Foundation (Weeks 1-2)
1. **Project Setup**
   - Initialize Replit Node.js project
   - Set up folder structure
   - Install essential dependencies
   - Configure environment variables

2. **Database Connection**
   - Set up MongoDB connection
   - Create basic schema models (Users, Contacts)
   - Implement basic CRUD operations

3. **Authentication System**
   - Implement user registration and login
   - Set up JWT authentication
   - Create protected routes
   - Implement password reset functionality

4. **Basic UI Framework**
   - Set up React with basic components
   - Implement responsive layout
   - Create navigation structure
   - Design login and dashboard screens

### Stage 2: Essential Features (Weeks 3-4)
1. **Contact Management**
   - Implement contact listing and filtering
   - Create contact detail view
   - Add contact creation and editing
   - Implement CSV import functionality

2. **Project Management**
   - Create project listing and filtering
   - Implement Kanban board view
   - Add project creation and editing
   - Implement task management

3. **Basic Communication Tools**
   - Set up email templates
   - Implement email sending functionality
   - Create call logging system
   - Add basic notification system

4. **File Management**
   - Set up file upload system
   - Implement file organization
   - Create file preview functionality
   - Add file sharing capabilities

### Stage 3: Advanced Features (Weeks 5-6)
1. **Client Portal**
   - Create client-facing views
   - Implement project status display
   - Add file sharing for clients
   - Create messaging system

2. **Sales Pipeline**
   - Implement deal tracking
   - Create proposal generation
   - Add quote builder
   - Implement contract management

3. **Reporting & Analytics**
   - Create dashboard widgets
   - Implement basic reports
   - Add revenue tracking
   - Create time analysis tools

4. **Customization Options**
   - Implement custom fields
   - Add workflow editor
   - Create form builder
   - Implement branding options

### Stage 4: Integrations & Optimization (Weeks 7-8)
1. **Third-Party Integrations**
   - Implement Zapier connector
   - Add webhook support
   - Create PulseHD dialler integration
   - Implement Wix integration

2. **Web Scraping & Lead Generation**
   - Create prospect finder
   - Implement lead scoring
   - Add automated research
   - Create lead nurturing workflows

3. **Performance Optimization**
   - Implement pagination
   - Optimize database queries
   - Add caching mechanisms
   - Improve load times

4. **Testing & Deployment**
   - Conduct unit and integration testing
   - Perform security audits
   - Prepare deployment pipeline
   - Deploy to production environment

## Data Storage Strategy

The CRM MVP will use a combination of storage solutions to optimize performance, cost, and scalability:

### 1. MongoDB Database (Primary Data Storage)
- **Usage**: Store all structured data including user information, contacts, projects, deals, and communications
- **Implementation**: 
  - Use MongoDB Atlas for cloud-hosted database
  - Configure connection string in environment variables
  - Implement mongoose schemas for data validation
  - Set up indexes for frequently queried fields
  - Enable automatic backups

### 2. Firebase Storage (File Storage)
- **Usage**: Store all uploaded files including project deliverables, contact attachments, and profile images
- **Implementation**:
  - Configure Firebase Storage bucket
  - Set up security rules to restrict access
  - Implement file upload/download functionality
  - Generate signed URLs for temporary access
  - Create folder structure for organization

### 3. Local Storage (Client-Side Caching)
- **Usage**: Cache frequently accessed data and user preferences
- **Implementation**:
  - Store authentication tokens
  - Cache dashboard data
  - Save user interface preferences
  - Implement expiration for cached data

### 4. Redis Cache (Optional for Performance)
- **Usage**: Cache database queries and API responses
- **Implementation**:
  - Set up Redis instance on Railway
  - Configure cache expiration policies
  - Implement cache invalidation on data updates
  - Use for high-frequency read operations

### Data Migration and Backup Strategy
1. **Regular Backups**:
   - Daily automated backups of MongoDB database
   - Weekly backups of Firebase Storage
   - Retention policy of 30 days for daily backups

2. **Export/Import Functionality**:
   - Implement data export to CSV/JSON
   - Create import functionality for data migration
   - Maintain data integrity during imports

3. **Versioning**:
   - Track changes to critical data
   - Implement soft delete for recoverable operations
   - Store activity logs for audit purposes

## Hosting Implementation

The CRM MVP can be deployed using several hosting options, with a recommended approach leveraging your existing services:

### Recommended Approach: Railway + Firebase

#### 1. Railway (Backend Hosting)
- **Components Hosted**:
  - Node.js/Express API server
  - MongoDB database (or connect to MongoDB Atlas)
  - Redis cache (optional)
- **Implementation Steps**:
  1. Create a new Railway project
  2. Connect to GitHub repository
  3. Configure environment variables
  4. Set up automatic deployments from main branch
  5. Configure custom domain (optional)

#### 2. Firebase (Frontend & Storage)
- **Components Hosted**:
  - React frontend application
  - User authentication
  - File storage
- **Implementation Steps**:
  1. Create a Firebase project
  2. Configure Firebase Authentication
  3. Set up Firebase Storage
  4. Deploy React app to Firebase Hosting
  5. Configure custom domain (optional)

#### 3. GitHub (Version Control & CI/CD)
- **Components**:
  - Source code repository
  - CI/CD pipeline
  - Issue tracking
- **Implementation Steps**:
  1. Create GitHub repository
  2. Set up branch protection rules
  3. Configure GitHub Actions for CI/CD
  4. Connect to Railway and Firebase for automatic deployments

### Alternative Approaches

#### Option 1: Full Firebase Stack
- **Pros**: Simplified management, integrated services
- **Cons**: Higher cost for database operations
- **Components**: Firebase Hosting, Firebase Authentication, Cloud Firestore, Firebase Storage, Cloud Functions

#### Option 2: Railway Full Stack
- **Pros**: Unified platform, simplified scaling
- **Cons**: Requires external file storage solution
- **Components**: Railway for Node.js, MongoDB, Redis, and static hosting

#### Option 3: Hybrid with Replit
- **Pros**: Development and production in same environment
- **Cons**: Limited resources on free tier
- **Components**: Replit for development and testing, Railway/Firebase for production

### Deployment Workflow
1. Develop and test on Replit
2. Push changes to GitHub repository
3. GitHub Actions runs tests and builds application
4. On successful build, deploy backend to Railway
5. Deploy frontend to Firebase Hosting
6. Run post-deployment tests

## Integration Details

### 1. Zapier Integration

Zapier allows connecting the CRM to thousands of other applications without custom code.

#### Implementation
```javascript
// src/controllers/integrations/zapier.js
const Integration = require('../../models/Integration');
const axios = require('axios');

// Register Zapier webhook
exports.registerWebhook = async (req, res) => {
  try {
    const { userId, webhookUrl, triggerEvent } = req.body;
    
    const integration = new Integration({
      userId,
      service: 'zapier',
      status: 'active',
      credentials: {
        webhookUrl
      },
      settings: {
        triggerEvent
      }
    });
    
    await integration.save();
    
    res.status(201).json({
      success: true,
      data: integration
    });
  } catch (error) {
    res.status(400).json({
      success: false,
      error: error.message
    });
  }
};

// Trigger Zapier webhook
exports.triggerWebhook = async (userId, triggerEvent, data) => {
  try {
    const integration = await Integration.findOne({
      userId,
      service: 'zapier',
      status: 'active',
      'settings.triggerEvent': triggerEvent
    });
    
    if (!integration) {
      return {
        success: false,
        error: 'No active Zapier integration found for this trigger'
      };
    }
    
    const response = await axios.post(integration.credentials.webhookUrl, {
      event: triggerEvent,
      data
    });
    
    return {
      success: true,
      data: response.data
    };
  } catch (error) {
    return {
      success: false,
      error: error.message
    };
  }
};
```

#### Supported Triggers
- New contact created
- Project status changed
- Deal stage updated
- Task completed
- Email sent/received

#### Supported Actions
- Create new contact
- Update project status
- Add task
- Send email
- Create calendar event

### 2. PulseHD Dialler Integration

PulseHD integration enables click-to-call functionality directly from the CRM.

#### Implementation
```javascript
// src/controllers/integrations/pulseHD.js
const Integration = require('../../models/Integration');
const axios = require('axios');

// Configure PulseHD integration
exports.configurePulseHD = async (req, res) => {
  try {
    const { userId, apiKey, apiSecret } = req.body;
    
    const integration = new Integration({
      userId,
      service: 'pulsehd',
      status: 'active',
      credentials: {
        apiKey,
        apiSecret
      }
    });
    
    await integration.save();
    
    res.status(201).json({
      success: true,
      data: integration
    });
  } catch (error) {
    res.status(400).json({
      success: false,
      error: error.message
    });
  }
};

// Make call via PulseHD
exports.makeCall = async (req, res) => {
  try {
    const { userId, contactId, phoneNumber } = req.body;
    
    const integration = await Integration.findOne({
      userId,
      service: 'pulsehd',
      status: 'active'
    });
    
    if (!integration) {
      return res.status(400).json({
        success: false,
        error: 'No active PulseHD integration found'
      });
    }
    
    // PulseHD API call
    const response = await axios.post('https://api.pulsehd.com/v1/calls', {
      phone_number: phoneNumber,
      contact_id: contactId
    }, {
      headers: {
        'Authorization': `Bearer ${integration.credentials.apiKey}`,
        'Content-Type': 'application/json'
      }
    });
    
    res.status(200).json({
      success: true,
      data: response.data
    });
  } catch (error) {
    res.status(400).json({
      success: false,
      error: error.message
    });
  }
};
```

#### Features
- Click-to-call from contact records
- Call logging and recording
- Sequential calling for outreach campaigns
- Call analytics and reporting

### 3. Wix Integration

Wix integration allows monitoring client websites built on the Wix platform.

#### Implementation
```javascript
// src/controllers/integrations/wix.js
const Integration = require('../../models/Integration');
const axios = require('axios');

// Configure Wix integration
exports.configureWix = async (req, res) => {
  try {
    const { userId, apiKey, siteId } = req.body;
    
    const integration = new Integration({
      userId,
      service: 'wix',
      status: 'active',
      credentials: {
        apiKey,
        siteId
      }
    });
    
    await integration.save();
    
    res.status(201).json({
      success: true,
      data: integration
    });
  } catch (error) {
    res.status(400).json({
      success: false,
      error: error.message
    });
  }
};

// Get Wix site analytics
exports.getSiteAnalytics = async (req, res) => {
  try {
    const { userId } = req.params;
    
    const integration = await Integration.findOne({
      userId,
      service: 'wix',
      status: 'active'
    });
    
    if (!integration) {
      return res.status(400).json({
        success: false,
        error: 'No active Wix integration found'
      });
    }
    
    // Wix API call
    const response = await axios.get(`https://www.wixapis.com/site-analytics/v1/analytics/site/${integration.credentials.siteId}`, {
      headers: {
        'Authorization': integration.credentials.apiKey,
        'Content-Type': 'application/json'
      }
    });
    
    res.status(200).json({
      success: true,
      data: response.data
    });
  } catch (error) {
    res.status(400).json({
      success: false,
      error: error.message
    });
  }
};
```

#### Features
- Monitor website traffic and performance
- Track form submissions
- View site health metrics
- Receive downtime alerts

## Starter Code

Here's a complete starter code package for Replit that includes the basic structure and core functionality:

### Project Structure
```
crm-mvp/
├── .env
├── .gitignore
├── package.json
├── index.js
├── public/
│   ├── css/
│   │   └── styles.css
│   ├── js/
│   │   └── main.js
│   └── images/
│       └── logo.png
├── src/
│   ├── config/
│   │   ├── database.js
│   │   └── passport.js
│   ├── controllers/
│   │   ├── authController.js
│   │   ├── contactController.js
│   │   └── projectController.js
│   ├── models/
│   │   ├── User.js
│   │   ├── Contact.js
│   │   └── Project.js
│   ├── routes/
│   │   ├── authRoutes.js
│   │   ├── contactRoutes.js
│   │   └── projectRoutes.js
│   ├── middleware/
│   │   └── auth.js
│   ├── utils/
│   │   └── errorHandler.js
│   └── services/
│       └── emailService.js
└── views/
    ├── layouts/
    │   └── main.ejs
    ├── partials/
    │   ├── header.ejs
    │   ├── footer.ejs
    │   └── sidebar.ejs
    └── pages/
        ├── login.ejs
        ├── dashboard.ejs
        ├── contacts.ejs
        └── projects.ejs
```

### Core Files

#### index.js (Entry Point)
```javascript
const express = require('express');
const mongoose = require('mongoose');
const dotenv = require('dotenv');
const cors = require('cors');
const path = require('path');
const authRoutes = require('./src/routes/authRoutes');
const contactRoutes = require('./src/routes/contactRoutes');
const projectRoutes = require('./src/routes/projectRoutes');

// Load environment variables
dotenv.config();

// Initialize Express app
const app = express();

// Middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(cors());

// Set static folder
app.use(express.static(path.join(__dirname, 'public')));

// Set view engine
app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));

// Connect to MongoDB
mongoose.connect(process.env.MONGODB_URI || 'mongodb+srv://username:password@cluster.mongodb.net/crm-mvp', {
  useNewUrlParser: true,
  useUnifiedTopology: true
})
.then(() => console.log('MongoDB Connected'))
.catch(err => console.log('MongoDB Connection Error:', err));

// Routes
app.use('/api/auth', authRoutes);
app.use('/api/contacts', contactRoutes);
app.use('/api/projects', projectRoutes);

// Frontend routes
app.get('/', (req, res) => {
  res.render('login');
});

app.get('/dashboard', (req, res) => {
  res.render('dashboard');
});

app.get('/contacts', (req, res) => {
  res.render('contacts');
});

app.get('/projects', (req, res) => {
  res.render('projects');
});

// Start server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

#### src/models/User.js
```javascript
const mongoose = require('mongoose');
const bcrypt = require('bcrypt');

const userSchema = new mongoose.Schema({
  email: {
    type: String,
    required: true,
    unique: true,
    trim: true,
    lowercase: true
  },
  password: {
    type: String,
    required: true
  },
  firstName: {
    type: String,
    required: true
  },
  lastName: {
    type: String,
    required: true
  },
  companyName: {
    type: String,
    default: ''
  },
  companyLogo: {
    type: String,
    default: ''
  },
  brandColors: {
    primary: {
      type: String,
      default: '#4a6cf7'
    },
    secondary: {
      type: String,
      default: '#6c757d'
    },
    accent: {
      type: String,
      default: '#ffc107'
    }
  },
  settings: {
    emailNotifications: {
      type: Boolean,
      default: true
    },
    defaultView: {
      type: String,
      default: 'dashboard'
    },
    timezone: {
      type: String,
      default: 'UTC'
    }
  }
}, { timestamps: true });

// Password hashing middleware
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  
  try {
    const salt = await bcrypt.genSalt(10);
    this.password = await bcrypt.hash(this.password, salt);
    next();
  } catch (error) {
    next(error);
  }
});

// Method to compare passwords
userSchema.methods.comparePassword = async function(candidatePassword) {
  return bcrypt.compare(candidatePassword, this.password);
};

module.exports = mongoose.model('User', userSchema);
```

#### src/routes/authRoutes.js
```javascript
const express = require('express');
const router = express.Router();
const jwt = require('jsonwebtoken');
const User = require('../models/User');
const auth = require('../middleware/auth');

// Register
router.post('/register', async (req, res) => {
  try {
    const user = new User(req.body);
    await user.save();
    
    const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET, {
      expiresIn: '7d'
    });
    
    res.status(201).send({ user, token });
  } catch (error) {
    res.status(400).send(error);
  }
});

// Login
router.post('/login', async (req, res) => {
  try {
    const { email, password } = req.body;
    const user = await User.findOne({ email });
    
    if (!user) {
      return res.status(401).send({ error: 'Login failed! Check authentication credentials' });
    }
    
    const isPasswordMatch = await user.comparePassword(password);
    
    if (!isPasswordMatch) {
      return res.status(401).send({ error: 'Login failed! Check authentication credentials' });
    }
    
    const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET, {
      expiresIn: '7d'
    });
    
    res.send({ user, token });
  } catch (error) {
    res.status(400).send(error);
  }
});

// Get user profile
router.get('/profile', auth, async (req, res) => {
  res.send(req.user);
});

// Update user profile
router.patch('/profile', auth, async (req, res) => {
  const updates = Object.keys(req.body);
  const allowedUpdates = ['firstName', 'lastName', 'companyName', 'companyLogo', 'brandColors', 'settings'];
  const isValidOperation = updates.every(update => allowedUpdates.includes(update));
  
  if (!isValidOperation) {
    return res.status(400).send({ error: 'Invalid updates!' });
  }
  
  try {
    updates.forEach(update => req.user[update] = req.body[update]);
    await req.user.save();
    res.send(req.user);
  } catch (error) {
    res.status(400).send(error);
  }
});

module.exports = router;
```

#### src/middleware/auth.js
```javascript
const jwt = require('jsonwebtoken');
const User = require('../models/User');

const auth = async (req, res, next) => {
  try {
    const token = req.header('Authorization').replace('Bearer ', '');
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    const user = await User.findOne({ _id: decoded.id });

    if (!user) {
      throw new Error();
    }

    req.token = token;
    req.user = user;
    next();
  } catch (error) {
    res.status(401).send({ error: 'Please authenticate.' });
  }
};

module.exports = auth;
```

#### views/layouts/main.ejs
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>CRM MVP</title>
  <link rel="stylesheet" href="/css/styles.css">
  <!-- Add any other CSS libraries here -->
</head>
<body>
  <div class="container">
    <%- include('../partials/header') %>
    <div class="main-content">
      <%- include('../partials/sidebar') %>
      <div class="content">
        <%- body %>
      </div>
    </div>
    <%- include('../partials/footer') %>
  </div>
  
  <script src="/js/main.js"></script>
  <!-- Add any other JS libraries here -->
</body>
</html>
```

#### package.json
```json
{
  "name": "crm-mvp",
  "version": "1.0.0",
  "description": "CRM MVP for one-person website design agency",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "dependencies": {
    "bcrypt": "^5.1.0",
    "cors": "^2.8.5",
    "dotenv": "^16.0.3",
    "ejs": "^3.1.9",
    "express": "^4.18.2",
    "jsonwebtoken": "^9.0.0",
    "mongoose": "^7.0.3",
    "multer": "^1.4.5-lts.1"
  },
  "devDependencies": {
    "nodemon": "^2.0.22"
  }
}
```

#### .env
```
PORT=3000
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/crm-mvp
JWT_SECRET=your_jwt_secret
NODE_ENV=development
```

## Testing Strategy

To ensure the CRM MVP functions correctly, implement the following testing strategy:

### 1. Unit Testing
- Test individual components and functions
- Use Jest for JavaScript testing
- Focus on models, controllers, and utility functions
- Aim for 70%+ code coverage

### 2. Integration Testing
- Test API endpoints
- Verify database operations
- Test authentication flows
- Ensure proper error handling

### 3. UI Testing
- Test responsive design
- Verify form submissions
- Test navigation flows
- Ensure accessibility compliance

### 4. Performance Testing
- Test load times
- Verify database query performance
- Test file upload/download speeds
- Ensure smooth UI interactions

### 5. Security Testing
- Test authentication and authorization
- Verify input validation
- Check for common vulnerabilities
- Test API rate limiting

## Deployment Guide

### Prerequisites
- Node.js and npm installed
- MongoDB Atlas account
- Firebase account
- Railway account
- GitHub account

### Deployment Steps

#### 1. Prepare the Application
```bash
# Install dependencies
npm install

# Build the application
npm run build

# Run tests
npm test
```

#### 2. Set Up MongoDB Atlas
1. Create a new cluster
2. Set up database user
3. Configure network access
4. Get connection string

#### 3. Set Up Firebase
1. Create a new project
2. Enable Authentication
3. Set up Storage
4. Get configuration keys

#### 4. Deploy to Railway
1. Connect GitHub repository
2. Configure environment variables
3. Set up automatic deployments
4. Verify deployment

#### 5. Deploy Frontend to Firebase
1. Install Firebase CLI
2. Initialize Firebase project
3. Deploy to Firebase Hosting
4. Configure custom domain (optional)

## Maintenance Considerations

### 1. Regular Updates
- Update dependencies monthly
- Apply security patches immediately
- Review and update API integrations quarterly

### 2. Backup Strategy
- Daily automated backups
- Weekly manual backups
- Test restore procedures quarterly

### 3. Monitoring
- Set up uptime monitoring
- Configure error logging
- Monitor database performance
- Track API usage

### 4. Scaling Considerations
- Implement database indexing
- Use caching for frequent queries
- Consider serverless functions for background tasks
- Implement pagination for large datasets

### 5. Future Enhancements
- Mobile application
- Advanced analytics
- AI-powered lead scoring
- Automated marketing campaigns
- Client self-service portal
