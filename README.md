# OrthoNow — GTM Tracking Schema & Strategy (Task 1)

This is the analytics setup and tracking architecture for OrthoNow's 9-clinic network to fix the attribution and funnel blind spots before rolling out paid campaigns.

---

## 1. GTM Event Schema Matrix

| Event Name | Trigger Type | Key Parameters | GA4 Report / Destination |
| :--- | :--- | :--- | :--- |
| `booking_step_1_view` | DOM Ready / History | `step`, `location_slug`, `specialty_selected`, `session_id` | Funnel Exploration (Step 1 Drop-off) |
| `booking_step_2_view` | DOM Ready / History | `step`, `location_slug`, `specialty_selected`, `referrer_source`, `session_id` | Funnel Exploration (Step 2 Drop-off) |
| `booking_completed` | Custom Event | `booking_id`, `location_slug`, `specialty_selected`, `appointment_date`, `new_vs_returning`, `session_id` | **Primary Conversion** (Google Ads Smart Bidding) |
| `call_now_click` | Click (Element Selector) | `button_position`, `page_type`, `clinic_slug`, `device_type` | Events Report & Caller Retargeting Lists |
| `whatsapp_chat_start` | Click (Visibility) | `widget_position`, `page_type`, `clinic_slug`, `session_id` | Events Report & Retargeting Lists |
| `guide_download_start` | Form Submit | `guide_name`, `form_location`, `content_category`, `user_email_hash` | Lead Gen / Content Engagement Report |
| `clinic_page_view` | Page View | `clinic_slug`, `clinic_name`, `city`, `page_type` | Pages and Screens (Geo Performance) |
| `blog_scroll_depth` | Scroll (90%) | `scroll_threshold`, `article_slug`, `article_category`, `read_time_estimate` | Engagement / Content Readership Report |

---

## 2. Complete dataLayer Snippets for Frontend

The MERN frontend needs to fire these exact dataLayer pushes programmatically when a user moves through the form steps.

### Step 1: Location & Specialty Selection

window.dataLayer = window.dataLayer || [];
window.dataLayer.push({
  'event': 'booking_step_1_view',
  'booking': {
    'step': 1,
    'clinic_slug': 'clinic-downtown',
    'specialty_selected': 'knee_replacement',
    'session_id': 'sess_a1b2c3d4'
  }
});

### Step 2: Patient Details Entry

window.dataLayer = window.dataLayer || [];
window.dataLayer.push({
  'event': 'booking_step_2_view',
  'booking': {
    'step': 2,
    'location_slug': 'clinic-downtown',
    'specialty_selected': 'knee_replacement',
    'referrer_source': 'google_ads',
    'session_id': 'sess_a1b2c3d4'
  }
});
### Step 3: Final Booking Confirmation
window.dataLayer = window.dataLayer || [];
window.dataLayer.push({
  'event': 'booking_completed',
  'booking': {
    'step': 3,
    'booking_id': 'BK-2025-00482',
    'location_slug': 'clinic-downtown',
    'specialty_selected': 'knee_replacement',
    'appointment_date': '2025-02-14',
    'new_vs_returning': true,
    'session_id': 'sess_a1b2c3d4'
  }
});

---
## 3. Quick Dev Note on Form Logic

**Why standard GTM won't work by default:** Since our multi-step booking form updates the UI dynamically via custom vanilla JavaScript (without changing the actual page URL or causing a hard reload), standard GTM pageview or generic element click triggers won't capture step drop-offs accurately.

To handle this cleanly, I will hook these custom dataLayer pushes directly into our custom multi-step navigation script. Right after a step passes validation and our vanilla JS function changes the display state of the form fields, the `window.dataLayer.push()` executes programmatically before updating the view.

## 4. Why Step 3 is My Primary Conversion Action

* **Primary Conversion:** `booking_completed` (Step 3)
* **The Logic:** For Google Ads Smart Bidding, we need to optimize for actual business outcomes—which is a confirmed clinical appointment. If we optimize bids for Step 1 or Step 2 views, the ad algorithm focuses on traffic that just starts filling forms but abandons midway, burning through the budget. 
* **Secondary Tracking:** Step 1 and Step 2 views are kept strictly as **Secondary Actions**. They are super useful for mapping funnel drop-offs and building custom remarketing lists to chase up intent-heavy users who didn't finish booking, but they shouldn't dictate bid optimization.


