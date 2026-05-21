## Database Schema and Table Structures

### 1. **Contact Table** (`app_contact`)

```sql
CREATE TABLE app_contact (
    id CHAR(32) PRIMARY KEY,  -- UUID without hyphens or CHAR(36) with hyphens
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    middle_name VARCHAR(100),
    date_of_birth DATE,
    gender VARCHAR(20),
    notes TEXT,
    created_at DATETIME NOT NULL,
    updated_at DATETIME NOT NULL
);

-- Sample records stored:
+--------------------------------------+------------+-----------+-------------+---------------+--------+----------------------------------------------------------+---------------------+---------------------+
| id                                   | first_name | last_name | middle_name | date_of_birth | gender | notes                                                    | created_at          | updated_at          |
+--------------------------------------+------------+-----------+-------------+---------------+--------+----------------------------------------------------------+---------------------+---------------------+
| 550e8400-e29b-41d4-a716-446655440000 | John       | Smith     | A.          | 1985-03-15    | M      | Prefers email communication, Available after 2 PM       | 2024-01-15 10:30:00 | 2024-01-15 10:30:00 |
| 6ba7b810-9dad-11d1-80b4-00c04fd430c8 | Sarah      | Johnson   | M.          | 1990-07-22    | F      | Bilingual - Spanish, VIP customer                       | 2024-01-15 10:30:01 | 2024-01-15 10:30:01 |
| 7c9e6679-7425-40de-944b-e07fc1f90ae7 | Michael    | Williams  | B.          | 1978-11-08    | M      | Work from home, Priority contact                         | 2024-01-15 10:30:02 | 2024-01-15 10:30:02 |
| 8e9e6679-7425-40de-944b-e07fc1f90ae8 | Emily      | Brown     | C.          | 1995-02-14    | F      | Special needs accommodation                              | 2024-01-15 10:30:03 | 2024-01-15 10:30:03 |
| 9f0e6679-7425-40de-944b-e07fc1f90ae9 | David      | Jones     |             | 1982-09-30    | M      |                                                          | 2024-01-15 10:30:04 | 2024-01-15 10:30:04 |
+--------------------------------------+------------+-----------+-------------+---------------+--------+----------------------------------------------------------+---------------------+---------------------+
```

### 2. **Address Table** (`app_address`)

```sql
CREATE TABLE app_address (
    id CHAR(32) PRIMARY KEY,
    contact_id CHAR(32) NOT NULL,
    address_type VARCHAR(20) NOT NULL,
    street VARCHAR(255) NOT NULL,
    street2 VARCHAR(255),
    city VARCHAR(100) NOT NULL,
    state VARCHAR(50),
    postal_code VARCHAR(20) NOT NULL,
    country VARCHAR(100) NOT NULL,
    is_default BOOLEAN NOT NULL,
    is_primary BOOLEAN NOT NULL,
    latitude DECIMAL(10,6),
    longitude DECIMAL(10,6),
    FOREIGN KEY (contact_id) REFERENCES app_contact(id) ON DELETE CASCADE
);

-- Sample records stored:
+--------------------------------------+--------------------------------------+--------------+----------------------+------------------+-----------+-------+-------------+---------+------------+------------+-------------+--------------+
| id                                   | contact_id                           | address_type | street               | street2          | city      | state | postal_code | country | is_default | is_primary | latitude    | longitude    |
+--------------------------------------+--------------------------------------+--------------+----------------------+------------------+-----------+-------+-------------+---------+------------+------------+-------------+--------------+
| a1b2c3d4-e5f6-7890-abcd-ef1234567890 | 550e8400-e29b-41d4-a716-446655440000 | home         | 742 Evergreen Terrace | Apt 4B           | Springfield | IL    | 62701       | USA     | 1          | 1          | 39.781721   | -89.650148   |
| b2c3d4e5-f6a7-8901-bcde-f234567890ab | 550e8400-e29b-41d4-a716-446655440000 | work         | 123 Corporate Drive   | Suite 500        | Chicago     | IL    | 60601       | USA     | 0          | 0          | 41.878114   | -87.629798   |
| c3d4e5f6-a7b8-9012-cdef-34567890abcd | 6ba7b810-9dad-11d1-80b4-00c04fd430c8 | work         | 456 Business Ave      | Floor 12         | New York    | NY    | 10001       | USA     | 1          | 1          | 40.712776   | -74.005974   |
| d4e5f6a7-b8c9-0123-defg-4567890abcde | 6ba7b810-9dad-11d1-80b4-00c04fd430c8 | billing      | 789 Financial Blvd    |                  | Los Angeles | CA    | 90001       | USA     | 0          | 0          | 34.052235   | -118.243683  |
| e5f6a7b8-c9d0-1234-efgh-567890abcdef | 7c9e6679-7425-40de-944b-e07fc1f90ae7 | home         | 321 Residential Ln    |                  | Houston     | TX    | 77001       | USA     | 1          | 1          | 29.760427   | -95.369804   |
+--------------------------------------+--------------------------------------+--------------+----------------------+------------------+-----------+-------+-------------+---------+------------+------------+-------------+--------------+
```

