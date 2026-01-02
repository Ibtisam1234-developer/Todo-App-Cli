# Quickstart: Interactive Todo Application

**Feature**: 001-todo-app
**Purpose**: Setup and run instructions for developers

## Prerequisites

- Python 3.11 or higher installed
- Neon PostgreSQL account (sign up at neon.tech)
- Git (for cloning repository)
- Terminal/command line access

## Setup Steps

### 1. Clone Repository

```bash
git clone <repository-url>
cd todo-hackathon
```

### 2. Create Neon Database

1. Go to https://neon.tech and sign in
2. Create a new project
3. Create a new database (or use default)
4. Copy the connection string from the dashboard
   - Format: `postgresql://user:password@ep-xxx.region.aws.neon.tech/dbname?sslmode=require`

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

**Required dependencies** (`requirements.txt`):
```txt
psycopg2-binary>=2.9.0
python-dotenv>=1.0.0
```

### 4. Configure Environment

Create a `.env` file in the project root:

```bash
# Linux/macOS
echo "DATABASE_URL=your_neon_connection_string_here" > .env

# Windows (PowerShell)
"DATABASE_URL=your_neon_connection_string_here" | Out-File -FilePath .env -Encoding utf8
```

Replace `your_neon_connection_string_here` with your actual Neon connection string.

**Example `.env` file**:
```
DATABASE_URL=postgresql://user:pass@ep-cool-name-123456.us-east-2.aws.neon.tech/neondb?sslmode=require
```

⚠️ **Important**: Add `.env` to `.gitignore` to prevent committing credentials:
```bash
echo ".env" >> .gitignore
```

### 5. Run Application

```bash
python src/main.py
```

## Usage

### Main Menu

```
========================================
       Interactive Todo Application
========================================
1. Create New Task
2. View All Tasks
3. Update Task
4. Toggle Task Completion
5. Delete Task
6. Exit
========================================
Enter your choice (1-6):
```

### Operations

**Create Task**:
1. Select option `1`
2. Enter task title when prompted
3. Enter description (or press Enter to skip)
4. Task is saved immediately

**View All Tasks**:
1. Select option `2`
2. All tasks are displayed with:
   - Task ID
   - Status (✓ complete, ○ incomplete)
   - Title
   - Description (if present)

**Update Task**:
1. Select option `3`
2. Enter task ID to update
3. Enter new title (or press Enter to keep current)
4. Enter new description (or press Enter to keep current)

**Toggle Completion**:
1. Select option `4`
2. Enter task ID to toggle
3. Status switches between complete/incomplete

**Delete Task**:
1. Select option `5`
2. Enter task ID to delete
3. Task is permanently removed

**Exit**:
1. Select option `6`
2. Application closes cleanly

## Troubleshooting

### Connection Error: "DATABASE_URL not found"

**Solution**: Create `.env` file in project root with DATABASE_URL

### Connection Error: "Could not connect to database"

**Possible causes**:
1. Invalid Neon connection string
2. Neon database suspended (free tier auto-suspends)
3. Network connectivity issues

**Solutions**:
1. Verify connection string in `.env` matches Neon dashboard
2. Wake up database in Neon dashboard (compute suspended)
3. Check internet connection

### Module Not Found: "No module named 'psycopg2'"

**Solution**: Install dependencies:
```bash
pip install -r requirements.txt
```

### Permission Error on .env file

**Solution**: Check file permissions:
```bash
# Linux/macOS
chmod 644 .env

# Windows
# Verify file is not read-only in Properties
```

### "Tasks table does not exist"

**Solution**: Application auto-creates table on first run. If error persists, check Neon database permissions.

## File Structure

```
todo-hackathon/
├── src/
│   ├── db.py              # Database layer (Database Agent)
│   ├── models.py          # Data models (Service Agent)
│   ├── services.py        # Business logic (Service Agent)
│   ├── menu.py            # User interface (Menu Agent)
│   └── main.py            # Entry point (Menu Agent)
├── specs/
│   └── 001-todo-app/
│       ├── spec.md        # Feature specification
│       ├── plan.md        # Implementation plan
│       ├── data-model.md  # Database schema
│       ├── research.md    # Technology decisions
│       ├── quickstart.md  # This file
│       └── contracts/     # Layer contracts
├── .env                   # Environment config (create this)
├── requirements.txt       # Python dependencies
└── README.md              # Project documentation
```

## Development Workflow

### 1. Database Agent Implementation
- Implement `src/db.py` with all database functions
- Test connection to Neon
- Verify schema creation

### 2. Service Logic Agent Implementation
- Implement `src/models.py` with Task dataclass
- Implement `src/services.py` with business logic
- Test validation rules

### 3. Menu/UX Agent Implementation
- Implement `src/menu.py` with menu functions
- Implement `src/main.py` with main loop
- Test user interaction flows

### 4. Guardian Reviews
- Submit each layer to Spec Guardian for review
- Fix any constitutional violations
- Proceed to next layer only after approval

## Performance Tips

- **Neon Cold Start**: First query after inactivity may be slow (Neon waking up compute)
- **Connection Reuse**: Application maintains single connection (no reconnection overhead)
- **Task Limit**: Designed for < 1000 tasks (no pagination needed)

## Security Notes

- **Never commit** `.env` file to version control
- **Use Neon SSL**: Connection string must include `?sslmode=require`
- **Parameterized queries**: All queries use placeholders (SQL injection prevention)

## Next Steps

After successful setup:
1. Run `/sp.tasks` to generate detailed implementation tasks
2. Begin implementation following sub-agent order
3. Submit to Spec Guardian after each layer
4. Complete Guardian reviews before delivery

---

**Quickstart Status**: ✅ COMPLETE
**Target Audience**: Developers implementing the application
**Next**: Implementation begins with Database Agent
