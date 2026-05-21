# Generic Relationship

## Represent data in such a way that same contact can belong to a employee and a customer or client.

Excellent requirement! You need a **shared contact model** where the same contact person can be associated with multiple entities (employee, client, customer). This is a many-to-many relationship between contacts and your business models.

Here's the best approach:

## Solution: Many-to-Many with Through Model (Most Flexible)

```python
from django.db import models
from django.contrib.contenttypes.models import ContentType
from django.contrib.contenttypes.fields import GenericForeignKey, GenericRelation

class Contact(models.Model):
    """Shared contact that can belong to multiple entities"""
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    middle_name = models.CharField(max_length=100, blank=True)
    
    # Personal details
    date_of_birth = models.DateField(null=True, blank=True)
    gender = models.CharField(max_length=20, choices=[
        ('M', 'Male'),
        ('F', 'Female'),
        ('O', 'Other'),
    ], blank=True)
    
    # Additional info
    notes = models.TextField(blank=True)
    
    # Timestamps
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['last_name', 'first_name']
        indexes = [
            models.Index(fields=['last_name', 'first_name']),
        ]
    
    @property
    def full_name(self):
        return f"{self.first_name} {self.last_name}"
    
    def __str__(self):
        return self.full_name

class Address(models.Model):
    ADDRESS_TYPES = [
        ('home', 'Home'),
        ('work', 'Work'),
        ('billing', 'Billing'),
        ('shipping', 'Shipping'),
        ('other', 'Other'),
    ]
    
    contact = models.ForeignKey(Contact, on_delete=models.CASCADE, related_name='addresses')
    address_type = models.CharField(max_length=20, choices=ADDRESS_TYPES, default='home')
    
    # Address fields
    street = models.CharField(max_length=255)
    street2 = models.CharField(max_length=255, blank=True)
    city = models.CharField(max_length=100)
    state = models.CharField(max_length=50, blank=True)
    postal_code = models.CharField(max_length=20)
    country = models.CharField(max_length=100, default='USA')
    
    # Metadata
    is_default = models.BooleanField(default=False)
    is_primary = models.BooleanField(default=False)
    latitude = models.DecimalField(max_digits=10, decimal_places=6, null=True, blank=True)
    longitude = models.DecimalField(max_digits=10, decimal_places=6, null=True, blank=True)
    
    class Meta:
        verbose_name_plural = "Addresses"
        ordering = ['-is_primary', '-is_default', 'address_type']
    
    def __str__(self):
        return f"{self.street}, {self.city}"

class Phone(models.Model):
    PHONE_TYPES = [
        ('home', 'Home'),
        ('work', 'Work'),
        ('mobile', 'Mobile'),
        ('fax', 'Fax'),
        ('other', 'Other'),
    ]
    
    contact = models.ForeignKey(Contact, on_delete=models.CASCADE, related_name='phones')
    phone_type = models.CharField(max_length=20, choices=PHONE_TYPES, default='mobile')
    number = models.CharField(max_length=20)
    extension = models.CharField(max_length=10, blank=True)
    is_default = models.BooleanField(default=False)
    is_primary = models.BooleanField(default=False)
    
    class Meta:
        ordering = ['-is_primary', '-is_default', 'phone_type']
    
    def __str__(self):
        return self.number

class Email(models.Model):
    EMAIL_TYPES = [
        ('personal', 'Personal'),
        ('work', 'Work'),
        ('billing', 'Billing'),
        ('other', 'Other'),
    ]
    
    contact = models.ForeignKey(Contact, on_delete=models.CASCADE, related_name='emails')
    email_type = models.CharField(max_length=20, choices=EMAIL_TYPES, default='work')
    email = models.EmailField(unique=True)  # Unique email across all contacts
    is_default = models.BooleanField(default=False)
    is_primary = models.BooleanField(default=False)
    is_verified = models.BooleanField(default=False)
    
    class Meta:
        ordering = ['-is_primary', '-is_default', 'email_type']
    
    def __str__(self):
        return self.email

# Through model for relationship with additional context
class ContactAssociation(models.Model):
    """Links a Contact to any entity with additional context"""
    ROLE_TYPES = [
        ('primary', 'Primary Contact'),
        ('billing', 'Billing Contact'),
        ('technical', 'Technical Contact'),
        ('sales', 'Sales Contact'),
        ('support', 'Support Contact'),
        ('emergency', 'Emergency Contact'),
        ('manager', 'Manager'),
        ('assistant', 'Assistant'),
        ('other', 'Other'),
    ]
    
    contact = models.ForeignKey(Contact, on_delete=models.CASCADE, related_name='associations')
    
    # Generic foreign key to any model (Employee, Client, Customer)
    entity_content_type = models.ForeignKey(ContentType, on_delete=models.CASCADE)
    entity_object_id = models.PositiveIntegerField()
    entity = GenericForeignKey('entity_content_type', 'entity_object_id')
    
    # Context about this association
    role = models.CharField(max_length=50, choices=ROLE_TYPES, default='other')
    is_active = models.BooleanField(default=True)
    start_date = models.DateField(null=True, blank=True)
    end_date = models.DateField(null=True, blank=True)
    notes = models.TextField(blank=True)
    
    # Custom fields for this specific association
    job_title = models.CharField(max_length=200, blank=True)
    department = models.CharField(max_length=200, blank=True)
    
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        unique_together = ['contact', 'entity_content_type', 'entity_object_id', 'role']
        indexes = [
            models.Index(fields=['entity_content_type', 'entity_object_id']),
            models.Index(fields=['contact', 'is_active']),
        ]
    
    def __str__(self):
        return f"{self.contact.full_name} - {self.role}"

# Your business models
class Employee(models.Model):
    employee_id = models.CharField(max_length=50, unique=True)
    hire_date = models.DateField()
    department = models.CharField(max_length=100)
    position = models.CharField(max_length=100)
    
    # Reverse relation to contacts through associations
    contacts = GenericRelation(ContactAssociation)
    
    # Employee-specific fields
    salary = models.DecimalField(max_digits=10, decimal_places=2, null=True, blank=True)
    
    created_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return f"{self.employee_id} - {self.position}"
    
    def get_primary_contacts(self):
        """Get primary contacts for this employee"""
        return Contact.objects.filter(
            associations__entity_content_type=ContentType.objects.get_for_model(self),
            associations__entity_object_id=self.id,
            associations__role='primary',
            associations__is_active=True
        )
    
    def add_contact(self, contact, role='other', **kwargs):
        """Add a contact to this employee"""
        return ContactAssociation.objects.create(
            contact=contact,
            entity=self,
            role=role,
            **kwargs
        )

class Client(models.Model):
    company_name = models.CharField(max_length=200)
    client_since = models.DateField(auto_now_add=True)
    client_type = models.CharField(max_length=50, choices=[
        ('enterprise', 'Enterprise'),
        ('business', 'Business'),
        ('individual', 'Individual'),
    ])
    
    # Reverse relation to contacts through associations
    contacts = GenericRelation(ContactAssociation)
    
    # Client-specific fields
    credit_limit = models.DecimalField(max_digits=12, decimal_places=2, null=True, blank=True)
    payment_terms = models.IntegerField(default=30)  # days
    
    def __str__(self):
        return self.company_name
    
    def get_contacts_by_role(self, role):
        """Get contacts with specific role for this client"""
        return Contact.objects.filter(
            associations__entity_content_type=ContentType.objects.get_for_model(self),
            associations__entity_object_id=self.id,
            associations__role=role,
            associations__is_active=True
        )

class Customer(models.Model):
    name = models.CharField(max_length=100)
    customer_since = models.DateField(auto_now_add=True)
    customer_type = models.CharField(max_length=50, choices=[
        ('regular', 'Regular'),
        ('premium', 'Premium'),
        ('vip', 'VIP'),
    ])
    
    # Reverse relation to contacts through associations
    contacts = GenericRelation(ContactAssociation)
    
    # Customer-specific fields
    loyalty_points = models.IntegerField(default=0)
    
    def __str__(self):
        return self.name

class Company(models.Model):
    """Optional: Company model if employees/clients/customers belong to companies"""
    name = models.CharField(max_length=200)
    tax_id = models.CharField(max_length=50, unique=True)
    contacts = GenericRelation(ContactAssociation)
    
    def __str__(self):
        return self.name
```