### 3. **Phone Table** (`app_phone`)

```sql
CREATE TABLE app_phone (
    id CHAR(32) PRIMARY KEY,
    contact_id CHAR(32) NOT NULL,
    phone_type VARCHAR(20) NOT NULL,
    number VARCHAR(20) NOT NULL,
    extension VARCHAR(10),
    is_default BOOLEAN NOT NULL,
    is_primary BOOLEAN NOT NULL,
    FOREIGN KEY (contact_id) REFERENCES app_contact(id) ON DELETE CASCADE
);

-- Sample records stored:
+--------------------------------------+--------------------------------------+------------+-----------------+-----------+------------+------------+
| id                                   | contact_id                           | phone_type | number          | extension | is_default | is_primary |
+--------------------------------------+--------------------------------------+------------+-----------------+-----------+------------+------------+
| f6a7b8c9-d0e1-2345-fghi-678901abcdef | 550e8400-e29b-41d4-a716-446655440000 | mobile     | +1-212-555-0123 |           | 1          | 1          |
| a7b8c9d0-e1f2-3456-ghij-789012bcdefg | 550e8400-e29b-41d4-a716-446655440000 | work       | +1-312-555-0456 | 1234      | 0          | 0          |
| b8c9d0e1-f2a3-4567-hijk-890123cdefgh | 6ba7b810-9dad-11d1-80b4-00c04fd430c8 | mobile     | +1-310-555-0789 |           | 1          | 1          |
| c9d0e1f2-a3b4-5678-ijkl-901234defghi | 6ba7b810-9dad-11d1-80b4-00c04fd430c8 | home       | +1-213-555-1011 |           | 0          | 0          |
| d0e1f2a3-b4c5-6789-jklm-012345efghij | 7c9e6679-7425-40de-944b-e07fc1f90ae7 | work       | +1-713-555-1213 | 5678      | 1          | 1          |
+--------------------------------------+--------------------------------------+------------+-----------------+-----------+------------+------------+
```

### 4. **Email Table** (`app_email`)

