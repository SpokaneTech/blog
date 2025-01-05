# **Building Spokane Tech: Part 5**

Welcome to part 5 of the "Building Spokane Tech" series! In this article, we'll explore the django views and templates used to server web pages in the initial web app. 


## **Pages**
The initial release of spokanetech.org includes a small set of pages, including the landing page, an about page, a calendar page, and event and group pages. Here is a brief description of each page:

**Index:** Home page of spokanetech.org

**About:** Informational page for spokanetech.org

**Calendar:** Monthly Calendar listing all events per month

**Group List:** Page listing all tech groups

**Event List:** Page listing all upcoming events

**Group Detail:** Page detailing a specific tech group

**Event Detail:** Page detailing a specific event


## **Views**
The views are designed to support both htmx and non-htmx requests. A different template will be used accordingly when the *"template_name"* and *"htmx_template_name"* parameters are provided. Selection of the template is handled by the base class.

For our main entities we have a detail, list, and modal views to present data as applicable on web pages. Below is a snapshot of the tech group views. See [github](https://github.com/SpokaneTech/SpokaneTechWeb/blob/main/src/django_project/web/views.py) for the latest and complete code.

```python

class TechGroupView(HtmxOptionDetailView):
    """Render detail page for a TechGroup instance"""
    model = TechGroup
    htmx_template_name = "web/partials/detail/group.htm"
    template_name = "web/full/detail/group.html"


class TechGroupsView(HtmxOptionMultiFilterView):
    """Render a list of TechGroup instances"""
    htmx_list_template_name = "web/partials/list/groups.htm"
    htmx_list_wrapper_template_name = "web/partials/list/wrapper_list.htm"
    htmx_template_name = "web/partials/marquee/groups.htm"
    queryset = TechGroup.objects.filter(enabled=True)
    template_name = "web/full/list/groups.html"


class TechGroupModalView(ModelDetailBootstrapModalView):
    """Render Bootstrap 5 modal displaying get details of a TechGroup instance"""
    modal_button_submit = None
    modal_size = "modal-lg"
    modal_template = "web/partials/modal/group_information.htm"
    modal_title = "Group Info"
    model = TechGroup
```


## **Templates**

Templates are split into to sections, full and partials. Templates in the full subdirectory hold the template that extend the django base template and are used when the request to the page is not htmx. Templates in the partials subdirectory are used by htmx requests. We use a filename convention where full pages have the .html extension and partials have the .htm extension.

Here is the directory structure of the templates directories:
```plaintext
.
├── src
│   ├── django_project
│   │   ├── core
│   │   │   └── templates
│   │   │       └── base.htm
│   │   ├── web
│   │   │   └── templates
│   │   │       └── web
│   │   │           ├──  full
│   │   │           │    ├──  custom
│   │   │           │    │    ├──  about.html
│   │   │           │    │    ├──  calendar.html
│   │   │           │    │    └──  index.html
│   │   │           │    ├──  detail
│   │   │           │    │    ├──  event.html
│   │   │           │    │    └──  group.html
│   │   │           │    └──  list
│   │   │           │         ├──  events.html
│   │   │           │         └──  groups.html
│   │   │           │
│   │   │           └──  partials
│   │   │                ├──  custom
│   │   │                │    ├──  about.htm
│   │   │                │    ├──  calendar.htm
│   │   │                │    └──  index.htm
│   │   │                ├──  detail
│   │   │                │    ├──  event.htm
│   │   │                │    └──  group.htm
│   │   │                ├──  list
│   │   │                │    ├──  events.htm
│   │   │                │    └──  groups.htm
│   │   │                ├──  marquee
│   │   │                │    ├──  events.htm
│   │   │                │    └──  groups.htm
│   │   │                └──  modal
│   │   │                     ├──  event.htm
│   │   │                     └──  group.htm
│   │   └── ...
```
