# django-gen-settings - a Settings Mixin for Django Models

Features:
* Save settings directly to a Instance
* Retrieve settings directly from an Instance
* Settings can be inherited from multiple predecessor fields

When to use:
* if you want to store values to a model, but dont want to create additional fields on the model

And when not:
* if you want to query these stored values
* if you often change this values

## Notes

**This Version should not be used in full production, since the API will may change in the future**

The settings are stored in an json object in the database, some caching is done, 
but settings a value leads to an write of the whole json object

## Installation

pypi package coming soon ...

Add django_gen_settings to your INSTALLED_APPS in your django settings.py:

```python
INSTALLED_APPS = (
    ...,
    'django_gen_settings',
)
```


## Usage

Lets say you have a model Client like this:
```python
class Client(models.Model, SettingsMixin):
    name = models.CharField(max_length=128, db_index=True, verbose_name=_('Name'), help_text=_('Name of the Client')) #Tag Name should be unique
```            
        
The SettingsMixin adds a additional field with the names 'settings' to your model.
Instead of accessing the field directly, use the getsetting and setsetting methods.
The setting name will be splitted on the dots ( '.' ), and saved as json dict.

```python

client = Client()
client.name = "Testclient"
client.setsetting("messages.inbox.displaymessages", 5 )
client.save()

nrmsg = client.getsetting("messages.inbox.displaymessages") # nrmsg contains 5

```            
   
Lets add a second class with also with a SettingsMixin:

```python
class Account(SettingsMixin, AbstractBaseUser, PermissionsMixin):
    client = models.ForeignKey(Client)
    username = models.CharField(max_length=64,  unique=True)
    email = models.EmailField(blank=True)
    is_staff = models.BooleanField(default=False)

    USERNAME_FIELD = 'username'

    class Meta:
        settings_predecessor_field = 'client'
    
    
```

Note the settings_predecessor_field in the Meta Class of Account. If a setting does not exist in the current instance,
the setting of the predecessor field is queried. You can also chain multiple models with a SettingsMixin.

If you create a Account instance linked to the created Client Instance before, you can do the following:
```python


nrmsg = account.getsetting("messages.inbox.displaymessages") # contains 5 from the predecessor field 'client'
account.setsetting("messages.inbox.displaymessages", 2) 

nrmsg = account.getsetting("messages.inbox.displaymessages") # contains now 2, since we overlayed the setting

```  