```sql
CREATE TABLE app_email (
    id CHAR(32) PRIMARY KEY,
    contact_id CHAR(32) NOT NULL,
    email_type VARCHAR(20) NOT NULL,
    email VARCHAR(254) NOT NULL UNIQUE,
    is_default BOOLEAN NOT NULL,
    is_primary BOOLEAN NOT NULL,
    is_verified BOOLEAN NOT NULL,
    FOREIGN KEY (contact_id) REFERENCES app_contact(id) ON DELETE CASCADE
);

-- Sample records stored:
+--------------------------------------+--------------------------------------+------------+---------------------------------+------------+------------+-------------+
| id                                   | contact_id                           | email_type | email                           | is_default | is_primary | is_verified |
+--------------------------------------+--------------------------------------+------------+---------------------------------+------------+------------+-------------+
| e1f2a3b4-c5d6-7890-klmn-123456fghijk | 550e8400-e29b-41d4-a716-446655440000 | work       | john.smith@company.com          | 1          | 1          | 1           |
| f2a3b4c5-d6e7-8901-lmno-234567ghijkl | 550e8400-e29b-41d4-a716-446655440000 | personal   | john.smith@gmail.com            | 0          | 0          | 1           |
| a3b4c5d6-e7f8-9012-mnop-345678hijklm | 6ba7b810-9dad-11d1-80b4-00c04fd430c8 | work       | sarah.johnson@techcorp.com      | 1          | 1          | 1           |
| b4c5d6e7-f8a9-0123-nopq-456789ijklmn | 6ba7b810-9dad-11d1-80b4-00c04fd430c8 | personal   | sarah.johnson@yahoo.com         | 0          | 0          | 1           |
| c5d6e7f8-a9b0-1234-opqr-567890jklmno | 7c9e6679-7425-40de-944b-e07fc1f90ae7 | work       | michael.williams@innovate.com   | 1          | 1          | 0           |
+--------------------------------------+--------------------------------------+------------+---------------------------------+------------+------------+-------------+
```

### 5. **Employee Table** (`app_employee`)

```sql
CREATE TABLE app_employee (
    id CHAR(32) PRIMARY KEY,
    employee_id VARCHAR(50) NOT NULL UNIQUE,
    hire_date DATE NOT NULL,
    department VARCHAR(100) NOT NULL,
    position VARCHAR(100) NOT NULL,
    salary DECIMAL(10,2),
    created_at DATETIME NOT NULL
);

-- Sample records stored:
+--------------------------------------+-------------+------------+----------------+------------------------+------------+---------------------+
| id                                   | employee_id | hire_date  | department     | position               | salary     | created_at          |
+--------------------------------------+-------------+------------+----------------+------------------------+------------+---------------------+
| f3a4b5c6-d7e8-9012-pqrs-678901klmnop | EMP12345    | 2020-01-15 | Engineering    | Senior Developer       | 95000.00   | 2024-01-15 10:30:00 |
| a4b5c6d7-e8f9-0123-qrst-789012lmnopq | EMP12346    | 2021-03-10 | Sales          | Account Executive      | 75000.00   | 2024-01-15 10:30:01 |
| b5c6d7e8-f9a0-1234-rstu-890123mnopqr | EMP12347    | 2019-11-20 | Marketing      | Marketing Specialist   | 68000.00   | 2024-01-15 10:30:02 |
| c6d7e8f9-a0b1-2345-stuv-901234nopqrs | EMP12348    | 2022-06-05 | HR             | HR Manager             | 82000.00   | 2024-01-15 10:30:03 |
| d7e8f9a0-b1c2-3456-tuvw-012345opqrst | EMP12349    | 2023-01-12 | Operations     | Operations Director    | 110000.00  | 2024-01-15 10:30:04 |
+--------------------------------------+-------------+------------+----------------+------------------------+------------+---------------------+
```

### 6. **Client Table** (`app_client`)

```sql
CREATE TABLE app_client (
    id CHAR(32) PRIMARY KEY,
    company_name VARCHAR(200) NOT NULL,
    client_since DATE NOT NULL,
    client_type VARCHAR(50) NOT NULL,
    credit_limit DECIMAL(12,2),
    payment_terms INTEGER NOT NULL
);

-- Sample records stored:
+--------------------------------------+----------------------+-------------+-------------+--------------+---------------+
| id                                   | company_name         | client_since| client_type | credit_limit | payment_terms |
+--------------------------------------+----------------------+-------------+-------------+--------------+---------------+
| e8f9a0b1-c2d3-4567-uvwx-123456pqrstu | TechCorp Solutions   | 2023-01-15  | enterprise  | 50000.00     | 45            |
| f9a0b1c2-d3e4-5678-vwxy-234567qrstuv | Global Industries    | 2023-03-20  | business    | 25000.00     | 30            |
| a0b1c2d3-e4f5-6789-wxyz-345678rstuvw | Innovative Systems   | 2022-11-10  | enterprise  | 75000.00     | 60            |
| b1c2d3e4-f5a6-7890-xyza-456789stuvwx | Premier Consulting   | 2024-01-05  | business    | 15000.00     | 30            |
| c2d3e4f5-a6b7-8901-yzab-567890tuvwxy | Apex Dynamics        | 2023-08-22  | individual  | 5000.00      | 15            |
+--------------------------------------+----------------------+-------------+-------------+--------------+---------------+
```

