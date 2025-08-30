# E-commerce Database Schema (MySQL)

## Database Information
- **Database Name**: ecomm_hat
- **Server Version**: MySQL 8.0.40-0ubuntu0.24.04.1
- **Host**: 13.126.168.80
- **Dump Date**: January 11, 2025
- **Character Set**: utf8mb4

## Table Structure

### 1. User Management

#### `users`
User authentication and basic information
```sql
CREATE TABLE `users` (
  `id` int NOT NULL AUTO_INCREMENT,
  `userid` varchar(255) NOT NULL,
  `status` varchar(255) NOT NULL,
  `currentStatus` enum('ACTIVE','INACTIVE','SUSPENDED') NOT NULL DEFAULT 'ACTIVE',
  `otp` int DEFAULT NULL,
  `password` varchar(255) DEFAULT NULL,
  `email` varchar(255) NOT NULL,
  `name` varchar(255) NOT NULL,
  `roletype` enum('USER','ADMIN','MERCHANT','SALESREPRESENTATIVE') NOT NULL DEFAULT 'USER',
  PRIMARY KEY (`id`),
  UNIQUE KEY `IDX_37b098e31baedfa2b76e787699` (`userid`),
  UNIQUE KEY `IDX_97672ac88f789774dd47f7c8be` (`email`)
);
```

#### `profile`
Extended user profile information
```sql
CREATE TABLE `profile` (
  `id` int NOT NULL AUTO_INCREMENT,
  `phone` bigint DEFAULT NULL,
  `email` varchar(255) NOT NULL,
  `userid` varchar(255) NOT NULL,
  `managerId` varchar(255) DEFAULT NULL,
  `name` varchar(255) NOT NULL,
  `aboutMe` varchar(255) NOT NULL DEFAULT 'Hi there! I am using eMetaComm',
  `roletype` enum('USER','ADMIN','MERCHANT','SALESREPRESENTATIVE') NOT NULL DEFAULT 'USER',
  `profileImage` varchar(255) DEFAULT NULL,
  `rewardWalletAddress` varchar(255) DEFAULT NULL,
  `avgSellerRating` decimal(2,1) NOT NULL DEFAULT '0.0',
  `rewardPoints` bigint NOT NULL DEFAULT '0',
  `userGroupId` int DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `IDX_3825121222d5c17741373d8ad1` (`email`)
);
```

#### `address_book`
User addresses
```sql
CREATE TABLE `address_book` (
  `id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `phone` bigint DEFAULT NULL,
  `address` varchar(255) NOT NULL,
  `city` varchar(255) NOT NULL,
  `district` varchar(255) NOT NULL,
  `state` varchar(255) NOT NULL,
  `pin` int NOT NULL,
  `country` varchar(255) NOT NULL,
  `profileId` int DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

#### `user_group`
User grouping for discounts and rewards
```sql
CREATE TABLE `user_group` (
  `id` int NOT NULL AUTO_INCREMENT,
  `userGroupName` varchar(255) NOT NULL,
  `fromDate` bigint DEFAULT NULL,
  `toDate` bigint DEFAULT NULL,
  `sellerDiscountPercent` int DEFAULT '0',
  `adminDiscountPercent` int DEFAULT '0',
  `sellerRewardPercent` int DEFAULT '0',
  `adminRewardPercent` int DEFAULT '0',
  PRIMARY KEY (`id`)
);
```

