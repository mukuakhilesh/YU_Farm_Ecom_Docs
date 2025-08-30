# DynamoDB Multi-Table Schema Design for E-commerce Platform (Clean)

## Table Design Strategy

Using **Multi-Table Design** with 9 core tables for essential e-commerce functionality.

## Table Structure

### 1. Users Table
**Primary Key**: `PK: userid`, `SK: PROFILE | ADDRESS#{id} | SESSION#{token}`
**GSI1**: Email Lookup - `GSI1PK: email`, `GSI1SK: userid`

### 2. Sellers Table
**Primary Key**: `PK: sellerId`, `SK: INFO | DOCUMENT#{type} | BANK#{id}`
**GSI1**: Sellers by Status - `GSI1PK: STATUS#{status}`, `GSI1SK: SELLER#{sellerId}`
**GSI2**: Sellers by Category - `GSI2PK: CATEGORY#{businessType}`, `GSI2SK: RATING#{rating}#SELLER#{sellerId}`

### 3. Products Table
**Primary Key**: `PK: productId`, `SK: INFO | SELLER#{sellerId} | IMAGE#{id} | SPEC#{name}`
**GSI1**: Category Browsing - `GSI1PK: CATEGORY#{categoryId}`, `GSI1SK: PRICE#{price}#RATING#{rating}`
**GSI2**: Seller Products - `GSI2PK: sellerId`, `GSI2SK: productId`

### 4. Orders Table
**Primary Key**: `PK: userid`, `SK: ORDER#{date}#{transactionId}`
**GSI1**: Order by Transaction ID - `GSI1PK: transactionId`, `GSI1SK: userid`
**GSI2**: Orders by Status - `GSI2PK: orderStatus`, `GSI2SK: ORDER#{date}#{transactionId}`
**LSI1**: Orders by Status (per user) - `PK: userid`, `LSI1SK: STATUS#{status}#ORDER#{date}#{transactionId}`

### 5. OrderItems Table
**Primary Key**: `PK: transactionId`, `SK: ITEM#{productId}#{sellerId}`
**GSI1**: Seller Orders - `GSI1PK: sellerId`, `GSI1SK: ORDER#{date}#{transactionId}`

### 6. Cart Table
**Primary Key**: `PK: userid`, `SK: ITEM#{productId}#{sellerId}`

### 7. Wishlist Table
**Primary Key**: `PK: userid`, `SK: WISHLIST#{wishlistId} | ITEM#{wishlistId}#{productId}`

### 8. Reviews Table
**Primary Key**: `PK: productId`, `SK: REVIEW#{date}#{reviewerId}`
**GSI1**: Reviews by Seller - `GSI1PK: sellerId`, `GSI1SK: REVIEW#{date}#{reviewerId}`

### 9. Rewards Table
**Primary Key**: `PK: userid`, `SK: REWARD#{date}#{type}#{id}`

### 10. Categories Table
**Primary Key**: `PK: categoryId`, `SK: INFO | SUBCATEGORY#{subcategoryId}`

### 11. Inventory Table
**Primary Key**: `PK: productId`, `SK: STOCK#{lotBatch}#{sellerId}`
**GSI1**: Stock by Seller - `GSI1PK: sellerId`, `GSI1SK: STOCK#{productId}#{lotBatch}`

### 12. UserGroups Table
**Primary Key**: `PK: userGroupId`, `SK: INFO`

## Enhanced Entity Details

### Products Table - With Embedded Terms
```javascript
// Product Base Info
{
  PK: "PROD123",
  SK: "INFO",
  name: "iPhone 15",
  description: "Latest iPhone model",
  brand: "Apple",
  categoryId: 1,
  subcategoryId: 5,
  thumbnail: "iphone15.jpg",
  productRewardPercent: 10,
  
  // Filters embedded for UI
  filterableAttributes: {
    brand: "Apple",
    color: ["Space Black", "Silver", "Gold"],
    storage: ["128GB", "256GB", "512GB"],
    priceRange: "100000-150000",
    features: ["5G", "Face ID", "Wireless Charging"]
  }
}

// Seller Listing - With Terms
{
  PK: "PROD123",
  SK: "SELLER#seller456",
  sellerId: "seller456",
  sellerName: "TechStore Mumbai",
  price: 125000,
  quantity: 50,
  isAvailable: true,
  
  // Terms embedded
  terms: {
    returnPolicy: "7 days return policy",
    warrantyPeriod: "1 year manufacturer warranty", 
    shippingPolicy: "Free shipping above ₹500",
    cancellationPolicy: "Cancel within 24 hours",
    conditions: [
      "Items must be in original packaging",
      "Return shipping paid by customer"
    ]
  }
}
```

