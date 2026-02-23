---
name: django-baton
description: Use when implementing Django ModelAdmin classes with django-baton features like form tabs, dropdown filters, form includes, inline configurations, or changelist customizations
---

# Django Baton

## Overview

Django Baton transforms Django admin with Bootstrap 5 and adds powerful features: tabbed forms, enhanced filters, form includes, and AI capabilities. Use django-baton specific features instead of complex Django workarounds.

## When to Use

- Implementing ModelAdmin classes with tabs
- Adding custom content to forms or changelists
- Using enhanced filters (dropdown, multiple choice, text input)
- Customizing changelist row attributes

## Form Tabs

Organize long forms into tabs using CSS classes. Organize fields semantically, for example:

- Main (title, slug, date)
- Content (abstract, content)
- Location (latitude, longitude, country, region, city)

**Three-step process:**

1. Initialize tabs in first fieldset: `"baton-tabs-init"`
2. Declare tabs in first fieldset: `"baton-tab-fs-{name}"` or `"baton-tab-inline-{related_name}"`
3. Link fieldsets to tabs: `"tab-fs-{name}"` in subsequent fieldsets

```python
from baton.admin import RelatedDropdownFilter

@admin.register(Page)
class PageAdmin(admin.ModelAdmin):
    fieldsets = (
        (_('Main'), {
            'fields': ('title', 'slug'),
            'classes': (
                'baton-tabs-init',          # Step 1: Initialize
                'baton-tab-fs-content',     # Step 2: Declare Content tab
                'baton-tab-fs-seo',         # Step 2: Declare SEO tab
                'baton-tab-inline-images',  # Step 2: Declare Images inline tab
            ),
        }),
        (_('Content'), {
            'fields': ('body', 'excerpt'),
            'classes': ('tab-fs-content',),  # Step 3: Link to Content tab
        }),
        (_('SEO'), {
            'fields': ('meta_title', 'meta_description'),
            'classes': ('tab-fs-seo',),      # Step 3: Link to SEO tab
        }),
    )
```

**Tab control classes:**

- `tab-fs-none` — Fieldset appears outside all tabs (always visible)
- `order-[NUMBER]` — Control tab order (0-based, add to first fieldset)
- `baton-tab-group-fs-tech--inline-feature` — Group multiple items (separate with `--`)

**Inlines:** Inline classes stay the same. Use `"collapse-entry"` and `"expand-first"` for collapsible inlines:

```python
class ImageInline(admin.StackedInline):
    model = Image
    extra = 1
    classes = ('collapse-entry', 'expand-first')
```

## List Filters

**Dropdown filters** (requires 3+ options, else uses default):

```python
from baton.admin import (
    DropdownFilter,              # For regular fields
    RelatedDropdownFilter,       # For ForeignKey
    RelatedOnlyDropdownFilter,   # ForeignKey (only used values)
    ChoicesDropdownFilter,       # For choice fields
)

class NewsAdmin(admin.ModelAdmin):
    list_filter = (
        ('category', RelatedDropdownFilter),
        ('status', ChoicesDropdownFilter),
        ('author', DropdownFilter),
    )
```

**Multiple choice filter:**

```python
from baton.admin import MultipleChoiceListFilter

class StatusFilter(MultipleChoiceListFilter):
    title = 'Status'
    parameter_name = 'status__in'

    def lookups(self, request, model_admin):
        return News.Status.choices
```

**Text input filter:**

```python
from baton.admin import InputFilter

class IdFilter(InputFilter):
    parameter_name = 'id'
    title = 'ID'

    def queryset(self, request, queryset):
        if self.value():
            return queryset.filter(id=self.value())
        return queryset
```

## Form Includes

Insert custom templates into forms. Templates receive `{{ original }}` variable:

```python
@admin.register(News)
class NewsAdmin(admin.ModelAdmin):
    baton_form_includes = [
        ('news/admin_stats.html', 'publication_date', 'above'),
        ('news/admin_help.html', 'content', 'top'),
        ('news/admin_preview.html', 'title', 'right'),
    ]
```

**Positions:** `'above'`, `'top'`, `'below'`, `'right'`

Form includes work with tabs — content appears in the field's tab.

## Changelist Customizations

**Row attributes** (add classes, styles, data attributes):

```python
def baton_cl_rows_attributes(self, request, cl):
    data = {}
    for item in cl.result_list:
        if item.is_featured:
            data[item.pk] = {
                'class': 'featured-row',
                'data-status': item.status,
            }
    return data
```

**Changelist includes:**

```python
baton_cl_includes = [
    ('news/admin_stats.html', 'top'),
]
```

**Filter sidebar includes:**

```python
baton_cl_filters_includes = [
    ('news/filter_help.html', 'top'),
]
```

## AI Features (Requires Subscription)

**Text summarization:**

```python
baton_summarize_fields = {
    'body': [{
        'target': 'summary',
        'words': 140,
        'useBulletedList': True,
        'language': 'en',
    }],
}
```

**AI image field:**

```python
from baton.fields import BatonAiImageField

class Article(models.Model):
    image = BatonAiImageField(
        upload_to='images/',
        subject_location_field='image_subject_location',
        alt_field='image_alt',
    )
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Inline doesn't appear in tab | Use `baton-tab-inline-{modelname}` not `baton-tab-fs-` |
| Tab classes on wrong fieldset | First fieldset gets `baton-tabs-init` and all `baton-tab-*` declarations |
| Forgot RelatedDropdownFilter import | Import from `baton.admin` not `django.contrib.admin` |
| Form include doesn't show | Check template path and field name match exactly |
| Dropdown shows default filter | Dropdown requires 3+ options to activate |

## Quick Reference

| Feature | Attribute | Example |
|---------|-----------|---------|
| Form tabs | fieldset classes | `'baton-tabs-init'`, `'tab-fs-content'` |
| Form includes | `baton_form_includes` | `[('template.html', 'field', 'above')]` |
| Inline tabs | fieldset classes | `'baton-tab-inline-modelname'` |
| Dropdown filters | `list_filter` | `('field', RelatedDropdownFilter)` |
| Row attributes | `baton_cl_rows_attributes` | Method returning `{pk: {'class': 'custom'}}` |
| Collapsible inlines | inline classes | `'collapse-entry'`, `'expand-first'` |
| Text summarization | `baton_summarize_fields` | `{'source': [{'target': 'dest'}]}` |

## Other admin paractices

- use `prepopulated_fields` attribute whenever a slug field exists. Prepopulate the slug using the name or title or similar field.

## Real-World Impact

Form tabs reduce scroll time on complex models from minutes to seconds. Form includes eliminate readonly field hacks. Enhanced filters make large datasets navigable. See <https://django-baton.readthedocs.io/> for complete documentation.
