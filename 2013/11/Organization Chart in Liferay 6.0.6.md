Liferay out of the box provides very flexible and powerful features to organize a portal. It allows creation of Communities, Organizations, Roles, User Groups, Teams and of course Users. All of these entites and relationships between them are described thoroughly in documentation (for example: [http://www.liferay.com/documentation/liferay-portal/6.0/administration/-/ai/portal-architectu-4](http://www.liferay.com/documentation/liferay-portal/6.0/administration/-/ai/portal-architectu-4)). These built-in features and excellent documentation allow fulfilment of many business requirements without any programming.

One thing, which in my opinion is missing, is possibility to see portal's organization structure as a whole. It is possible to display "list of Xes", "details of X" or "list of Xes associated with Y". However, it is not possible to see multi-level hierarchies or simply all entites at once. This makes it difficult to get the overall picture or to verify if everything has been configured correctly.

Because of that I created a simple portlet plugin, which adds a new portlet - Organization Chart. This portlet displays hierarchy of Communities, Organizations, User Groups, Teams and Users as a tree. This is how it looks in action:

![lf-org-chart](https://raw.github.com/mateuszwenus/blog-backup/master/2013/11/lf-org-chart0.png)

The plugin can be downloaded from [https://github.com/mateuszwenus/lf-org-chart](https://github.com/mateuszwenus/lf-org-chart). It is compatible with Liferay 6.0.6. It is licensed under MIT license, so you can freely use it in both personal and commercial projects. 

I hope you will find this plugin useful. If you have any comments, suggestions or problems with it, let me know.
