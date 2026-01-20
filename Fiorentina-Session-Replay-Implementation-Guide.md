# ACF Fiorentina - Amplitude Session Replay Implementation Guide

> A comprehensive guide for implementing Amplitude Browser SDK with Session Replay, Autocapture, and GDPR-compliant consent handling.

**Author:** Giuliano Giannini  
**Date:** January 2026  
**Demo URL:** `http://localhost:8888/fiorentina-demo.html`

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Implementation Architecture](#implementation-architecture)
4. [Step-by-Step Implementation](#step-by-step-implementation)
5. [GDPR Compliance](#gdpr-compliance)
6. [Autocapture Configuration](#autocapture-configuration)
7. [Session Replay Setup](#session-replay-setup)
8. [Running the Demo](#running-the-demo)
9. [Events Tracked](#events-tracked)
10. [Best Practices](#best-practices)
11. [Troubleshooting](#troubleshooting)

---

## Overview

This guide documents the implementation of Amplitude's Browser SDK with Session Replay for the ACF Fiorentina demo website. The implementation follows GDPR best practices by ensuring analytics only initialize after explicit user consent.

### Key Features

- ✅ **Session Replay** - Full session recording at 100% sample rate (demo)
- ✅ **Autocapture** - Automatic event tracking without manual instrumentation
- ✅ **Frustration Analytics** - Detect rage clicks and dead clicks
- ✅ **GDPR Compliance** - SDK loads only after user consent
- ✅ **Element Interactions** - Track all button and link clicks automatically

---

## Prerequisites

Before implementing, ensure you have:

1. **Amplitude Account** with a project created
2. **API Key** from your Amplitude project settings
3. **Session Replay** enabled in your Amplitude project
4. Basic understanding of HTML/JavaScript

### Amplitude Resources

- [Browser SDK 2.0 Documentation](https://amplitude.com/docs/sdks/analytics/browser/browser-sdk-2)
- [Session Replay Documentation](https://amplitude.com/docs/session-replay/session-replay-standalone-sdk)

---

## Implementation Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Page Load                               │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              Check localStorage for consent                  │
│                    (cookie_consent)                          │
└─────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              ▼                               ▼
┌─────────────────────────┐     ┌─────────────────────────────┐
│   No consent stored     │     │    Consent = 'accepted'     │
│   Show Cookie Modal     │     │    Load Amplitude SDK       │
│   (blur main content)   │     │    Initialize tracking      │
└─────────────────────────┘     └─────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────────┐
│                    User clicks ACCEPT                        │
│              Store consent in localStorage                   │
│              Load & Initialize Amplitude SDK                 │
│              Hide modal, remove blur                         │
└─────────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Implementation

### Step 1: Define Your API Key

```javascript
const AMPLITUDE_API_KEY = 'your-api-key-here';
```

> ⚠️ **Security Note:** Never commit API keys to public repositories. Use environment variables in production.

### Step 2: Create the Cookie Consent Modal (HTML)

```html
<!-- Cookie Consent Modal -->
<div class="cookie-modal" id="cookieModal">
    <div class="cookie-modal-content">
        <div class="cookie-modal-icon">🔒</div>
        <h2 class="cookie-modal-title">Rispettiamo la tua privacy</h2>
        <p class="cookie-modal-text">
            Utilizziamo cookie e tecnologie simili per migliorare la tua esperienza sul nostro sito, 
            analizzare il traffico e personalizzare i contenuti. I tuoi dati, le tue scelte.
        </p>
        <div class="cookie-modal-buttons">
            <button class="cookie-btn cookie-btn-info">Scopri di più</button>
            <button class="cookie-btn cookie-btn-reject" id="rejectCookies">Rifiuta</button>
            <button class="cookie-btn cookie-btn-accept" id="acceptCookies">Accetta</button>
        </div>
    </div>
</div>

<!-- Main Content Wrapper (will be blurred) -->
<div id="mainContent">
    <!-- Your website content here -->
</div>
```

### Step 3: Style the Modal (CSS)

```css
.cookie-modal {
    display: none;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.7);
    z-index: 10000;
    justify-content: center;
    align-items: center;
}

.cookie-modal.active {
    display: flex;
}

.blur-content {
    filter: blur(5px);
    pointer-events: none;
}
```

### Step 4: Implement Consent Logic (JavaScript)

```javascript
// Check consent on page load
function checkCookieConsent() {
    const consent = localStorage.getItem('cookie_consent');
    if (consent === 'accepted') {
        loadAndInitializeAmplitude();
    } else if (consent === 'rejected') {
        console.log('Cookie consent was previously rejected');
    } else {
        showCookieModal();
    }
}

// Show/Hide Modal Functions
function showCookieModal() {
    document.getElementById('cookieModal').classList.add('active');
    document.getElementById('mainContent').classList.add('blur-content');
}

function hideCookieModal() {
    document.getElementById('cookieModal').classList.remove('active');
    document.getElementById('mainContent').classList.remove('blur-content');
}
```

### Step 5: Load Amplitude SDK Dynamically

```javascript
function loadAndInitializeAmplitude() {
    // Create script element for Amplitude Script Loader
    const script = document.createElement('script');
    script.src = `https://cdn.amplitude.com/script/${AMPLITUDE_API_KEY}.js`;
    script.async = true;
    
    script.onload = function() {
        // Add Session Replay plugin
        window.amplitude.add(window.sessionReplay.plugin({
            sampleRate: 1  // 100% for demo, use 0.1-0.3 in production
        }));

        // Initialize Amplitude with autocapture configuration
        window.amplitude.init(AMPLITUDE_API_KEY, {
            fetchRemoteConfig: true,
            autocapture: {
                attribution: true,
                sessions: true,
                pageViews: true,
                formInteractions: false,
                fileDownloads: false,
                elementInteractions: true,
                frustrationInteractions: true
            }
        });

        console.log('✅ Amplitude initialized with Session Replay');
    };

    document.head.appendChild(script);
}
```

### Step 6: Wire Up Event Listeners

```javascript
document.addEventListener('DOMContentLoaded', function() {
    // Accept button
    document.getElementById('acceptCookies').addEventListener('click', function() {
        localStorage.setItem('cookie_consent', 'accepted');
        hideCookieModal();
        loadAndInitializeAmplitude();
    });

    // Reject button
    document.getElementById('rejectCookies').addEventListener('click', function() {
        localStorage.setItem('cookie_consent', 'rejected');
        hideCookieModal();
    });

    // Check consent on load
    checkCookieConsent();
});
```

---

## GDPR Compliance

### Why Dynamic Loading Matters

There are two ways to load the Amplitude SDK:

| Method | How It Works | GDPR Compliant? |
|--------|--------------|-----------------|
| **Script Loader (in HTML)** | SDK auto-initializes on page load | ❌ No |
| **Dynamic Loading** | SDK loads only when called | ✅ Yes |

### The Problem with Static Script Tags

```html
<!-- ❌ DON'T DO THIS - SDK initializes immediately -->
<script src="https://cdn.amplitude.com/script/API_KEY.js"></script>
<script>
    window.amplitude.init('API_KEY');
</script>
```

### The GDPR-Compliant Approach

```javascript
// ✅ DO THIS - SDK loads only after consent
function loadAndInitializeAmplitude() {
    const script = document.createElement('script');
    script.src = `https://cdn.amplitude.com/script/${AMPLITUDE_API_KEY}.js`;
    script.onload = function() {
        window.amplitude.init(AMPLITUDE_API_KEY, { /* config */ });
    };
    document.head.appendChild(script);
}

// Only call this AFTER user accepts cookies
document.getElementById('acceptCookies').addEventListener('click', loadAndInitializeAmplitude);
```

### Consent Storage

```javascript
// Store consent decision
localStorage.setItem('cookie_consent', 'accepted'); // or 'rejected'

// Check on subsequent visits
const consent = localStorage.getItem('cookie_consent');
```

---

## Autocapture Configuration

### Available Options

| Setting | Description | Our Config |
|---------|-------------|------------|
| `attribution` | Track UTM parameters and referrers | ✅ `true` |
| `sessions` | Track session start/end | ✅ `true` |
| `pageViews` | Track page views automatically | ✅ `true` |
| `formInteractions` | Track form field interactions | ❌ `false` |
| `fileDownloads` | Track file download clicks | ❌ `false` |
| `elementInteractions` | Track clicks on buttons/links | ✅ `true` |
| `frustrationInteractions` | Track rage clicks & dead clicks | ✅ `true` |

### Configuration Code

```javascript
autocapture: {
    attribution: true,           // UTM tracking
    sessions: true,              // Session tracking
    pageViews: true,             // Page view tracking
    formInteractions: false,     // Disabled
    fileDownloads: false,        // Disabled
    elementInteractions: true,   // Button/link clicks
    frustrationInteractions: true // Rage/dead clicks
}
```

---

## Session Replay Setup

### Adding the Session Replay Plugin

```javascript
// Add Session Replay BEFORE init()
window.amplitude.add(window.sessionReplay.plugin({
    sampleRate: 1  // 1 = 100%, 0.1 = 10%
}));

// Then initialize
window.amplitude.init(AMPLITUDE_API_KEY, { /* config */ });
```

### Sample Rate Guidelines

| Environment | Recommended Rate | Notes |
|-------------|------------------|-------|
| Demo/Testing | `1` (100%) | Capture everything |
| Production (Low Traffic) | `0.3` (30%) | Good coverage |
| Production (High Traffic) | `0.1` (10%) | Cost-effective |

### What Session Replay Captures

- ✅ Mouse movements and clicks
- ✅ Scroll behavior
- ✅ Form interactions (masked by default)
- ✅ Page navigation
- ✅ Rage clicks and dead clicks
- ✅ Console errors

---

## Running the Demo

### Start Local Server

```bash
cd /Users/giuliano.giannini/Desktop
python3 -m http.server 8888
```

### Open in Browser

```
http://localhost:8888/fiorentina-demo.html
```

### Verify Implementation

1. Open browser Developer Tools (F12)
2. Go to **Console** tab
3. Accept cookies on the modal
4. Look for these messages:

```
✅ Amplitude initialized with Session Replay and Frustration Analytics
📊 Autocapture enabled: attribution, sessions, pageViews, elementInteractions, frustrationInteractions
🎬 Session Replay sample rate: 100%
```

---

## Events Tracked

With our autocapture configuration, these events are tracked automatically:

### Amplitude Autocapture Events

| Event Name | Trigger | Properties |
|------------|---------|------------|
| `[Amplitude] Page Viewed` | Page load | `page_title`, `page_url`, `referrer` |
| `[Amplitude] Session Start` | New session begins | `session_id` |
| `[Amplitude] Session End` | Session ends | `session_length` |
| `[Amplitude] Element Clicked` | Click on button/link | `element_tag`, `element_text`, `element_href` |
| `[Amplitude] Rage Click` | Rapid repeated clicks | `element_tag`, `click_count` |
| `[Amplitude] Dead Click` | Click with no response | `element_tag`, `element_text` |

### Attribution Properties (Automatic)

- `utm_source`
- `utm_medium`
- `utm_campaign`
- `utm_term`
- `utm_content`
- `referrer`
- `referring_domain`

---

## Best Practices

### 1. Always Load SDK After Consent

```javascript
// ✅ Correct
acceptButton.onclick = () => {
    saveConsent();
    loadAmplitudeSDK();
};

// ❌ Wrong - SDK loads before consent
<script src="https://cdn.amplitude.com/script/KEY.js"></script>
```

### 2. Use Appropriate Sample Rates

```javascript
// Demo/Testing
sampleRate: 1

// Production
sampleRate: 0.1 // Start low, increase if needed
```

### 3. Don't Track Unnecessary Events

We intentionally disabled:
- `formInteractions: false` - Too noisy for this demo
- `fileDownloads: false` - Not relevant for this site

### 4. Keep Console Logs for Debugging

```javascript
console.log('✅ Amplitude initialized');
console.log('📊 Autocapture enabled: ...');
console.log('🎬 Session Replay sample rate: ...');
```

### 5. Test Consent Flow Thoroughly

1. Clear localStorage: `localStorage.removeItem('cookie_consent')`
2. Refresh page
3. Verify modal appears
4. Test both Accept and Reject paths

---

## Troubleshooting

### SDK Not Initializing

**Symptom:** No console logs, no events in Amplitude

**Solution:** Check browser console for errors. Verify API key is correct.

```javascript
// Debug: Check if amplitude object exists
console.log(window.amplitude); // Should not be undefined
```

### Session Replay Not Recording

**Symptom:** Events appear but no replays

**Solution:** Verify Session Replay is enabled in Amplitude project settings and plugin is added before `init()`.

```javascript
// Must be BEFORE init()
window.amplitude.add(window.sessionReplay.plugin({ sampleRate: 1 }));
window.amplitude.init(API_KEY, config);
```

### Consent Modal Not Showing

**Symptom:** Page loads without modal

**Solution:** Check if consent is already stored in localStorage.

```javascript
// Clear stored consent for testing
localStorage.removeItem('cookie_consent');
location.reload();
```

### Events Not Appearing in Amplitude

**Symptom:** Console shows success but no data in Amplitude

**Solution:** 
1. Wait 1-2 minutes (data processing delay)
2. Check correct project is selected in Amplitude
3. Verify API key matches the project

---

## Summary

This implementation provides:

1. **GDPR-compliant tracking** - SDK only loads after explicit consent
2. **Zero manual instrumentation** - Autocapture handles all events
3. **Full session visibility** - Session Replay at 100% sample rate
4. **User frustration detection** - Rage clicks and dead clicks tracked automatically
5. **Easy maintenance** - No custom events to maintain

For questions or issues, refer to the [Amplitude Documentation](https://amplitude.com/docs) or contact your Amplitude representative.

---

**Forza Viola!** ⚜️💜

