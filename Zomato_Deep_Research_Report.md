# Zomato Deep Research Report
## Complete UI/UX & Technical Analysis for Cloning

**Prepared:** April 25, 2026  
**Scope:** Consumer App (Android/iOS), Restaurant Discovery, Ordering Flow, Menu System, Promotions, Payments, Tracking  
**Goal:** Provide exhaustive technical and UX specification for building a Zomato clone

---

## 1. EXECUTIVE SUMMARY

Zomato is India's largest food delivery and restaurant discovery platform. It operates as a three-sided marketplace connecting **Users**, **Restaurants**, and **Delivery Partners**. The app handles ~23-25 million orders monthly (as of 2025), with a Gross Order Value (GOV) exceeding ₹20,000 crore quarterly.

**Key Differentiators:**
- Dual discovery model: Food Delivery + Dining Out
- Hyper-local restaurant density (even tier-3 cities)
- Zomato Gold subscription program (40% of food delivery GOV)
- Real-time order tracking with live rider location
- Dynamic pricing with surge, platform fees, and distance-based delivery charges

---

## 2. APP ARCHITECTURE & TECH STACK

### 2.1 Backend Architecture
Based on available documentation and system design analyses:

```
Client (iOS/Android/Web)
    ↓
API Gateway (Rate limiting, Auth, Routing)
    ↓
Microservices:
├── User Service (Auth, Profiles, Preferences)
├── Restaurant Service (Listings, Menus, Hours)
├── Search Service (Elasticsearch/OpenSearch)
├── Pricing Service (Dynamic pricing, Offers, Taxes)
├── Cart Service (Session management)
├── Order Service (Order lifecycle, State machine)
├── Payment Service (Multiple gateways, UPI)
├── Delivery Service (Rider matching, ETAs, Tracking)
├── Notification Service (Push, SMS, Email)
└── Review Service (Ratings, Photos, Moderation)
    ↓
Data Layer:
├── PostgreSQL/MySQL (Transactional data)
├── Redis (Caching, Sessions, Cart)
├── Elasticsearch (Search index)
├── Kafka (Event streaming)
└── S3/Cloud Storage (Images, Documents)
```

### 2.2 Mobile Apps
- **iOS:** Swift, native UIKit
- **Android:** Kotlin, custom view system
- **Design System:** "Sushi" — modular, server-configurable UI framework
  - Can push new UI elements without app updates
  - Lego-style reusable components
  - A/B testing enabled at component level

### 2.3 Key Third-Party Integrations
- Maps: Google Maps (primary), MapMyIndia (backup)
- Payments: Razorpay, PayU, UPI apps, Wallets (Paytm, PhonePe)
- Push Notifications: Firebase Cloud Messaging
- SMS: Twilio/Exotel
- CDN: Cloudflare/Akamai for image delivery

---

## 3. HOME SCREEN / DISCOVERY EXPERIENCE

### 3.1 Layout Structure
The home screen is the primary discovery surface. It uses a **feed-based architecture** with multiple content rails.

**Screen Sections (top to bottom):**

| Section | Description |
|---------|-------------|
| **Header Bar** | Location selector (dropdown), Search icon, Profile avatar |
| **Search Bar** | Prominent search with placeholder "Restaurant name, cuisine, or a dish..." |
| **Delivery/Dining Toggle** | Tab switcher: "Delivery" / "Dining Out" / "Nightlife" |
| **Promo Banner** | Horizontal scrolling carousel of promotional banners (full-width cards) |
| **Quick Filters** | Horizontal rail: "Pure Veg", "Bestseller", "Rating 4.0+", "Fastest Delivery" |
| **Cuisine Rails** | Horizontal scrolling cuisine category chips: "Biryani", "Pizza", "Burger", "North Indian", etc. |
| **Featured Collections** | Curated lists: "Trending Now", "Newly Opened", "Hidden Gems" |
| **Restaurant List** | Vertical infinite scroll of restaurant cards |
| **Bottom Nav** | Home / Orders / Gold / Cart / Profile |

### 3.2 Personalization Engine
- **Suggested Filters:** Contextual filter rails powered by a graph of user intent
  - If user searches "Cafes" → suggests "WiFi", "Outdoor Seating", "Nearest to me"
  - If ordering → suggests "Previously Ordered", "Express Delivery"
- **Server-Configurable UI:** The layout can be rearranged per user segment, city, or experiment