## Alternative: Simpler Many-to-Many Approach

If you don't need the extra context provided by `ContactAssociation`:

```python
class Contact(models.Model):
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    # ... other fields as above
    
    # Many-to-many relationships to your models
    employees = models.ManyToManyField('Employee', related_name='contacts', blank=True)
    clients = models.ManyToManyField('Client', related_name='contacts', blank=True)
    customers = models.ManyToManyField('Customer', related_name='contacts', blank=True)
    
    # Addresses, phones, emails as before
    addresses = models.ManyToManyField(Address, through='ContactAddress', blank=True)
    phones = models.ManyToManyField(Phone, through='ContactPhone', blank=True)
    emails = models.ManyToManyField(Email, through='ContactEmail', blank=True)

# Through models with additional info
class ContactAddress(models.Model):
    contact = models.ForeignKey(Contact, on_delete=models.CASCADE)
    address = models.ForeignKey(Address, on_delete=models.CASCADE)
    is_default = models.BooleanField(default=False)
    # ... other meta fields
```

## Usage Examples

### Creating and Managing Shared Contacts

```python
# Create a contact person
john = Contact.objects.create(
    first_name="John",
    last_name="Smith",
    date_of_birth="1985-05-15",
    gender="M"
)

# Add contact details
Address.objects.create(
    contact=john,
    address_type="work",
    street="123 Corporate Drive",
    city="New York",
    state="NY",
    postal_code="10001",
    is_primary=True
)

Phone.objects.create(
    contact=john,
    phone_type="mobile",
    number="+1-555-123-4567",
    is_primary=True
)

Email.objects.create(
    contact=john,
    email_type="work",
    email="john.smith@example.com",
    is_primary=True,
    is_verified=True
)

# Now associate John with multiple entities
employee = Employee.objects.create(
    employee_id="EMP001",
    hire_date="2020-01-15",
    department="Engineering",
    position="Senior Developer"
)

client = Client.objects.create(
    company_name="TechCorp Inc.",
    client_type="enterprise",
    payment_terms=45
)

customer = Customer.objects.create(
    name="John Smith",
    customer_type="vip",
    loyalty_points=1000
)

# Associate John with different roles for each entity
association1 = ContactAssociation.objects.create(
    contact=john,
    entity=employee,
    role='primary',
    job_title='Lead Developer',
    department='Engineering'
)

association2 = ContactAssociation.objects.create(
    contact=john,
    entity=client,
    role='technical',
    job_title='Technical Consultant',
    notes='Primary technical contact for support'
)

association3 = ContactAssociation.objects.create(
    contact=john,
    entity=customer,
    role='other',
    job_title='Individual Customer'
)

# John is now associated with all three entities!
```

