# DynamoDB Multi-Table Schema - Entity Relationship Diagram (Clean)

```mermaid
graph TB
    %% Core Tables
    subgraph "Users Table"
        U1["PK: userid<br/>SK: PROFILE<br/>• User Profile<br/>• Authentication<br/>• Reward Points"]
        U2["PK: userid<br/>SK: ADDRESS#id<br/>• User Addresses<br/>• Contact Info"]
        U3["PK: userid<br/>SK: SESSION#token<br/>• User Sessions<br/>• TTL: expiresAt"]
        U4["PK: userid<br/>SK: OTP#type<br/>• OTP Verification<br/>• TTL: 300s"]
    end
    
    subgraph "Sellers Table"
        SE1["PK: sellerId<br/>SK: INFO<br/>• Business Profile<br/>• Default Terms<br/>• KYC Details"]
        SE2["PK: sellerId<br/>SK: DOCUMENT#type<br/>• Business Documents<br/>• Verification Status"]
        SE3["PK: sellerId<br/>SK: BANK#id<br/>• Bank Details<br/>• Payment Setup"]
    end
    
    subgraph "Products Table"
        P1["PK: productId<br/>SK: INFO<br/>• Product Details<br/>• Filterable Attributes<br/>• Category, Brand"]
        P2["PK: productId<br/>SK: SELLER#sellerId<br/>• Seller Listings<br/>• Price, Stock<br/>• Embedded Terms"]
        P3["PK: productId<br/>SK: IMAGE#id<br/>• Product Images<br/>• Media Files"]
        P4["PK: productId<br/>SK: SPEC#name<br/>• Specifications<br/>• Technical Details"]
    end
    
    subgraph "Orders Table"
        O1["PK: userid<br/>SK: ORDER#date#txnId<br/>• Order Header<br/>• Payment Info<br/>• Status Tracking"]
    end
    
    subgraph "OrderItems Table"
        OI1["PK: transactionId<br/>SK: ITEM#productId#sellerId<br/>• Order Line Items<br/>• Individual Tracking<br/>• Courier Details"]
    end
    
    subgraph "Cart Table"
        C1["PK: userid<br/>SK: ITEM#productId#sellerId<br/>• Cart Items<br/>• Quantity, Price<br/>• Availability"]
    end
    
    subgraph "Wishlist Table"
        W1["PK: userid<br/>SK: WISHLIST#wishlistId<br/>• Wishlist Header<br/>• Name, Description"]
        W2["PK: userid<br/>SK: ITEM#wishlistId#productId<br/>• Wishlist Items<br/>• Priority, Notes"]
    end
    
    subgraph "Reviews Table"
        R1["PK: productId<br/>SK: REVIEW#date#reviewerId<br/>• Product Reviews<br/>• Ratings, Comments<br/>• Verified Purchase"]
    end
    
    subgraph "Rewards Table"
        RW1["PK: userid<br/>SK: REWARD#date#type#id<br/>• Reward Transactions<br/>• Earn/Redeem<br/>• Blockchain Integration"]
    end
    
    subgraph "Categories Table"
        CT1["PK: categoryId<br/>SK: INFO<br/>• Category Details<br/>• Images, Metadata"]
        CT2["PK: categoryId<br/>SK: SUBCATEGORY#id<br/>• Subcategories<br/>• Hierarchy"]
    end
    
    subgraph "Inventory Table"
        I1["PK: productId<br/>SK: STOCK#lotBatch#sellerId<br/>• Stock Details<br/>• Lot Tracking<br/>• Expiry Dates"]
    end
    
    subgraph "UserGroups Table"
        UG1["PK: userGroupId<br/>SK: INFO<br/>• Group Details<br/>• Discount Rules<br/>• Reward Rates"]
    end
    
    %% GSI Indexes
    subgraph "Global Secondary Indexes"
        GSI1["Users GSI1<br/>Email Lookup<br/>PK: email, SK: userid"]
        GSI2["Sellers GSI1<br/>Status Lookup<br/>PK: STATUS#status, SK: sellerId"]
        GSI3["Sellers GSI2<br/>Category Browse<br/>PK: CATEGORY#type, SK: RATING#rating"]
        GSI4["Products GSI1<br/>Category Browse<br/>PK: CATEGORY#id, SK: PRICE#amount"]
        GSI5["Products GSI2<br/>Seller Products<br/>PK: sellerId, SK: productId"]
        GSI6["Orders GSI1<br/>Transaction Lookup<br/>PK: transactionId, SK: userid"]
        GSI7["Orders GSI2<br/>Status Tracking<br/>PK: orderStatus, SK: date"]
        GSI8["Reviews GSI1<br/>Seller Reviews<br/>PK: sellerId, SK: date"]
        GSI9["Inventory GSI1<br/>Seller Stock<br/>PK: sellerId, SK: productId"]
    end
    
    %% LSI Indexes
    subgraph "Local Secondary Indexes"
        LSI1["Orders LSI1<br/>User Orders by Status<br/>PK: userid, SK: STATUS#status"]
        LSI2["Orders LSI2<br/>User Orders by Amount<br/>PK: userid, SK: AMOUNT#amount"]
    end
    
    %% Access Patterns
    subgraph "Core Access Patterns"
        AP1["🔐 User Login<br/>GSI1: email → userid"]
        AP2["🏪 Seller Dashboard<br/>GSI2: status → active sellers"]
        AP3["🛍️ Product Catalog<br/>GSI4: category → products"]
        AP4["👤 Seller Products<br/>GSI5: sellerId → products"]
        AP5["📦 Order Tracking<br/>GSI6: transactionId → order"]
        AP6["📊 Order Management<br/>GSI7: status → orders"]
        AP7["⭐ Product Reviews<br/>PK: productId → reviews"]
        AP8["🛒 Shopping Cart<br/>PK: userid → cart items"]
        AP9["💝 User Wishlist<br/>PK: userid → wishlists"]
        AP10["💰 Reward History<br/>PK: userid → rewards"]
        AP11["📦 Inventory Management<br/>GSI9: sellerId → stock"]
        AP12["📋 Seller KYC<br/>PK: sellerId → documents"]
    end
    
    %% Data Flow Relationships
    U1 -.-> O1
    U1 -.-> C1
    U1 -.-> W1
    U1 -.-> RW1
    
    SE1 -.-> P2
    SE1 -.-> OI1
    SE1 -.-> I1
    
    P1 -.-> P2
    P1 -.-> C1
    P1 -.-> W2
    P1 -.-> R1
    P1 -.-> I1
    
    O1 -.-> OI1
    O1 -.-> RW1
    
    P2 -.-> OI1
    P2 -.-> R1
    
    CT1 -.-> CT2
    CT1 -.-> P1
    
    UG1 -.-> U1
    UG1 -.-> O1
    
    %% GSI Connections
    U1 -.-> GSI1
    SE1 -.-> GSI2
    SE1 -.-> GSI3
    P2 -.-> GSI4
    P2 -.-> GSI5
    O1 -.-> GSI6
    O1 -.-> GSI7
    R1 -.-> GSI8
    I1 -.-> GSI9
    
    %% LSI Connections
    O1 -.-> LSI1
    O1 -.-> LSI2
    
    %% TTL Configuration
    subgraph "TTL Configuration"
        TTL1["🕐 User Sessions<br/>Auto-expire based on expiresAt"]
        TTL2["🔐 OTP Codes<br/>Auto-expire after 300 seconds"]
    end
    
    U3 -.-> TTL1
    U4 -.-> TTL2
    
    %% Design Benefits
    subgraph "Clean Architecture Benefits"
        B1["🎯 Minimal Complexity<br/>Only essential tables"]
        B2["📦 Embedded Data<br/>Terms and filters in context"]
        B3["🎨 UI-Driven Filters<br/>Frontend handles filter logic"]
        B4["⚡ Better Performance<br/>Related data together"]
        B5["🔧 Easier Maintenance<br/>Fewer tables to manage"]
        B6["🏪 Complete Seller Management<br/>KYC and business data"]
        B7["🚀 Focused Features<br/>Core e-commerce functionality"]
    end
    
    %% Frontend Integration
    subgraph "Frontend Integration"
        FE1["🎨 Dynamic Filters<br/>Built from product attributes"]
        FE2["📋 Terms Display<br/>Shown from product/seller data"]
        FE3["🔧 Configuration<br/>Hardcoded in UI components"]
        FE4["📊 Analytics<br/>Computed from order data"]
    end
    
    P1 -.-> FE1
    P2 -.-> FE2
    SE1 -.-> FE2
    O1 -.-> FE4
    
    %% Styling
    classDef userTable fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef sellerTable fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    classDef productTable fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef orderTable fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
    classDef systemTable fill:#fce4ec,stroke:#c2185b,stroke-width:2px
    classDef gsiIndex fill:#e0f2f1,stroke:#00695c,stroke-width:2px
    classDef lsiIndex fill:#e8eaf6,stroke:#3f51b5,stroke-width:2px
    classDef accessPattern fill:#f1f8e9,stroke:#558b2f,stroke-width:1px
    classDef benefit fill:#fff8e1,stroke:#ff8f00,stroke-width:1px
    classDef frontend fill:#f3e5f5,stroke:#8e24aa,stroke-width:1px
    classDef ttl fill:#ffebee,stroke:#d32f2f,stroke-width:1px
    
    class U1,U2,U3,U4 userTable
    class SE1,SE2,SE3 sellerTable
    class P1,P2,P3,P4 productTable
    class O1,OI1,C1,W1,W2,R1,RW1 orderTable
    class CT1,CT2,I1,UG1 systemTable
    class GSI1,GSI2,GSI3,GSI4,GSI5,GSI6,GSI7,GSI8,GSI9 gsiIndex
    class LSI1,LSI2 lsiIndex
    class AP1,AP2,AP3,AP4,AP5,AP6,AP7,AP8,AP9,AP10,AP11,AP12 accessPattern
    class B1,B2,B3,B4,B5,B6,B7 benefit
    class FE1,FE2,FE3,FE4 frontend
    class TTL1,TTL2 ttl
```
