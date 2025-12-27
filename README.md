# FastAPI-Tutorial-Fron-Zero-tu-database
# FastAPI Tutorial: From Zero to Database

A progressive tutorial for building REST APIs with FastAPI, designed for beginners.

---

## Prerequisites

- Python 3.11+ installed
- Basic Python knowledge (variables, functions, dictionaries)
- Terminal/Command line basics

---

## Step 1: Project Setup with UV

UV is a fast Python package manager. Let's use it to create our project.

### 1.1 Install UV (if not installed)

```bash
# On macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# On Windows (PowerShell)
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

```

### 1.2 Create Project

```bash
# Create a new project
uv init student-api

# Navigate into the project
cd student-api

# Create venv
uv venv

# Activate with: 
.venv\Scripts\activate
```

This creates:

```
student-api/
├── pyproject.toml    # Project configuration
├── README.md
├── main.py     
└── .python-version   # Python version

```

### 1.3 Verify Setup

```bash
# Check UV is working
uv --version

# Check Python version
uv run python --version

```

✅ **Checkpoint:** You should see UV and Python versions printed.

---

## Step 2: Install FastAPI

### 2.1 Add FastAPI Package

```bash
# Install FastAPI with standard dependencies
uv add fastapi[standard]

```

This installs:

- `fastapi` - The web framework
- `uvicorn` - The server to run our API
- `pydantic` - Data validation (included with FastAPI)

### 2.2 Verify Installation

```bash
uv run python -c "import fastapi; print(f'FastAPI version: {fastapi.__version__}')"

```

✅ **Checkpoint:** You should see the FastAPI version printed.

---

## Step 3: Your First API Endpoint

### 3.1 Create the Main File

open `main.py`:

```python
from fastapi import FastAPI

# Create the app
app = FastAPI()

# Create a simple endpoint
@app.get("/")
def home():
    return {"message": "Hello, World!"}

```

**What's happening here?**

- `FastAPI()` creates our application
- `@app.get("/")` says "when someone visits `/`, run this function"
- We return a dictionary, FastAPI converts it to JSON automatically

### 3.2 Run the Server

```bash
# Option 1
fastapi dev

# Option 2
uv run uvicorn main:app --reload

```

**Understanding the command:**

- `uvicorn` - The server
- `main:app` - File `main.py`, variable `app`
- `-reload` - Auto-restart when code changes

### 3.3 Test Your API

Open your browser and visit:

- http://127.0.0.1:8000 → See your JSON response
- http://127.0.0.1:8000/docs → Interactive API documentation (Swagger UI)

✅ **Checkpoint:** You should see `{"message": "Hello, World!"}` in your browser.

---

## Step 4: Building More Endpoints with Claude Code

Now we'll use Claude Code to progressively add more functionality.

### 4.1 Add Demo Data and List All Students

First, let's add some dummy data so we have something to work with:

**Prompt:**

```
Add a list of 3 demo students with Pakistani names (include id, name, age, email). Add an endpoint to get all students.

```

**Result:**

```python
# Demo data
students = [
    {"id": 1, "name": "Ali Khan", "age": 20, "email": "ali@example.com"},
    {"id": 2, "name": "Fatima Ahmed", "age": 22, "email": "fatima@example.com"},
    {"id": 3, "name": "Hassan Raza", "age": 19, "email": "hassan@example.com"},
]

@app.get("/students")
def get_all_students():
    return students

```

Now visit http://127.0.0.1:8000/students - you'll see all 3 students!

### 4.2 Get Single Student

**Prompt:**

```
Add an endpoint to get one student by their id. Return error if student not found.

```

**Result:**

```python
@app.get("/students/{student_id}")
def get_student(student_id: int):
    for student in students:
        if student["id"] == student_id:
            return student
    return {"error": "Student not found"}

```

### 4.3 Create a Student

**Prompt:**

```
Add an endpoint to create a new student. Accept name, age, email. Return the created student.

```

**Result:**

```python
from pydantic import BaseModel

class Student(BaseModel):
    name: str
    age: int
    email: str

@app.post("/students")
def create_student(student: Student):
    return {"message": "Student created", "student": student}

```

### 4.4 Update and Delete Student

**Prompt:**

```
Add endpoints to update a student and delete a student by their id.

```

### 4.5 Test All Endpoints

After each addition, test using:

- Browser: http://127.0.0.1:8000/docs
- Use the "Try it out" button in Swagger UI

⚠️ **Note:** Data is lost when server restarts! That's why we need a database.

✅ **Checkpoint:** You can now list, view, create, update, and delete students.

---

## Step 5: Introduction to SQLModel

SQLModel combines Pydantic (validation) + SQLAlchemy (database) in one library.

### 5.1 Install SQLModel

```bash
uv add sqlmodel

```

### 5.2 Understanding SQLModel

```python
from sqlmodel import SQLModel, Field

# This is BOTH a Pydantic model AND a database table!
class Student(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str
    age: int
    email: str

```

**Key Concepts:**

- `SQLModel` - Base class for models
- `table=True` - Makes it a database table
- `Field(primary_key=True)` - Makes `id` the primary key
- `id: int | None = Field(default=None, ...)` - Auto-generated by database

### 5.3 Claude Code Prompt

**Prompt:**

```
Prepare the Student model for database. Use SQLModel library. Keep in-memory storage working for now.

```

✅ **Checkpoint:** App works exactly the same, but now using SQLModel classes.

---

## Step 6: Database Integration

Now let's connect to a real SQLite database.

### 6.1 Claude Code Prompt - Database Setup

**Prompt:**

```
Add SQLite database connection. Create the database file and tables when app starts. Don't change endpoints yet.

```

### 6.2 Claude Code Prompt - Database CRUD