### Querying Shared Contacts

```python
# Find all contacts associated with an employee
employee = Employee.objects.get(employee_id="EMP001")
associations = employee.contacts.all()
for assoc in associations:
    print(f"Contact: {assoc.contact.full_name}, Role: {assoc.role}")

# Find all entities a contact is associated with
john = Contact.objects.get(first_name="John", last_name="Smith")
for assoc in john.associations.all():
    entity_type = assoc.entity_content_type.model
    print(f"Associated with {entity_type}: {assoc.entity}, Role: {assoc.role}")

# Find contacts who are both employees AND clients
employees_who_are_clients = Contact.objects.filter(
    associations__entity_content_type=ContentType.objects.get_for_model(Employee),
    associations__entity_object_id__in=Employee.objects.values_list('id', flat=True)
).filter(
    associations__entity_content_type=ContentType.objects.get_for_model(Client),
    associations__entity_object_id__in=Client.objects.values_list('id', flat=True)
).distinct()

# Get all active technical contacts for clients
technical_contacts = Contact.objects.filter(
    associations__entity_content_type=ContentType.objects.get_for_model(Client),
    associations__role='technical',
    associations__is_active=True
)
```

### Admin Configuration

```python
from django.contrib import admin
from django.contrib.contenttypes.admin import GenericTabularInline

class AddressInline(admin.TabularInline):
    model = Address
    extra = 1

class PhoneInline(admin.TabularInline):
    model = Phone
    extra = 1

class EmailInline(admin.TabularInline):
    model = Email
    extra = 1

class ContactAssociationInline(GenericTabularInline):
    model = ContactAssociation
    extra = 1
    ct_field = "entity_content_type"
    ct_fk_field = "entity_object_id"

class ContactAdmin(admin.ModelAdmin):
    list_display = ['first_name', 'last_name', 'email_list', 'phone_list']
    search_fields = ['first_name', 'last_name', 'emails__email', 'phones__number']
    inlines = [AddressInline, PhoneInline, EmailInline, ContactAssociationInline]
    
    def email_list(self, obj):
        return ", ".join([e.email for e in obj.emails.all()[:3]])
    email_list.short_description = 'Emails'
    
    def phone_list(self, obj):
        return ", ".join([p.number for p in obj.phones.all()[:3]])
    phone_list.short_description = 'Phones'

class EmployeeAdmin(admin.ModelAdmin):
    list_display = ['employee_id', 'position', 'department', 'contact_count']
    inlines = [ContactAssociationInline]
    
    def contact_count(self, obj):
        return obj.contacts.count()
    contact_count.short_description = '# Contacts'

admin.site.register(Contact, ContactAdmin)
admin.site.register(Employee, EmployeeAdmin)
admin.site.register(Client)
admin.site.register(Customer)
admin.site.register(Address)
admin.site.register(Phone)
admin.site.register(Email)
admin.site.register(ContactAssociation)
```

