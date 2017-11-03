---
title:  "How To Fix AttributeError: 'NoneType' object has no attribute '_meta'"
date: 2017-03-11 09:09:00
categories: [django, python]
tags: [django, python]
---
A few hours ago i spend my time for debugging cause a shit error above, and theeennn after a long time, finally i can solved this problem, with a simple decorator from django.utils.encoding lib, i just added ```python @python_2_unicode_compatible()```

Why this decorator can solve my problem??

Actually, cause while you use ``` python def __str__(self)``` it will not work on python 2 without a decorator, so it's why you must use this decorators if you want to use python 2.

As documentations mentioned:
>A decorator that defines __unicode__ and __str__ methods under Python 2. Under Python 3 it does nothing.
>To support Python 2 and 3 with a single code base, define a __str__ method returning text (use six.text_type() if you’re doing some casting) and apply this decorator to the class.

### Example:
``` python
@python_2_unicode_compatible
class Comments(StatusDeletedModel, DateTimeHandlerModel, StatusManager):
    user_id = models.ForeignKey(User, on_delete=models.CASCADE,
                                db_column='user_id')
    post_id = models.ManyToManyField('Posts', db_column='post_id')
    comment = models.CharField(max_length=200)
    deleted = models.IntegerField(choices=DELETED_STATUS, default=NOT_DELETED,
                                  db_column='deleted')

    class Meta:
        db_table = 'comments'

    def __str__(self):
        return '{}'.format(self.comment)

    def save(self, *args, **kwargs):
        if self.created_at is None:
            self.created_at = timezone.now()
        if self.updated_at is not None:
            self.updated_at = timezone.now()
        super(Comments, self).save(*args, **kwargs)
```
