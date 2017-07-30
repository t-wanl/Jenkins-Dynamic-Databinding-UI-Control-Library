# tags

Jenkins UI controller library support databinding.

# Configuration

Add `<repository>` and `<dependency>`in pom.xml:

```
<repositories>
    <repository>
    <id>tags</id>
    <name>Custom Tags</name>
    <url>https://raw.githubusercontent.com/t-wanl/tags/master/jar/</url>
    </repository>
</repositories>

<dependencies>
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <version>1.5.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
    <groupId>albertxavier</groupId>
    <artifactId>tags</artifactId>
    <version>1.0</version>
    </dependency>
</dependencies>
```


# Usage

Please refer to
[Demo-Jenkins-UI](https://github.com/t-wanl/Demo-Jenkins-UI) for detailed demo.

## General
In <j:jelly> defines `xmlns:ui="/ui"`, then use namesapce `ui` in tag ddefined in this library. E.g. `<ui:dropdownList>`

## &lt;downdownList&gt;

### Support emitting selected option from dropdownList.

The value of the selected `<dropdownListBlock>` in `<dropdownList>` can be obtained just like `<select>` tag.

#### jelly file

```
...
<ui:dropdownList title="Fruit" name="dropdownListSelected" tablename="dropdownListContent">
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

A certain option in <dropdownList> and its <dropdownListBlock> can be trigered by other UI controllers.

#### jelly file

```
<f:entry title="Select Fruit" field="preSelect">
  <f:select/>
</f:entry>
<ui:dropdownList title="Fruit" name="dropdownListSelected2" tablename="dropdownListContent2">
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