### 3.3 Universal Tracking
Every user interaction registers with a `search_id`. When navigating to a new page, it carries `previous_search_id` + new `search_id`, enabling tap-by-tap behavior analysis and funnel optimization.

---

## 4. RESTAURANT LISTING CARD

### 4.1 Card Design
The restaurant card is the atomic unit of discovery. It uses a **vertical card layout** with rich metadata.

**Card Elements (top to bottom):**

```
┌─────────────────────────────────────┐
│  [Hero Image - 16:9 aspect ratio]   │
│  ┌────────────────────────────┐     │
│  │ ⏱ 30-40 min    🛵 Free    │     │
│  │ [Delivery time] [Delivery  │     │
│  │              fee indicator]│     │
│  └────────────────────────────┘     │
├─────────────────────────────────────┤
│ 🏷️ FLAT ₹100 OFF                    │
│ Restaurant Name                    │
│ ⭐ 4.3 (1,234 ratings)             │
│ North Indian • Chinese • ₹200 for one│
│ 📍 2.5 km away                      │
│                                     │
│ ┌─────────────────────────────┐     │
│ │ 🎁 Free delivery on Gold    │     │
│ │ 🏆 Pure Veg restaurant      │     │
│ └─────────────────────────────┘     │
└─────────────────────────────────────┘
```

### 4.2 Card Metadata Fields
| Field | Description |
|-------|-------------|
| **Hero Image** | Restaurant cover photo (often food collage) |
| **Offer Tag** | Top-left or top-right promotional badge (e.g., "FLAT ₹100 OFF", "50% OFF") |
| **Delivery ETA** | Time range (e.g., "30-40 min"), color-coded by speed |
| **Delivery Fee** | Shows "Free" for Gold members or distance-based fee |
| **Restaurant Name** | Bold, truncated if too long |
| **Rating** | Star icon + numerical rating + review count in parentheses |
| **Cuisines** | Comma-separated list (max 3 shown) |
| **Cost for One** | Estimated per-person cost (₹150 for one, ₹200 for one, etc.) |
| **Distance** | km from user's location |
| **Additional Badges** | "Pure Veg", "Bestseller", "Must Try", "Gold Partner" |

### 4.3 Dynamic Elements
- **Hero Text:** Server-driven promotional message on cards (e.g., "Free dessert with orders above ₹500")
- **Distance/Delivery Time Updates:** Real-time based on rider availability and traffic
- **Surge Indicators:** Visual cue when demand is high (e.g., "High demand - longer wait")

---

## 5. SEARCH & FILTER SYSTEM

### 5.1 Search Interface
- **Global Search:** Accessible from header
- **Search Suggestions:** Autocomplete with recent searches, trending searches, popular restaurants
- **Cuisine Search:** Searching "Biryani" shows both restaurants AND dish-level results
- **Dish-Level Discovery:** Users can search for specific dishes, not just restaurants

### 5.2 Filter Modal (Bottom Sheet)
When tapping "Filters", a bottom sheet opens with:

| Filter Category | Options |
|----------------|---------|
| **Sort By** | Relevance, Rating, Delivery Time, Cost (low to high), Cost (high to low) |
| **Cuisines** | Multi-select chips: North Indian, Chinese, South Indian, Biryani, Pizza, etc. |
| **Rating** | 4.5+, 4.0+, 3.5+ |
| **Cost for Two** | ₹0-300, ₹300-600, ₹600-1000, ₹1000+ |
| **Dietary** | Pure Veg, Non-Veg, Vegan, Jain |
| **Offers** | Flat discount, Free delivery, Gold offers |
| **Delivery Time** | Under 30 min, Under 45 min |
| **More Filters** | Open now, Accepts online payment, Pure veg only |

### 5.3 Search Results Display
- **Tabbed Results:** "Restaurants" | "Dishes" — lets users toggle between restaurant-level and dish-level results
- **Dish-Level Results:** Shows individual menu items with "Add" buttons, restaurant name below
- **Empty States:** "No results found" with suggested alternatives

---

## 6. RESTAURANT DETAIL PAGE

### 6.1 Page Structure
The restaurant page is a **scrollable vertical page** with sticky elements.

**Sections (top to bottom):**