### 7. **Customer Table** (`app_customer`)

```sql
CREATE TABLE app_customer (
    id CHAR(32) PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    customer_since DATE NOT NULL,
    customer_type VARCHAR(50) NOT NULL,
    loyalty_points INTEGER NOT NULL
);

-- Sample records stored:
+--------------------------------------+------------------+----------------+---------------+-----------------+
| id                                   | name             | customer_since | customer_type | loyalty_points  |
+--------------------------------------+------------------+----------------+---------------+-----------------+
| d3e4f5a6-b7c8-9012-zabc-678901uvwxyz | John's Retail    | 2023-05-15     | premium       | 1250            |
| e4f5a6b7-c8d9-0123-abcd-789012vwxyza | Smith Family     | 2023-09-30     | regular       | 450             |
| f5a6b7c8-d9e0-1234-bcde-890123wxyzab | Williams Trading | 2024-01-10     | vip           | 3200            |
| a6b7c8d9-e0f1-2345-cdef-901234xyzabc | Brown Enterprises| 2022-12-01     | premium       | 2100            |
| b7c8d9e0-f1a2-3456-defg-012345yzabcd | Jones & Assoc    | 2023-07-18     | regular       | 780             |
+--------------------------------------+------------------+----------------+---------------+-----------------+
```

### 8. **ContactAssociation Table** (`app_contactassociation`) - The Critical Junction Table

This is the most important table for shared contacts:

```sql
CREATE TABLE app_contactassociation (
    id CHAR(32) PRIMARY KEY,
    contact_id CHAR(32) NOT NULL,
    entity_content_type_id INTEGER NOT NULL,  -- References django_content_type
    entity_object_id INTEGER NOT NULL,        -- The actual record ID (NOTE: This is INTEGER, not UUID!)
    role VARCHAR(50) NOT NULL,
    is_active BOOLEAN NOT NULL,
    start_date DATE,
    end_date DATE,
    job_title VARCHAR(200),
    department VARCHAR(200),
    notes TEXT,
    created_at DATETIME NOT NULL,
    updated_at DATETIME NOT NULL,
    FOREIGN KEY (contact_id) REFERENCES app_contact(id) ON DELETE CASCADE,
    FOREIGN KEY (entity_content_type_id) REFERENCES django_content_type(id),
    UNIQUE KEY unique_association (contact_id, entity_content_type_id, entity_object_id, role)
);

-- django_content_type table (Django built-in)
-- +----+---------------------+-------------+
-- | id | app_label           | model       |
-- +----+---------------------+-------------+
-- | 7  | app                 | employee    |
-- | 8  | app                 | client      |
-- | 9  | app                 | customer    |
-- +----+---------------------+-------------+

-- Sample records stored in app_contactassociation:
+--------------------------------------+--------------------------------------+------------------------+------------------+-----------+-----------+------------+------------+------------------------+------------------+--------------------------------------------------+---------------------+
| id                                   | contact_id                           | entity_content_type_id | entity_object_id | role      | is_active | start_date | end_date    | job_title              | department       | notes                                            | created_at          |
+--------------------------------------+--------------------------------------+------------------------+------------------+-----------+-----------+------------+------------+------------------------+------------------+--------------------------------------------------+---------------------+
| c8d9e0f1-a2b3-4567-efgh-123456abcdef | 550e8400-e29b-41d4-a716-446655440000 | 7                      | 1                | primary   | 1         | 2020-01-15 | NULL       | Senior Developer       | Engineering      | Employee contact record for John Smith           | 2024-01-15 10:35:00 |
| d9e0f1a2-b3c4-5678-fghi-234567bcdefg | 550e8400-e29b-41d4-a716-446655440000 | 8                      | 1                | technical | 1         | 2023-01-15 | NULL       | Technical Consultant   | Client Services  | Technical contact for TechCorp Solutions         | 2024-01-15 10:35:01 |
| e0f1a2b3-c4d5-6789-ghij-345678cdefgh | 550e8400-e29b-41d4-a716-446655440000 | 9                      | 1                | primary   | 1         | 2023-05-15 | NULL       | Customer Owner         | N/A              | Primary customer contact                         | 2024-01-15 10:35:02 |
| f1a2b3c4-d5e6-7890-hijk-456789defghi | 6ba7b810-9dad-11d1-80b4-00c04fd430c8 | 8                      | 2                | billing   | 1         | 2023-03-20 | NULL       | Billing Specialist     | Finance          | Handles all billing inquiries                    | 2024-01-15 10:35:03 |
| a2b3c4d5-e6f7-8901-ijkl-567890efghij | 6ba7b810-9dad-11d1-80b4-00c04fd430c8 | 7                      | 2                | manager   | 1         | 2021-03-10 | NULL       | Sales Manager          | Sales            | Team lead for sales department                   | 2024-01-15 10:35:04 |
| b3c4d5e6-f7a8-9012-jklm-678901fghijk | 6ba7b810-9dad-11d1-80b4-00c04fd430c8 | 9                      | 3                | primary   | 0         | 2024-01-10 | 2024-03-15  | Former Customer        | N/A              | Customer relationship ended                      | 2024-01-15 10:35:05 |
+--------------------------------------+--------------------------------------+------------------------+------------------+-----------+-----------+------------+------------+------------------------+------------------+--------------------------------------------------+---------------------+
```

