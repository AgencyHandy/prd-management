# PRD: Order Form Builder

**Priority:** HIGH | **Impact:** Revenue Generation | **Timeline:** Q2 2026

## Context & Problem

Agencies need flexible, customizable order forms to capture leads and process orders efficiently. Current static forms limit:
- Lead qualification and segmentation
- Dynamic pricing based on selections
- Complex service configuration capture
- Integration with existing workflows
- Conversion optimization through form testing

**Revenue Impact:** $30K+ MRR - improved conversion rates, better lead qualification, expanded self-service capabilities.

## Solution Overview

Build a drag-and-drop order form builder that allows agencies to create customized forms for:
1. Service inquiries and lead qualification
2. Package selection with dynamic pricing
3. Project specifications and requirements gathering
4. Automated onboarding and client data collection

## Core Features

### 1. Visual Form Builder
- Drag-and-drop interface with pre-built components
- Real-time preview with mobile responsiveness
- Template library for common agency use cases
- Custom CSS and branding options
- Multi-step form creation with progress indicators

### 2. Smart Form Components
- **Service Selector**: Radio buttons, dropdowns for package/service selection
- **Conditional Logic**: Show/hide fields based on previous answers
- **Pricing Calculator**: Dynamic price updates based on selections
- **File Upload**: Portfolio uploads, project briefs, assets
- **Payment Integration**: Deposit collection, full payment options
- **Calendar Integration**: Meeting scheduling within forms

### 3. Advanced Logic Engine
- Conditional field display/hide rules
- Dynamic pricing calculations
- Service compatibility validations
- Required field logic based on selections
- Custom validation rules and error messages

### 4. Integration Capabilities
- CRM integration for lead management
- Payment gateway connection for deposits/payments
- Email automation trigger setup
- Project management tool integration
- Analytics and conversion tracking

## Technical Requirements

### Backend Architecture
```
/form-builder/
├── models/
│   ├── FormTemplate.js          // Form structure and config
│   ├── FormSubmission.js        // Captured responses
│   ├── FormComponent.js         // Reusable components
│   └── ConditionalRule.js       // Logic rules engine
├── services/
│   ├── builder.service.js       // Form creation/editing
│   ├── submission.service.js    // Response processing
│   ├── pricing.service.js       // Dynamic pricing calculations
│   └── integration.service.js   // Third-party connections
├── processors/
│   ├── lead-processor.js        // Lead qualification
│   └── order-processor.js       // Order creation from forms
└── controllers/
    └── form-builder.controller.js
```

### Form Template Schema
```javascript
FormTemplate: {
  workspaceId: ObjectId,
  name: String,
  slug: String,
  components: [{
    type: 'text' | 'select' | 'radio' | 'checkbox' | 'file' | 'payment' | 'calendar',
    id: String,
    label: String,
    required: Boolean,
    options: Array,
    validation: Object,
    conditionalRules: Array,
    pricing: Object
  }],
  styling: {
    theme: String,
    customCSS: String,
    branding: Object
  },
  integrations: {
    crmWebhook: String,
    emailAutomation: Object,
    paymentGateway: String
  }
}
```

### Frontend Components
- Visual form builder with component palette
- Form preview with responsive testing
- Submission management dashboard
- Analytics and conversion tracking interface
- Public form rendering engine

## User Experience Flow

### Agency Admin Flow
1. Access form builder from admin panel
2. Choose template or start from scratch
3. Drag components from palette to canvas
4. Configure component properties and validation
5. Set up conditional logic and pricing rules
6. Preview form across devices
7. Configure integrations and automation
8. Publish and get embeddable form URL/code

### Client Submission Flow
1. Access form via direct link or website embed
2. Complete multi-step form with progress indication
3. See dynamic pricing updates as selections change
4. Upload required files and documents
5. Schedule meetings if calendar integration enabled
6. Make deposit/payment if required
7. Receive confirmation and next steps automation

## Form Component Library

### Basic Components
- Text Input (single/multi-line)
- Number Input with validation
- Email with format validation
- Phone with country code selection
- Date/Time picker
- Dropdown select (single/multi)
- Radio button groups
- Checkbox groups
- File upload (with restrictions)

### Advanced Components
- Service/Package selector with pricing
- Dynamic pricing calculator display
- Payment gateway integration
- Calendar/meeting scheduler
- Digital signature capture
- Location/address autocomplete
- Rating/satisfaction scales

### Logic Components
- Conditional visibility rules
- Progressive disclosure sections
- Form step/page breaks
- Validation rule builders
- Custom calculation fields

## Integration Specifications

### CRM Integration
- Lead creation with form data mapping
- Tag application based on form responses
- Lead scoring integration
- Duplicate detection and merging

### Payment Gateway
- Stripe Elements for secure payment collection
- PayPal integration for alternative payments
- Deposit vs full payment options
- Prorated pricing for selected services

### Email Automation
- Form submission confirmations
- Lead nurturing sequences based on responses
- Admin notifications for new submissions
- Follow-up scheduling

## Success Metrics

- **Primary:** 35% improvement in form conversion rates
- **Secondary:** 50% reduction in manual data entry time
- **Revenue:** $25K+ monthly revenue from improved lead qualification
- **User:** 90% agency satisfaction with form flexibility

## Acceptance Criteria

### MVP Phase 1: Basic Builder
- [ ] Drag-and-drop form builder interface
- [ ] Essential form components (text, select, radio, checkbox)
- [ ] Basic conditional logic (show/hide based on selection)
- [ ] Form preview and responsive testing
- [ ] Public form publishing with custom URLs
- [ ] Submission management dashboard

### Phase 2: Advanced Features
- [ ] Dynamic pricing calculator components
- [ ] File upload with restrictions and previews
- [ ] Payment gateway integration
- [ ] Multi-step forms with progress indicators
- [ ] Template library for common use cases
- [ ] Custom CSS and advanced styling

### Phase 3: Integrations & Analytics
- [ ] CRM integration (HubSpot, Salesforce, etc.)
- [ ] Email automation integration
- [ ] Calendar scheduling integration
- [ ] Advanced analytics and A/B testing
- [ ] White-label embedding options
- [ ] API for custom integrations

## Technical Considerations

### Performance
- Client-side form validation for immediate feedback
- Lazy loading of conditional components
- Optimized form rendering for mobile devices
- Efficient submission processing and storage

### Security
- CSRF protection for all form submissions
- File upload security scanning
- PCI compliance for payment collection
- Data encryption for sensitive information

### Scalability
- CDN-hosted form assets for global performance
- Database indexing for form lookup and submissions
- Caching strategy for frequently accessed forms
- Rate limiting for form submissions

## Risks & Mitigations

**Risk:** Complex conditional logic causing form errors
**Mitigation:** Comprehensive testing framework and validation rules

**Risk:** Integration failures causing data loss
**Mitigation:** Backup submission storage and retry mechanisms

**Risk:** Performance issues with complex forms
**Mitigation:** Component optimization and progressive loading

## Dependencies

- Payment gateway APIs and compliance requirements
- Email service provider integrations
- File storage and security scanning services
- Analytics and tracking infrastructure

## Questions for Review

1. What's the maximum number of form components per form?
2. Should we support custom JavaScript in forms?
3. What file types and sizes should we allow for uploads?
4. How long should we retain form submission data?

---
**Created:** 2026-03-10 | **Owner:** Product Team | **Reviewer:** @mehzabeen_c