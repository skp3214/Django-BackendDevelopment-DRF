## Filtering, Searching and Ordering in DRF

In Django REST Framework (DRF), filtering and searching are key features that allow you to refine and query your API's data more efficiently. They make it easier to implement dynamic filtering and search functionality for users.

### 1. **Filtering in DRF**

DRF provides several ways to filter querysets in your API views. The most commonly used methods are:
- **Manual Filtering**
- **DjangoFilterBackend**
- **Overriding `get_queryset()`**

#### a. **Manual Filtering**
You can manually filter a queryset based on query parameters passed in the request.

##### Example:

```python
from rest_framework import generics
from .models import Product
from .serializers import ProductSerializer

class ProductListView(generics.ListAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer

    def get_queryset(self):
        queryset = super().get_queryset()
        price = self.request.query_params.get('price', None)
        if price:
            queryset = queryset.filter(price__lte=price)  # Filter products by price
        return queryset
```

Here, the API will accept a `price` query parameter and filter products with a price less than or equal to that value. The query would look like:

```
GET /api/products/?price=100
```

#### b. **DjangoFilterBackend**
DRF supports `django-filter`, a more powerful way to filter querysets by using fields defined in models. First, install `django-filter`:

```bash
pip install django-filter
```

Then, add it to the `REST_FRAMEWORK` settings:

```python
REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': ['django_filters.rest_framework.DjangoFilterBackend']
}
```

You can now use `filterset_fields` in your views to automatically enable filtering based on model fields.

##### Example:

```python
from rest_framework import generics
from django_filters.rest_framework import DjangoFilterBackend
from .models import Product
from .serializers import ProductSerializer

class ProductListView(generics.ListAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [DjangoFilterBackend]
    filterset_fields = ['category', 'price']
```

Now, users can filter products by `category` and `price`:

```
GET /api/products/?category=electronics&price=100
```

#### c. **Overriding `get_queryset()` for Custom Filtering**
You can override the `get_queryset()` method to apply custom filtering logic based on the query parameters.

```python
class ProductListView(generics.ListAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer

    def get_queryset(self):
        queryset = super().get_queryset()
        category = self.request.query_params.get('category')
        price_min = self.request.query_params.get('price_min')
        price_max = self.request.query_params.get('price_max')

        if category:
            queryset = queryset.filter(category=category)
        if price_min and price_max:
            queryset = queryset.filter(price__gte=price_min, price__lte=price_max)

        return queryset
```

### 2. **Searching in DRF**

DRF provides a simple way to enable search functionality in API views using the `SearchFilter` class.

#### a. **Using `SearchFilter`**

To enable search functionality, you must use the `SearchFilter` class in the view's `filter_backends`. You also need to specify the `search_fields` attribute, which defines the model fields that can be searched.

##### Example:

```python
from rest_framework import generics
from rest_framework.filters import SearchFilter
from .models import Product
from .serializers import ProductSerializer

class ProductListView(generics.ListAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [SearchFilter]
    search_fields = ['name', 'description', 'category__name']  # Specify searchable fields
```

Now, you can search for products by name, description, or category:

```
GET /api/products/?search=phone
```

The `search_fields` can also include relationships (e.g., `category__name`) for more advanced search functionality.

#### b. **Customizing Search Behavior**

By default, the `SearchFilter` performs an `icontains` query, which is case-insensitive and searches for a substring match. If you need custom search logic, you can override the `filter_queryset()` method in the view.

### 3. **Combining Filtering and Searching**

You can combine both filtering and searching by adding both `DjangoFilterBackend` and `SearchFilter` to the `filter_backends`.

##### Example:

```python
from rest_framework import generics
from rest_framework.filters import SearchFilter
from django_filters.rest_framework import DjangoFilterBackend
from .models import Product
from .serializers import ProductSerializer

class ProductListView(generics.ListAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [DjangoFilterBackend, SearchFilter]
    filterset_fields = ['category', 'price']
    search_fields = ['name', 'description']
```

Now, users can both filter and search at the same time:

```
GET /api/products/?category=electronics&search=phone
```

### 4. **Ordering Results**

To enable ordering of the results, you can use `OrderingFilter` from DRF.

#### a. **Using `OrderingFilter`**

Add the `OrderingFilter` to your `filter_backends` and specify `ordering_fields` to define which fields can be ordered.

##### Example:

```python
from rest_framework import generics
from rest_framework.filters import SearchFilter, OrderingFilter
from django_filters.rest_framework import DjangoFilterBackend
from .models import Product
from .serializers import ProductSerializer

class ProductListView(generics.ListAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [DjangoFilterBackend, SearchFilter, OrderingFilter]
    filterset_fields = ['category', 'price']
    search_fields = ['name', 'description']
    ordering_fields = ['price', 'name']  # Specify the fields you can order by
```

Users can now order the products by price or name:

```
GET /api/products/?ordering=price
GET /api/products/?ordering=-name  # Descending order
```

### Summary

- **Filtering**: Use `DjangoFilterBackend` to allow users to filter results based on specific fields.
- **Searching**: Use `SearchFilter` to allow full-text search on fields.
- **Ordering**: Use `OrderingFilter` to allow users to order results based on fields.
  
By combining filtering, searching, and ordering, you can offer a highly flexible and user-friendly API experience.