## CRITICAL NOTE About UUID and GenericForeignKey

**⚠️ Important Limitation**: Django's `GenericForeignKey` requires `PositiveIntegerField` for `object_id`, which cannot store UUIDs. This creates a mismatch if your Employee/Client/Customer models use UUID primary keys.

### Solution: Store both Integer and UUID references

```python
# Enhanced ContactAssociation model to handle UUIDs
class ContactAssociation(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    contact = models.ForeignKey(Contact, on_delete=models.CASCADE, related_name='associations')
    
    # For GenericForeignKey (requires integer)
    entity_content_type = models.ForeignKey(ContentType, on_delete=models.CASCADE)
    entity_object_id = models.PositiveIntegerField()  # Stores integer ID
    entity = GenericForeignKey('entity_content_type', 'entity_object_id')
    
    # ALSO store the UUID for direct reference (for performance)
    entity_uuid = models.UUIDField(null=True, blank=True, db_index=True)
    
    # Rest of fields...
    
    def save(self, *args, **kwargs):
        # Store the UUID if the entity has one
        if self.entity and hasattr(self.entity, 'id'):
            if isinstance(self.entity.id, uuid.UUID):
                self.entity_uuid = self.entity.id
        super().save(*args, **kwargs)
```

### Alternative: Use Integer Primary Keys for Business Models

If you must use UUIDs for business models, consider this workaround:

```sql
-- Add a surrogate integer ID alongside UUID
CREATE TABLE app_employee (
    id INTEGER PRIMARY KEY AUTO_INCREMENT,  -- For GenericForeignKey
    uuid CHAR(32) UNIQUE NOT NULL,          -- Your UUID for external use
    employee_id VARCHAR(50) NOT NULL UNIQUE,
    -- other fields...
);
```

