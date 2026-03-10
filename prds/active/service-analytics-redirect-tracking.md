# PRD: Service Analytics & Redirect Tracking  

**Priority**: CRITICAL (Revenue Intelligence Blind Spot)  
**Impact**: Revenue - Cannot optimize conversion funnels  
**Effort**: Medium  

## Problem Statement

Service redirection feature is technically complete but analytically blind. Zero PostHog tracking means we cannot measure redirect effectiveness, conversion rates, or revenue attribution from redirect destinations.

**Evidence**: Complete redirection module exists (`/server/redirection/`) with frontend UI, but no analytics events tracked. Cannot optimize redirect strategies or measure ROI.

## Business Impact

### Revenue Impact
- **Blind funnel optimization**: Can't improve redirect conversion rates
- **Lost attribution**: Revenue from redirects not measurable
- **Strategic decisions**: No data to guide redirect strategy
- **ROI mystery**: Investment in redirect feature unmeasurable

### Analytics Gap
- **No conversion data**: Pre/post redirect purchase tracking missing  
- **No A/B testing**: Cannot test different redirect strategies
- **No user journey**: Redirect impact on customer lifecycle unknown
- **No optimization**: Can't identify high/low performing redirects

## Success Metrics

### Primary KPIs
- **Redirect engagement**: 30%+ of users interact with redirects
- **Conversion attribution**: Track 100% of redirect → purchase journeys  
- **Redirect performance**: Identify top 10% performing redirect strategies
- **Revenue attribution**: Measure redirect impact on customer LTV

### Tracking Events
- `redirect_triggered` (service_id, redirect_type, destination_url)
- `redirect_completed` (user_agent, success/failure, error_type)
- `redirect_conversion` (time_to_purchase, purchase_amount, service_id)
- `redirect_abandon` (time_on_redirect, bounce_rate, return_behavior)

## Technical Implementation

### Analytics Integration
1. **PostHog event tracking** for all redirect interactions
2. **Revenue attribution** linking redirects to purchases
3. **User journey mapping** across redirect touchpoints
4. **Performance monitoring** for redirect success/failure rates
5. **A/B testing framework** for redirect strategies

### Enhanced Redirect System
1. **Service-level customization** beyond global company settings
2. **Stripe integration completion** (currently only PayPal works)
3. **Dynamic redirect logic** based on user behavior/profile
4. **Redirect performance dashboard** for agencies
5. **Conversion funnel visualization** with redirect impact

### Data Schema Enhancements
1. **Redirect tracking table** for analytics persistence
2. **Attribution models** for multi-touch redirect journeys  
3. **Performance metrics** per redirect configuration
4. **User session linking** across redirect boundaries

## Current System Analysis

### What Exists ✅
- Complete backend redirection module (`/server/redirection/`)
- Frontend UI with conditional logic builder
- PayPal integration with redirect logic
- Support for preIntake, postIntake, failedPurchase redirects
- Global company-level redirect settings

### Critical Gaps ❌
- **Zero analytics tracking** - completely blind to performance
- **Missing Stripe integration** - only PayPal has redirect logic
- **No service-level customization** - only global settings
- **No conversion attribution** - cannot link redirects to revenue
- **No optimization data** - cannot improve redirect strategies

## Enhanced Features

### Service-Specific Redirects
- **Per-service redirect URLs** instead of global-only
- **Service category redirects** for grouped targeting
- **Dynamic redirect selection** based on purchase amount/type
- **Conditional redirects** based on client profile data

### Advanced Analytics
- **Redirect heatmaps** showing user interaction patterns
- **Conversion funnel analysis** with redirect impact measurement
- **Revenue attribution reports** linking redirects to purchases
- **A/B testing dashboard** for redirect strategy optimization

### Performance Optimization
- **Redirect speed monitoring** for user experience impact
- **Success/failure rate tracking** with error categorization
- **Load time optimization** for redirect destinations
- **Mobile vs desktop redirect performance** comparison

## Implementation Plan

