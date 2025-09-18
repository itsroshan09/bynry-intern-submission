
**Part 2: Database Design**
Here’s my proposed database schema for the StockFlow platform. I've written it as Django models because it’s a very clear way to define the structure, data types, and relationships.

**Proposed Schema (models.py)**
**Python**

from django.db import models

# I started with the core entities: Company, Warehouse, Supplier, and Product.

class Company(models.Model):
    name = models.CharField(max_length=255, unique=True)
    created_at = models.DateTimeField(auto_now_add=True)

class Warehouse(models.Model):
    company = models.ForeignKey(Company, on_delete=models.CASCADE, related_name='warehouses')
    name = models.CharField(max_length=255)
    location = models.CharField(max_length=500)

class Supplier(models.Model):
    name = models.CharField(max_length=255)
    contact_email = models.EmailField(unique=True)
    phone_number = models.CharField(max_length=20, blank=True)

class Product(models.Model):
    company = models.ForeignKey(Company, on_delete=models.CASCADE, related_name='products')
    name = models.CharField(max_length=255)
    sku = models.CharField(max_length=100, unique=True, db_index=True) # Indexed for fast lookups
    price = models.DecimalField(max_digits=10, decimal_places=2)
    suppliers = models.ManyToManyField(Supplier, related_name='products')
    low_stock_threshold = models.PositiveIntegerField(default=10)
    
    # A simple way to handle bundles is to have a product link to other products.
    bundle_components = models.ManyToManyField('self', symmetrical=False, blank=True)

# This table connects Products and Warehouses to store the actual quantity on hand.
class Inventory(models.Model):
    product = models.ForeignKey(Product, on_delete=models.CASCADE)
    warehouse = models.ForeignKey(Warehouse, on_delete=models.CASCADE)
    quantity = models.PositiveIntegerField(default=0)
    last_updated = models.DateTimeField(auto_now=True)

    class Meta:
        # Ensures we can't have duplicate inventory entries for the same product in the same warehouse.
        unique_together = ('product', 'warehouse')

# A table to track sales, which is needed for the "recent sales activity" rule.
class Sale(models.Model):
    inventory = models.ForeignKey(Inventory, on_delete=models.PROTECT)
    quantity_sold = models.PositiveIntegerField()
    sale_date = models.DateTimeField(auto_now_add=True)
Questions I'd Ask the Product Team
The requirements were a great starting point, but to build a truly robust system, I'd need to ask a few clarifying questions:

About Bundles: When a bundle is sold, should we automatically decrease the stock of all its component products? How is the "available quantity" of a bundle determined?

About Thresholds: Does the low-stock threshold ever change based on seasonality or sales trends, or is it always a fixed number per product?

About Suppliers: If a product can have multiple suppliers, is there a "preferred" supplier we should list on the alert for re-ordering?

About "Recent Sales": How do we define "recent"? Is it the last 7 days, 30 days, or something else?

Why I Designed It This Way
Single Source of Truth: I separated everything into its own table (e.g., Supplier, Product) to avoid storing the same information in multiple places. This keeps the data clean and reliable.

Performance: I added a db_index=True to the sku field on the Product model. Since people will be searching for products by SKU all the time, this will make those lookups much faster.

Data Integrity: I used a unique_together constraint on the Inventory model. This is a database rule that stops anyone from accidentally creating two different stock counts for the same product in the same warehouse.