| Section | Content |
|---------|---------|
| **Hero Banner** | Large restaurant image with gradient overlay, back button, share button, wishlist heart |
| **Restaurant Info** | Name, rating badge, cuisine tags, cost-for-two, address |
| **Offer Carousel** | Horizontal scrolling offers specific to this restaurant |
| **Info Bar** | Delivery time • Distance • Delivery fee • Open/Closed status |
| **Search Within Menu** | Sticky search bar: "Search within menu" |
| **Menu Tabs / Rail** | Horizontal scrolling category tabs (e.g., "Recommended", "Starters", "Main Course") |
| **Menu Content** | Vertical list of menu sections and items |
| **Reviews Section** | Rating breakdown, top reviews, photo gallery |
| **Restaurant Info** | Address, map, opening hours, safety certifications |
| **Similar Restaurants** | "You might also like" carousel |

### 6.2 Key Features
- **Sticky Category Tabs:** As user scrolls, the category tabs stick to top and auto-highlight the current section
- **Menu Search:** Real-time filtering of menu items within the restaurant
- **Offers Applied:** Visual indicator showing which offers apply to this restaurant
- **Safety Badges:** "Zomato Hygiene Rated", "Temperature Check", "Masks Mandatory"

### 6.3 Delivery Info Bar
```
🕒 30-40 min    🛵 Free Delivery    📍 2.5 km    🟢 Open now
```
- If closed: Shows "Opens at 11:00 AM tomorrow" with pre-order option

---

## 7. MENU DISPLAY & CATEGORIZATION

### 7.1 Menu Organization Structure
Zomato menus are organized in a **hierarchical category system**:

```
Menu
├── Category (e.g., "Starters", "Main Course", "Breads")
│   ├── Sub-category (optional, e.g., "Veg Starters", "Non-Veg Starters")
│   │   ├── Menu Item (dish)
│   │   │   ├── Variants (e.g., "Half", "Full", "Family Pack")
│   │   │   └── Modifier Groups (Add-ons, Customizations)
│   │   │       └── Modifier Items ("Extra Cheese", "Double Patty")
```

### 7.2 Category Types
**Standard Categories:**
- Recommended / Most Popular / Bestsellers
- Starters / Appetizers
- Soups & Salads
- Main Course
- Breads / Rice / Biryani
- Desserts
- Beverages
- Combos / Thalis
- Accompaniments

**Special Sections:**
- "T20 World Cup Combo" (event-based)
- "Navratri Special" (festival-based)
- "Healthy Options"
- "Under ₹200"

### 7.3 Menu Section Header
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Starters
7 items
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
- Category name with item count
- Collapsible (tap to expand/collapse)
- Visual separator (subtle divider or background color change)

### 7.4 Menu Auto-Tagging System
Zomato auto-detects dietary tags based on keywords in item names:
- **Veg:** Items with no non-veg keywords
- **Non-Veg:** Auto-tagged based on keywords ("chicken", "mutton", "fish", "egg")
- **Egg:** Separate category for egg-only items
- **Vegan:** Can only be applied to items already tagged as veg
- **Jain:** Special dietary option

---

## 8. MENU ITEM CARD

### 8.1 Item Card Layout
Each menu item is displayed as a **horizontal card** with image on right or left.

```
┌─────────────────────────────────────┐
│ 🟢 (veg indicator)                  │
│ Butter Chicken                      │
│ ⭐ Bestseller                       │
│ Barbequed paneer tikka cooked in    │
│ rich Indian gravy.                  │
│ ₹340                                │
│                                     │
│ ┌──────────┐  ┌─────────────────┐   │
│ │  - 1 +   │  │    Customize    │   │
│ │ (Add)    │  │    (dropdown)   │   │
│ └──────────┘  └─────────────────┘   │
│                        ┌─────────┐  │
│                        │ [IMAGE] │  │
│                        │  1:1    │  │
│                        └─────────┘  │
└─────────────────────────────────────┘
```

### 8.2 Menu Item Fields
| Field | Description |
|-------|-------------|
| **Dietary Indicator** | Green dot (Veg), Red dot (Non-Veg), Brown dot (Egg) |
| **Item Name** | Bold, max 70 characters |
| **Badges** | "Bestseller", "Must Try", "Chef's Special", "Spicy", "New" |
| **Description** | 4-500 characters. Truncated with "... read more" |
| **Price** | Primary price in ₹. Shows "₹340" or range "₹340-₹450" |
| **Image** | Square (1:1) or 4:3. Optional — not all items have images |
| **Add Button** | Primary action: "+" or "Add". Changes to quantity stepper after adding |
| **Customize** | Appears if item has variants or add-ons |
| **Calorie Count** | Optional: "320 kcal" |
| **Serving Size** | Optional: "Serves 2" |

