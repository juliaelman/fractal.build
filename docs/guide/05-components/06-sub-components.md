---
label: Including sub-components
title: Including sub-components
---

Often you will want to include one component within another component. For instance, you may have a 'button' component that you want to include as a sub-component of a 'search box' component (as well as potentially within a number of other components).

Using the default Handlebars template engine, that might look something like this:

{% raw %}
```handlebars
<!-- button.hbs -->
<button type="button" name="submit">Search</button>
```

```handlebars
<!-- search-box.hbs -->
<div class="searchbox">
    <input type="search" name="keywords">
    {{> @button }}
</div>
```
{% endraw %}

As you can see, the `search-box.hbs` component view file uses the standard [Handlebars partial include](http://handlebarsjs.com/#partials) syntax to include the button template, with one difference - instead of using a *path* to the template, it uses the {{ link('@naming#referencing', '`@handle` syntax') }} to refer to the button component by it's handle.

It's important to note that the syntax for including one component's view template within another **will depend on which {{ link('@template-engines', 'template engine') }} you are using**. For instance, if you were using the [Nunjucks engine](https://github.com/frctl/nunjucks) you would instead use Nunjuck's `include` tag to include sub components. For example:
{% raw %}
```html
<!-- parent.nunj -->
<div class="parent">
    {% include '@child' %}
</div>
```
{% endraw %}
## Providing context data to sub-components

When you include a sub-component in the manner described above, it's important to note that you are effectively just including the contents sub-component's view file. **It will not automatically include any {{ link('@context-data', 'context data') }} that you may have defined for that sub-component.**

To handle passing context data for your included sub-components, you have a number of options:

* A) Use the {{ link('@views#handlebars-helpers', 'render helper') }} to render the sub-component in-place, using it's own context data.
* B) Define a complete set of context data in the parent component's config file
* C) Reference context data for sub-components using the '@handle' {{ link('@naming#referencing', 'static data reference syntax') }}  from within the parent component's context data.

Let's look at how each of these might work in turn.

### A) Using the `render` helper

Both the default {{ link('@views#template-rendering', 'Handlebars template engine') }} and the [Nunjucks engine](https://github.com/frctl/nunjucks#helpers) come with a `render` helper to use in your templates, instead of the standard partial/include syntax.

Unlike the standard 'include' syntax, the `render` helper renders a sub-component in-place, using it's own context data (unless overridden by data passed into it). This is a good solution if you want to **render a sub-component with it's own context data**, AND you don't mind using a custom helper function to handle the rendering.

> If you are planning on consuming your template files outside of Fractal, you may **not** want to use a custom helper if it is will not be supported in your target environment.

An example might look like this:
{% raw %}
```handlebars
<!-- button.hbs -->
<button type="button" name="submit">{{ button.text }}</button>
```

```yaml
# button.config.yml
context:
  button:
    text: A Button
```

```handlebars
<!-- search-box.hbs -->
<div class="searchbox">
    <label for="search">{{ label }}</label>
    <input type="search" name="keywords" id="search">
    {{ render '@button' }}
</div>
```

```yaml
# search-box.config.yml
context:
  label: Search
```


You can see in this case, we don't need to specify any context data for the `button` sub-component in the `search-box.config.yml` - instead the `{{ render }}` helper will automatically render the `button` using the context data defined in it's own `button.config.yml`.

{% endraw %}

You can also override all or a just a sub-set of the sub-component's default context data - see the {{ link('@views#handlebars-helpers', 'Handlebars helpers') }} or the [Nunjucks helpers](https://github.com/frctl/nunjucks) documentation for more info.

### B) Define all context data in the parent component's config

This is a good approach for when you want to render an instance of the sub-component that should use a **different set of context data** to that which is provided in the sub-component's config file.

For example, you may want to render a 'prose' component on it's own with a set of Lorem Ipsum default content, but when pulled into a 'homepage' component you may want to override that with page-specific content.

Using this method, our example would look like this:

{% raw %}
```handlebars
<!-- button.hbs -->
<button type="button" name="submit">{{ button.text }}</button>
```

```yaml
# button.config.yml
context:
  button:
    text: A Button
```

```handlebars
<!-- search-box.hbs -->
<div class="searchbox">
    <label for="search">{{ label }}</label>
    <input type="search" name="keywords" id="search">
    {{> @button }}
</div>
```

```yaml
# search-box.config.yml
context:
  label: Search    
  button:
    text: Go!
```
You can see that in this case, the parent (`search-box`) component is providing a whole new set of context data, including the `button` object with it's `text` property. The rendered output of both of these components would look like:

```html
<!-- @button -->
<button type="button" name="submit">A Button</button>
```

```html
<!-- @search-box -->
<div class="searchbox">
    <label for="search">Search</label>
    <input type="search" name="keywords" id="search">
    <button type="button" name="submit">Go!</button>
</div>
```
So in this case the parent component is responsible for defining the context data used by both itself *and* and data needed by any sub-components.

{% endraw %}

### C) Use context data '@handle' references

If you *don't* want to specify new data for all the sub-components in the parent's configuration file (and you don't want to use the `render` helper) you can use the  '@handle' {{ link('@naming#referencing', 'static data reference syntax') }} to dynamically pull context from the sub-component's configuration in to the parent component's context.

If we re-work the example above to use this technique it would look like this:

{% raw %}
```handlebars
<!-- button.hbs -->
<button type="button" name="submit">{{ button.text }}</button>
```

```yaml
# button.config.yml
context:
  button:
    text: A Button
```

```handlebars
<!-- search-box.hbs -->
<div class="searchbox">
    <label for="search">{{ label }}</label>
    <input type="search" name="keywords" id="search">
    {{> @button }}
</div>
```

```yaml
# search-box.config.yml
context:
  label: Search    
  button: "@button"
```
{% endraw %}

The last line in the `search-box.config.yml` file (`button: "@button"`) ensures that the `button` context data property will be set to the value of the context data object defined in the button component; this is a reference that is resolved when the parent component is rendered.

In this case, when rendered the components will look like:

{% raw %}
```html
<!-- @button -->
<button type="button" name="submit">A Button</button>
```

```html
<!-- @search-box -->
<div class="searchbox">
    <label for="search">Search</label>
    <input type="search" name="keywords" id="search">
    <button type="button" name="submit">A Button</button>
</div>
```
{% endraw %}

You can see that in the `@search-box` component the button text is now the same as the text defined in the `button.config.yml` file.

See the documentation on {{ link('@context-data#data-references', 'referencing other component\'s data') }} for more information on using this technique.