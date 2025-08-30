# DynamoDB Multi-Table Schema Design for E-commerce Platform

## Table Design Strategy

Using **Multi-Table Design** with separate tables for each domain, optimized for flexibility and future extensions.

## Table Structure

### 1. Users Table

**Primary Key**: `PK: userid`, `SK: PROFILE | ADDRESS#{id} | SESSION#{token}`

```javascript
// User Profile
{
  PK: "userid123",
  SK: "PROFILE",
  email: "user@example.com",
  name: "John Doe",
  phone: 1234567890,
  roletype: "USER",
  status: "ACTIVE",
  rewardPoints: 1000,
  avgSellerRating: 4.5,
  userGroupId: 1
}

// User Address
{
  PK: "userid123",
  SK: "ADDRESS#home",
  name: "Home",
  address: "123 Main St",
  city: "Mumbai",
  state: "Maharashtra",
  pin: 400001,
  country: "India"
}

// User Session
{
  PK: "userid123",
  SK: "SESSION#token123",
  accessToken: "jwt_token",
  refreshToken: "refresh_token",
  createdAt: "2025-01-11T10:22:43Z",
  expiresAt: "2025-01-12T10:22:43Z",
  TTL: 1736683363
}
```

**GSI1**: Email Lookup
- `GSI1PK: email`, `GSI1SK: userid`

### 2. Products Table

**Primary Key**: `PK: productId`, `SK: INFO | SELLER#{sellerId} | IMAGE#{id} | SPEC#{name}`

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
  productRewardPercent: 10
}

// Seller Listing
{
  PK: "PROD123",
  SK: "SELLER#seller456",
  sellerId: "seller456",
  sellerProductName: "iPhone 15 Pro Max",
  price: 125000,
  quantity: 50,
  isAvailable: true,
  avgProductRating: 4.5,
  sellerRewardPercent: 10
}

// Product Image
{
  PK: "PROD123",
  SK: "IMAGE#1",
  url: "iphone15_1.jpg",
  type: "image"
}

// Product Specification
{
  PK: "PROD123",
  SK: "SPEC#RAM",
  spec: "RAM",
  specValue: "8GB"
}
```

**GSI1**: Category Browsing
- `GSI1PK: CATEGORY#{categoryId}`, `GSI1SK: PRICE#{price}#RATING#{rating}`

**GSI2**: Seller Products
- `GSI2PK: sellerId`, `GSI2SK: productId`

### 3. Orders Table

**Primary Key**: `PK: userid`, `SK: ORDER#{date}#{transactionId}`

```javascript
// Order Header
{
  PK: "userid123",
  SK: "ORDER#2025-01-11#TXN123",
  transactionId: "TXN123",
  purchasedAt: 1705000000000,
  totalAmount: 125000,
  netPayable: 147500,
  paymentType: "UPI",
  paymentStatus: "Paid",
  orderStatus: "Delivered",
  buyerName: "John Doe",
  buyerAddress: "123 Main St, Mumbai"
}
```

**GSI1**: Order by Transaction ID
- `GSI1PK: transactionId`, `GSI1SK: userid`

**GSI2**: Orders by Status
- `GSI2PK: orderStatus`, `GSI2SK: ORDER#{date}#{transactionId}`

**LSI1**: Orders by Status (per user)
- `PK: userid`, `LSI1SK: STATUS#{status}#ORDER#{date}#{transactionId}`

### 4. OrderItems Table

**Primary Key**: `PK: transactionId`, `SK: ITEM#{productId}#{sellerId}`

```javascript
{
  PK: "TXN123",
  SK: "ITEM#PROD123#seller456",
  productId: "PROD123",
  sellerId: "seller456",
  productName: "iPhone 15",
  quantity: 1,
  price: 125000,
  orderStatus: "Delivered",
  courierName: "BlueDart",
  trackingCode: "BD123456"
}
```

**GSI1**: Seller Orders
- `GSI1PK: sellerId`, `GSI1SK: ORDER#{date}#{transactionId}`

### 5. Cart Table

**Primary Key**: `PK: userid`, `SK: ITEM#{productId}#{sellerId}`

```javascript
{
  PK: "userid123",
  SK: "ITEM#PROD123#seller456",
  productId: "PROD123",
  sellerId: "seller456",
  productName: "iPhone 15",
  price: 125000,
  quantity: 2,
  addedAt: 1705000000000
}
```

### 6. Wishlist Table

**Primary Key**: `PK: userid`, `SK: WISHLIST#{wishlistId} | ITEM#{wishlistId}#{productId}`

```javascript
// Wishlist Header
{
  PK: "userid123",
  SK: "WISHLIST#WISH123",
  wishlistId: "WISH123",
  name: "My Favorites",
  description: "Items I want to buy"
}

// Wishlist Item
{
  PK: "userid123",
  SK: "ITEM#WISH123#PROD123",
  wishlistId: "WISH123",
  productId: "PROD123",
  productName: "iPhone 15",
  price: 125000,
  addedAt: 1705000000000
}
```

### 7. Reviews Table

**Primary Key**: `PK: productId`, `SK: REVIEW#{date}#{reviewerId}`

```javascript
{
  PK: "PROD123",
  SK: "REVIEW#2025-01-11#userid123",
  reviewerId: "userid123",
  reviewerName: "John Doe",
  sellerId: "seller456",
  productRating: 5,
  productDeliveryRating: 4,
  sellerRating: 5,
  remarks: "Excellent product",
  createdAt: 1705000000000
}
```

**GSI1**: Reviews by Seller
- `GSI1PK: sellerId`, `GSI1SK: REVIEW#{date}#{reviewerId}`

### 8. Rewards Table

**Primary Key**: `PK: userid`, `SK: REWARD#{date}#{type}#{id}`