### 8.3 Menu Score System
Zomato provides restaurants a "Menu Score" to optimize conversion:
- **Menu Appeal:** Visual delight and clarity
- **Completeness:** Descriptions, images, dietary tags
- **Navigability:** Category organization
- Higher menu score correlates with higher menu-to-cart ratio

### 8.4 Image Requirements (from API docs)
- **Min size:** 400x400px
- **Max size:** 15MB
- **Formats:** JPEG, PNG
- **AR Media:** Max 5MB (for 3D/AR features)

---

## 9. CUSTOMIZATIONS & ADD-ONS

### 9.1 Modifier Group Structure
When a user taps "Customize" or "Add", a **bottom sheet modal** opens.

```
┌─────────────────────────────────────┐
│ Butter Chicken          ┃    ×     │
│ ₹340                                │
├─────────────────────────────────────┤
│ Select Variant * (required)       │
│ ○ Half          ₹240                │
│ ● Full          ₹340                │
│ ○ Family Pack   ₹550                │
├─────────────────────────────────────┤
│ Choose Preparation (optional)       │
│ □ Less Spicy                        │
│ □ Extra Gravy                       │
├─────────────────────────────────────┤
│ Add-ons (optional, max 2)           │
│ □ Extra Butter    +₹30              │
│ □ Extra Naan      +₹20              │
├─────────────────────────────────────┤
│                    Total: ₹340      │
│           ┌──────────────┐           │
│           │  Add Item    │           │
│           └──────────────┘           │
└─────────────────────────────────────┘
```

### 9.2 Modifier Rules
| Rule | Description |
|------|-------------|
| **Required** | User must select one option (marked with *) |
| **Optional** | User can skip (min=0) |
| **Min/Max** | "Choose at least 1, at most 3" |
| **Max Selection Per Item** | Limit on how many of each add-on |
| **Price Impact** | Each modifier shows price delta (+₹30) |
| **Nested Modifiers** | Modifier items themselves can't have further customizations |

### 9.3 Smart Modifier Validation
- Items priced < ₹30: Cannot have non-mandatory modifier groups
- Items priced ₹30-50: Modifier item price cannot exceed root item price
- Items priced > ₹50: Modifier item price cannot exceed 4x root item price

---

## 10. PROMOS, DISCOUNTS & OFFERS SYSTEM

### 10.1 Offer Display Locations

| Location | Offer Type |
|----------|-----------|
| **Home Banner** | Platform-wide promos: "FLAT ₹100 OFF", "50% OFF up to ₹100" |
| **Restaurant Card** | Restaurant-specific offers (hero text) |
| **Restaurant Page** | Offer carousel with detailed terms |
| **Menu Items** | Strike-through pricing, "20% OFF" badges |
| **Cart Page** | Applicable coupons, auto-applied offers |
| **Checkout** | Final offer validation, savings summary |

### 10.2 Offer Types

| Type | Description |
|------|-------------|
| **Flat Discount** | "FLAT ₹100 OFF on orders above ₹300" |
| **Percentage Off** | "50% OFF up to ₹100" |
| **Free Delivery** | Waived delivery fee (Gold member benefit) |
| **BOGO** | Buy One Get One (historically part of Gold) |
| **Item-Level** | Specific dishes discounted |
| **Bank Offers** | Credit/debit card specific ("20% OFF up to ₹500 on HDFC Cards") |
| **Scratch Cards** | Gamified rewards after transaction |
| **Surprise Offers** | Randomly assigned personalized offers |

### 10.3 Offer Carousel on Restaurant Page
```
┌─────────────────────────────────────┐
│ 🎁 Offers for you                   │
│                                     │
│ ┌──────────┐ ┌──────────┐ ┌───────┐ │
│ │FLAT ₹100 │ │ 50% OFF  │ │Bank   │ │
│ │OFF       │ │upto ₹150 │ │Offer  │ │
│ │On orders │ │On select │ │20% off│ │
│ │>₹299    │ │items     │ │      │ │
│ └──────────┘ └──────────┘ └───────┘ │
└─────────────────────────────────────┘
```
- Horizontal scrolling cards
- Each card shows: Offer title, brief description, "Terms & Conditions" link
- Tapping shows full terms in a bottom sheet

