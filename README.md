**Zoho CRM — Quote to Sales Order: Selling Price Override**
A Zoho Deluge automation that overrides the default Quote-to-Sales Order conversion behaviour by substituting the Selling Price (marked-up price) as the List Price in the Sales Order line items — specifically for the Technical - UAE layout.
**Problem**
When converting a Quote to a Sales Order in Zoho CRM using the default button, the List Price from the Quote is copied as-is into the SO line items.
However, in some quotes a Markup % is applied on top of the List Price to calculate a Selling Price. In these cases, the SO should reflect the Selling Price — not the original List Price.
Selling Price = List Price + (List Price × Markup% / 100)

**Solution**
A custom Deluge function runs after the Quote-to-SO conversion and:

Fetches the Quote using the passed quoteid
Reads the Layout to determine if the override should apply
Fetches existing SO line items and marks them all for deletion
Rebuilds all line items fresh from the Quote — using Selling_Price as List_Price where applicable
Sends a single PUT request to the Zoho CRM v8 API


**Why delete and re-insert?**
Zoho CRM does not allow updating List_Price on line items created during Quote-to-SO conversion. Deleting and re-inserting as fresh items (without IDs) is the only reliable approach.
**Price Logic**
Price Logic
Layout                   Selling Price present?          SO List Price used 
Technical - UAE          Yes (non-zero)                  Selling_Price from Quote
Technical - UAE          No / zero                       List_Price from Quote (fallback)
Any other layout            —                            List_Price from Quote (default)

Fields Used
Quote Line Items (Quoted_Items)
API Field                  Description
Product_Name.id          Product / Service record ID
Description              Line item display labe
lList_Price              Base price before markup
Selling_Price            Marked-up price
Markup_Percent           Markup percentage
Quantity                 Quantity ordered
Discount                 Discount value
Layout.display_label     Layout name (e.g. Technical - UAE)


Sales Order Line Items (Ordered_Items)
API Field                  Description
Product_Name              Product / Service record ID
List_Price                Set to Selling_Price for Technical - UAE
Quantity                  Quantity
Description               Line item label
Discount                  Discount (included only if not null)
_delete                   Set to "true" to remove existing item

