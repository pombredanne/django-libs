Views Mixins
===========

AccessMixin
-----------

Use this mixin if you want to allow users of your app to decide if the views
of your app should be accessible to anonymous users or only to authenticated
users::

    form django_libs.views_mixins import AccessMixin

    class YourView(AccessMixin, TemplateView):
        access_mixin_setting_name = 'YOURAPP_ALLOW_ANONYMOUS'

        # your view code here

Given the above example, users of your app would have to set
``YOURAPP_ALLOW_ANONYMOUS`` to ``True`` or ``False``.


AjaxResponseMixin
-----------------

Use this with views that can be called normally or from an AJAX call. Usually,
when you call a view normally, you will have ``{% extends "base.html" %}`` at
the beginning of the view's template. However, when you call the same view
from an AJAX call you just want to update a partial region of your page,
therefore the view needs to return that partial template only.

You can use this by inheriting the class::

    from django_libs.views_mixins import AjaxResponseMixin

    class MyView(AjaxResponseMixin, CreateView):
        ajax_template_prefix = 'partials/ajax_'

The attribute ``ajax_template_prefix`` defaults to ``ajax_``. If you would
like to store your app's ajax templates in a different way, for example in
a subfolder called ``partials``, you can override that attribute in your
class.

DetailViewWithPostAction
------------------------

This view enhances the class-based generic detail view with even more
generic post actions.
In order to use it, import it like all the other generic class based views
and view mixins.

* Create a Mixin or View which inherits from this action mixin.
* Be sure to add a general ``get_success_url()`` function or custom success
  functions for each post action.
* Create your post actions
* Make sure to add this action names to the name attribute of an input field.


Basic usage in a html template::

    <form method="post" action=".">
        {% csrf_token %}
        <input name="post_verify" type="submit" value="Verify" />
    </form>


Usage in a views.py::

    from django_libs.views_mixins import DetailViewWithPostAction

    class NewsDetailBase(DetailViewWithPostAction):
        def post_verify(self):
            self.object.is_verified = True
            self.object.save()
    
        def post_reject(self):
            self.object.is_verified = False
            self.object.save()


    class NewsEntryDetailView(NewsDetailBase):
        model = NewsEntry

    def get_success_url(self):
        return reverse('newsentry_detail', kwargs={'pk': self.object.pk})

        def post_verify(self):
            super(NewsEntryDetailView, self).post_verify()
            ctx_dict = {
                'verified': True,
                'entry': self.object,
            }
            self.send_mail_to_editor(ctx_dict)

        def get_success_url_post_reject(self):
            return reverse('newsentry_list')


JSONResponseMixin
-----------------

You can find out more about the ``JSONResponseMixin`` in the official Django
docs:
https://docs.djangoproject.com/en/dev/topics/class-based-views/#more-than-just-html

In order to use it, just import it like all the other generic class based views
and view mixins::

    from django.views.generic import View
    from django_libs.views_mixins import JSONResponseMixin

    class MyAPIView(JSONResponseMixin, View):
        pass
