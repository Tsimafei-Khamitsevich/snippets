# snippets
Models we are working with:

```
class Women(models.Model):
    title = models.CharField(max_length=255, verbose_name="Заголовок")
    slug = models.SlugField(max_length=255, unique=True, db_index=True, verbose_name="URL")
    content = models.TextField(blank=True, verbose_name="Текст статьи")
    photo = models.ImageField(upload_to="photos/%Y/%m/%d/", verbose_name="Фото")
    time_create = models.DateTimeField(auto_now_add=True, verbose_name="Время создания")
    time_update = models.DateTimeField(auto_now=True, verbose_name="Время изменения")
    is_published = models.BooleanField(default=True, verbose_name="Публикация")
    cat = models.ForeignKey('Category', on_delete=models.PROTECT, verbose_name="Категории")

    def __str__(self):
        return self.title

    def get_absolute_url(self):
        return reverse('post', kwargs={'post_slug': self.slug})

    class Meta:
        verbose_name = 'Известные женщины'
        verbose_name_plural = 'Известные женщины'
        ordering = ['-time_create', 'title']


class Category(models.Model):
    name = models.CharField(max_length=100, db_index=True, verbose_name="Категория")
    slug = models.SlugField(max_length=255, unique=True, db_index=True, verbose_name="URL")

    def __str__(self):
        return self.name

    def get_absolute_url(self):
        return reverse('category', kwargs={'cat_slug': self.slug})

    class Meta:
        verbose_name = 'Категория'
        verbose_name_plural = 'Категории'
        ordering = ['id']
```

See recent SQL queries
```
from django.db import connection
from django.db import reset_queries

connection.queries
reset_queries()
```

Prefetch related objects to avoid extra queries
```
Women.objects.select_related('cat').all()
Category.objects.prefetch_related('women_set').all()
```

Sorting in reverse order
```
Woment.objects.all().reverse()
```

AND &, OR |, NOT ~
```
from django.db.models import Q

Women.objects.filter(Q(title__endswith='r') | Q(cat__name='signer'))
```

DateTime
Earliest, Latest date
Previous Next by date
```
Women.objects.earliest('time_update')
Women.objects.latest('time_update')

w = Women.objects.get(pk=7)
# get_previous_by_ datetime field
w.get_previous_by_time_update()
w.get_next_by_time_update(#condition)
```

Exists
```
Women.objects.filter(title='Ryana').exists()
```

Distinct
```
Category.objects.filter(women__title_contains='ли').distinct()
```

Agregate
```
from django.db.models import *
Women.objects.agregate(Max('cat_id'))
# Min, Sum, Count, Avg
```

Get Women.title and coresponding cat.name
```
Women.objects.values('title', 'cat__name')
```

Group by 
```
# Count women in each category
Category.objects.annotate(Count('women'))
# category id and name. Count women in each category.
Category.objects.values('id', 'name').annotate(total=Count('women'))
# get category and women count that greater than 0
Category.objects.annotate(total=Count('women')).filter(total__gt=0)
```

Reference field from same model 
```
from django.db.models import F
# women has 'views' field - times post is shown
Women.objects.get(pk=1).update(views=F('views')+1)
# or
w = Women.objects.get(pk=1)
w.views = F('views') + 1
```

Raw SQL
```
Women.objects.raw('SELECT * FROM women_women')
Women.objects.raw('SELECT id, title FROM women_women')
# fields must include 'id' in SELECT
```
