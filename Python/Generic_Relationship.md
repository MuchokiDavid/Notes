## Generic Foreign Keys (Most Flexible)

This approach allows any model to have contacts with minimal coupling.

```python
from django.db import models
from django.contrib.contenttypes.models import ContentType
from django.contrib.contenttypes.fields import GenericForeignKey, GenericRelation

class Contact(models.Model):
    """Represents a contact person linked to any model (Employee, Client, Customer)"""
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    title = models.CharField(max_length=100, blank=True)
    department = models.CharField(max_length=100, blank=True)
    is_primary = models.BooleanField(default=False)
    notes = models.TextField(blank=True)
    
    # Generic foreign key to any model
    content_type = models.ForeignKey(ContentType, on_delete=models.CASCADE)
    object_id = models.PositiveIntegerField()
    content_object = GenericForeignKey('content_type', 'object_id')
    
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        indexes = [
            models.Index(fields=['content_type', 'object_id']),
        ]
        ordering = ['-is_primary', 'last_name', 'first_name']
    
    def __str__(self):
        return f"{self.first_name} {self.last_name}"

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
    street = models.CharField(max_length=255)
    city = models.CharField(max_length=100)
    state = models.CharField(max_length=50, blank=True)
    postal_code = models.CharField(max_length=20)
    country = models.CharField(max_length=100)
    is_default = models.BooleanField(default=False)
    
    class Meta:
        verbose_name_plural = "Addresses"
        ordering = ['-is_default', 'address_type']
    
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
    
    class Meta:
        ordering = ['-is_default', 'phone_type']
    
    def __str__(self):
        return self.number

class Email(models.Model):
    EMAIL_TYPES = [
        ('personal', 'Personal'),
        ('work', 'Work'),
        ('other', 'Other'),
    ]
    
    contact = models.ForeignKey(Contact, on_delete=models.CASCADE, related_name='emails')
    email_type = models.CharField(max_length=20, choices=EMAIL_TYPES, default='work')
    email = models.EmailField()
    is_default = models.BooleanField(default=False)
    
    class Meta:
        ordering = ['-is_default', 'email_type']
    
    def __str__(self):
        return self.email

# Your business models
class Employee(models.Model):
    name = models.CharField(max_length=100)
    employee_id = models.CharField(max_length=50, unique=True)
    contacts = GenericRelation(Contact)  # Allows reverse lookup
    # other employee fields...
    
    def __str__(self):
        return self.name

class Client(models.Model):
    company_name = models.CharField(max_length=200)
    contacts = GenericRelation(Contact)
    # other client fields...
    
    def __str__(self):
        return self.company_name

class Customer(models.Model):
    name = models.CharField(max_length=100)
    customer_since = models.DateField(auto_now_add=True)
    contacts = GenericRelation(Contact)
    # other customer fields...
    
    def __str__(self):
        return self.name
```

## Usage Examples

### Creating and Querying (Solution 1)

```python
# Create an employee and add a contact
employee = Employee.objects.create(name="John Smith", employee_id="EMP001")

# Add a primary contact
contact = Contact.objects.create(
    first_name="Jane",
    last_name="Doe",
    title="Assistant",
    is_primary=True,
    content_object=employee
)

# Add contact details
Address.objects.create(
    contact=contact,
    address_type="work",
    street="123 Business Ave",
    city="New York",
    state="NY",
    postal_code="10001",
    country="USA",
    is_default=True
)

Phone.objects.create(
    contact=contact,
    phone_type="mobile",
    number="+1-555-123-4567",
    is_default=True
)

Email.objects.create(
    contact=contact,
    email_type="work",
    email="jane.doe@company.com",
    is_default=True
)

# Querying
employee = Employee.objects.get(id=1)
for contact in employee.contacts.all():
    print(f"{contact.first_name} {contact.last_name}")
    for phone in contact.phones.all():
        print(f"  Phone: {phone.number}")
    for email in contact.emails.all():
        print(f"  Email: {email.email}")

# Get primary contact
primary_contact = employee.contacts.filter(is_primary=True).first()
```

### Reverse Query Example

```python
from django.contrib.contenttypes.models import ContentType

# Find all contacts for a specific model type
employee_ct = ContentType.objects.get_for_model(Employee)
employee_contacts = Contact.objects.filter(content_type=employee_ct)

# Find all contacts with a specific phone number
phone = "+1-555-123-4567"
contacts = Contact.objects.filter(phones__number=phone)
```

## Admin Integration

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

class ContactAdmin(admin.ModelAdmin):
    list_display = ['first_name', 'last_name', 'title', 'is_primary']
    list_filter = ['is_primary']
    search_fields = ['first_name', 'last_name', 'emails__email', 'phones__number']
    inlines = [AddressInline, PhoneInline, EmailInline]

admin.site.register(Contact, ContactAdmin)
admin.site.register(Address)
admin.site.register(Phone)
admin.site.register(Email)
```

## Performance Considerations

For better performance with Generic Foreign Keys, consider:

1. **Database indexes** on `content_type` and `object_id` (already included)
2. **Prefetching** related objects:
   ```python
   employees = Employee.objects.prefetch_related(
       'contacts__addresses',
       'contacts__phones', 
       'contacts__emails'
   )
   ```

3. **For large datasets**, consider denormalizing or using a dedicated search solution

## Recommendation

**Use Solution 1 (Generic Foreign Keys)** because:
- Maximum flexibility - any model can have contacts
- Centralized contact management
- Easy to add new model types without migrations
- Allows cross-model queries
- Clean separation of concerns

The only downside is slightly slower queries compared to direct foreign keys, but with proper indexing it's usually negligible for most applications.