#### `session`
User session management
```sql
CREATE TABLE `session` (
  `id` int NOT NULL AUTO_INCREMENT,
  `userid` varchar(255) NOT NULL,
  `createdAt` bigint DEFAULT NULL,
  `expiresAt` bigint DEFAULT NULL,
  `accessToken` varchar(255) DEFAULT NULL,
  `refreshToken` varchar(255) DEFAULT NULL,
  `isBlacklisted` tinyint NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`)
);
```

#### `user_otp`
OTP verification
```sql
CREATE TABLE `user_otp` (
  `otp_pk` int NOT NULL AUTO_INCREMENT,
  `otp_email_mobile` varchar(255) NOT NULL,
  `otp` int NOT NULL,
  `otp_userid` varchar(255) NOT NULL,
  `otp_no` int DEFAULT NULL,
  `crt_dt` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`otp_pk`)
);
```

### 2. Product Catalog

#### `product`
Base product information
```sql
CREATE TABLE `product` (
  `id` int NOT NULL AUTO_INCREMENT,
  `productId` varchar(255) NOT NULL,
  `name` varchar(255) NOT NULL,
  `arimagedata` varchar(255) DEFAULT NULL,
  `arimageurl` varchar(255) DEFAULT NULL,
  `posted_date` bigint NOT NULL,
  `description` text NOT NULL,
  `brand` varchar(255) NOT NULL,
  `productRewardPercent` int NOT NULL DEFAULT '10',
  `discountPercent` int NOT NULL DEFAULT '0',
  `thumbnail` varchar(255) DEFAULT NULL,
  `criticalStockQty` int NOT NULL DEFAULT '5',
  `categoryId` int DEFAULT NULL,
  `subcategoryId` int DEFAULT NULL,
  `productGroupId` int DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `IDX_429540a50a9f1fbf87efd047f3` (`productId`)
);
```

#### `product_records`
Seller-specific product listings
```sql
CREATE TABLE `product_records` (
  `id` int NOT NULL AUTO_INCREMENT,
  `ParentId` varchar(255) NOT NULL,
  `recordId` varchar(255) NOT NULL,
  `sellerProductName` varchar(255) NOT NULL,
  `quantity` int unsigned NOT NULL DEFAULT '0',
  `unsettledQuantity` int unsigned NOT NULL DEFAULT '0',
  `returnDate` bigint DEFAULT NULL,
  `rentPeriod` int DEFAULT NULL,
  `productId` int DEFAULT NULL,
  `isAvailable` tinyint NOT NULL DEFAULT '1',
  `purpose` varchar(255) NOT NULL,
  `itemDescription` text NOT NULL,
  `price` int NOT NULL DEFAULT '0',
  `productStatus` varchar(255) NOT NULL,
  `avgProductRating` decimal(2,1) NOT NULL DEFAULT '0.0',
  `countOfPeopleVoted` int NOT NULL DEFAULT '0',
  `sellerRewardPercent` int NOT NULL DEFAULT '10',
  `sellerDiscountPercent` int NOT NULL DEFAULT '0',
  `categoryId` int NOT NULL,
  `subcategoryId` int NOT NULL,
  `sellerDetailsId` int DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

#### `product_stock`
Inventory management
```sql
CREATE TABLE `product_stock` (
  `id` int NOT NULL AUTO_INCREMENT,
  `lotBatch` varchar(255) NOT NULL,
  `stockQty` int NOT NULL,
  `productPrice` int NOT NULL,
  `productUnit` varchar(255) NOT NULL,
  `mfgDate` bigint NOT NULL,
  `expiryDate` bigint NOT NULL,
  `stockPrice` int NOT NULL,
  `taxName1` varchar(255) DEFAULT NULL,
  `taxPercentage1` int DEFAULT '0',
  `stockPointId` int NOT NULL,
  `productRecordsId` int DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

#### `category`
Product categories
```sql
CREATE TABLE `category` (
  `id` int NOT NULL AUTO_INCREMENT,
  `categoryName` varchar(255) NOT NULL,
  `image` varchar(255) NOT NULL,
  `categoryFor` enum('RealEstate','Grocery','Electronics','Fashion','Jewellery','Interior','Pharmacy','Temporary') NOT NULL DEFAULT 'Electronics',
  PRIMARY KEY (`id`)
);
```

#### `subcategory`
Product subcategories
```sql
CREATE TABLE `subcategory` (
  `id` int NOT NULL AUTO_INCREMENT,
  `subcategoryName` varchar(255) NOT NULL,
  `categoryId` int DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

#### `brand`
Product brands
```sql
CREATE TABLE `brand` (
  `id` int NOT NULL AUTO_INCREMENT,
  `brandName` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
);
```

#### `image`
Product images and media
```sql
CREATE TABLE `image` (
  `id` int NOT NULL AUTO_INCREMENT,
  `url` varchar(255) NOT NULL,
  `type` enum('image','video','text','360','ar','vr') NOT NULL DEFAULT 'image',
  `productId` int NOT NULL,
  PRIMARY KEY (`id`)
);
```

#### `specifications`
Product specifications
```sql
CREATE TABLE `specifications` (
  `id` int NOT NULL AUTO_INCREMENT,
  `spec` varchar(255) NOT NULL,
  `specValue` varchar(255) NOT NULL,
  `productId` int NOT NULL,
  PRIMARY KEY (`id`)
);
```

#### `product_group`
Product grouping for promotions
```sql
CREATE TABLE `product_group` (
  `id` int NOT NULL AUTO_INCREMENT,
  `productGroupName` varchar(255) NOT NULL,
  `sellerDiscountPercent` int DEFAULT NULL,
  `adminDiscountPercent` int DEFAULT NULL,
  `fromDate` bigint DEFAULT NULL,
  `toDate` bigint DEFAULT NULL,
  `sellerRewardPercent` int DEFAULT NULL,
  `adminRewardPercent` int DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

### 3. Orders & Purchases

#### `purchase`
Order transactions
```sql
CREATE TABLE `purchase` (
  `id` int NOT NULL AUTO_INCREMENT,
  `userid` varchar(255) NOT NULL,
  `transactionId` varchar(255) NOT NULL,
  `purchasedAt` bigint NOT NULL,
  `sellerRewardPercent` int DEFAULT '0',
  `sellerDiscountPercent` int DEFAULT '0',
  `adminDiscountPercent` int DEFAULT '0',
  `adminRewardPercent` int DEFAULT '0',
  `totalRewardAmount` int DEFAULT '0',
  `totalDiscountAmount` int DEFAULT '0',
  `totalAmount` int DEFAULT '0',
  `netPayable` int DEFAULT '0',
  `paidAmount` int DEFAULT '0',
  `paymentType` varchar(255) NOT NULL DEFAULT 'COD',
  `paymentStatus` varchar(255) NOT NULL DEFAULT 'Unpaid',
  `orderStatus` varchar(255) NOT NULL DEFAULT 'Initiated',
  `buyerAddress` varchar(500) DEFAULT NULL,
  `buyerPhone` bigint DEFAULT NULL,
  `buyerName` varchar(255) DEFAULT NULL,
  `buyerEmail` varchar(255) DEFAULT NULL,
  `paymentTransactionId` varchar(255) DEFAULT NULL,
  `couponId` int DEFAULT NULL,
  `userGroupId` int DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

#### `purchase_history`
Detailed purchase records
```sql
CREATE TABLE `purchase_history` (
  `id` int NOT NULL AUTO_INCREMENT,
  `purchaseId` int DEFAULT NULL,
  `purchaseQuantity` int DEFAULT '1',
  `userid` varchar(255) NOT NULL,
  `purchasedAt` bigint NOT NULL,
  `productPrice` int NOT NULL,
  `lotBatch` varchar(255) DEFAULT NULL,
  `stockpointId` int DEFAULT NULL,
  `expiryDate` bigint DEFAULT NULL,
  `totalAmount` int NOT NULL DEFAULT '0',
  `orderStatus` varchar(255) NOT NULL DEFAULT 'Initiated',
  `initiatedOn` bigint DEFAULT NULL,
  `processingOn` bigint DEFAULT NULL,
  `packedOn` bigint DEFAULT NULL,
  `shippedOn` bigint DEFAULT NULL,
  `outForDeliveryOn` bigint DEFAULT NULL,
  `deliveredOn` bigint DEFAULT NULL,
  `courierName` varchar(255) DEFAULT NULL,
  `courierTrackingCode` varchar(255) DEFAULT NULL,
  `sellerName` varchar(255) DEFAULT NULL,
  `categoryId` int DEFAULT NULL,
  `subcategoryId` int DEFAULT NULL,
  `sellerProductName` varchar(255) DEFAULT NULL,
  `productRecordsId` int DEFAULT NULL,
  `sellerId` int DEFAULT NULL,
  `productGroupId` int DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

#### `payment`
Payment processing
```sql
CREATE TABLE `payment` (
  `id` int NOT NULL AUTO_INCREMENT,
  `paymentId` varchar(255) NOT NULL,
  `createdAt` bigint DEFAULT NULL,
  `userId` varchar(255) NOT NULL,
  `amount` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
);
```

### 4. Shopping Experience

#### `cart`
Shopping cart
```sql
CREATE TABLE `cart` (
  `id` int NOT NULL AUTO_INCREMENT,
  `userid` varchar(255) NOT NULL,
  `modifiedAt` bigint NOT NULL,
  `purpose` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
);
```

#### `cart_details`
Cart items
```sql
CREATE TABLE `cart_details` (
  `id` int NOT NULL AUTO_INCREMENT,
  `quantity` int NOT NULL,
  `cartId` int DEFAULT NULL,
  `productRecordsId` int DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

#### `wishlist`
User wishlists
```sql
CREATE TABLE `wishlist` (
  `id` int NOT NULL AUTO_INCREMENT,
  `wishlistId` varchar(255) NOT NULL,
  `wishlistName` varchar(255) DEFAULT NULL,
  `wishlistDescription` varchar(255) DEFAULT NULL,
  `userid` varchar(255) NOT NULL,
  `modifiedAt` bigint NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `IDX_22652c49f2672c0fa2fb52bbe8` (`wishlistId`)
);
```

#### `wishlist_detail`
Wishlist items
```sql
CREATE TABLE `wishlist_detail` (
  `id` int NOT NULL AUTO_INCREMENT,
  `wishlistId` int DEFAULT NULL,
  `productRecordsId` int DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

### 5. Reviews & Ratings

#### `comment`
Product reviews and ratings
```sql
CREATE TABLE `comment` (
  `id` int NOT NULL AUTO_INCREMENT,
  `productRating` varchar(255) DEFAULT NULL,
  `productDeliveryRating` varchar(255) DEFAULT NULL,
  `productQualityRating` varchar(255) DEFAULT NULL,
  `productPackagingRating` varchar(255) DEFAULT NULL,
  `sellerRating` varchar(255) DEFAULT NULL,
  `sellerCommunicationRating` varchar(255) DEFAULT NULL,
  `remarks` text,
  `reviewerName` varchar(255) DEFAULT NULL,
  `reviewerId` int NOT NULL,
  `seller` int NOT NULL,
  `createdAt` bigint DEFAULT NULL,
  `productRecordId` int DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

### 6. Rewards & Promotions

#### `reward_earn`
Reward earning transactions
```sql
CREATE TABLE `reward_earn` (
  `id` int NOT NULL AUTO_INCREMENT,
  `rewardDate` bigint NOT NULL,
  `rewardFor` varchar(255) NOT NULL,
  `rewardValue` int NOT NULL,
  `rewardType` varchar(255) NOT NULL,
  `rewardValidity` varchar(255) NOT NULL,
  `userId` int NOT NULL,
  `purchaseId` int DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

#### `reward_redeem`
Reward redemption transactions
```sql
CREATE TABLE `reward_redeem` (
  `id` int NOT NULL AUTO_INCREMENT,
  `currentRewardPoint` bigint DEFAULT NULL,
  `redeemRewardPoint` bigint DEFAULT NULL,
  `token` varchar(255) DEFAULT NULL,
  `redeemDate` bigint NOT NULL,
  `toAddress` varchar(255) DEFAULT NULL,
  `chainType` varchar(255) DEFAULT NULL,
  `transactionId` int DEFAULT NULL,
  `redeemType` varchar(255) DEFAULT NULL,
  `redeemFor` varchar(255) DEFAULT NULL,
  `userId` int DEFAULT NULL,
  `purchaseId` int DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

#### `coupon`
Coupon management
```sql
CREATE TABLE `coupon` (
  `id` int NOT NULL AUTO_INCREMENT,
  `couponCode` varchar(255) NOT NULL,
  `expiryDate` bigint NOT NULL,
  `applicableFor` enum('USER','USERGROUP','PRODUCTGROUP','Temporary') NOT NULL DEFAULT 'USER',
  `applicableRefId` int NOT NULL,
  `couponType` enum('AMOUNT','PERCENT') NOT NULL DEFAULT 'AMOUNT',
  `couponValue` int NOT NULL,
  `limit` int NOT NULL,
  `usedTimes` int NOT NULL,
  PRIMARY KEY (`id`)
);
```

#### `coupon_redeem`
Coupon usage tracking
```sql
CREATE TABLE `coupon_redeem` (
  `id` int NOT NULL AUTO_INCREMENT,
  `redeemDate` bigint NOT NULL,
  `redeemType` varchar(255) DEFAULT NULL,
  `redeemFor` varchar(255) DEFAULT NULL,
  `couponId` int NOT NULL,
  `couponValue` int NOT NULL,
  `userId` int DEFAULT NULL,
  `purchaseId` int DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

### 7. Marketing & Campaigns

#### `campaign`
Marketing campaigns
```sql
CREATE TABLE `campaign` (
  `id` int NOT NULL AUTO_INCREMENT,
  `campaignName` varchar(255) NOT NULL,
  `startDate` bigint NOT NULL,
  `endDate` bigint NOT NULL,
  `campaignType` varchar(255) NOT NULL,
  `campaignStatus` varchar(255) NOT NULL,
  `expectedRevenue` int NOT NULL,
  `expectedResponse` int NOT NULL,
  `budgetCost` int NOT NULL,
  `actualCost` int NOT NULL,
  `campaignDescription` varchar(255) NOT NULL,
  `targetType` varchar(255) NOT NULL,
  `targetAgeGroup` varchar(255) NOT NULL,
  `targetGender` varchar(255) NOT NULL,
  `clipartId` int NOT NULL,
  `mailMessageHeader` varchar(255) NOT NULL,
  `mailMessageBody` varchar(255) NOT NULL,
  `mailMessageFooter` varchar(255) NOT NULL,
  `targetLocation` varchar(255) NOT NULL,
  `sellerId` int DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

#### `campaign_users`
Campaign target users
```sql
CREATE TABLE `campaign_users` (
  `id` int NOT NULL AUTO_INCREMENT,
  `userId` int NOT NULL,
  `campaignId` int NOT NULL,
  PRIMARY KEY (`id`)
);
```

#### `advertisement`
Advertisement management
```sql
CREATE TABLE `advertisement` (
  `id` int NOT NULL AUTO_INCREMENT,
  `heading` varchar(255) NOT NULL,
  `title` varchar(255) NOT NULL,
  `description` varchar(255) NOT NULL,
  `price` double NOT NULL DEFAULT '0',
  `usedFor` enum('FourBox','ThreeBox','Carousel') NOT NULL DEFAULT 'Carousel',
  `imageUrl` varchar(255) NOT NULL,
  `categoryFor` enum('RealEstate','Grocery','Electronics','Fashion','Jewellery','Interior','Pharmacy','Temporary') NOT NULL DEFAULT 'Fashion',
  `linkUrl` varchar(255) NOT NULL DEFAULT '#',
  `bgColor` varchar(255) NOT NULL DEFAULT 'bg-blue-100',
  PRIMARY KEY (`id`)
);
```

#### `clipart`
Marketing assets
```sql
CREATE TABLE `clipart` (
  `id` int NOT NULL AUTO_INCREMENT,
  `clipartName` varchar(255) NOT NULL,
  `image` varchar(255) NOT NULL,
  `clipartFor` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
);
```

### 8. CRM & Lead Management

#### `leads`
Lead management
```sql
CREATE TABLE `leads` (
  `id` int NOT NULL AUTO_INCREMENT,
  `status` varchar(255) NOT NULL DEFAULT 'Created',
  `leadDate` bigint NOT NULL,
  `leadPrice` int NOT NULL DEFAULT '0',
  `leadDescription` varchar(255) NOT NULL,
  `location` varchar(255) NOT NULL,
  `expectedOrderDate` bigint NOT NULL,
  `expectedDeliveryDate` bigint NOT NULL,
  `leadSource` varchar(255) NOT NULL,
  `currentSetup` varchar(255) NOT NULL,
  `leadType` varchar(255) NOT NULL,
  `fundType` varchar(255) NOT NULL,
  `leadNo` varchar(255) NOT NULL,
  `referenceCode` varchar(255) NOT NULL,
  `zone` varchar(255) NOT NULL,
  `baseCurrencyCode` varchar(255) NOT NULL,
  `ownerId` varchar(255) DEFAULT NULL,
  `customerId` int DEFAULT NULL,
  `customerName` varchar(255) DEFAULT NULL,
  `customerPhone` varchar(255) DEFAULT NULL,
  `customerEmail` varchar(255) DEFAULT NULL,
  `customerCompanyName` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

#### `lead_status`
Lead status tracking
```sql
CREATE TABLE `lead_status` (
  `id` int NOT NULL AUTO_INCREMENT,
  `leadStatus` varchar(255) NOT NULL,
  `givenBy` varchar(255) NOT NULL,
  `givenByType` varchar(255) NOT NULL,
  `nextFollowupDate` bigint NOT NULL,
  `givenOn` bigint NOT NULL,
  `remarks` varchar(255) NOT NULL,
  `leadType` varchar(255) NOT NULL,
  `priority` varchar(255) NOT NULL,
  `reminderType` varchar(255) NOT NULL,
  `leadId` int NOT NULL,
  PRIMARY KEY (`id`)
);
```

#### `lead_item`
Lead items
```sql
CREATE TABLE `lead_item` (
  `id` int NOT NULL AUTO_INCREMENT,
  `productQty` int NOT NULL,
  `productPrice` int NOT NULL,
  `taxName1` varchar(255) DEFAULT NULL,
  `taxPercentage1` int DEFAULT '0',
  `remarks` varchar(255) DEFAULT NULL,
  `totalPrice` int NOT NULL DEFAULT '0',
  `leadId` int NOT NULL,
  `productId` int NOT NULL,
  PRIMARY KEY (`id`)
);
```

### 9. Inventory Management

#### `stock_point`
Warehouse/stock locations
```sql
CREATE TABLE `stock_point` (
  `id` int NOT NULL AUTO_INCREMENT,
  `parentId` int DEFAULT NULL,
  `stockPointAddress` varchar(255) NOT NULL,
  `stockPointName` varchar(255) NOT NULL,
  `stockPointType` varchar(255) NOT NULL,
  `sellerId` int DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

### 10. System Configuration

#### `filter`
Product filtering options
```sql
CREATE TABLE `filter` (
  `id` int NOT NULL AUTO_INCREMENT,
  `filterName` varchar(255) NOT NULL,
  `subcategoryId` int DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

#### `filter_data`
Filter option values
```sql
CREATE TABLE `filter_data` (
  `id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(255) NOT NULL,
  `filterFieldId` int DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

#### `lookup`
System lookup values
```sql
CREATE TABLE `lookup` (
  `id` int NOT NULL AUTO_INCREMENT,
  `lookupType` varchar(255) DEFAULT NULL,
  `lookupName` varchar(255) DEFAULT NULL,
  `lookupFor` varchar(255) DEFAULT NULL,
  `lookupNameView` varchar(255) DEFAULT NULL,
  `lookupSort` int DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

#### `term`
Terms and conditions
```sql
CREATE TABLE `term` (
  `id` int NOT NULL AUTO_INCREMENT,
  `termsName` varchar(255) NOT NULL,
  `userId` int DEFAULT NULL,
  PRIMARY KEY (`id`)
);
```

#### `terms_detail`
Terms details
```sql
CREATE TABLE `terms_detail` (
  `id` int NOT NULL AUTO_INCREMENT,
  `termsNo` int NOT NULL,
  `termsId` int NOT NULL,
  `termsDetails` varchar(255) NOT NULL,
  `termsType` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
);
```

## Key Relationships

### Foreign Key Constraints
- `users.id` ↔ `profile.id` (One-to-One)
- `profile.userGroupId` → `user_group.id`
- `address_book.profileId` → `profile.id`
- `product.categoryId` → `category.id`
- `product.subcategoryId` → `subcategory.id`
- `product.productGroupId` → `product_group.id`
- `product_records.productId` → `product.id`
- `product_records.sellerDetailsId` → `profile.id`
- `product_stock.productRecordsId` → `product_records.id`
- `product_stock.stockPointId` → `stock_point.id`
- `purchase.couponId` → `coupon.id`
- `purchase.userGroupId` → `user_group.id`
- `purchase_history.purchaseId` → `purchase.id`
- `purchase_history.productRecordsId` → `product_records.id`
- `cart_details.cartId` → `cart.id`
- `cart_details.productRecordsId` → `product_records.id`
- `wishlist_detail.wishlistId` → `wishlist.id`
- `wishlist_detail.productRecordsId` → `product_records.id`

## Business Logic Features

### Multi-vendor Support
- Sellers can list products with their own pricing
- Separate reward and discount percentages for sellers and admin
- Stock management per seller location

### Reward System
- Points earned on purchases
- Blockchain wallet integration for reward redemption
- Multiple chain support (SOL, ETH, BNB, ICT, Polygon)

### Flexible Pricing
- Product-level and seller-level discounts
- User group-based pricing
- Coupon system with various applicability rules

### Order Management
- Comprehensive order status tracking
- Courier integration with tracking codes
- Multi-item orders with individual item status

### CRM Integration
- Lead management with status tracking
- Customer relationship management
- Sales pipeline tracking

### Marketing Tools
- Campaign management with targeting
- Advertisement placement system
- Coupon and promotion management
