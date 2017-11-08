---
title:  "Extend User Field on Django Rest Framework"
date: 2017-11-08 23:06
categories: [django, drf, django-rest-framework]
tags: [django, django-rest-framework]
---
There ara 4 trick for extend users field, but on this example i use one to one realtionship.
You can read [here](https://simpleisbetterthancomplex.com/tutorial/2016/07/22/how-to-extend-django-user-model.html) why i use one to one relation, and you will get the difference between the four methods.

It's simple trick for extends users field on drf, ask me on facebook if you are confused.

## Models
``` python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

from django.db import models
from django.utils import timezone
from django.contrib.auth.models import User
from django.db.models.signals import post_save
from django.dispatch import receiver

from managers.custom_managers import (
    DateTimeHandlerModel,
    StatusManager
)


class Profile(DateTimeHandlerModel, StatusManager):
    user = models.OneToOneField(User, on_delete=models.CASCADE, db_column='user')
    address = models.CharField(max_length=100)
    phone = models.CharField(max_length=50)
    birth_date = models.DateField(null=True)
    description = models.CharField(max_length=200)

    class meta:
        db_table = 'profile'

    def __str__(self):
        return self.title

    def save(self, *args, **kwargs):
        if self.created_at is None:
            self.created_at = timezone.now()
        if self.created_at is not None:
            self.updated_at = timezone.now()
        super(Profile, self).save(*args, **kwargs)
```

## Serializers
``` python
from django.contrib.auth.models import User
from .models import Profile
# from .models import Posts, Comments, Likes, Alarams
from rest_framework.serializers import HyperlinkedModelSerializer, CharField


class ProfileSerializer(HyperlinkedModelSerializer):
    username = CharField(source="user.username")
    email = CharField(source="user.email")
    password = CharField(source="user.password")

    class Meta:
        model = Profile
        fields = ('username', 'email', 'password', 'address', 'phone',
                  'description', 'birth_date')

    def create(self, validated_data):
        #validated_data = Get all valid data
        #validated_data.pop('user') = Gell data by user data only
        #key "user" is one to one relation from User models
        user_data = validated_data.pop("user")
        #User.objects.create(**user_data) = Create data by user_data
        #'**' = for get all data like dict, it's will be key:value format
        user = User.objects.create(**user_data)
        #create data profile, user above as instance for user one to one relation on Profile models
        #like **user_data above, **validated_data, for get all data, key:value formats(dict)
        profile = Profile.objects.create(user=user, **validated_data)
        return Profile
```

## Api
``` python
from rest_framework.viewsets import ModelViewSet
from django.contrib.auth.models import User
# from .models import Posts, Comments, Likes, Alarams
from .serializers import ProfileSerializer
from .models import Profile

class ProfileViewSet(ModelViewSet):
    queryset = Profile.objects.all()
    serializer_class = ProfileSerializer
```
