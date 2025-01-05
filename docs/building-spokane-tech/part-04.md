# **Building Spokane Tech: Part 4**

Welcome to part 4 of the "Building Spokane Tech" series! In this article, we look at the initial modeling for the web app. 


## **Initial Django Models**
The initial release of spokanetech.org will include five models in the web application: Event, Link, SocialPlatform, Tag, and TechGroup. Let's breakdown the role of each.

**Event:** A tech event hosted by a TechGroup

**Link:** A link associated with a TechGroup, such as the group home page

**SocialPlatform:** The platform hosting the event, such as Meetup or EventBright

**Tag:** An attributes of a Event or a TechGroup

**TechGroup:** A local tech group that organizes events

Each of these models use the HandyHelperBaseModel which includes created_at and updated_at datetime fields and a model manager that provides a number of useful methods.

## **Model Code**
Below is a snapshot of the model code. See [github](https://github.com/SpokaneTech/SpokaneTechWeb/blob/main/src/django_project/web/models.py) for the latest and complete model code.

### ***Event***
```python
class Event(HandyHelperBaseModel):
    """An event on a specific day and time"""
    name = models.CharField(max_length=255)
    description = models.TextField(blank=True)
    start_datetime = models.DateTimeField(
        auto_now=False, auto_now_add=False, help_text="date and time the event starts"
    )
    end_datetime = models.DateTimeField(
        auto_now=False, auto_now_add=False, null=True, help_text="date and time the event ends"
    )
    location_name = models.CharField(
        max_length=64,
        blank=True,
        help_text="name of location where this event is being hosted",
    )
    location_address = models.CharField(
        max_length=256,
        blank=True,
        help_text="address of location where this event is being hosted",
    )
    map_link = models.URLField(
        blank=True,
        help_text="link to a map showing the location of the event",
    )
    url = models.URLField(
        blank=True,
        help_text="URL to the event details",
    )
    social_platform_id = models.CharField(
        max_length=128,
        blank=True,
        help_text="unique identifier provided by the social platform hosting the event",
    )
    group = models.ForeignKey("TechGroup", blank=True, null=True, on_delete=models.SET_NULL)
    tags = models.ManyToManyField("Tag", blank=True)
    image = models.ImageField(upload_to="tech_events/", blank=True, null=True)
```

### ***Link***
```python
class Link(HandyHelperBaseModel):
    """A link to a resource associated with a TechGroup or Event"""
    name = models.CharField(max_length=64, blank=True)
    description = models.CharField(max_length=255, blank=True)
    url = models.URLField()
```

### ***SocialPlatform***
```python
class SocialPlatform(HandyHelperBaseModel):
    """The social platform (such as Meetup) that hosts the group and events"""
    name = models.CharField(max_length=64, unique=True, help_text="service where this tech group is hosted")
    enabled = models.BooleanField(default=True)
    base_url = models.URLField(blank=True, help_text="base url of provider")
```

### ***Tag***
```python
class Tag(HandyHelperBaseModel):
    """A Tag that describes attributes of a Event"""
    value = models.CharField(max_length=64, unique=True, null=False)
```

### ***TechGroup***
```python
class TechGroup(HandyHelperBaseModel):
    """A group that organizes events"""
    name = models.CharField(max_length=128, unique=True)
    description = models.TextField(blank=True)
    enabled = models.BooleanField(default=True)
    platform = models.ForeignKey("SocialPlatform", on_delete=models.CASCADE)
    icon = models.CharField(
        max_length=255,
        blank=True,
        help_text="Font Awesome CSS icon class(es) to represent the group.",
    )
    tags = models.ManyToManyField("Tag", blank=True)
    links = models.ManyToManyField("Link", blank=True)
    image = models.ImageField(upload_to="techgroups/", blank=True, null=True)

```
