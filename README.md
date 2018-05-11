# myindextestproject

My Lucene Index Test Hippo CMS Project

## How to Customize the Default Lucene Indexing Behavior of Jackrabbit in Hippo CMS?

Copy the following file to ```cms/src/main/resources/org/hippoecm/repository/query/lucene/indexing_configuration.xml```:

- https://code.onehippo.org/cms-community/hippo-repository/blob/hippo-repository-5.2.0/resources/src/main/resources/org/hippoecm/repository/query/lucene/indexing_configuration.xml

Then the ```cms/src/main/resources/org/hippoecm/repository/query/lucene/indexing_configuration.xml``` file will override the default setting in the resource of hippo-repository-resources-x.x.x.jar.

Change the tag name/version (e.g, ```hippo-repository-5.2.0```) for your project.

References:

- https://www.onehippo.org/library/administration/searchindex-configuration.html
- https://wiki.apache.org/jackrabbit/IndexingConfiguration

## How to Exclude Rich Text Content field in a specific document type?

Suppose you want to exclude the Rich Text Content field (```hippostd:html``` child node) under a News Article document
(e.g, ```myindextestproject:newsdocument```).

You can simply add the following in ```cms/src/main/resources/org/hippoecm/repository/query/lucene/indexing_configuration.xml``` file for the purpose:

```xml
<configuration xmlns:hippostd="http://www.onehippo.org/jcr/hippostd/nt/2.0" xmlns:hippostdpubwf="http://www.onehippo.org/jcr/hippostdpubwf/nt/1.0"
               xmlns:jcr="http://www.jcp.org/jcr/1.0"
               xmlns:myindextestproject="http://www.onehippo.org/myindextestproject/nt/1.0">

    <!-- Custom Index Rule in order to exclude hippostd:html nodes under myindextestproject:newsdocument node. -->
    <index-rule nodeType="hippostd:html"
                condition="parent::element(*, myindextestproject:newsdocument)/@jcr:primaryType = '{http://www.onehippo.org/myindextestproject/nt/1.0}newsdocument'">
    </index-rule>

    <!-- SNIP -->

```

Note the following:

- You should define the namespace prefixes including ```jcr``` and your project's namespace (e.g, ```myindextestproject```).
- In the above example, the **index-rule** element works for ```hippostd:html``` JCR nodes on the **condition** that
  the ```hippostd:html``` node's *parent* node's primary node type name is exactly "myindextestproject:newsdocument".
- The property section in the **condition** is *redundant* but it's necessary to make it work.
  You have to use full namespace URI name (e.g, ```{http://www.onehippo.org/myindextestproject/nt/1.0}newsdocument```) for the literal
  when comparing with ```@jcr:primaryType``` property value.
  Prefixed literal like ```myindextestproject:newsdocument``` never works because ```org.apache.jackrabbit.core.query.lucene.IndexingConfigurationImpl.PathExpression#evaluate(NodeState)``` does the following internally:

```java
InternalValue[] values = propState.getValues();
for (InternalValue value : values) {
    if (value.toString().equals(propertyValue)) {
        return true;
    }
}
```

### Demo

- Build and run this demo project using ```mvn clean verify && mvn -Pcargo.run```.
- Visit http://localhost:8080/site/.
- In the Search text box on the top, enter "chickenbone" and initiate the search.
- It doesn't show any result even if you can find "chickenbone" in [The medusa news](http://localhost:8080/site/news/2018/05/the-medusa-news.html) page which renders the ```hippostd:html``` content.
- See the configuration in [indexing_configuration.xml](cms/src/main/resources/org/hippoecm/repository/query/lucene/indexing_configuration.xml).