### 10.4 Coupon Code System
- **Coupon Input:** Text field on cart/checkout
- **Auto-Apply:** Best coupon auto-applied
- **Validation:** Real-time API call to validate coupon
- **Error States:** "Minimum order value not met", "Already used", "Expired"
- **Savings Tracker:** Shows total savings from all offers

### 10.5 Gold Member Discounts
- **Delivery:** Free delivery on orders >₹199 (restaurants within 10km)
- **Dining Out:** 15-25% off at partner restaurants
- **Extra Discounts:** Up to 30% additional on delivery orders from partner restaurants
- **VIP Access:** Priority ordering during rush hours/festivals

---

## 11. CART & CHECKOUT FLOW

### 11.1 Cart Page

```
┌─────────────────────────────────────┐
│ 🛒 Your Cart (3 items)    ┃  ₹1,247 │
├─────────────────────────────────────┤
│                                     │
│ ▼ Domino's Pizza                    │
│ ┌─────────────────────────────────┐ │
│ │ 🟢 Margherita Pizza (Large)     │ │
│ │    ₹349    │  -  1  +  │  ×   │ │
│ └─────────────────────────────────┘ │
│ ▼ McDonald's                        │
│ ┌─────────────────────────────────┐ │
│ │ 🔴 McChicken Burger             │ │
│ │    ₹189    │  -  1  +  │  ×   │ │
│ │ Custom: Extra Cheese +₹30       │ │
│ └─────────────────────────────────┘ │
├─────────────────────────────────────┤
│ 🎁 Apply Coupon                     │
│ [ Enter coupon code           > ]   │
├─────────────────────────────────────┤
│ 📄 Bill Details                     │
│ Item Total              ₹1,086      │
│ Restaurant Packaging    +₹45        │
│ Restaurant Charges      +₹28        │
│ Delivery Fee            ₹39         │
│ Delivery Partner Tip    +₹0         │
│ Platform Fee            +₹12.50     │
│ GST                     +₹58.50     │
│                                     │
│ Offer Applied: FLAT50               │
│ -₹100                               │
│                                     │
│ TO PAY                  ₹1,169      │
├─────────────────────────────────────┤
│ 🎗️ Donate ₹3 to Feed India        │
│ [☑️] (opt-out checkbox)            │
├─────────────────────────────────────┤
│     ┌─────────────────────────┐     │
│     │  Proceed to Pay ₹1,169  │     │
│     └─────────────────────────┘     │
└─────────────────────────────────────┘
```

### 11.2 Bill Breakdown Components

| Component | Description |
|-----------|-------------|
| **Item Total** | Sum of all menu items (before any charges) |
| **Restaurant Packaging** | Packaging fee charged by restaurant |
| **Restaurant Charges** | Additional restaurant fees |
| **Delivery Fee** | Distance-based: Base ₹39 + ₹2/km. Free for Gold |
| **Small Order Fee** | ₹15 (waived for orders >₹150) |
| **Platform Fee** | Flat fee per order (currently ₹14.90 including GST as of 2026) |
| **Delivery Partner Tip** | Optional, user-selectable amount |
| **GST** | 5% on food (services), 18% on delivery charges |
| **Discounts** | Sum of all applied offers |
| **Feeding India Donation** | ₹3 default (opt-out available) |

### 11.3 Checkout Flow
```
Cart Review
    ↓
Delivery Address Selection
    ├── Saved addresses (Home, Work, etc.)
    ├── Add new address (map pin + manual entry)
    └── Address validation (delivery zone check)
    ↓
Payment Method Selection
    ├── UPI (Google Pay, PhonePe, Paytm, any UPI ID)
    ├── Credit/Debit Cards
    ├── Wallets (Paytm, Amazon Pay, Mobikwik)
    ├── Net Banking
    ├── Cash on Delivery (if available)
    └── Zomato Credits
    ↓
Order Confirmation
    ├── Final bill review
    ├── "Place Order" CTA
    └── Order placed screen with ETA
```

### 11.4 Payment Processing
- **Pre-auth:** Some payment methods do pre-authorization
- **Failed Payment:** Retry with different method, or COD option offered
- **Zomato Credits:** Refund credits stored in wallet for future orders
- **COD Verification:** Phone number verification for first-time COD users

---

## 12. ORDER TRACKING SYSTEM

