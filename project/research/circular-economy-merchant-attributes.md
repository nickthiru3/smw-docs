# Merchants Search Service: Comprehensive Data Attributes Framework  

Based on the SMW document and circular economy best practices, here's a consolidated data attribute specification for merchant listings:  

## Data Attributes Table  

| Attribute | Description | Example Values | Priority | Source/Notes |  
|-----------|-------------|----------------|----------|--------------|  
| **BUSINESS IDENTITY** |||||  
| Business Legal Name | Registered company name | "ABC Repair Sdn Bhd" | **Mandatory** | SSM verification required |  
| Trading Name (DBA) | Display name if different | "Mr. Fix-It Workshop" | Optional | User-facing brand name |  
| SSM Registration Number | Malaysia Companies Commission ID | "201901234567 (1234567-X)" | **Mandatory** | Required for Silver+ packages; verification badge |  
| Business Type | Legal entity structure | Sole proprietor, Sdn Bhd, NGO, Social Enterprise, Cooperative | **Mandatory** | Affects credibility and compliance |  
| Logo Image | Business branding | URL or file upload | **Mandatory** | 1 logo for Basic, more for premium |  
| **CONTACT & LOCATION** |||||  
| Primary Address | Physical location | "123 Jalan Raja, 50050 KL" | **Mandatory** | Required for all listings |  
| Service Area Radius | Geographic coverage | "5km", "10km", "Statewide", "Nationwide" | **Mandatory** | Matches user distance filters |  
| GPS Coordinates | Lat/Long for mapping | 3.1390, 101.6869 | **Mandatory** | Google Maps/Waze integration |  
| Additional Branches | Multiple locations | Array of addresses | Optional | 3-10% discount scheme |  
| Phone Number | Contact number | "+60123456789" | **Mandatory** | Listed in basic info |  
| WhatsApp Number | Messaging contact | "+60123456789" | Optional | Common in Malaysia |  
| Email Address | Business email | "info@business.com" | **Mandatory** | For account management |  
| Website URL | Company website | "https://business.com" | Optional | Link provided in listings |  
| Facebook Page | Social media link | "facebook.com/business" | Optional | Check-in integration |  
| Instagram Handle | Social media | "@business_ig" | Optional | Sharing functionality |  
| **OPERATIONAL DETAILS** |||||  
| Operating Hours | Business hours by day | "Mon-Fri: 9AM-6PM, Sat: 9AM-1PM, Sun: Closed" | **Mandatory** | Listed in all packages |  
| Operating Days | Days of operation | For markets: specific dates/weekly schedule | **Mandatory** | Essential for pasar malam, local markets |  
| Seasonal Schedule | Temporary closures | "Closed during Hari Raya", "Summer only" | Optional | Improves customer experience |  
| Appointment Required | Walk-in vs booking | Boolean: Yes/No | Optional | Important for repair services |  
| Lead Time | Service turnaround | "Same day", "2-3 days", "1 week+" | Optional | User planning needs |  
| Minimum Order/Fee | Transaction requirements | "RM50 minimum", "No minimum" | Optional | Sets expectations |  
| Delivery Available | Delivery service | Boolean + coverage area | Optional | Especially for bulk/grocery |  
| Pickup Service | Collection service | Boolean + terms | Optional | Relevant for donations, composting |  
| **CIRCULAR ECONOMY CLASSIFICATION** |||||  
| Primary CE Category | Main circular activity | Repair, Refill, Recycle, Donate/Divert, Rent, Subscription, 2nd-Hand, Green Retail | **Mandatory** | Core app taxonomy |  
| Secondary CE Categories | Additional activities | Array of categories | Optional | Multi-service providers |  
| Specific Service Types | Detailed offerings | For Repair: "Tailor", "Small Electronics", "Large Appliances", "Furniture", "Watch" | **Mandatory** | Enables granular search |  
| Products/Services List | Itemized offerings | Up to 25/50/100 items based on package | Tier-dependent | Silver: 25, Gold: 50, Platinum: 100 |  
| Accepted Waste Types | Materials handled | "HDPE", "PP", "Corrugated cardboard", "E-waste", "Textiles", "Organic waste" | **Mandatory** (Recycling) | Multi-item search |  
| Product Categories Sold | Retail inventory types | "Biodegradable packaging", "Bamboo products", "Upcycled goods", "Local produce" | Optional | For green retailers |  
| Rental Item Categories | Rentable products | "Furniture", "Electronics", "Party props", "Plants", "Instruments" | Optional | Category-specific |  
| **SUSTAINABILITY CREDENTIALS** |||||  
| Environmental Certifications | Third-party validation | "ISO 14001", "Green Building Index", "MyHIJAU", "B Corp", "Cradle to Cradle" | Optional | Trust signals; no specific requirement in doc |  
| Zero Waste Practices | Specific initiatives | "BYO container discount", "Packaging-free", "Composting program", "Take-back scheme" | Optional | Differentiator for cafes/restaurants |  
| Sustainable Packaging | Packaging approach | "Biodegradable containers", "Reusable packaging", "No packaging" | Optional | Restaurant/cafe filter |  
| Local Sourcing | Product origin | "100% local", "Majority local", "Some local" | Optional | Buy local mandate |  
| Social Enterprise Status | Impact business | Boolean + mission statement | Optional | Social enterprise category |  
| Waste Diversion Stats | Impact metrics | "500kg recycled monthly", "30% food waste reduced" | Optional | Phase 3 carbon tracking alignment |  
| EPR Compliance | Extended Producer Responsibility | "Take-back program active", "EPR registration #" | Optional | Subscription services |  
| **VERIFICATION & TRUST SIGNALS** |||||  
| Verification Status | Admin approval | "Pending", "Verified", "Rejected" | System-managed | Internal verification |  
| SSM Verification Badge | Business legitimacy | Boolean display | Auto (Silver+) | Tied to subscription tier |  
| User Rating | Average customer rating | 1.0-5.0 stars | System-calculated | Displayed in search results |  
| Number of Reviews | Review count | Integer | System-calculated | Social proof metric |  
| Verification Date | Last verified | ISO 8601 date | System-managed | Data freshness indicator |  
| Claimed Listing | Owner-managed | Boolean | System-flag | Business owner signed in |  
| Years in Operation | Business longevity | "Since 2015" or calculated | Optional | Trust signal |  
| **SEARCH & DISCOVERY METADATA** |||||  
| Keywords/Tags | Search terms | Up to 25/50/100 based on package | Tier-dependent | SEO optimization |  
| Search Ranking Tier | Paid placement | "Top 5", "Top 10", "Standard" | Package-based | Silver: Top 10, Gold/Platinum: Top 5 |  
| Featured Listing | Priority visibility | Boolean | Package-based | Revenue driver |  
| Description (Short) | Brief summary | 100-150 characters | **Mandatory** | Search result snippet |  
| Description (Long) | Detailed information | 500-1000 characters | Optional | Detail page content |  
| Popular Services | Most-used offerings | Array of frequently searched items | System-calculated | Improves relevance |  
| **MEDIA ASSETS** |||||  
| Photos | Business images | URLs/files, quantity by tier | Tier-dependent | Silver: 25, Gold: 50, Platinum: 100 |  
| Videos | Promotional videos | YouTube embeds | Tier-dependent | Silver: 1, Gold: 2, Platinum: 4 |  
| User-Generated Photos | Customer uploads | Array of image URLs | System-managed | Review feature |  
| 360° Virtual Tour | Immersive view | URL or embed code | Optional | Future enhancement |  
| **PROMOTIONS & MARKETING** |||||  
| Current Promotions | Special offers | Text + validity period | Optional | Premium feature |  
| Banner Advertisements | Visual ads | Image URLs for top/side placement | Package-based | Gold: Side, Platinum: Top+Side |  
| Loyalty Program | Customer rewards | Program description | Optional | Phase 2 integration |  
| Referral Discount | Word-of-mouth incentive | "Bring a friend: 10% off" | Optional | Acquisition strategy |  
| **ACCOUNT MANAGEMENT** |||||  
| Subscription Package | Tier level | "Basic (Free)", "Silver (RM600/yr)", "Gold (RM1200/yr)", "Platinum (RM1680/yr)" | System-managed | Access control |  
| Package Start Date | Subscription began | ISO 8601 date | System-managed | Billing cycle |  
| Package End Date | Renewal date | ISO 8601 date | System-managed | Renewal reminders (3-10% discount) |  
| Content Update Allowance | Edit frequency | Silver: 2x/yr, Gold: 4x/yr, Platinum: Unlimited | Package-based | Rate limiting |  
| Last Content Update | Recent modification | ISO 8601 timestamp | System-tracked | Admin monitoring |  
| Payment Status | Billing status | "Active", "Pending", "Expired", "Trial" | System-managed | Access enforcement |  
| Account Owner Email | Admin contact | Email address | **Mandatory** | Verified email required |  
| Account Owner Name | Primary contact | Full name | **Mandatory** | Communication |  
| **REGULATORY & COMPLIANCE** |||||  
| Business License | Local authority permit | License number + expiry | Optional | Depends on category |  
| Health Permit | Food establishment | MOH registration (for cafes/restaurants) | Category-specific | Restaurant/cafe mandatory |  
| Waste Handling License | DOE authorization | For recycling centers/collectors | Category-specific | Recycling businesses |  
| Fire Safety Certificate | Premise safety | BOMBA certification | Optional | Large operations |  
| Halal Certification | Food compliance | JAKIM cert number (for F&B) | Optional | Muslim-majority market |  
| Insurance Coverage | Liability protection | Type + coverage amount | Optional | Professional services |  
| **USER ENGAGEMENT METRICS** |||||  
| Total Check-ins | User visits logged | Integer count | System-tracked | Facebook integration |  
| Total Saves/Favorites | Bookmark count | Integer count | System-tracked | Popularity indicator |  
| Profile Views | Page impressions | Integer count | System-tracked | Analytics for owners |  
| Click-through Rate | Search → detail page | Percentage | System-calculated | Search optimization |  
| Conversion Events | User actions taken | "Called", "Visited website", "Got directions" | System-tracked | ROI metrics |  