```javascript
{
  PK: "userid123",
  SK: "REWARD#2025-01-11#EARN#TXN123",
  rewardType: "EARN",
  rewardFor: "PURCHASE",
  rewardValue: 1250,
  purchaseId: "TXN123",
  rewardDate: 1705000000000
}
```

### 9. Coupons Table

**Primary Key**: `PK: couponCode`, `SK: INFO | USAGE#{userid}#{transactionId}`

```javascript
// Coupon Info
{
  PK: "SAVE10",
  SK: "INFO",
  couponType: "AMOUNT",
  couponValue: 1000,
  expiryDate: 1735689600000,
  limit: 100,
  usedTimes: 25
}

// Coupon Usage
{
  PK: "SAVE10",
  SK: "USAGE#userid123#TXN123",
  userId: "userid123",
  transactionId: "TXN123",
  redeemDate: 1705000000000
}
```

### 10. Campaigns Table

**Primary Key**: `PK: campaignId`, `SK: INFO | USER#{userid}`

```javascript
// Campaign Info
{
  PK: "CAMP123",
  SK: "INFO",
  campaignName: "Summer Sale",
  startDate: 1704067200000,
  endDate: 1706745600000,
  campaignType: "EMAIL",
  sellerId: "seller456"
}

// Campaign Target User
{
  PK: "CAMP123",
  SK: "USER#userid123",
  userId: "userid123",
  targetedAt: 1705000000000
}
```

### 11. Leads Table

**Primary Key**: `PK: leadId`, `SK: INFO | ITEM#{productId} | STATUS#{date}#{id}`

```javascript
// Lead Info
{
  PK: "LEAD001",
  SK: "INFO",
  customerId: "userid123",
  customerName: "John Doe",
  status: "Created",
  leadPrice: 100000,
  leadDate: 1705000000000
}

// Lead Item
{
  PK: "LEAD001",
  SK: "ITEM#PROD123",
  productId: "PROD123",
  quantity: 10,
  price: 10000,
  totalPrice: 100000
}

// Lead Status
{
  PK: "LEAD001",
  SK: "STATUS#2025-01-11#1",
  status: "Created",
  givenBy: "Sales Rep",
  givenOn: 1705000000000,
  remarks: "Initial lead"
}
```

**GSI1**: Leads by Customer
- `GSI1PK: customerId`, `GSI1SK: leadId`

### 12. Categories Table

**Primary Key**: `PK: categoryId`, `SK: INFO | SUBCATEGORY#{subcategoryId}`

```javascript
// Category Info
{
  PK: "1",
  SK: "INFO",
  categoryName: "Electronics",
  image: "electronics.jpg",
  categoryFor: "Electronics"
}

// Subcategory
{
  PK: "1",
  SK: "SUBCATEGORY#5",
  subcategoryId: 5,
  subcategoryName: "Smartphones"
}
```

### 13. Inventory Table

**Primary Key**: `PK: productId`, `SK: STOCK#{lotBatch}#{sellerId}`

```javascript
{
  PK: "PROD123",
  SK: "STOCK#LOT001#seller456",
  sellerId: "seller456",
  lotBatch: "LOT001",
  stockQty: 25,
  productPrice: 125000,
  mfgDate: 1704067200000,
  expiryDate: 1735689600000,
  stockPointId: 1
}
```

**GSI1**: Stock by Seller
- `GSI1PK: sellerId`, `GSI1SK: STOCK#{productId}#{lotBatch}`

### 14. UserGroups Table

**Primary Key**: `PK: userGroupId`, `SK: INFO`

```javascript
{
  PK: "1",
  SK: "INFO",
  userGroupName: "Premium",
  sellerDiscountPercent: 10,
  adminDiscountPercent: 5,
  sellerRewardPercent: 15,
  adminRewardPercent: 10
}
```

### 15. System Tables

#### Filters Table
**Primary Key**: `PK: subcategoryId`, `SK: FILTER#{filterName} | DATA#{filterName}#{value}`

#### Lookups Table
**Primary Key**: `PK: lookupType`, `SK: lookupName`

#### Terms Table
**Primary Key**: `PK: termId`, `SK: INFO | DETAIL#{detailId}`

## Access Patterns by Table

### Users Table
- Get user profile: `PK = userid123, SK = PROFILE`
- Get user addresses: `PK = userid123, SK begins_with ADDRESS#`
- Get user by email: `GSI1PK = email`

### Products Table
- Get product info: `PK = PROD123, SK = INFO`
- Get seller listings: `PK = PROD123, SK begins_with SELLER#`
- Browse by category: `GSI1PK = CATEGORY#1`
- Get seller products: `GSI2PK = seller456`

### Orders Table
- Get user orders: `PK = userid123, SK begins_with ORDER#`
- Get order by transaction: `GSI1PK = TXN123`
- Get orders by status: `GSI2PK = Delivered`

## Benefits of Multi-Table Approach

### Advantages
- **Flexibility**: Easy to add new entity types
- **Clarity**: Each table has clear purpose
- **Independent scaling**: Scale tables based on usage
- **Easier migrations**: Modify one entity without affecting others
- **Team collaboration**: Different teams can own different tables
- **Simpler queries**: More intuitive access patterns

### Trade-offs
- **More tables to manage**: Higher operational overhead
- **Cross-table queries**: Need multiple API calls for related data
- **Higher costs**: More RCU/WCU consumption
- **No atomic transactions**: Across different tables

## When to Use Multi-Table

**Use when:**
- Requirements change frequently
- Multiple teams working on different domains
- Need flexibility for future extensions
- Clear entity boundaries
- Independent scaling requirements

This approach provides better flexibility for your e-commerce platform while still leveraging DynamoDB's performance benefits within each table.