### 12.1 Order Lifecycle States
```
Order Placed
    ↓
Payment Confirmed
    ↓
Order Accepted by Restaurant
    ↓
Food Being Prepared
    ↓
Ready for Pickup
    ↓
Delivery Partner Assigned
    ↓
Partner Reached Restaurant
    ↓
Food Picked Up
    ↓
En Route
    ↓
Nearby
    ↓
Delivered
```

### 12.2 Tracking Screen UI

```
┌─────────────────────────────────────┐
│ 🔴 Live Tracking                    │
│                                     │
│ ETA: 12:24 PM (18 min remaining)    │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │    🗺️ [LIVE MAP VIEW]           │ │
│ │         🏠 ←──🛵── 📍          │ │
│ └─────────────────────────────────┘ │
│                                     │
│ Status: Food is being prepared      │
│                                     │
│ Timeline:                           │
│ ✅ Order placed          11:45 AM   │
│ ✅ Payment confirmed     11:46 AM   │
│ ⏳ Food preparation      11:50 AM   │
│ ○ Out for delivery                  │
│ ○ Delivered                         │
├─────────────────────────────────────┤
│ 👤 Delivery Partner                 │
│ Ramesh K. ⭐ 4.8                    │
│ 📞 Call    💬 Chat                  │
├─────────────────────────────────────┤
│ 📋 Order Details                    │
│ [Item list with quantities]          │
│                                     │
│ Total Paid: ₹1,169                  │
└─────────────────────────────────────┘
```

### 12.3 Live Tracking Features
- **Real-time Map:** Rider location updated every few seconds
- **ETA Calculation:** Dynamic based on distance, traffic, rider speed
- **Contact Options:** Call button (masked phone number), In-app chat
- **Safe Delivery:** "No Contact Delivery" option, OTP verification for delivery
- **Dynamic Island / Live Activities:** iOS-specific live tracking in lock screen/notifications

### 12.4 Notifications
| Event | Notification Type |
|-------|------------------|
| Order Placed | Push + In-app |
| Order Accepted | Push |
| Food Ready | Push |
| Rider Assigned | Push with rider details |
| Rider Nearby | Push + SMS |
| Delivered | Push + "Rate your order" prompt |
| Delayed | Push: "Sorry for the delay, here's ₹100 coupon" |

---

## 13. RATING & REVIEW SYSTEM

### 13.1 Dual Rating System
Zomato uses **two separate ratings**:

| Type | Icon Color | Criteria |
|------|-----------|----------|
| **Delivery Rating** | Red stars | Taste, Quantity, Food Packaging |
| **Dining Rating** | Black stars | Food, Service, Ambiance, Value |

- Contextual display: Delivery mode shows red stars, Dining out shows black stars

### 13.2 Review Submission
After order delivery, user prompted to rate:

```
┌─────────────────────────────────────┐
│ Rate your order                     │
│                                     │
│ How was the food?                   │
│ ⭐⭐⭐⭐⭐ (select stars)             │
│                                     │
│ What went well?                     │
│ [Tasty] [Good Packaging] [On Time]  │
│ [Good Quantity] [Value for Money]   │
│                                     │
│ Any issues?                         │
│ [Late Delivery] [Wrong Item]        │
│ [Missing Item] [Spilled]            │
│                                     │
│ Add Photos (optional)               │
│ [📷 Upload]                         │
│                                     │
│ Write a review (optional)           │
│ [Text input...                      │
│                                     │
│ [ Submit Review ]                   │
└─────────────────────────────────────┘
```

### 13.3 Review Display on Restaurant Page
- **Overall Rating:** Large number with star icon (e.g., "4.3 ★")
- **Rating Breakdown:** 5-star, 4-star, 3-star, 2-star, 1-star distribution bar chart
- **Review Cards:** Reviewer name, date, rating, review text, photos, helpful count
- **Photos Section:** Grid of user-uploaded food photos
- **Top Reviews:** "Most Helpful", "Most Recent" tabs

---

## 14. ZOMATO GOLD / MEMBERSHIP PROGRAM

### 14.1 Program Structure

| Aspect | Details |
|--------|---------|
| **Price** | ₹149 for 3 months (introductory), varies by user (₹30-₹99 for targeted users) |
| **Free Trial** | 14-day free trial available |
| **Members** | 2+ million subscribers (2025) |
| **GOV Contribution** | ~40% of food delivery Gross Order Value |

### 14.2 Benefits

