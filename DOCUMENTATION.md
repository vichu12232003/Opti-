# File Uploader and Tracker - Documentation

## Project Overview

This is a Full Stack web application built with FastAPI and SQLAlchemy that allows users to upload files, store them locally, and track their metadata in a SQLite database. The application consists of two main pages: an upload page where users can upload files, and a history page that displays all previously uploaded files with their metadata.

## Local Setup Guide

### Prerequisites
- Python 3.8 or higher installed on your system
- pip (Python package installer)

### Step-by-Step Setup Instructions

1. **Clone the Repository**
   ```bash
   git clone <repository-url>
   cd intern
   ```

2. **Create a Virtual Environment**
   
   On Windows:
   ```bash
   python -m venv venv
   venv\Scripts\activate
   ```
   
   On macOS/Linux:
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```

3. **Install Dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Run the Application**
   ```bash
   python run.py
   ```

   Alternatively, you can use uvicorn directly:
   ```bash
   uvicorn app.main:app --reload
   ```

5. **Access the Application**
   - Open your web browser and navigate to `http://localhost:8000`
   - You should see the File Uploader page

6. **Create the Database**
   - The SQLite database (`file_uploader.db`) and the necessary tables will be created automatically when you first run the application
   - No additional database setup is required

### Required Dependencies

The following packages are listed in `requirements.txt`:
- `fastapi==0.104.1` - Web framework for building APIs
- `uvicorn[standard]==0.24.0` - ASGI server for running FastAPI applications
- `sqlalchemy==2.0.23` - SQL toolkit and ORM
- `python-multipart==0.0.6` - For handling file uploads in FastAPI
- `aiofiles==23.2.1` - For asynchronous file operations

## Project Structure & Rationale

### Directory Structure
```
intern/
├── app/
│   ├── __init__.py          # Makes app a Python package
│   ├── main.py              # FastAPI application and API endpoints
│   ├── database.py          # Database connection and session management
│   └── models.py            # SQLAlchemy models for database tables
├── static/
│   ├── upload.html          # Upload page frontend
│   ├── files.html           # File history page frontend
│   └── styles.css           # CSS styling for both pages
├── uploaded_files/          # Directory where uploaded files are stored (created automatically)
├── file_uploader.db        # SQLite database (created automatically)
├── requirements.txt        # Python dependencies
├── run.py                  # Entry point to run the application
└── DOCUMENTATION.md        # This file
```

### Design Choices Explained

#### 1. Project Organization
The project follows a modular structure:
- **`app/` directory**: Contains all backend logic (models, database configuration, and API routes)
- **`static/` directory**: Contains all frontend files (HTML and CSS)
- **Separation of concerns**: Database logic (`database.py`), data models (`models.py`), and API routes (`main.py`) are in separate files for better maintainability

#### 2. Unique System Filename Generation
I chose to use **UUID (Universally Unique Identifier)** for generating unique system filenames because:
- UUIDs are extremely unlikely to collide, even with millions of files
- They provide strong uniqueness guarantees across time and space
- Python's `uuid` module makes implementation straightforward
- The original file extension is preserved by extracting it and appending it to the UUID

**Implementation**: `system_filename = f"{uuid.uuid4()}{file_extension}"`

**Example**: A file `document.pdf` might be stored as `a3f8d92e-4b7c-4d1e-9f2a-8b5c3d7e9f1a.pdf`

#### 3. File Storage and Database Synchronization
To ensure data consistency between physical file storage and database records:

**Atomic-like behavior**:
1. First, the file is read and its content is stored in memory
2. The file size is calculated from the content length
3. A unique system filename is generated
4. The file is written to disk in the `uploaded_files/` directory
5. **Only if the file write succeeds**, the metadata is inserted into the database
6. Both operations happen in a try-except block to catch any errors

**Why this approach?**
- If file writing fails, no database record is created
- If database insertion fails, the file can be identified for cleanup (by its system filename)
- The database transaction ensures atomicity for the metadata insertion
- Files are stored before database insertion to ensure we have a file to reference

**Potential improvement**: In a production system, I would implement transactional file operations with rollback capabilities for complete atomicity.