---  

## Circular Economy Framework Sources  

### 1. **Repair/Reuse/Refurbish**  
The document explicitly addresses repair services for extending product lifecycles:  
- Tailoring (textiles)  
- Small electronics (phones, tablets, speakers)  
- Large appliances (fridges, washing machines, TVs)  
- Furniture refurbishing  
- Watches and small items  

**CE Principle:** Product life extension (Inner loop of circularity)  

### 2. **Refill/Package-Free Retail**  
Zero waste shopping model where consumers bring containers:  
- Refill stations for liquids/bulk goods  
- Packaging-free stores  
- Bulk suppliers  
- Supermarket refill counters  

**CE Principle:** Waste prevention, reusable packaging systems  

### 3. **Recycling & Material Recovery**  
Traditional end-of-life material processing:  
- Council recycling centers  
- Private recycling facilities  
- NGO drop bins (Erth, Icycle, RSSS)  
- Reverse vending machines  
- Buy-back centers  

**CE Principle:** Material cycling (outer loop, last resort)  

### 4. **Divert/Donate**  
Alternative end-of-life pathways to avoid landfill:  
- Composting sites (organic waste → soil amendment)  
- Donation centers (children's homes, refugee camps)  
- Textile recycling (Kloth bins)  
- Animal shelters (food waste)  

**CE Principle:** Value retention, waste hierarchy  

### 5. **Sharing Economy/Rental**  
Access over ownership model:  
- Equipment rental (projectors, instruments, party props)  
- Furniture rental  
- Plant rental  
- Short-term use items  

**CE Principle:** Intensive product use, dematerialization  

### 6. **Subscription/Product-as-Service**  
Producer retains ownership and responsibility:  
- Container/packaging take-back programs  
- Refillable product delivery (water, milk)  
- Printer ink refill services  

**CE Principle:** Producer responsibility, circular supply chains  

### 7. **Second-Hand/Re-commerce**  
Pre-owned product markets:  
- Bundle shops  
- Used furniture stores  
- Second-hand electronics  
- Vintage/thrift retail  

**CE Principle:** Product life extension, value retention  

### 8. **Green/Sustainable Retail**  
New products designed for circularity:  
- Locally made sustainable goods  
- Biodegradable products  
- Social enterprise items  
- Upcycled products  

**CE Principle:** Circular design, regenerative materials  

### 9. **Local Markets**  
Short supply chains, minimal packaging:  
- Pasar (traditional markets)  
- Pasar malam (night markets)  
- Farmers markets  
- Car boot sales  

**CE Principle:** Localization, packaging reduction  

### 10. **Cafes/Restaurants (Zero Waste)**  
Food service with circular practices:  
- BYO container acceptance  
- Food waste management  
- Eco-friendly takeaway packaging  

**CE Principle:** Food waste prevention, packaging elimination  

### 11. **Community Groups/Activities**  
Behavior change and collective action:  
- Tree planting events  
- Beach cleanups  
- DIY/upcycling workshops  
- CE education talks  

**CE Principle:** Systemic change, community engagement  

---  

## External Framework References  

While the SMW document doesn't cite external sources, here are industry-standard frameworks to validate the taxonomy:  

| Framework | Relevant Categories | URL | Notes |  
|-----------|---------------------|-----|-------|  
| **Ellen MacArthur Foundation - Butterfly Diagram** | Biological/Technical cycles, Inner/Outer loops | [ellenmacarthurfoundation.org/circular-economy-diagram](https://ellenmacarthurfoundation.org/circular-economy-diagram) | Gold standard CE framework; SMW categories map to maintenance, reuse, refurbish, remanufacture, recycle loops |  
| **EU Waste Hierarchy (Directive 2008/98/EC)** | Prevention, Reuse, Recycling, Recovery, Disposal | [ec.europa.eu/environment/waste](https://ec.europa.eu/environment/waste/framework/) | Legal framework; SMW prioritizes prevention/reuse over recycling |  
| **ISO 59020:2024 - Circular Economy** | Measuring circularity in organizations | [iso.org/standard/80650.html](https://www.iso.org/standard/80650.html) | New standard; verification metrics for waste diversion stats |  
| **MyHIJAU Directory (Malaysia)** | Green product/service certification | [myhijau.my](https://www.myhijau.my/) | Local relevance; potential integration for verified badges |  
| **Sharing Economy Association** | Rental, subscription, access models | [sharingeconomyuk.com](https://www.sharingeconomyuk.com/) | Best practices for rental service classification |  

---  

## Mandatory vs. Optional Attribute Summary  

### **Mandatory at Sign-Up (Basic Free Listing)**  
1. Business Legal Name + SSM Number (if paid tier)  
2. Primary Address + GPS Coordinates  
3. Service Area Radius  
4. Phone Number + Email  
5. Operating Hours  
6. Primary CE Category  
7. Specific Service Types  
8. Short Description  
9. Logo (1 image minimum)  
10. Account Owner Email (verified)  

### **Tier-Dependent (Paid Packages)**  
1. Number of Products/Services (25/50/100)  
2. Number of Images (25/50/100)  
3. Number of Keywords (25/50/100)  
4. Videos (1/2/4)  
5. Content Update Frequency (2x, 4x, unlimited/year)  
6. Search Ranking Placement (Top 10 vs Top 5)  
7. Banner Ad Slots (None, Side, Top+Side)  
8. SSM Verification Badge  

### **Optional Enhancements (Value-Add)**  
1. Secondary CE Categories  
2. Sustainability Certifications  
3. Waste Diversion Stats  
4. Promotions/Special Offers  
5. Social Media Links  
6. Delivery/Pickup Options  
7. User-Generated Content  
8. 360° Tours  
9. Loyalty Program Details  
10. Regulatory Certifications  

---  

## Critical Notes for Development  

### Search Relevance Priorities  
1. **Location proximity** is paramount - 3 distance filters explicitly required  
2. **Multi-filter support** - users search by activity AND waste type AND product simultaneously  
3. **Tiered search results** - paid listings appear in Top 5 or Top 10  
4. **Keyword matching** - 25-100 keywords per listing  

### Operational Considerations  
1. **Update frequency limits** prevent spam and encourage annual renewals  
2. **Verification workflow** requires admin approval before publishing  
3. **SSM verification** is automated for Silver+ tiers  
4. **Multi-branch discounts** need sibling-listing logic  

### Data Quality & Trust  
1. **No explicit greenwashing prevention** - major gap requiring policy  
2. **User reviews** are the primary trust mechanism  
3. **Check-in integration** provides social proof  
4. **Claimed vs unclaimed** listings affect credibility  

This framework balances the document's explicit requirements with circular economy best practices, creating a robust merchant data model for search and discovery.