| Benefit | Description |
|---------|-------------|
| **Free Delivery** | Unlimited free delivery on orders >₹199 (restaurants within 10km) |
| **Extra Discounts** | Up to 30% off on delivery from 20,000+ partner restaurants |
| **Dining Discounts** | 15-25% off (up to 40% at some) at 25,000+ partner restaurants |
| **VIP Access** | Priority access during rush hours, festivals, high demand |
| **No Delay Guarantee** | ₹100 coupon if order is delayed (phased out for new subscribers 2023) |

### 14.3 Gold Member Indicators
- **Gold Badge:** Gold crown icon on restaurant cards that are Gold partners
- **Savings Tracker:** Shows "You saved ₹847 with Gold" in order history
- **Member-Only Restaurants:** Some restaurants visible only to Gold members during peak times

### 14.4 Platform Fee Note
Even Gold members pay the platform fee (currently ₹14.90/order). The free delivery benefit only waives the delivery fee, not the platform fee.

---

## 15. BILL BREAKUP & PRICING ARCHITECTURE

### 15.1 Fee Evolution
| Date | Platform Fee |
|------|-------------|
| Aug 2023 | ₹2 |
| Late 2023 | ₹3 |
| Jan 2024 | ₹4 |
| Apr 2024 | ₹5 |
| Jul 2024 | ₹6 |
| Sep 2024 | ₹10 |
| Sep 2025 | ₹12 (excl. GST) |
| Nov 2025 | ₹14.90 (incl. GST) |

### 15.2 Complete Cost Breakdown Example
```
Item Total:              ₹1,086
Restaurant Packaging:    +₹45
Restaurant Charges:      +₹28
Delivery Fee:            +₹39  (or FREE for Gold)
Small Order Fee:         +₹15  (waived if >₹150)
Platform Fee:            +₹12.50
Delivery Partner Tip:    +₹0   (user optional)
GST (5% on food):        +₹54.30
GST (18% on delivery):   +₹7.02
Feeding India Donation:  +₹3   (opt-out)
────────────────────────────────
Subtotal:                ₹1,289.82
Discount Applied:        -₹100
────────────────────────────────
TO PAY:                  ₹1,189.82
```

### 15.3 GST Rules
- **Food items (services):** 5% GST
- **Delivery charges:** 18% GST (post Sep 2025 reform)
- **GST-unregistered restaurants:** Cannot sell "goods" tagged items
- **Composite dealers:** Special GST rules apply

---

## 16. NOTIFICATION & COMMUNICATION SYSTEM

### 16.1 Push Notification Types
| Type | Trigger | Content |
|------|---------|---------|
| **Order Status** | State change | "Your order from Domino's is being prepared" |
| **Promotional** | Scheduled | "50% OFF on your favorite biryani!" |
| **Abandoned Cart** | Cart idle >10 min | "Don't forget your cart! Complete your order" |
| **Reorder** | Time-based | "Hungry? Reorder your usual from McDonald's" |
| **Gold** | Membership | "Your Gold membership expires in 3 days" |
| **Location** | Geo-fence | "You're near a great restaurant - 20% OFF" |
| **Referral** | Program | "Rahul used your code! You earned ₹50" |

### 16.2 In-App Messaging
- **Order Chat:** Dedicated chat channel between user, restaurant, and delivery partner
- **Support Chat:** AI-powered + human escalation
- **System Messages:** "Restaurant is busy, order may take 10 min extra"

---

## 17. RESTAURANT PARTNER DASHBOARD (For Reference)

### 17.1 Key Features for Restaurant Owners
- **Menu Management:** Upload/edit menu, prices, availability, images
- **Order Management:** Accept/reject orders, preparation time settings
- **Live Orders:** Real-time order stream with print integration
- **Analytics:** Sales reports, peak hours, popular items, Menu Score
- **Promotions:** Create restaurant-level offers
- **Reconciliation:** Payouts, invoice generation, tax reports

### 17.2 Menu Score for Restaurants
Zomato provides a score (0-100) based on:
- Image quality and coverage
- Description completeness
- Category organization
- Item naming and searchability
- Modifier group setup

---

## 18. KEY UX PATTERNS & DESIGN PRINCIPLES

### 18.1 Design Philosophy
- **Speed First:** Every screen optimized for <2s load time
- **Density:** Information-rich screens with minimal whitespace waste
- **Color Coding:**
  - Green = Veg, Red = Non-Veg, Brown = Egg
  - Red stars = Delivery, Black stars = Dining