## Visual Representation of Data Relationships

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           DATABASE RELATIONSHIP DIAGRAM                       │
└─────────────────────────────────────────────────────────────────────────────┘

                                    ┌──────────────────┐
                                    │   Contact        │
                                    │  ┌────────────┐  │
                                    │  │ id (UUID)  │  │
                                    │  │ first_name │  │
                                    │  │ last_name  │  │
                                    │  └────────────┘  │
                                    └────────┬─────────┘
                                             │
                    ┌────────────────────────┼────────────────────────┐
                    │                        │                        │
                    ▼                        ▼                        ▼
         ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
         │   Address        │    │   Phone          │    │   Email          │
         │  ┌────────────┐  │    │  ┌────────────┐  │    │  ┌────────────┐  │
         │  │ contact_id │  │    │  │ contact_id │  │    │  │ contact_id │  │
         │  │ street     │  │    │  │ number     │  │    │  │ email      │  │
         │  │ city       │  │    │  │ type       │  │    │  │ type       │  │
         │  └────────────┘  │    │  └────────────┘  │    │  └────────────┘  │
         └──────────────────┘    └──────────────────┘    └──────────────────┘

                                    │
                                    │
                    ┌───────────────┼───────────────┐
                    │               │               │
                    ▼               ▼               ▼
         ┌─────────────────────────────────────────────────────┐
         │          ContactAssociation (Junction Table)         │
         │  ┌──────────────────────────────────────────────┐   │
         │  │ contact_id (UUID)                             │   │
         │  │ entity_content_type_id (FK to ContentType)   │   │
         │  │ entity_object_id (INTEGER - NOT UUID!!!)      │   │
         │  │ role, is_active, start_date, end_date         │   │
         │  │ job_title, department, notes                  │   │
         │  └──────────────────────────────────────────────┘   │
         └──────────────┬──────────────┬──────────────┬────────┘
                        │              │              │
                        ▼              ▼              ▼
              ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
              │  Employee   │  │   Client    │  │  Customer   │
              │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │
              │ │id (INT) │ │  │ │id (INT) │ │  │ │id (INT) │ │
              │ │uuid     │ │  │ │uuid     │ │  │ │uuid     │ │
              │ │emp_id   │ │  │ │company  │ │  │ │name     │ │
              │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │
              └─────────────┘  └─────────────┘  └─────────────┘
```

## Example: How John Smith is Stored as a Shared Contact

Here's the actual data showing how John Smith appears as Employee, Client Contact, and Customer:

```sql
-- 1. John Smith in Contact table
INSERT INTO app_contact VALUES (
    '550e8400-e29b-41d4-a716-446655440000', 
    'John', 'Smith', 'A.', '1985-03-15', 'M',
    'Multi-role contact: Employee, Client Contact, and Customer',
    '2024-01-15 10:30:00', '2024-01-15 10:30:00'
);

-- 2. John's contact details
INSERT INTO app_address VALUES (
    'a1b2c3d4-e5f6-7890-abcd-ef1234567890',
    '550e8400-e29b-41d4-a716-446655440000',
    'home', '742 Evergreen Terrace', 'Apt 4B', 'Springfield', 'IL', '62701', 'USA',
    1, 1, 39.781721, -89.650148
);

INSERT INTO app_phone VALUES (
    'f6a7b8c9-d0e1-2345-fghi-678901abcdef',
    '550e8400-e29b-41d4-a716-446655440000',
    'mobile', '+1-212-555-0123', NULL, 1, 1
);

INSERT INTO app_email VALUES (
    'e1f2a3b4-c5d6-7890-klmn-123456fghijk',
    '550e8400-e29b-41d4-a716-446655440000',
    'work', 'john.smith@company.com', 1, 1, 1
);

-- 3. John as EMPLOYEE (via ContactAssociation)
-- First, Employee record:
INSERT INTO app_employee VALUES (
    1,  -- Integer ID for GenericForeignKey
    'EMP12345', '2020-01-15', 'Engineering', 'Senior Developer', 95000.00, '2024-01-15 10:30:00'
);

