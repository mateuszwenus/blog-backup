In [my recent Liferay project](https://github.com/mateuszwenus/blog-backup/blob/master/2013/11/Organization%20Chart%20in%20Liferay%206.0.6.md) I wanted to load list of all top-level organizations (organizations, which are not members of a community or another organization). There is no built-in method in any *LocalServiceUtil which returns what I needed. I also couldn't use DynamicQuery, because the query I wanted to run included groups_orgs table which is not mapped to any entity. 

My first approach was the following code:
```java
List&lt;Group&gt; communities = GroupLocalServiceUtil.search(PortalUtil.getCompanyId(req), null, null, null, QueryUtil.ALL_POS, QueryUtil.ALL_POS);
Collection&lt;Object&gt; ids = new HashSet&lt;Object&gt;();
for (Group community : communities) {
    List&lt;Organization&gt; communityOrganizations = OrganizationLocalServiceUtil.getGroupOrganizations(community.getPrimaryKey());
    for (Organization org : communityOrganizations) {
        ids.add(org.getPrimaryKey());
    }
}
DynamicQuery query = DynamicQueryFactoryUtil.forClass(Organization.class, PortalClassLoaderUtil.getClassLoader());
query.add(RestrictionsFactoryUtil.eq("parentOrganizationId", Long.valueOf(OrganizationConstants.DEFAULT_PARENT_ORGANIZATION_ID)));
query.add(RestrictionsFactoryUtil.not(RestrictionsFactoryUtil.in("id", ids)));
return OrganizationLocalServiceUtil.dynamicQuery(query);
```
This is obviously not optimal - it executes too many queries and loads too much data from DB. After some searching and experimenting I came up with better solution, which is general enough to be useful in other contexts. 

First I created the following service.xml file:

```xml
&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;!DOCTYPE service-builder PUBLIC "-//Liferay//DTD Service Builder 6.0.0//EN" "http://www.liferay.com/dtd/liferay-service-builder_6_0_0.dtd"&gt;
&lt;service-builder package-path="com.github.mateuszwenus.lf_org_chart.service_builder"&gt;
    &lt;author&gt;Mateusz Wenus&lt;/author&gt;
    &lt;namespace&gt;lf_org_chart&lt;/namespace&gt;
    &lt;entity name="SessionCustomAction" local-service="true" remote-service="false"&gt;&lt;/entity&gt;
&lt;/service-builder&gt;
```
The file contains entity with no columns, which makes service builder generate @Transactional service layer, but no dao/persistence layer. I ran service builder and then I added the following method to SessionCustomActionLocalServiceImpl:
```java
public Object execute(SessionCallback callback) {
  SessionFactory sessionFactory = (SessionFactory) PortalBeanLocatorUtil.locate("liferaySessionFactory");
  Session session = null;
  try {
    session = sessionFactory.openSession();
    return callback.execute(session, sessionFactory.getDialect());
  } catch (Exception e) {
    throw handleException(e);
  } finally {
    if (session != null) {
      sessionFactory.closeSession(session);
    }
  }
}
```
SessionCallback is a simple interface which allows to execute any action with a Session:
```java
public interface SessionCallback {
  Object execute(Session session, Dialect dialect) throws Exception;
}
```
Then I ran service builder again and execute() method was added to SessionCustomActionLocalServiceUtil. Finally in my portlet I could use the following code to find top level organizations:
```java
SessionCustomActionLocalServiceUtil.execute(new SessionCallback() {
  public Object execute(Session session, Dialect dialect) throws Exception {
    String sql = "SELECT {org.*} FROM Organization_ org WHERE org.parentOrganizationId = 0 "
      + "and org.organizationId not in (select gr_org.organizationId from Groups_Orgs gr_org)";
    SQLQuery query = session.createSQLQuery(sql);
    query.addEntity("org", PortalClassLoaderUtil.getClassLoader().loadClass("com.liferay.portal.model.impl.OrganizationImpl"));
    return QueryUtil.list(query, dialect, QueryUtil.ALL_POS, QueryUtil.ALL_POS);
  }
});
```
BTW, if this solution looks similar to [HibernateTemplate.execute()](http://docs.spring.io/spring/docs/3.2.5.RELEASE/javadoc-api/org/springframework/orm/hibernate3/HibernateTemplate.html#execute%28org.springframework.orm.hibernate3.HibernateCallback%29) to you, then you are right - that method was my inspiration.