- **Bottom Sheets:** Used for filters, customizations, offer details, and confirmations
- **Sticky Elements:** Category tabs, search bars, and CTAs stick during scroll
- **Skeleton Loading:** Shimmer effects while content loads

### 18.2 Gestures & Interactions
| Gesture | Action |
|---------|--------|
| Pull to refresh | Restaurant list refresh |
| Horizontal swipe | Image carousels, cuisine rails |
| Tap on category tab | Jump to menu section |
| Long press | Quick preview (experimental) |
| Swipe down | Dismiss bottom sheet |

### 18.3 Accessibility
- Dietary indicators use color + shape (not just color)
- Alt text for images
- Screen reader optimized card layouts
- High contrast mode support

---

## 19. CLONE IMPLEMENTATION ROADMAP

### 19.1 Phase 1: MVP (Months 1-3)
- [ ] User auth (phone OTP, social login)
- [ ] Restaurant listing with basic cards
- [ ] Menu display with categories
- [ ] Cart functionality
- [ ] Checkout with COD and UPI
- [ ] Basic order tracking (status updates)

### 19.2 Phase 2: Core Features (Months 4-6)
- [ ] Search with autocomplete
- [ ] Filter system
- [ ] Rating and reviews
- [ ] Live delivery tracking (map integration)
- [ ] Promo code system
- [ ] Push notifications

### 19.3 Phase 3: Advanced (Months 7-9)
- [ ] Subscription program (Gold equivalent)
- [ ] Advanced personalization
- [ ] Restaurant partner dashboard
- [ ] Delivery partner app
- [ ] AI-powered recommendations
- [ ] Dish-level search

### 19.4 Phase 4: Scale (Months 10-12)
- [ ] Multi-city expansion
- [ ] Dynamic pricing engine
- [ ] A/B testing framework
- [ ] Real-time analytics
- [ ] Advanced fraud detection

### 19.5 Tech Stack Recommendation for Clone

| Layer | Technology |
|-------|-----------|
| **Mobile Apps** | React Native / Flutter (for speed) or native Swift/Kotlin |
| **Backend API** | Node.js / Python (FastAPI/Django) / Go |
| **Database** | PostgreSQL (primary), Redis (cache/sessions) |
| **Search** | Elasticsearch or Algolia |
| **Queue** | RabbitMQ / Apache Kafka |
| **Maps** | Google Maps SDK / Mapbox |
| **Payments** | Razorpay / Stripe (India) |
| **Push** | Firebase Cloud Messaging |
| **Hosting** | AWS / GCP / Azure |
| **CDN** | CloudFront / Cloudflare |
| **Images** | AWS S3 + CloudFront |

---

## 20. KEY METRICS TO TRACK

| Metric | Description |
|--------|-------------|
| **Menu-to-Cart Ratio** | % of menu views that result in items added to cart |
| **Cart Conversion** | % of carts that result in completed orders |
| **Average Order Value (AOV)** | Average ₹ per order |
| **Repeat Order Rate** | % of users who reorder within 30 days |
| **Delivery ETA Accuracy** | % of orders delivered within promised window |
| **App Load Time** | Time to interactive on key screens |
| **Search Success Rate** | % of searches resulting in a restaurant/dish selection |
| **Gold Penetration** | % of active users with subscription |
| **Take Rate** | % of GOV retained as revenue |

---

## 21. COMPETITIVE DIFFERENTIATORS (What Makes Zomato Unique)

1. **Dine-in Discovery:** Unlike pure delivery apps, Zomato retains the original restaurant discovery DNA
2. **Menu Density:** Even small-town restaurants have complete menus with descriptions
3. **Gold Program:** Subscription model that drives 40% of GOV with low churn
4. **Photo Reviews:** Heavy emphasis on user-generated food photos
5. **Dual Ratings:** Separate delivery and dining ratings for same restaurant
6. **Platform Fee Model:** Transparent fee structure that users accept
7. **Live Activities:** iOS-specific deep integration for order tracking

---

*Report compiled from public documentation, API references, user experience analysis, and industry reports. For internal technical specifications, refer to Zomato's developer documentation and engineering blog posts.*

**Sources:** Zomato Engineering Blog, Zomato Developer API Docs, Zomato Investor Relations (Q2 FY24, Q3 FY25), Industry Analysis (Inc42, Economic Times, Business Today), System Design Publications, User Review Analysis (Trustpilot, App Store reviews)