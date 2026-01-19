# SQLAlchemy Joins and Field Selection Guide

A comprehensive guide to selecting specific fields and performing different types of joins in SQLAlchemy/SQLModel.

## Table of Contents
1. [Selecting Specific Fields](#selecting-specific-fields)
2. [Join Types](#join-types)
3. [Practical Examples](#practical-examples)
4. [Best Practices](#best-practices)

---

## Selecting Specific Fields

### Method 1: Select Specific Columns

```python
from sqlmodel import select

# Select only specific fields from one table
query = select(
    Assignment.id,
    Assignment.client_name,
    Assignment.assignment_title
).where(Assignment.id == 1)

result = await session.execute(query)
rows = result.all()  # Returns list of tuples: [(id, client_name, title), ...]

# Access data
for row in rows:
    id, client_name, title = row
    print(f"{id}: {client_name} - {title}")
```

**SQL Generated:**
```sql
SELECT assignment.id, assignment.client_name, assignment.assignment_title 
FROM assignment 
WHERE assignment.id = 1;
```

### Method 2: Select from Multiple Tables

```python
# Select specific fields from multiple tables
query = select(
    AssignmentApplication.id,
    AssignmentApplication.status,
    Assignment.client_name,
    Assignment.assignment_title
).join(Assignment).where(AssignmentApplication.consultant_id == 10)

result = await session.execute(query)
rows = result.all()

# Access data
for row in rows:
    app_id, status, client, title = row
    print(f"Application {app_id}: {client} - {title} ({status})")
```

**SQL Generated:**
```sql
SELECT 
    assignmentapplication.id,
    assignmentapplication.status,
    assignment.client_name,
    assignment.assignment_title
FROM assignmentapplication 
JOIN assignment ON assignment.id = assignmentapplication.assignment_id
WHERE assignmentapplication.consultant_id = 10;
```

### Method 3: Using Column Objects

```python
from sqlalchemy import column

# When you need to use column expressions
query = select(
    AssignmentApplication.id.label("app_id"),
    Assignment.client_name.label("client"),
    (Assignment.minimum_salary + Assignment.maximum_salary) / 2  # Calculated field
).join(Assignment)

result = await session.execute(query)
```

### Method 4: Select with Aggregations

```python
from sqlmodel import func, select

# Count, sum, avg, etc.
query = select(
    Assignment.client_name,
    func.count(AssignmentApplication.id).label("application_count"),
    func.avg(AssignmentApplication.expected_salary).label("avg_salary")
).join(AssignmentApplication).group_by(Assignment.client_name)

result = await session.execute(query)
for client, count, avg_salary in result:
    print(f"{client}: {count} applications, avg salary: {avg_salary}")
```

**SQL Generated:**
```sql
SELECT 
    assignment.client_name,
    COUNT(assignmentapplication.id) AS application_count,
    AVG(assignmentapplication.expected_salary) AS avg_salary
FROM assignment
JOIN assignmentapplication ON assignment.id = assignmentapplication.assignment_id
GROUP BY assignment.client_name;
```

---

## Join Types

### 1. INNER JOIN (Default)

**Description**: Returns only rows where there's a match in both tables.

```python
from sqlmodel import select

# Method A: Using .join() - INNER JOIN by default
query = select(AssignmentApplication).join(Assignment)

# Method B: Explicit ON condition
query = select(AssignmentApplication).join(
    Assignment, 
    AssignmentApplication.assignment_id == Assignment.id
)

# Method C: Select specific fields
query = select(
    AssignmentApplication.id,
    Assignment.client_name
).join(Assignment)
```

**SQL Generated:**
```sql
SELECT assignmentapplication.*, assignment.client_name
FROM assignmentapplication
INNER JOIN assignment ON assignment.id = assignmentapplication.assignment_id;
```

**Use Case**: Get applications with existing assignments only.

---

### 2. LEFT JOIN (LEFT OUTER JOIN)

**Description**: Returns all rows from left table, and matching rows from right table. If no match, NULL for right table columns.

```python
from sqlmodel import select

# Method A: Using .join() with isouter=True
query = select(AssignmentApplication).join(
    Assignment,
    isouter=True  # LEFT JOIN
)

# Method B: Using .outerjoin() - clearer syntax
query = select(AssignmentApplication).outerjoin(Assignment)

# Method C: With specific fields
query = select(
    AssignmentApplication.id,
    AssignmentApplication.status,
    Assignment.client_name,
    Assignment.assignment_title
).outerjoin(
    Assignment,
    AssignmentApplication.assignment_id == Assignment.id
)
```

**SQL Generated:**
```sql
SELECT 
    assignmentapplication.id,
    assignmentapplication.status,
    assignment.client_name,
    assignment.assignment_title
FROM assignmentapplication
LEFT OUTER JOIN assignment ON assignment.id = assignmentapplication.assignment_id;
```

**Use Case**: Get all applications even if assignment was deleted.

---

### 3. RIGHT JOIN (RIGHT OUTER JOIN)

**Description**: Returns all rows from right table, and matching rows from left table. If no match, NULL for left table columns.

```python
from sqlmodel import select
from sqlalchemy import select as sa_select

# SQLAlchemy 2.0 style - join from the "right" table
query = select(Assignment).join(
    AssignmentApplication,
    Assignment.id == AssignmentApplication.assignment_id,
    isouter=True  # Make Assignment the "kept" table
)

# Alternative: Use LEFT JOIN but flip the tables
query = select(Assignment).outerjoin(AssignmentApplication)

# Or use raw SQL-style with select_from
from sqlalchemy import text

query = sa_select(
    Assignment.id,
    Assignment.client_name,
    AssignmentApplication.id.label("app_id")
).select_from(Assignment).outerjoin(
    AssignmentApplication,
    Assignment.id == AssignmentApplication.assignment_id
)
```

**SQL Generated (conceptually):**
```sql
SELECT 
    assignment.id,
    assignment.client_name,
    assignmentapplication.id AS app_id
FROM assignment
LEFT OUTER JOIN assignmentapplication 
    ON assignment.id = assignmentapplication.assignment_id;
-- Note: RIGHT JOIN is equivalent to LEFT JOIN with tables flipped
```

**Use Case**: Get all assignments even if no one applied.

**Note**: In SQLAlchemy, you typically use LEFT JOIN with flipped table order instead of RIGHT JOIN.

---

### 4. FULL OUTER JOIN

**Description**: Returns all rows from both tables. If no match, NULL for missing side.

```python
from sqlmodel import select
from sqlalchemy import full_join

# Method A: Using full() parameter
query = select(
    AssignmentApplication.id.label("app_id"),
    Assignment.id.label("assignment_id"),
    Assignment.client_name
).select_from(
    AssignmentApplication
).join(
    Assignment,
    AssignmentApplication.assignment_id == Assignment.id,
    full=True  # FULL OUTER JOIN
)

# Method B: Using select_from with join
from sqlalchemy.orm import outerjoin

query = select(
    AssignmentApplication,
    Assignment
).select_from(
    full_join(
        AssignmentApplication,
        Assignment,
        AssignmentApplication.assignment_id == Assignment.id
    )
)
```

**SQL Generated:**
```sql
SELECT 
    assignmentapplication.id AS app_id,
    assignment.id AS assignment_id,
    assignment.client_name
FROM assignmentapplication
FULL OUTER JOIN assignment 
    ON assignment.id = assignmentapplication.assignment_id;
```

**Use Case**: Get all applications and all assignments, showing which are matched and which aren't.

**Note**: Not all databases support FULL OUTER JOIN (MySQL doesn't). PostgreSQL does.

---

## Practical Examples

### Example 1: Get Application with Assignment Details (INNER JOIN)

```python
async def get_application_with_assignment(
    session: AsyncSession, 
    application_id: int
) -> Optional[Dict[str, Any]]:
    """Get application with assignment details using INNER JOIN"""
    
    query = select(
        # Application fields
        AssignmentApplication.id,
        AssignmentApplication.status,
        AssignmentApplication.expected_salary,
        AssignmentApplication.created_at,
        # Assignment fields
        Assignment.client_name,
        Assignment.assignment_title,
        Assignment.minimum_salary,
        Assignment.maximum_salary,
        Assignment.location_city
    ).join(
        Assignment,  # INNER JOIN - only if assignment exists
        AssignmentApplication.assignment_id == Assignment.id
    ).where(
        AssignmentApplication.id == application_id
    )
    
    result = await session.execute(query)
    row = result.first()
    
    if not row:
        return None
    
    return {
        "id": row[0],
        "status": row[1],
        "expected_salary": row[2],
        "created_at": row[3],
        "assignment": {
            "client_name": row[4],
            "assignment_title": row[5],
            "minimum_salary": row[6],
            "maximum_salary": row[7],
            "location_city": row[8],
        }
    }
```

---

### Example 2: Get All Applications Even If Assignment Deleted (LEFT JOIN)

```python
async def get_all_applications_safe(
    session: AsyncSession,
    consultant_id: int
) -> List[Dict[str, Any]]:
    """Get all applications, even if assignment was deleted"""
    
    query = select(
        AssignmentApplication.id,
        AssignmentApplication.status,
        AssignmentApplication.assignment_id,
        Assignment.client_name,  # Might be NULL
        Assignment.assignment_title  # Might be NULL
    ).outerjoin(  # LEFT OUTER JOIN
        Assignment,
        AssignmentApplication.assignment_id == Assignment.id
    ).where(
        AssignmentApplication.consultant_id == consultant_id
    )
    
    result = await session.execute(query)
    rows = result.all()
    
    applications = []
    for row in rows:
        applications.append({
            "id": row[0],
            "status": row[1],
            "assignment_id": row[2],
            "assignment": {
                "client_name": row[3] if row[3] else "Deleted",
                "assignment_title": row[4] if row[4] else "N/A"
            }
        })
    
    return applications
```

---

### Example 3: Multiple Joins with Specific Fields

```python
async def get_application_with_match_details(
    session: AsyncSession,
    consultant_id: int
) -> List[Dict[str, Any]]:
    """Get applications with assignment and match details"""
    
    query = select(
        # From AssignmentApplication
        AssignmentApplication.id,
        AssignmentApplication.status,
        # From Assignment
        Assignment.client_name,
        Assignment.assignment_title,
        # From Match
        AssignmentConsultantMatch.score_overall,
        AssignmentConsultantMatch.status.label("match_status")
    ).join(
        Assignment,  # INNER JOIN
        AssignmentApplication.assignment_id == Assignment.id
    ).outerjoin(  # LEFT JOIN - not all applications have matches
        AssignmentConsultantMatch,
        (AssignmentConsultantMatch.assignment_id == AssignmentApplication.assignment_id) &
        (AssignmentConsultantMatch.consultant_id == AssignmentApplication.consultant_id)
    ).where(
        AssignmentApplication.consultant_id == consultant_id
    )
    
    result = await session.execute(query)
    rows = result.all()
    
    items = []
    for row in rows:
        items.append({
            "id": row[0],
            "status": row[1],
            "assignment": {
                "client_name": row[2],
                "assignment_title": row[3]
            },
            "match": {
                "score": row[4] if row[4] else None,
                "status": row[5] if row[5] else None
            }
        })
    
    return items
```

---

### Example 4: Join with Filtering and Aggregation

```python
async def get_consultant_statistics(
    session: AsyncSession,
    consultant_id: int
) -> Dict[str, Any]:
    """Get consultant statistics with joins and aggregations"""
    
    query = select(
        func.count(AssignmentApplication.id).label("total_applications"),
        func.count(
            func.distinct(AssignmentApplication.assignment_id)
        ).label("unique_assignments"),
        func.avg(AssignmentApplication.expected_salary).label("avg_expected_salary"),
        func.avg(Assignment.minimum_salary).label("avg_offered_salary"),
        func.count(AssignmentConsultantMatch.id).label("match_count")
    ).select_from(
        AssignmentApplication
    ).join(
        Assignment,
        AssignmentApplication.assignment_id == Assignment.id
    ).outerjoin(
        AssignmentConsultantMatch,
        (AssignmentConsultantMatch.assignment_id == Assignment.id) &
        (AssignmentConsultantMatch.consultant_id == consultant_id)
    ).where(
        AssignmentApplication.consultant_id == consultant_id
    )
    
    result = await session.execute(query)
    row = result.first()
    
    return {
        "total_applications": row[0],
        "unique_assignments": row[1],
        "avg_expected_salary": float(row[2]) if row[2] else 0,
        "avg_offered_salary": float(row[3]) if row[3] else 0,
        "match_count": row[4]
    }
```

---

### Example 5: Subquery Join

```python
from sqlalchemy import alias

async def get_assignments_with_application_count(
    session: AsyncSession
) -> List[Dict[str, Any]]:
    """Get assignments with application count using subquery"""
    
    # Subquery to count applications per assignment
    app_count_subq = (
        select(
            AssignmentApplication.assignment_id,
            func.count(AssignmentApplication.id).label("app_count")
        )
        .group_by(AssignmentApplication.assignment_id)
    ).subquery()
    
    # Main query
    query = select(
        Assignment.id,
        Assignment.client_name,
        Assignment.assignment_title,
        func.coalesce(app_count_subq.c.app_count, 0).label("application_count")
    ).outerjoin(
        app_count_subq,
        Assignment.id == app_count_subq.c.assignment_id
    )
    
    result = await session.execute(query)
    rows = result.all()
    
    return [
        {
            "id": row[0],
            "client_name": row[1],
            "assignment_title": row[2],
            "application_count": row[3]
        }
        for row in rows
    ]
```

---

## Best Practices

### 1. **Choose the Right Join Type**

```python
# ✅ Use INNER JOIN when you need matching records only
query = select(AssignmentApplication).join(Assignment)

# ✅ Use LEFT JOIN when you want all left records, even without matches
query = select(AssignmentApplication).outerjoin(Assignment)

# ✅ Use multiple LEFT JOINs for optional relationships
query = (
    select(AssignmentApplication)
    .join(Assignment)  # Must have assignment
    .outerjoin(AssignmentConsultantMatch)  # May have match
)
```

### 2. **Select Only What You Need**

```python
# ❌ Bad - fetches entire objects with all fields
query = select(AssignmentApplication, Assignment).join(Assignment)

# ✅ Good - fetches only needed fields
query = select(
    AssignmentApplication.id,
    AssignmentApplication.status,
    Assignment.client_name,
    Assignment.assignment_title
).join(Assignment)
```

### 3. **Use Labels for Clarity**

```python
# ✅ Use labels when selecting from multiple tables
query = select(
    AssignmentApplication.id.label("app_id"),
    Assignment.id.label("assignment_id"),
    Assignment.client_name
).join(Assignment)

result = await session.execute(query)
for row in result:
    print(f"App: {row.app_id}, Assignment: {row.assignment_id}")
```

### 4. **Handle NULL Values from Outer Joins**

```python
# ✅ Always check for NULL when using LEFT/RIGHT/FULL JOINs
query = select(
    AssignmentApplication.id,
    Assignment.client_name
).outerjoin(Assignment)

result = await session.execute(query)
for row in result:
    client = row[1] if row[1] is not None else "Unknown"
    print(f"Application {row[0]}: Client = {client}")
```

### 5. **Use Exists for Filtering**

```python
from sqlalchemy import exists

# ✅ Use exists() for efficient existence checks
has_match_subq = (
    select(1)
    .where(AssignmentConsultantMatch.assignment_id == Assignment.id)
    .where(AssignmentConsultantMatch.consultant_id == 10)
    .exists()
)

query = select(Assignment).where(has_match_subq)
```

---

## Comparison: Explicit vs Relationship Loading

### Explicit Field Selection (Recommended for APIs)

```python
# Pros: Fine-grained control, select only needed fields
query = select(
    AssignmentApplication.id,
    Assignment.client_name,
    Assignment.assignment_title
).join(Assignment).where(AssignmentApplication.consultant_id == 10)

result = await session.execute(query)
rows = result.all()

# Returns tuples of specific fields
for app_id, client, title in rows:
    print(f"{app_id}: {client} - {title}")
```

### Relationship Loading (Recommended for ORM objects)

```python
# Pros: Type-safe, ORM objects with all features
from sqlalchemy.orm import selectinload

query = (
    select(AssignmentApplication)
    .options(selectinload(AssignmentApplication.assignment))
    .where(AssignmentApplication.consultant_id == 10)
)

result = await session.execute(query)
applications = result.scalars().all()

# Returns full ORM objects
for app in applications:
    print(f"{app.id}: {app.assignment.client_name} - {app.assignment.assignment_title}")
```

---

## Quick Reference Table

| Join Type | SQLAlchemy Method | SQL Equivalent | Use Case |
|-----------|-------------------|----------------|----------|
| **Inner** | `.join()` | `INNER JOIN` | Only matching rows |
| **Left** | `.outerjoin()` or `.join(isouter=True)` | `LEFT JOIN` | All left rows + matches |
| **Right** | Flip tables + `.outerjoin()` | `RIGHT JOIN` | All right rows + matches |
| **Full** | `.join(full=True)` | `FULL OUTER JOIN` | All rows from both |

---

## Common Patterns

```python
# Pattern 1: Select specific fields from joined tables
query = select(Table1.col1, Table2.col2).join(Table2)

# Pattern 2: Left join with NULL handling
query = select(Table1, Table2.col).outerjoin(Table2)

# Pattern 3: Multiple joins
query = select(Table1).join(Table2).outerjoin(Table3)

# Pattern 4: Join with complex condition
query = select(Table1).join(
    Table2, 
    (Table1.id == Table2.table1_id) & (Table2.status == "active")
)

# Pattern 5: Subquery join
subq = select(Table2).subquery()
query = select(Table1).join(subq, Table1.id == subq.c.table1_id)
```

---

## Conclusion

- **Select specific fields** for API responses (better performance)
- **Use INNER JOIN** when both sides must exist
- **Use LEFT JOIN** when left side is required, right is optional
- **Flip tables** instead of using RIGHT JOIN
- **Use FULL OUTER JOIN** rarely (not all DBs support it)
- **Handle NULLs** when using outer joins
- **Label columns** for clarity in multi-table selects
