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
