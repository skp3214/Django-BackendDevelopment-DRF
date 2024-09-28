Pagination in Django REST Framework (DRF) is a way to divide large datasets into smaller, manageable chunks. It is especially useful when dealing with large collections of data, allowing users to retrieve data in pages rather than loading everything at once.

### Types of Pagination in DRF

1. **PageNumberPagination**
   - The simplest and most common type of pagination.
   - The client specifies a page number, and the API returns that page of results.

   **Example:**
   ```python
   from rest_framework.pagination import PageNumberPagination

   class CustomPageNumberPagination(PageNumberPagination):
       page_size = 10  # Number of items per page
       page_size_query_param = 'page_size'  # Allows client to modify page size
       max_page_size = 100  # Maximum page size limit
   ```

   **Usage in views:**
   ```python
   from rest_framework import generics
   from myapp.models import Item
   from myapp.serializers import ItemSerializer
   from myapp.pagination import CustomPageNumberPagination

   class ItemListView(generics.ListAPIView):
       queryset = Item.objects.all()
       serializer_class = ItemSerializer
       pagination_class = CustomPageNumberPagination
   ```

   - You can pass `page` and optionally `page_size` as query parameters in the URL to control pagination:
     ```
     /api/items/?page=2&page_size=5
     ```

2. **LimitOffsetPagination**
   - The client specifies an offset (i.e., how many items to skip) and a limit (i.e., how many items to retrieve).
   - Useful when you need more flexibility than page numbers.

   **Example:**
   ```python
   from rest_framework.pagination import LimitOffsetPagination

   class CustomLimitOffsetPagination(LimitOffsetPagination):
       default_limit = 10  # Default number of items to return
       max_limit = 50  # Maximum limit allowed
   ```

   **Usage in views:**
   ```python
   class ItemListView(generics.ListAPIView):
       queryset = Item.objects.all()
       serializer_class = ItemSerializer
       pagination_class = CustomLimitOffsetPagination
   ```

   - You can pass `limit` and `offset` as query parameters in the URL:
     ```
     /api/items/?limit=10&offset=20
     ```

3. **CursorPagination**
   - Cursor-based pagination uses a record's position in the database for pagination, which allows for efficient pagination with large datasets.
   - The cursor is encoded and opaque to the client, making it impossible for the user to manually set offsets or page numbers.

   **Example:**
   ```python
   from rest_framework.pagination import CursorPagination

   class CustomCursorPagination(CursorPagination):
       page_size = 10
       ordering = 'created'  # Order by created field (newest first)
   ```

   **Usage in views:**
   ```python
   class ItemListView(generics.ListAPIView):
       queryset = Item.objects.all()
       serializer_class = ItemSerializer
       pagination_class = CustomCursorPagination
   ```

   - You can pass the `cursor` query parameter in the URL to retrieve the next set of results:
     ```
     /api/items/?cursor=some_encoded_value
     ```

### Global Pagination Settings

To apply a default pagination style across all views in your project, you can define pagination settings in the `settings.py` file.

**Example:**
```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10  # Default page size for all paginated views
}
```

This setting ensures that all views use `PageNumberPagination` by default with a page size of 10 items.

### Example API Response with Pagination

When pagination is applied, DRF automatically adds additional metadata to the API response. Here's an example of how paginated responses look:

**PageNumberPagination Response:**
```json
{
    "count": 50,
    "next": "http://api.example.com/items/?page=3",
    "previous": "http://api.example.com/items/?page=1",
    "results": [
        {
            "id": 21,
            "name": "Item 21"
        },
        {
            "id": 22,
            "name": "Item 22"
        }
        // More items
    ]
}
```

### Customizing Pagination

You can fully customize pagination behavior by overriding the `get_paginated_response()` method in your custom pagination class.

**Example of Custom Paginated Response:**
```python
from rest_framework.pagination import PageNumberPagination
from rest_framework.response import Response

class CustomPageNumberPagination(PageNumberPagination):
    page_size = 10

    def get_paginated_response(self, data):
        return Response({
            'total_items': self.page.paginator.count,
            'total_pages': self.page.paginator.num_pages,
            'current_page': self.page.number,
            'next_page': self.get_next_link(),
            'previous_page': self.get_previous_link(),
            'items': data
        })
```

### Conclusion

Pagination is an essential feature in DRF to handle large datasets efficiently. DRF provides built-in pagination classes such as:
- `PageNumberPagination`
- `LimitOffsetPagination`
- `CursorPagination`

These classes can be easily customized and applied globally or per view, giving you flexibility over how you manage paginated data in your API.