### Phase 1: Analytics Foundation (1 week)
- Add PostHog tracking for all redirect events
- Implement redirect → purchase attribution logic
- Create basic analytics dashboard for redirect performance
- Set up conversion funnel tracking with redirect steps

### Phase 2: Stripe Integration (3 days)
- Complete Stripe redirect integration (match PayPal functionality)
- Add redirect logic to Stripe webhook processing
- Test redirect flows across payment providers
- Ensure consistent redirect behavior regardless of payment method

### Phase 3: Service-Level Controls (1 week)  
- Add service-specific redirect configuration UI
- Implement service-level redirect logic in backend
- Create redirect inheritance: Service → Company → Global defaults
- Build redirect strategy templates for common use cases

### Phase 4: Advanced Analytics (1 week)
- Build comprehensive redirect analytics dashboard
- Implement A/B testing framework for redirect strategies
- Create revenue attribution reports with redirect impact
- Add redirect performance optimization recommendations

## User Flows

### Agency Redirect Configuration
1. Agency goes to Service Settings → Redirects
2. Can set service-specific redirects or use company defaults
3. Configure different redirects for success/failure scenarios
4. Preview redirect behavior with test transactions
5. View redirect performance analytics and optimization suggestions

### Client Redirect Experience (Enhanced)
1. Client completes purchase or encounters payment failure
2. System determines appropriate redirect based on service + company rules
3. **NEW**: PostHog tracking fires for redirect_triggered event
4. Client redirected with UTM parameters for attribution
5. **NEW**: Return visits tracked for conversion attribution

### Agency Analytics Dashboard
1. Agency views Redirects Analytics page
2. See redirect engagement rates by service
3. View conversion attribution from redirect destinations
4. Compare redirect performance across payment methods
5. Get optimization recommendations based on data

## Acceptance Criteria

### Must Have
- [ ] PostHog tracking for all redirect interactions
- [ ] Stripe integration with redirect functionality (parity with PayPal)
- [ ] Service-specific redirect configuration UI
- [ ] Revenue attribution linking redirects to purchases
- [ ] Basic redirect performance dashboard

### Should Have
- [ ] A/B testing framework for redirect strategies
- [ ] Redirect performance optimization recommendations
- [ ] Mobile vs desktop redirect analytics
- [ ] Error rate monitoring and alerting for failed redirects

### Could Have
- [ ] Dynamic redirect selection based on user behavior
- [ ] Redirect heatmaps and user interaction analysis
- [ ] Integration with Google Analytics for cross-platform tracking
- [ ] Automated redirect optimization based on ML recommendations

## Dependencies & Risks

### Technical Dependencies
- **PostHog integration**: Event tracking setup and attribution models
- **Stripe webhook enhancement**: Adding redirect logic to existing payment flows
- **Frontend dashboard**: New analytics UI for redirect performance

### Business Risks
- **Performance impact**: Additional tracking may slow redirect experience
- **Data complexity**: Attribution across multiple redirect touchpoints
- **User confusion**: Too many redirect options may overwhelm agencies

### Technical Risks
- **Attribution accuracy**: Tracking users across redirect boundaries
- **Redirect failures**: Network issues or destination unavailability
- **Privacy compliance**: Tracking user behavior across domains

## Success Measurement

### Week 1-2: Foundation Metrics
- 100% of redirects tracked via PostHog
- Stripe redirect integration achieving parity with PayPal
- Service-level redirect configuration adopted by 20% of agencies

### Month 1: Performance Insights
- Clear conversion attribution for 80%+ of redirect interactions
- Identification of top/bottom performing redirect strategies
- Baseline redirect engagement and conversion rates established

### Month 2-3: Optimization Impact
- 15%+ improvement in redirect conversion rates via optimization
- 25%+ increase in redirect feature utilization
- Clear ROI measurement for redirect investments

---

**Owner**: Analytics + Backend Teams  
**Reviewer**: @mehzabeen_c  
**Est. Completion**: 3 weeks  
**Revenue Impact**: High (conversion optimization + attribution)