**Prompt:**

```
Update all endpoints to save and read students from the database instead of memory.

```

**Expected Final Structure:**

```python
from fastapi import FastAPI, HTTPException, Depends
from sqlmodel import SQLModel, Field, Session, create_engine, select

# Database setup
DATABASE_URL = "sqlite:///students.db"
engine = create_engine(DATABASE_URL)

def create_db():
    SQLModel.metadata.create_all(engine)

def get_session():
    with Session(engine) as session:
        yield session

# Models
class StudentBase(SQLModel):
    name: str
    age: int
    email: str

class Student(StudentBase, table=True):
    id: int | None = Field(default=None, primary_key=True)

class StudentCreate(StudentBase):
    pass

# App
app = FastAPI()

@app.on_event("startup")
def on_startup():
    create_db()

@app.post("/students", response_model=Student)
def create_student(student: StudentCreate, session: Session = Depends(get_session)):
    db_student = Student.model_validate(student)
    session.add(db_student)
    session.commit()
    session.refresh(db_student)
    return db_student

@app.get("/students")
def list_students(session: Session = Depends(get_session)):
    students = session.exec(select(Student)).all()
    return students

@app.get("/students/{student_id}")
def get_student(student_id: int, session: Session = Depends(get_session)):
    student = session.get(Student, student_id)
    if not student:
        raise HTTPException(status_code=404, detail="Student not found")
    return student

@app.put("/students/{student_id}")
def update_student(
    student_id: int,
    student_data: StudentCreate,
    session: Session = Depends(get_session)
):
    student = session.get(Student, student_id)
    if not student:
        raise HTTPException(status_code=404, detail="Student not found")

    student.name = student_data.name
    student.age = student_data.age
    student.email = student_data.email

    session.commit()
    session.refresh(student)
    return student

@app.delete("/students/{student_id}")
def delete_student(student_id: int, session: Session = Depends(get_session)):
    student = session.get(Student, student_id)
    if not student:
        raise HTTPException(status_code=404, detail="Student not found")

    session.delete(student)
    session.commit()
    return {"message": "Student deleted"}

```

### 6.3 Test Database Integration

1. Restart server
2. Create students
3. Stop server, start again
4. Students should still exist! 🎉

You can also check the database directly:

```bash
uv run python -c "
from sqlmodel import Session, select, create_engine
from main import Student, engine

with Session(engine) as session:
    students = session.exec(select(Student)).all()
    for s in students:
        print(f'{s.id}: {s.name} ({s.email})')
"

```

✅ **Checkpoint:** Data persists across server restarts!

---

## Step 7: Cloud Database with Neon

SQLite stores data in a file on your computer. But what if you want:

- Data accessible from anywhere (cloud)
- Multiple apps connecting to same database
- Professional production setup

**Neon** is a free cloud PostgreSQL database - perfect for this!

### 8.1 Create Neon Account & Database

1. Go to [neon.tech](https://neon.tech/) and sign up (free)
2. Create a new project
3. Copy your connection string (looks like: `postgresql://user:pass@ep-xxx.region.aws.neon.tech/neondb`)

### 8.2 Install PostgreSQL Driver

SQLModel works with any database, but needs a **driver** to connect:

| Database | Driver |
| --- | --- |
| SQLite | Built-in (no install) |
| PostgreSQL | `psycopg2-binary` |

```bash
uv add psycopg2-binary python-dotenv
```

Your code stays the same! Only the connection string changes.

### 8.3 Claude Code Prompt

**Prompt:**

```
Change database from SQLite to Neon PostgreSQL. Load connection string from environment variable DATABASE_URL.

```

### 8.4 What Changes

Only ONE line changes - the connection string:

```python
import os

# Before (SQLite - local file)
DATABASE_URL = "sqlite:///students.db"

# After (Neon - cloud database)
DATABASE_URL = os.environ.get("DATABASE_URL")

```

### 8.5 Run with Neon

Create a `.env` file:

```
DATABASE_URL=postgresql://user:password@ep-xxx.region.aws.neon.tech/neondb?sslmode=require

```

Run the server:

```bash
uv run uvicorn main:app --reload

```

### 8.6 Why Neon?

| SQLite | Neon |
| --- | --- |
| File on your computer | Cloud database |
| Only you can access | Access from anywhere |
| Good for learning | Good for production |
| Free | Free tier available |

✅ **Checkpoint:** Your data is now in the cloud!

## Summary

| Step | What You Learned |
| --- | --- |
| 1 | UV project setup |
| 2 | Installing packages |
| 3 | Basic GET endpoint |
| 4 | Path params, query params, POST, PUT, DELETE |
| 5 | In-memory storage, HTTPException |
| 6 | SQLModel basics |
| 7 | Database integration with SQLite |

---

## What's Next?

- Add data validation (email format, age range)
- Add relationships (Students → Courses)
- Add authentication
- Deploy to production

---

## Quick Reference: Claude Code Prompts

### Pattern

```
[What you want] + [Simple details]

```

### Example Prompts

**Add field:**

```
Add a grade field to students. It should be optional.

```

**Add validation:**

```
Validate student data: age between 5-100, email must be valid, name at least 2 characters.

```

**Add filtering:**

```
Let me filter students by age range and search by name.

```

**Add pagination:**

```
Add pagination to student list. Let me skip records and set a limit.

```

---

## Troubleshooting

**"Module not found" error:**

```bash
uv sync  # Reinstall dependencies

```

**"Address already in use":**

```bash
# Find and kill the process using port 8000
lsof -i :8000
kill -9 <PID>

```

**Database locked:**

```bash
# Delete database and restart
rm students.db
uv run uvicorn main:app --reload

```

---

Happy coding! 🚀
