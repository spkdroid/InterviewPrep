# InterviewPrep

HTTP verbs (or methods) are actions that clients can request to perform on a resource, and they play a key role in how web applications communicate. Django, a popular Python web framework, provides built-in support for handling HTTP methods, allowing you to define specific views to handle different HTTP verbs.

Here's a breakdown of common HTTP verbs and how they’re typically used in Django:

### 1. **GET** 
   - **Purpose**: Retrieve data from the server.
   - **Django Usage**: This is the default method for most Django views. When you open a URL in the browser, you’re sending a `GET` request.
   - **Example**:
     ```python
     from django.http import HttpResponse
     from django.shortcuts import render
     
     def my_view(request):
         if request.method == "GET":
             return HttpResponse("This is a GET request")
         return HttpResponse(status=405)  # Method not allowed for other requests
     ```

### 2. **POST**
   - **Purpose**: Submit data to the server, usually to create a new resource.
   - **Django Usage**: Commonly used for submitting forms, creating new records, or processing actions with sensitive data.
   - **Example**:
     ```python
     from django.http import HttpResponse
     from django.views.decorators.csrf import csrf_exempt
     
     @csrf_exempt  # Temporarily disabling CSRF for the example; not recommended for production
     def my_view(request):
         if request.method == "POST":
             # Access form data
             data = request.POST.get('data')
             return HttpResponse(f"Received POST request with data: {data}")
         return HttpResponse(status=405)
     ```

### 3. **PUT**
   - **Purpose**: Update an existing resource.
   - **Django Usage**: Less common in standard Django apps but useful in RESTful APIs (e.g., when using Django REST Framework).
   - **Example**:
     ```python
     from django.http import JsonResponse
     import json
     
     def my_view(request):
         if request.method == "PUT":
             data = json.loads(request.body)
             # Process the data to update an existing resource
             return JsonResponse({"message": "Resource updated", "data": data})
         return JsonResponse({"error": "Method not allowed"}, status=405)
     ```

### 4. **DELETE**
   - **Purpose**: Delete an existing resource.
   - **Django Usage**: Often used in RESTful APIs to delete records.
   - **Example**:
     ```python
     from django.http import JsonResponse
     
     def my_view(request):
         if request.method == "DELETE":
             # Process the delete request
             return JsonResponse({"message": "Resource deleted"})
         return JsonResponse({"error": "Method not allowed"}, status=405)
     ```

### 5. **PATCH**
   - **Purpose**: Partially update an existing resource.
   - **Django Usage**: Useful when only some fields need updating.
   - **Example**:
     ```python
     def my_view(request):
         if request.method == "PATCH":
             data = json.loads(request.body)
             # Process partial update
             return JsonResponse({"message": "Resource partially updated", "data": data})
         return JsonResponse({"error": "Method not allowed"}, status=405)
     ```

### Handling HTTP Verbs with Django Views

In Django’s class-based views, you can define methods directly for each HTTP verb by implementing methods like `get`, `post`, `put`, etc.

```python
from django.http import JsonResponse
from django.views import View

class MyView(View):
    def get(self, request):
        return JsonResponse({"message": "This is a GET request"})

    def post(self, request):
        data = request.POST.get("data")
        return JsonResponse({"message": "POST request received", "data": data})
    
    def put(self, request):
        data = json.loads(request.body)
        return JsonResponse({"message": "PUT request received", "data": data})

    def delete(self, request):
        return JsonResponse({"message": "DELETE request received"})
```

### HTTP Verbs in Django REST Framework (DRF)

With Django REST Framework, you can use the `APIView` class or viewsets, where HTTP verbs are even more streamlined:

```python
from rest_framework.views import APIView
from rest_framework.response import Response

class MyAPIView(APIView):
    def get(self, request):
        return Response({"message": "This is a GET request"})

    def post(self, request):
        return Response({"message": "POST request received", "data": request.data})
```

### Summary
Each HTTP verb corresponds to a specific type of action in the HTTP protocol, and Django provides mechanisms to handle them, making it possible to create powerful, RESTful applications.

# Setup

Here’s a practical example where we use Django views to interact with a database. We’ll create a simple API to manage "books" in a library. This API will use various HTTP methods to create, retrieve, update, and delete book records.


1. **Create a Django Project and App**  
   First, create a Django project and app, if you haven’t already:
   ```bash
   django-admin startproject library
   cd library
   python manage.py startapp books
   ```