-- Association record linking John to Employee:
INSERT INTO app_contactassociation VALUES (
    'c8d9e0f1-a2b3-4567-efgh-123456abcdef',
    '550e8400-e29b-41d4-a716-446655440000',  -- John's contact UUID
    7,   -- ContentType ID for Employee model
    1,   -- Employee record ID (Integer)
    'primary', 1, '2020-01-15', NULL,
    'Senior Developer', 'Engineering',
    'Employee contact record',
    '2024-01-15 10:35:00', '2024-01-15 10:35:00'
);

-- 4. John as CLIENT CONTACT
-- Client record:
INSERT INTO app_client VALUES (
    1,  -- Integer ID for GenericForeignKey
    'TechCorp Solutions', '2023-01-15', 'enterprise', 50000.00, 45
);

-- Association record linking John to Client:
INSERT INTO app_contactassociation VALUES (
    'd9e0f1a2-b3c4-5678-fghi-234567bcdefg',
    '550e8400-e29b-41d4-a716-446655440000',  -- John's contact UUID
    8,   -- ContentType ID for Client model
    1,   -- Client record ID (Integer)
    'technical', 1, '2023-01-15', NULL,
    'Technical Consultant', 'Client Services',
    'Technical contact for TechCorp Solutions',
    '2024-01-15 10:35:01', '2024-01-15 10:35:01'
);

-- 5. John as CUSTOMER
-- Customer record:
INSERT INTO app_customer VALUES (
    1,  -- Integer ID for GenericForeignKey
    "John's Retail", '2023-05-15', 'premium', 1250
);

-- Association record linking John to Customer:
INSERT INTO app_contactassociation VALUES (
    'e0f1a2b3-c4d5-6789-ghij-345678cdefgh',
    '550e8400-e29b-41d4-a716-446655440000',  -- John's contact UUID
    9,   -- ContentType ID for Customer model
    1,   -- Customer record ID (Integer)
    'primary', 1, '2023-05-15', NULL,
    'Customer Owner', 'N/A',
    'Primary customer contact',
    '2024-01-15 10:35:02', '2024-01-15 10:35:02'
);
```

## Query Examples to See Stored Data

```sql
-- Find all contacts for Employee #1
SELECT c.* FROM app_contact c
JOIN app_contactassociation ca ON c.id = ca.contact_id
WHERE ca.entity_content_type_id = 7  -- Employee content type
  AND ca.entity_object_id = 1;       -- Employee ID 1

-- Find all entities associated with John Smith
SELECT 
    c.first_name, c.last_name,
    CASE ca.entity_content_type_id
        WHEN 7 THEN 'Employee'
        WHEN 8 THEN 'Client'
        WHEN 9 THEN 'Customer'
    END as entity_type,
    ca.role,
    ca.job_title,
    ca.is_active
FROM app_contact c
JOIN app_contactassociation ca ON c.id = ca.contact_id
WHERE c.id = '550e8400-e29b-41d4-a716-446655440000';

-- Results:
+------------+-----------+-------------+-----------+---------------------+-----------+
| first_name | last_name | entity_type | role      | job_title           | is_active |
+------------+-----------+-------------+-----------+---------------------+-----------+
| John       | Smith     | Employee    | primary   | Senior Developer    | 1         |
| John       | Smith     | Client      | technical | Technical Consultant| 1         |
| John       | Smith     | Customer    | primary   | Customer Owner      | 1         |
+------------+-----------+-------------+-----------+---------------------+-----------+
```

## Summary of How Data is Stored

1. **Base Contact Data**: One row in `app_contact` table per person
2. **Contact Details**: Multiple rows in `app_address`, `app_phone`, `app_email` tables linked by `contact_id`
3. **Business Entities**: Separate tables for `app_employee`, `app_client`, `app_customer` (using Integer PKs for GenericForeignKey compatibility)
4. **Associations**: The `app_contactassociation` table creates the many-to-many relationships, storing:
   - Which contact (UUID)
   - Which entity type (via ContentType)
   - Which specific entity (by its Integer ID)
   - Role and context metadata

**Key Insight**: The same contact UUID can appear in multiple association records, linking to different entity types and IDs, enabling a single person to be an employee, client contact, and customer simultaneously in the database.