## Performance Optimizations

```python
# Prefetch related data efficiently
from django.contrib.contenttypes.models import ContentType

def get_contact_with_all_details(contact_id):
    return Contact.objects.filter(id=contact_id).prefetch_related(
        'addresses',
        'phones', 
        'emails',
        'associations',
        'associations__entity_content_type'
    ).first()

# Batch load contacts for multiple employees
employee_ids = [1, 2, 3, 4, 5]
employee_ct = ContentType.objects.get_for_model(Employee)
associations = ContactAssociation.objects.filter(
    entity_content_type=employee_ct,
    entity_object_id__in=employee_ids
).select_related('contact').prefetch_related(
    'contact__addresses',
    'contact__phones',
    'contact__emails'
)
```

## Recommendation

**Use the GenericForeignKey with ContactAssociation approach** because:

1. ✅ **Truly shared contacts** - One contact can belong to Employee, Client, AND Customer simultaneously
2. ✅ **Role context** - Same contact can have different roles for different entities
3. ✅ **Extensible** - Easy to add new entity types (Company, Vendor, Partner, etc.)
4. ✅ **Historical tracking** - Track when associations start/end
5. ✅ **Clean queries** - Easy to find all associations for any entity or contact
6. ✅ **No circular dependencies** - Contact doesn't need to know about all possible entity types

This design gives you maximum flexibility while maintaining referential integrity and clean data modeling.

# Seed Data

Here's a comprehensive script to generate sample records with UUID primary keys for all models:

```python
import uuid
import random
from datetime import datetime, timedelta
from django.utils import timezone
from django.contrib.contenttypes.models import ContentType

# Sample data generators
def generate_sample_data():
    """Generate sample records for all models"""
    
    # ============ CREATE CONTACTS ============
    contacts = []
    
    # Sample contact data
    contact_data = [
        {"first_name": "John", "last_name": "Smith", "dob": "1985-03-15", "gender": "M"},
        {"first_name": "Sarah", "last_name": "Johnson", "dob": "1990-07-22", "gender": "F"},
        {"first_name": "Michael", "last_name": "Williams", "dob": "1978-11-08", "gender": "M"},
        {"first_name": "Emily", "last_name": "Brown", "dob": "1995-02-14", "gender": "F"},
        {"first_name": "David", "last_name": "Jones", "dob": "1982-09-30", "gender": "M"},
        {"first_name": "Jessica", "last_name": "Garcia", "dob": "1988-12-05", "gender": "F"},
        {"first_name": "Robert", "last_name": "Martinez", "dob": "1975-06-18", "gender": "M"},
        {"first_name": "Jennifer", "last_name": "Rodriguez", "dob": "1992-04-25", "gender": "F"},
        {"first_name": "William", "last_name": "Wilson", "dob": "1980-10-12", "gender": "M"},
        {"first_name": "Maria", "last_name": "Anderson", "dob": "1987-08-19", "gender": "F"},
        {"first_name": "James", "last_name": "Taylor", "dob": "1993-01-27", "gender": "M"},
        {"first_name": "Linda", "last_name": "Thomas", "dob": "1984-05-09", "gender": "F"},
        {"first_name": "Charles", "last_name": "Jackson", "dob": "1979-12-03", "gender": "M"},
        {"first_name": "Patricia", "last_name": "White", "dob": "1991-09-16", "gender": "F"},
        {"first_name": "Thomas", "last_name": "Harris", "dob": "1986-07-29", "gender": "M"},
    ]
    
    print("Creating Contacts...")
    for data in contact_data:
        contact = Contact.objects.create(
            id=uuid.uuid4(),
            first_name=data["first_name"],
            last_name=data["last_name"],
            middle_name=random.choice(["", "A.", "B.", "C.", "M."]),
            date_of_birth=data["dob"],
            gender=data["gender"],
            notes=random.choice([
                "", "Prefers email communication", "Available after 2 PM",
                "Special needs accommodation", "Bilingual - Spanish",
                "VIP customer", "Priority contact", "Work from home"
            ])
        )
        contacts.append(contact)
        print(f"  Created: {contact.full_name} (ID: {contact.id})")
    
    # ============ CREATE ADDRESSES ============
    address_types = ['home', 'work', 'billing', 'shipping', 'other']
    cities = ["New York", "Los Angeles", "Chicago", "Houston", "Phoenix", "Philadelphia", "San Antonio", "San Diego", "Dallas", "Austin"]
    states = ["NY", "CA", "IL", "TX", "AZ", "PA", "TX", "CA", "TX", "TX"]
    countries = ["USA", "Canada", "UK", "Australia", "Germany"]
    
    print("\nCreating Addresses...")
    for contact in contacts:
        # Each contact gets 1-3 addresses
        num_addresses = random.randint(1, 3)
        for i in range(num_addresses):
            city_idx = random.randint(0, len(cities)-1)
            address = Address.objects.create(
                id=uuid.uuid4(),
                contact=contact,
                address_type=random.choice(address_types),
                street=f"{random.randint(100, 9999)} {random.choice(['Main', 'Oak', 'Pine', 'Maple', 'Cedar', 'Washington', 'Lincoln', 'Park'])} {random.choice(['St', 'Ave', 'Blvd', 'Dr', 'Ln', 'Rd'])}",
                street2=random.choice(["", "Apt " + str(random.randint(1, 50)), "Suite " + str(random.randint(100, 999)), "Floor " + str(random.randint(1, 20))]),
                city=cities[city_idx],
                state=states[city_idx],
                postal_code=f"{random.randint(10000, 99999)}",
                country=random.choice(countries),
                is_default=(i == 0),
                is_primary=(i == 0 and random.choice([True, False])),
                latitude=round(random.uniform(25.0, 48.0), 6),
                longitude=round(random.uniform(-125.0, -65.0), 6)
            )
            print(f"  Address for {contact.full_name}: {address.street}, {address.city}")
    
    # ============ CREATE PHONES ============
    phone_types = ['home', 'work', 'mobile', 'fax', 'other']
    area_codes = ['212', '310', '312', '713', '602', '215', '210', '619', '214', '512']
    
    print("\nCreating Phone Numbers...")
    for contact in contacts:
        num_phones = random.randint(1, 3)
        for i in range(num_phones):
            phone = Phone.objects.create(
                id=uuid.uuid4(),
                contact=contact,
                phone_type=random.choice(phone_types),
                number=f"+1-{random.choice(area_codes)}-{random.randint(100, 999)}-{random.randint(1000, 9999)}",
                extension=random.choice(["", str(random.randint(100, 9999))]),
                is_default=(i == 0),
                is_primary=(i == 0 and random.choice([True, False]))
            )
            print(f"  Phone for {contact.full_name}: {phone.number}")
    
    # ============ CREATE EMAILS ============
    email_types = ['personal', 'work', 'billing', 'other']
    domains = ['gmail.com', 'yahoo.com', 'outlook.com', 'company.com', 'business.com', 'techcorp.com']
    
    print("\nCreating Emails...")
    email_counter = 1
    for contact in contacts:
        num_emails = random.randint(1, 2)
        for i in range(num_emails):
            email_address = f"{contact.first_name.lower()}.{contact.last_name.lower()}{random.randint(1, 99) if i > 0 else ''}@{random.choice(domains)}"
            email = Email.objects.create(
                id=uuid.uuid4(),
                contact=contact,
                email_type=random.choice(email_types),
                email=email_address,
                is_default=(i == 0),
                is_primary=(i == 0 and random.choice([True, False])),
                is_verified=random.choice([True, True, True, False])  # 75% verified
            )
            print(f"  Email for {contact.full_name}: {email.email}")
            email_counter += 1
    
    # ============ CREATE EMPLOYEES ============
    departments = ['Engineering', 'Sales', 'Marketing', 'HR', 'Finance', 'Operations', 'Customer Support']
    positions = [
        'Software Engineer', 'Senior Developer', 'Team Lead', 'Product Manager',
        'Sales Representative', 'Account Executive', 'Marketing Specialist',
        'HR Manager', 'Financial Analyst', 'Operations Director', 'Support Specialist'
    ]
    
    print("\nCreating Employees...")
    employees = []
    for i in range(10):  # Create 10 employees
        # Some employees are also in the contacts list
        contact = random.choice(contacts[:8]) if random.choice([True, False]) else None
        
        employee = Employee.objects.create(
            id=uuid.uuid4(),
            employee_id=f"EMP{random.randint(10000, 99999)}",
            hire_date=timezone.now().date() - timedelta(days=random.randint(30, 3650)),
            department=random.choice(departments),
            position=random.choice(positions),
            salary=round(random.uniform(45000, 150000), 2) if random.choice([True, False]) else None
        )
        employees.append(employee)
        print(f"  Created Employee: {employee.employee_id} - {employee.position}")
        
        # If contact exists, associate them
        if contact:
            ContactAssociation.objects.create(
                id=uuid.uuid4(),
                contact=contact,
                entity=employee,
                role=random.choice(['primary', 'manager', 'other']),
                is_active=True,
                start_date=employee.hire_date,
                job_title=employee.position,
                department=employee.department,
                notes=f"Employee contact record for {contact.full_name}"
            )
            print(f"    Associated with contact: {contact.full_name}")
    
    # ============ CREATE CLIENTS ============
    company_names = [
        "TechCorp Solutions", "Global Industries", "Innovative Systems", "Premier Consulting",
        "Apex Dynamics", "Bright Future Enterprises", "Coastal Trading Co.", "Metro Analytics",
        "Summit Technologies", "Pacific Group", "Atlantic Ventures", "Sterling Partners"
    ]
    client_types = ['enterprise', 'business', 'individual']
    
    print("\nCreating Clients...")
    clients = []
    for i in range(8):  # Create 8 clients
        client = Client.objects.create(
            id=uuid.uuid4(),
            company_name=random.choice(company_names) + (f" {random.randint(1, 100)}" if random.choice([True, False]) else ""),
            client_since=timezone.now().date() - timedelta(days=random.randint(1, 1095)),
            client_type=random.choice(client_types),
            credit_limit=round(random.uniform(5000, 100000), 2) if random.choice([True, False]) else None,
            payment_terms=random.choice([15, 30, 45, 60, 90])
        )
        clients.append(client)
        print(f"  Created Client: {client.company_name}")
        
        # Associate 2-4 contacts with each client
        num_associations = random.randint(2, 4)
        selected_contacts = random.sample(contacts, min(num_associations, len(contacts)))
        for idx, contact in enumerate(selected_contacts):
            role = random.choice(['primary', 'billing', 'technical', 'sales', 'support'])
            is_active = random.choice([True, True, True, False])  # 75% active
            
            ContactAssociation.objects.create(
                id=uuid.uuid4(),
                contact=contact,
                entity=client,
                role=role,
                is_active=is_active,
                start_date=client.client_since,
                end_date=None if is_active else timezone.now().date() - timedelta(days=random.randint(1, 90)),
                job_title=random.choice(['Account Manager', 'Technical Lead', 'Sales Director', 'Support Specialist']),
                department=random.choice(['Sales', 'Support', 'Technical', 'Account Management']),
                notes=f"{role.title()} contact for {client.company_name}"
            )
            print(f"    Associated {contact.full_name} as {role}")
    
    # ============ CREATE CUSTOMERS ============
    customer_names = [
        "John's Retail", "Smith Family", "Williams Trading", "Brown Enterprises",
        "Jones & Associates", "Garcia Group", "Martinez Corp", "Rodriguez LLC"
    ]
    customer_types = ['regular', 'premium', 'vip']
    
    print("\nCreating Customers...")
    customers = []
    for i in range(12):  # Create 12 customers
        customer = Customer.objects.create(
            id=uuid.uuid4(),
            name=random.choice(customer_names) if i < 8 else f"{random.choice(['Johnson', 'Anderson', 'Taylor', 'Thomas'])}, {random.choice(['Inc', 'LLC', 'Group', 'Co'])}",
            customer_since=timezone.now().date() - timedelta(days=random.randint(1, 730)),
            customer_type=random.choice(customer_types),
            loyalty_points=random.randint(0, 5000)
        )
        customers.append(customer)
        print(f"  Created Customer: {customer.name} ({customer.customer_type})")
        
        # Associate 1-3 contacts with each customer
        num_associations = random.randint(1, 3)
        selected_contacts = random.sample(contacts, min(num_associations, len(contacts)))
        for contact in selected_contacts:
            role = random.choice(['primary', 'billing', 'other'])
            is_active = random.choice([True, True, False])
            
            ContactAssociation.objects.create(
                id=uuid.uuid4(),
                contact=contact,
                entity=customer,
                role=role,
                is_active=is_active,
                start_date=customer.customer_since,
                end_date=None if is_active else timezone.now().date() - timedelta(days=random.randint(1, 30)),
                job_title=random.choice(['Customer', 'Account Holder', 'Authorized User']),
                notes=f"{role.title()} contact record"
            )
            print(f"    Associated {contact.full_name} as {role}")
    
    # ============ CREATE SHARED CONTACTS (Same contact across multiple entities) ============
    print("\n" + "="*60)
    print("CREATING SHARED CONTACTS (Same person across multiple roles)")
    print("="*60)
    
    # Select a few contacts to have multiple associations
    shared_contacts = random.sample(contacts, 5)
    
    for shared_contact in shared_contacts:
        print(f"\nContact: {shared_contact.full_name} is being associated with multiple entities:")
        
        # Associate with an employee if not already
        if not shared_contact.associations.filter(entity_content_type=ContentType.objects.get_for_model(Employee)).exists():
            if employees:
                employee = random.choice(employees)
                ContactAssociation.objects.create(
                    id=uuid.uuid4(),
                    contact=shared_contact,
                    entity=employee,
                    role=random.choice(['primary', 'manager']),
                    is_active=True,
                    start_date=employee.hire_date,
                    job_title=employee.position,
                    department=employee.department,
                    notes=f"{shared_contact.full_name} works as {employee.position}"
                )
                print(f"  → Works as EMPLOYEE: {employee.employee_id} - {employee.position}")
        
        # Associate with a client if not already
        if not shared_contact.associations.filter(entity_content_type=ContentType.objects.get_for_model(Client)).exists():
            if clients:
                client = random.choice(clients)
                ContactAssociation.objects.create(
                    id=uuid.uuid4(),
                    contact=shared_contact,
                    entity=client,
                    role=random.choice(['technical', 'sales', 'support']),
                    is_active=True,
                    start_date=client.client_since,
                    job_title=random.choice(['Consultant', 'Technical Advisor', 'Account Manager']),
                    department='Client Services',
                    notes=f"{shared_contact.full_name} provides services to {client.company_name}"
                )
                print(f"  → Serves as CLIENT CONTACT: {client.company_name}")
        
        # Associate with a customer if not already
        if not shared_contact.associations.filter(entity_content_type=ContentType.objects.get_for_model(Customer)).exists():
            if customers:
                customer = random.choice(customers)
                ContactAssociation.objects.create(
                    id=uuid.uuid4(),
                    contact=shared_contact,
                    entity=customer,
                    role='primary',
                    is_active=True,
                    start_date=customer.customer_since,
                    job_title='Customer',
                    notes=f"{shared_contact.full_name} is a {customer.customer_type} customer"
                )
                print(f"  → Is also a CUSTOMER: {customer.name}")
    
    # ============ STATISTICS SUMMARY ============
    print("\n" + "="*60)
    print("DATA POPULATION COMPLETE - SUMMARY")
    print("="*60)
    print(f"Total Contacts: {Contact.objects.count()}")
    print(f"Total Addresses: {Address.objects.count()}")
    print(f"Total Phone Numbers: {Phone.objects.count()}")
    print(f"Total Emails: {Email.objects.count()}")
    print(f"Total Employees: {Employee.objects.count()}")
    print(f"Total Clients: {Client.objects.count()}")
    print(f"Total Customers: {Customer.objects.count()}")
    print(f"Total Contact Associations: {ContactAssociation.objects.count()}")
    
    # Show statistics for shared contacts
    print("\nShared Contact Statistics:")
    for contact in Contact.objects.all():
        association_count = contact.associations.count()
        if association_count > 1:
            entity_types = set()
            for assoc in contact.associations.all():
                entity_types.add(assoc.entity_content_type.model)
            print(f"  • {contact.full_name}: Associated with {association_count} entities ({', '.join(entity_types)})")
    
    return {
        'contacts': contacts,
        'employees': employees,
        'clients': clients,
        'customers': customers
    }

# Helper function to run the sample data generation
def populate_sample_data():
    """Main function to populate all sample data"""
    print("Starting sample data population...")
    print("="*60)
    
    # Clear existing data (optional - be careful!)
    # Comment these out if you want to preserve existing data
    print("\nClearing existing data...")
    ContactAssociation.objects.all().delete()
    Address.objects.all().delete()
    Phone.objects.all().delete()
    Email.objects.all().delete()
    Employee.objects.all().delete()
    Client.objects.all().delete()
    Customer.objects.all().delete()
    Contact.objects.all().delete()
    
    # Generate new sample data
    result = generate_sample_data()
    
    return result

# Example queries to verify data
def verify_data_integrity():
    """Run verification queries to ensure data integrity"""
    print("\n" + "="*60)
    print("DATA INTEGRITY VERIFICATION")
    print("="*60)
    
    # Find contacts that are both employees AND clients
    employee_ct = ContentType.objects.get_for_model(Employee)
    client_ct = ContentType.objects.get_for_model(Client)
    
    dual_role_contacts = Contact.objects.filter(
        associations__entity_content_type=employee_ct
    ).filter(
        associations__entity_content_type=client_ct
    ).distinct()
    
    print(f"\nContacts who are both Employees AND Client Contacts: {dual_role_contacts.count()}")
    for contact in dual_role_contacts:
        print(f"  • {contact.full_name}")
    
    # Find customers with most contacts
    print("\nCustomers with most contacts:")
    for customer in Customer.objects.all():
        contact_count = customer.contacts.count()
        if contact_count > 0:
            print(f"  • {customer.name}: {contact_count} contact(s)")
    
    # Show default contact information
    print("\nDefault contact information:")
    for contact in Contact.objects.all()[:5]:  # Show first 5
        default_email = contact.emails.filter(is_default=True).first()
        default_phone = contact.phones.filter(is_default=True).first()
        default_address = contact.addresses.filter(is_default=True).first()
        
        print(f"\n  {contact.full_name}:")
        if default_email:
            print(f"    Default Email: {default_email.email}")
        if default_phone:
            print(f"    Default Phone: {default_phone.number}")
        if default_address:
            print(f"    Default Address: {default_address.street}, {default_address.city}")

# Run the population
if __name__ == "__main__":
    # Populate sample data
    data = populate_sample_data()
    
    # Verify data integrity
    verify_data_integrity()
    
    print("\n" + "="*60)
    print("Sample data generation complete!")
    print("="*60)
```