2. **Define the Book Model**  
   In `books/models.py`, define a simple `Book` model:
   ```python
   from django.db import models

   class Book(models.Model):
       title = models.CharField(max_length=255)
       author = models.CharField(max_length=255)
       published_date = models.DateField()
       isbn = models.CharField(max_length=13, unique=True)
       available = models.BooleanField(default=True)

       def __str__(self):
           return self.title
   ```

3. **Create and Apply Migrations**
   Run the following commands to create and apply the migration for the `Book` model:
   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

4. **Add Books App to INSTALLED_APPS**
   In `library/settings.py`, add `"books"` to the `INSTALLED_APPS` list.

5. **Create URLs for Book Views**
   In `books/urls.py`, set up URL patterns for our views:
   ```python
   from django.urls import path
   from . import views

   urlpatterns = [
       path("books/", views.BookListView.as_view(), name="book_list"),
       path("books/<int:pk>/", views.BookDetailView.as_view(), name="book_detail"),
   ]
   ```

   Then include these URLs in the project’s `urls.py`:
   ```python
   from django.contrib import admin
   from django.urls import path, include

   urlpatterns = [
       path('admin/', admin.site.urls),
       path('api/', include('books.urls')),  # Include book app URLs
   ]
   ```

### Writing Views for HTTP Verbs

Now we’ll write views to handle each HTTP verb.

#### 1. **List and Create Books** (GET and POST)

In `books/views.py`:

```python
from django.http import JsonResponse
from django.views import View
from .models import Book
import json

class BookListView(View):
    def get(self, request):
        books = Book.objects.all().values("id", "title", "author", "published_date", "isbn", "available")
        return JsonResponse(list(books), safe=False)

    def post(self, request):
        data = json.loads(request.body)
        book = Book.objects.create(
            title=data["title"],
            author=data["author"],
            published_date=data["published_date"],
            isbn=data["isbn"],
            available=data.get("available", True),
        )
        return JsonResponse({"id": book.id, "message": "Book created successfully"}, status=201)
```

#### 2. **Retrieve, Update, and Delete a Book** (GET, PUT, and DELETE)

In `books/views.py`:

```python
from django.shortcuts import get_object_or_404

class BookDetailView(View):
    def get(self, request, pk):
        book = get_object_or_404(Book, pk=pk)
        return JsonResponse({
            "id": book.id,
            "title": book.title,
            "author": book.author,
            "published_date": book.published_date,
            "isbn": book.isbn,
            "available": book.available,
        })

    def put(self, request, pk):
        book = get_object_or_404(Book, pk=pk)
        data = json.loads(request.body)
        book.title = data.get("title", book.title)
        book.author = data.get("author", book.author)
        book.published_date = data.get("published_date", book.published_date)
        book.isbn = data.get("isbn", book.isbn)
        book.available = data.get("available", book.available)
        book.save()
        return JsonResponse({"message": "Book updated successfully"})

    def delete(self, request, pk):
        book = get_object_or_404(Book, pk=pk)
        book.delete()
        return JsonResponse({"message": "Book deleted successfully"}, status=204)
```

### Explanation of Each HTTP Verb in This Example

- **GET `/api/books/`** – Fetches all books.
- **POST `/api/books/`** – Creates a new book with data from the request body.
- **GET `/api/books/<id>/`** – Fetches details of a single book based on its ID.
- **PUT `/api/books/<id>/`** – Updates a book’s information with the data provided.
- **DELETE `/api/books/<id>/`** – Deletes a specific book based on its ID.

### Testing the API

You can use a tool like **Postman** or **curl** to test each endpoint:

- **List Books**: `GET http://localhost:8000/api/books/`
- **Create Book**: `POST http://localhost:8000/api/books/` with JSON body:
  ```json
  {
      "title": "The Great Gatsby",
      "author": "F. Scott Fitzgerald",
      "published_date": "1925-04-10",
      "isbn": "9780743273565",
      "available": true
  }
  ```
- **Retrieve Book**: `GET http://localhost:8000/api/books/<id>/`
- **Update Book**: `PUT http://localhost:8000/api/books/<id>/` with JSON body:
  ```json
  {
      "title": "The Great Gatsby (Updated Edition)"
  }
  ```
- **Delete Book**: `DELETE http://localhost:8000/api/books/<id>/`

### Summary

This example demonstrates how to use Django’s class-based views to interact with a database for basic CRUD operations using HTTP verbs. It’s a practical pattern for building RESTful APIs in Django, especially when combined with Django REST Framework for more extensive functionality.
