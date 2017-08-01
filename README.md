# Jenkins Dynamic Databinding UI Control Library

Jenkins UI controller library supports dynamic databinding, including some useful jelly tags like &lt;dropdownList&gt; and &lt;radioBlock&gt;.

# Configuration

Add `<repository>` and `<dependency>`in pom.xml:

```
<repositories>
    <repository>
        <id>tags</id>
        <name>Custom Tags</name>
        <url>https://raw.githubusercontent.com/t-wanl/Jenkins-Dynamic-Databinding-UI-Control-Library/master/jar/</url>
    </repository>
</repositories>

<dependencies>
    <dependency>
        <groupId>albertxavier</groupId>
        <artifactId>jenkins-dynamic-databinding-ui-controller-library</artifactId>
        <version>1.0</version>
    </dependency>
</dependencies>
```


# Usage

Please refer to
[Demo-Jenkins-UI](https://github.com/t-wanl/Demo-Jenkins-UI) for demo.

## General
In `<j:jelly>` defines `xmlns:ui="/ui"`, then use namespace `ui` in tag defined in this library. E.g. `<ui:dropdownList>`

## &lt;downdownList&gt;

### Support emitting selected option from dropdownList.

The value of the selected `<dropdownListBlock>` in `<dropdownList>` can be obtained just like `<select>` tag.

#### jelly file

```
...
<ui:dropdownList title="Fruit" selectedName="dropdownListSelected" name="dropdownListContent">
  <f:dropdownListBlock title="Apple" value="apple" selected="true">
  	<f:entry title="Number of apple" field="number">
	    <f:textbox/>
	</f:entry>
  </f:dropdownListBlock>
</ui:dropdownList>
...
```
#### json
```
{
	...
	"dropdownListSelected": "apple",
	"dropdownListContent": {
		"number": "1"
	},
	...
}
```


### Support databinding from other UI controllers to dropdownList.

A certain option in <dropdownList> and its <dropdownListBlock> can be triggered by other UI controllers.

#### jelly file

```
<f:entry title="Select Fruit" field="preSelect">
  <f:select/>
</f:entry>
<ui:dropdownList title="Fruit" selectedName="dropdownListSelected2" name="dropdownListContent">
  <f:dropdownListBlock title="Apple" value="apple" selected="true">
      <f:entry title="Number of apple">
          <f:textbox/>
      </f:entry>
  </f:dropdownListBlock>
  <f:dropdownListBlock title="Banana" value="banana" selected="false">
      <f:entry title="">
          <f:checkbox name="yellow" checked="true" title="Yellow?"/>
      </f:entry>
  </f:dropdownListBlock>
</ui:dropdownList>
```
Note that `selectedName` defines the variable stores the selected option and `name` defines the variable stores the content in `<dropdownListBlock>`

#### java file

Add options to `<select>` tag.
```
public ListBoxModel doFillPreSelectItems() throws IOException, ServletException  {
    ListBoxModel model = new ListBoxModel();
    model.add("Apple", "apple");
    model.add("Banana", "banana");
    return model;
}
```

Define trigger rules.

```
public String doFillDropdownListSelected2Items(
        @QueryParameter String preSelect
) throws IOException, ServletException {
    if (preSelect.equalsIgnoreCase("apple")) {
        return "apple";
    }
    return "banana";
}
```

`doFillDropdownListSelected2Items` function depends on `preSelect`, and return the value of the `dropdownListBlock` which you what to select.

## &lt;radioBlock&gt;

### support nested radioblocks

#### jelly file

```
<ui:radioBlock name="fruit" value="apple" title="Apple" checked="true">
    <ui:radioBlock name="color" value="red" title="Red" checked="false">
        <ui:tab><p>Everyone likes red apples.</p></ui:tab>
        <ui:radioBlock name="size" value="small" title="Small" checked="true">

            <ui:tab><p>Tom like small apple.</p></ui:tab>
        </ui:radioBlock>
        <ui:radioBlock name="size" value="big" title="Big" checked="false">
            <ui:tab><p>Mike like big apple.</p></ui:tab>
        </ui:radioBlock>
    </ui:radioBlock>
    <ui:radioBlock name="color" value="green" title="Green" checked="true">
        <ui:tab><p>Who likes green apples?</p></ui:tab>
    </ui:radioBlock>
</ui:radioBlock>
```

# Development

## How to make a tag support dynaminc databinding

In order to support dynaminc databinding, you need to modify both your jelly tag definition, Java code and Javascript code.

1. `fillUrl` attribute defines tag itself, and Stapler can render it with `doFillXyzItems` dunction.
2. `fillDependsOn` attribute defines the tag field which the &lt;dropdownList&gt; depends on.
3. `MorphTagLibrary` makes a tag support dynamically set attributes:
    e.g. In &lt;select&gt;
    ```
    <m:select xmlns:m="jelly:hudson.util.jelly.MorphTagLibrary"
           class="setting-input ${attrs.checkUrl!=null?'validated':''} select ${attrs.clazz}"
           name="${attrs.name ?: '_.'+attrs.field}"
           value="${value}"
           ATTRIBUTES="${attrs}" EXCEPT="field clazz">
    ```
    Please refer to [MorphTagLibrary](https://github.com/jenkinsci/jenkins/blob/08def67a18eee51de9f3f99bc2a792fee1c160e0/core/src/main/java/hudson/util/jelly/MorphTagLibrary.java) for more details.
4. Defines `doFillXyzItems` with `@QueryParameter` that represent `fillDependsOn` attribute.
5. `calFillSettings` in `Descriptor.java` computes the list of other form fields that the given field depends on, via the `doFillXyzItems` method, and sets that as the `fillDependsOn` attribute. Also computes the URL of the `doFillXyzItems` and sets that as the `fillUrl` attribute.
    Add `${descriptor.calcFillSettings(field,attrs)}` to your jelly code to call this function.
    Please refer to [calFillSettings](https://github.com/jenkinsci/jenkins/blob/51c46c6cf22a57860c71c7d7236ae30f6baa6651/core/src/main/java/hudson/model/Descriptor.java) and [select.js](https://github.com/jenkinsci/jenkins/blob/master/core/src/main/resources/lib/form/select.jelly) for more details.
6. Update view in Javascript.
    [hudson-behavior.js](https://github.com/jenkinsci/jenkins/blob/8f8b058548a4b912d6a9e6fa1a4a0873a70598f7/war/src/main/webapp/scripts/hudson-behavior.js) defines `jenkinsRules`, while [behavior.js](https://github.com/jenkinsci/jenkins/blob/08def67a18eee51de9f3f99bc2a792fee1c160e0/war/src/main/webapp/scripts/behavior.js) uses css selectors to apply javascript behaviors to enable unobtrusive javascript in html documents.
    You need to write your own Javascript code for updating your custom jelly tag.
    Please refer to [select.js](https://github.com/jenkinsci/jenkins/blob/master/core/src/main/resources/lib/form/select/select.js) for more details.
    Finally reference your Javascript code like this in custom jelly tag:
    `<st:adjunct includes="lib.form.select.select"/>`.
    Please refer to [select.jelly](https://github.com/jenkinsci/jenkins/blob/master/core/src/main/resources/lib/form/select.jelly) for more details.