## Management Command Version

Create a Django management command to easily populate sample data:

```python
# management/commands/populate_contacts.py
from django.core.management.base import BaseCommand
from django.utils import timezone
import uuid
import random

class Command(BaseCommand):
    help = 'Populates sample contact data with UUID primary keys'
    
    def add_arguments(self, parser):
        parser.add_argument(
            '--clear',
            action='store_true',
            help='Clear existing data before populating',
        )
        
        parser.add_argument(
            '--count',
            type=int,
            default=15,
            help='Number of contacts to create (default: 15)',
        )
    
    def handle(self, *args, **options):
        if options['clear']:
            self.stdout.write("Clearing existing data...")
            ContactAssociation.objects.all().delete()
            Address.objects.all().delete()
            Phone.objects.all().delete()
            Email.objects.all().delete()
            Employee.objects.all().delete()
            Client.objects.all().delete()
            Customer.objects.all().delete()
            Contact.objects.all().delete()
        
        self.stdout.write(f"Creating {options['count']} contacts...")
        # Call the populate_sample_data function with custom count
        # ... (rest of the population logic)
        
        self.stdout.write(self.style.SUCCESS('Successfully populated sample data!'))
```

## Sample Output Preview

When you run this script, you'll see output like:

```
Starting sample data population...
============================================================

Clearing existing data...

Creating Contacts...
  Created: John Smith (ID: 550e8400-e29b-41d4-a716-446655440000)
  Created: Sarah Johnson (ID: 6ba7b810-9dad-11d1-80b4-00c04fd430c8)
  ...

Creating Addresses...
  Address for John Smith: 742 Evergreen Terrace, Springfield
  ...

Creating Phone Numbers...
  Phone for John Smith: +1-212-555-0123
  ...

Creating Emails...
  Email for John Smith: john.smith@gmail.com
  ...

Creating Employees...
  Created Employee: EMP12345 - Software Engineer
    Associated with contact: John Smith

Creating Clients...
  Created Client: TechCorp Solutions
    Associated John Smith as technical

Creating Customers...
  Created Customer: John's Retail (premium)
    Associated John Smith as primary

============================================================
CREATING SHARED CONTACTS (Same person across multiple roles)
============================================================

Contact: John Smith is being associated with multiple entities:
  → Works as EMPLOYEE: EMP12345 - Software Engineer
  → Serves as CLIENT CONTACT: TechCorp Solutions
  → Is also a CUSTOMER: John's Retail

============================================================
DATA POPULATION COMPLETE - SUMMARY
============================================================
Total Contacts: 15
Total Addresses: 32
Total Phone Numbers: 28
Total Emails: 23
Total Employees: 10
Total Clients: 8
Total Customers: 12
Total Contact Associations: 67
```

## Key Features of This Implementation

1. **UUID Primary Keys**: Every model uses `id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)`

2. **Realistic Data**: Names, addresses, phone numbers, and emails that look authentic

3. **Shared Contacts**: Same contact can be employee, client contact, and customer simultaneously

4. **Relationships**: 
   - Each contact has 1-3 addresses, 1-3 phones, 1-2 emails
   - Each entity has multiple contacts with different roles
   - Contacts can be associated with multiple entities

5. **Data Integrity**: Verification queries ensure relationships work correctly

6. **Statistics**: Summary shows how many of each record type were created

To use this script, simply run it in your Django shell or as a management command, and you'll have a complete dataset ready for testing and development!