### Sellers Table - With Default Terms
```javascript
{
  PK: "seller456",
  SK: "INFO",
  businessName: "TechStore Pvt Ltd",
  displayName: "TechStore Mumbai",
  
  // Default terms for all products
  defaultTerms: {
    returnPolicy: "7 days return policy",
    warrantyPolicy: "As per manufacturer warranty",
    shippingPolicy: "Free shipping above ₹500",
    cancellationPolicy: "24 hours cancellation window"
  },
  
  // Business configuration
  paymentMethods: ["COD", "UPI", "Card", "Wallet"],
  orderStatuses: ["Initiated", "Processing", "Packed", "Shipped", "Delivered"],
  businessHours: "9 AM - 9 PM"
}
```

## Access Patterns by Table

### Users Table
- Get user profile: `PK = userid123, SK = PROFILE`
- Get user addresses: `PK = userid123, SK begins_with ADDRESS#`
- Get user by email: `GSI1PK = email`

### Sellers Table
- Get seller profile: `PK = seller456, SK = INFO`
- Get seller documents: `PK = seller456, SK begins_with DOCUMENT#`
- Get active sellers: `GSI1PK = STATUS#ACTIVE`

### Products Table
- Get product info: `PK = PROD123, SK = INFO`
- Get seller listings: `PK = PROD123, SK begins_with SELLER#`
- Browse by category: `GSI1PK = CATEGORY#1`
- Get seller products: `GSI2PK = seller456`

### Orders Table
- Get user orders: `PK = userid123, SK begins_with ORDER#`
- Get order by transaction: `GSI1PK = TXN123`
- Get orders by status: `GSI2PK = Delivered`

### OrderItems Table
- Get order items: `PK = TXN123, SK begins_with ITEM#`
- Get seller order items: `GSI1PK = seller456`

### Cart Table
- Get user cart: `PK = userid123, SK begins_with ITEM#`

### Wishlist Table
- Get user wishlists: `PK = userid123, SK begins_with WISHLIST#`

### Reviews Table
- Get product reviews: `PK = PROD123, SK begins_with REVIEW#`
- Get seller reviews: `GSI1PK = seller456`

### Rewards Table
- Get user rewards: `PK = userid123, SK begins_with REWARD#`

### Categories Table
- Get category info: `PK = 1, SK = INFO`
- Get subcategories: `PK = 1, SK begins_with SUBCATEGORY#`

### Inventory Table
- Get product stock: `PK = PROD123, SK begins_with STOCK#`
- Get seller stock: `GSI1PK = seller456`

### UserGroups Table
- Get user group benefits: `PK = 1, SK = INFO`

## Benefits of Clean Architecture

### Advantages
- **Minimal Complexity**: Only essential tables for core functionality
- **Embedded Data**: Terms and filters where they belong contextually
- **UI-Driven Filters**: Frontend handles filter logic and presentation
- **Reduced Overhead**: Fewer tables to manage and maintain
- **Better Performance**: Related data stored together
- **Cleaner Code**: Simpler relationships and access patterns

### Core Features Supported
- ✅ User management and authentication
- ✅ Multi-vendor seller onboarding with KYC
- ✅ Product catalog with rich metadata
- ✅ Order processing and tracking
- ✅ Shopping cart and wishlist
- ✅ Review and rating system
- ✅ Reward points system
- ✅ Inventory management
- ✅ User segmentation and pricing tiers

### Removed Complexity
- ❌ System configuration tables
- ❌ Separate terms management
- ❌ Complex filter data structures
- ❌ Lookup value management
- ❌ Over-engineered relationships

This clean approach focuses on core business functionality while keeping the architecture simple and maintainable.
