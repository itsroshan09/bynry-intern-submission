
Bynry Backend Intern Case Study Submission
Applicant: Roshan Kishor Patil
Date: September 19, 2025

Hello Bynry Team,

Thank you for the opportunity to work on this case study. I found the problem of building an inventory management system really interesting, and I enjoyed thinking through the challenges.

I've structured my response into the three parts as requested and spent about 90 minutes on the task. Below are my solutions and the reasoning behind my decisions.

**Part 1: Code Review & Debugging**
Looking at the initial code for creating a product, a few things immediately stood out to me from a safety and reliability perspective. While it might work in a perfect scenario, it could cause some serious issues in a live production environment.

**My Findings**
**The "Two Commits" Problem:** The biggest risk I see is the use of two separate db.session.commit(). If the first commit to create the Product works, but the second one for Inventory fails for any reason (e.g., a brief database hiccup), we end up with a product in our system that has no inventory. This inconsistent data can be very difficult to track down and fix later.

Trusting the Input Data: The code directly uses data['name'] and other keys. If an API call comes in without one of those keys, the whole application will crash. We should always validate incoming data.

Ignoring Uniqueness: The prompt mentioned that SKUs must be unique, but the code doesn't check if a product with that SKU already exists. This could easily lead to duplicate products in the database, causing chaos for inventory tracking and sales.

No Safety Net (Error Handling): There's no try...except block. Any unexpected error would result in a generic 500 server error, which isn't a great experience for the user of the API.

My Corrected Version
To fix these issues, I've rewritten the function using Django and the Django Rest Framework, as it provides great tools for building robust APIs.

views.py

Python

# I'm using Django Rest Framework as it makes handling API logic cleaner.
from django.db import transaction
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from .models import Product, Inventory, Warehouse

class ProductCreateAPIView(APIView):
    """
    Creates a new product and its initial inventory record in a single, safe transaction.
    """
    def post(self, request, *args, **kwargs):
        data = request.data

        # 1. First, I validate the incoming data to make sure we have everything we need.
        required_fields = ['name', 'sku', 'price', 'warehouse_id', 'initial_quantity']
        if not all(field in data for field in required_fields):
            return Response(
                {"error": "Missing required fields. Please include name, sku, price, warehouse_id, and initial_quantity."},
                status=status.HTTP_400_BAD_REQUEST
            )

        # 2. I check if the product already exists to enforce the unique SKU rule.
        if Product.objects.filter(sku=data['sku']).exists():
            return Response(
                {"error": f"A product with SKU '{data['sku']}' already exists."},
                status=status.HTTP_409_CONFLICT # 409 Conflict is a good status code for this
            )
        
        try:
            # 3. I wrap the database operations in a single atomic transaction.
            # This means either everything succeeds, or everything fails and rolls back. No more inconsistent data!
            with transaction.atomic():
                warehouse = Warehouse.objects.get(id=data['warehouse_id'])

                product = Product.objects.create(
                    name=data['name'],
                    sku=data['sku'],
                    price=data['price'],
                )
                
                Inventory.objects.create(
                    product=product,
                    warehouse=warehouse,
                    quantity=data['initial_quantity']
                )

            # 4. On success, I return a 201 Created status, which is the standard for this kind of request.
            return Response(
                {"message": "Product created successfully", "product_id": product.id},
                status=status.HTTP_201_CREATED
            )

        except Warehouse.DoesNotExist:
            return Response({"error": "That warehouse does not exist."}, status=status.HTTP_400_BAD_REQUEST)
        except Exception as e:
            # A general catch-all for any other unexpected server errors.
            return Response({"error": "Something went wrong on our end."}, status=status.HTTP_500_INTERNAL_SERVER_ERROR)

