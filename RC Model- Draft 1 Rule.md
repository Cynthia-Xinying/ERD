**Attribute of Entity**  
**1.Customer**  
PK: Customer\_id,   
Customer\_Address  
Customer\_Email  
Customer\_Revenue  
Date\_Added
## Cleaned / Normalized Entities and Attributes (RC Model)

Notes: I made these modeling decisions while normalizing:
- Use singular entity names (Customer, Product, Invoice, etc.).
- Surrogate integer primary keys (e.g., Customer_ID) for every main entity.
- Keep line items and PO lines as their own entities with a surrogate PK and FKs to parent + product. Enforce uniqueness of (Invoice_ID, Line_Number) or (PO_ID, PO_Line_Number) where needed.
- Removed Product_ID from `Invoice` (products belong on `Line_Item`).
- Product name is an attribute, not a foreign key.


### Customer
- PK: Customer_ID (INT)
- Customer_Name
- Customer_Email
- Date_Added
- Customer_Default_Billing_Address_ID (FK -> Address.Address_ID)  -- optional
- Customer_Default_Shipping_Address_ID (FK -> Address.Address_ID) -- optional
- Customer_Revenue (DECIMAL)  -- recommended: derived (compute) rather than source of truth


### Address
- PK: Address_ID (INT)
- Customer_ID (FK -> Customer.Customer_ID)  -- if addresses are owned by customers
- Address_Type (ENUM: billing, shipping, other)
- Street
- City
- State_Province
- Postal_Code
- Country

Reason: customers often have multiple addresses and invoices need a shipping/billing address.


### Manufacturer
- PK: Manufacturer_ID (INT)
- Manufacturer_Name
- Country
- Active_Flag (BOOLEAN)


### Product
- PK: Product_ID (INT)
- Manufacturer_ID (FK -> Manufacturer.Manufacturer_ID)
- SKU (business key, string)
- Product_Name
- Product_Model
- Product_Description
- Unit_Price (DECIMAL)
- Product_Status (ENUM: active, discontinued, etc.)
- Min_Quantity (INT)
- Note: Quantity_On_Hand should be modeled per location (see Warehouse / InventoryTransaction)


### Purchase_Order
- PK: PO_ID (INT)
- Manufacturer_ID (FK -> Manufacturer.Manufacturer_ID)
- PO_Date
- PO_Total (DECIMAL)  -- derived from PO_Line totals; can be stored for reporting but must be kept consistent
- Order_Status (ENUM)


### PO_Line
- PK: PO_Line_ID (INT)
- PO_ID (FK -> Purchase_Order.PO_ID)
- Line_Number (INT)  -- for ordering lines within a PO
- Product_ID (FK -> Product.Product_ID)
- Quantity_Ordered (INT)
- Unit_Cost (DECIMAL)
- Line_Total (DECIMAL)  -- Quantity_Ordered * Unit_Cost (derived)

Notes: Use a surrogate PK and a unique constraint on (PO_ID, Line_Number) or (PO_ID, Product_ID) based on business rules. Do not duplicate a composite PK unless you need it.


### Invoice
- PK: Invoice_ID (INT)
- Customer_ID (FK -> Customer.Customer_ID)
- Invoice_Date
- Billing_Address_ID (FK -> Address.Address_ID)
- Shipping_Address_ID (FK -> Address.Address_ID)
- Shipping_Charge (DECIMAL)
- Invoice_Total (DECIMAL)  -- derived (sum of Line_Total + shipping +/- taxes)
- Order_ID (FK -> Sales_Order.Sales_Order_ID)  -- optional; include only if you model Sales Orders

Removed Product_ID from Invoice because invoices can have many products via `Line_Item`.


### Line_Item
- PK: Line_Item_ID (INT)
- Invoice_ID (FK -> Invoice.Invoice_ID)
- Line_Number (INT)
- Product_ID (FK -> Product.Product_ID)
- Quantity (INT)
- Unit_Price (DECIMAL)  -- store the price used at invoice time for historical accuracy
- Line_Total (DECIMAL)  -- Quantity * Unit_Price (derived)

Constraint: Each invoice must have 1..N line items. Enforce (Invoice_ID, Line_Number) unique.


### Payment (suggested)
- PK: Payment_ID (INT)
- Invoice_ID (FK -> Invoice.Invoice_ID)
- Payment_Date
- Payment_Method (ENUM)
- Amount (DECIMAL)
- Payment_Status

Reason: Keeps invoice lifecycle and payment records cleanly separated.


### Shipment (suggested)
- PK: Shipment_ID (INT)
- Invoice_ID (FK -> Invoice.Invoice_ID)
- Shipper
- Tracking_Number
- Shipped_Date
- Delivered_Date
- Shipment_Status


### Warehouse (suggested)
- PK: Warehouse_ID (INT)
- Warehouse_Name
- Location


### InventoryTransaction (suggested)
- PK: InventoryTxn_ID (INT)
- Product_ID (FK -> Product.Product_ID)
- Warehouse_ID (FK -> Warehouse.Warehouse_ID)
- Txn_Type (ENUM: receipt, sale, adjustment, transfer)
- Quantity_Change (INT)
- Txn_Date
- Reference_ID (e.g., Invoice_ID, PO_ID)

Reason: Use transactions to compute Quantity_On_Hand per warehouse instead of storing a single aggregated attribute on Product.


### Category (optional)
- PK: Category_ID
- Category_Name
- Product — Category relationship: Product.Category_ID (FK)


## Key Relationships and Cardinalities (summary)

1) Customer 1 — 0..* Invoice
- FK: Invoice.Customer_ID

2) Invoice 1 — 1..* Line_Item
- FK: Line_Item.Invoice_ID

3) Product 1 — 0..* Line_Item
- FK: Line_Item.Product_ID

4) Manufacturer 1 — 0..* Product
- FK: Product.Manufacturer_ID

5) Manufacturer 1 — 0..* Purchase_Order
- FK: Purchase_Order.Manufacturer_ID

6) Purchase_Order 1 — 1..* PO_Line
- FK: PO_Line.PO_ID

7) Product 1 — 0..* PO_Line
- FK: PO_Line.Product_ID


## Modeling decisions & assumptions
- Surrogate keys used for simplicity and stable PKs.
- Line numbers chosen for ordering lines within invoices/POs and to allow a unique business constraint (Invoice_ID, Line_Number).
- Price at time of transaction is stored on Line_Item (Unit_Price) to keep historical invoices accurate even if product price changes later.
- Totals (Line_Total, Invoice_Total, PO_Total, Customer_Revenue) are derived and ideally calculated at query time or maintained by the application/service to avoid inconsistencies.
- Address is a first-class entity to support multiple addresses per customer and to link invoices to specific shipping/billing addresses.


## Next steps / options
1. If you want, I can:
	- produce a cleaned ERD diagram (Crowsfoot) from this normalized model; or
	- create a SQL DDL script (Postgres/ MySQL) for these tables; or
	- add constraints/index suggestions (unique keys, FK actions).
2. Answer these quick questions to finalize choices:
	- Do you track Sales Orders separately from Invoices? (Yes/No)
	- Do you require per-warehouse inventory? (Yes/No)
	- Do you track returns/credit notes? (Yes/No)

Please tell me which next action you'd like and answer the three questions above if you can.