#### 4. Database Choice: SQLite
SQLite was chosen because:
- Perfect for this assignment's scope (small to medium-scale application)
- No setup required - creates a file-based database automatically
- Excellent for development and demos
- Still supports all the SQL operations we need
- Easy to inspect and debug with standard tools

#### 5. FastAPI Framework Selection
FastAPI was chosen for its:
- Excellent performance (comparable to Node.js and Go)
- Built-in automatic API documentation (Swagger UI at `/docs`)
- Native support for async operations
- Pydantic integration for data validation
- Type hints for better code quality

## Challenges Faced and Solutions

### Challenge 1: Handling File Uploads in FastAPI
**Problem**: Initially had trouble with the multipart form data handling  
**Solution**: Used FastAPI's `UploadFile` type which handles multipart/form-data encoding automatically, and included `python-multipart` in requirements

### Challenge 2: Database Session Management
**Problem**: Ensuring database sessions are properly closed to prevent connection leaks  
**Solution**: Implemented a dependency function `get_db()` that yields a database session and ensures it's closed in a `finally` block, following FastAPI's dependency injection pattern

### Challenge 3: Relative Path Issues
**Problem**: HTML files and CSS couldn't find their resources when served  
**Solution**: Mounted the static directory using FastAPI's `StaticFiles` mount point, ensuring that files in the `static/` directory are accessible via `/static/` URL prefix

### Challenge 4: Serving HTML Files
**Problem**: Needed to serve HTML files as proper HTML responses, not JSON  
**Solution**: Used `response_class=HTMLResponse` in the route decorators to ensure proper content-type headers

## API Endpoints

### GET `/`
Returns the upload page HTML.

### GET `/files`
Returns the file history page HTML.

### POST `/upload-document`
Uploads a file to the server.

**Request**: Multipart form data with a `file` field  
**Response**: JSON object containing:
- `message`: Success message
- `file_id`: Database ID of the uploaded file
- `original_filename`: Original name of the uploaded file
- `system_filename`: System-generated unique filename

### GET `/api/files`
Retrieves all file metadata from the database.

**Response**: JSON object containing an array of file objects with:
- `id`: Database ID
- `original_filename`: Original filename
- `system_filename`: System filename
- `file_size_bytes`: File size in bytes
- `uploaded_at`: ISO format timestamp

## Code Quality and Best Practices

### Code Commenting
- All API endpoints have docstrings explaining their purpose
- Complex logic sections are commented for clarity
- SQLAlchemy model fields are documented
- Database operations include explanatory comments

### Error Handling
- Try-except blocks catch and handle errors gracefully
- HTTP exceptions are raised with appropriate status codes
- User-friendly error messages are returned to the frontend

### Security Considerations
- File uploads are restricted to prevent potential security issues
- System-generated filenames prevent directory traversal attacks
- File extensions are preserved for type identification

## Frontend Features

### Upload Page
- Drag-and-drop style file selection
- Real-time file information display
- Loading states during upload
- Success and error message display
- Clean, modern UI with gradient background

### File History Page
- Tabular display of all uploaded files
- Responsive design for mobile devices
- Empty state when no files are uploaded
- Proper date and file size formatting
- Sorting by upload date (newest first)

## Future Enhancements

Potential improvements for production deployment:
1. **File validation**: Add file type validation and size limits
2. **User authentication**: Implement user accounts and permissions
3. **File download**: Add ability to download uploaded files
4. **File deletion**: Allow users to delete their uploaded files
5. **Search and filter**: Add search functionality for file history
6. **Pagination**: Implement pagination for large file lists
7. **Cloud storage**: Store files in cloud storage (AWS S3, etc.)
8. **Database migration**: Use Alembic for database schema management

## Adaptation & AI Tools

As per the assignment requirements, AI tools were used in the following ways:
- **Initial setup**: AI assistance was used to generate the FastAPI boilerplate and initial project structure
- **CSS styling**: AI helped create modern, responsive CSS with gradient backgrounds and smooth animations
- **Error handling**: AI suggestions helped implement robust error handling patterns
- **Documentation**: This documentation was written manually to explain the thought process and design decisions

The core logic, database design, and integration decisions were made independently to demonstrate understanding of the technologies and requirements.
