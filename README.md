[![Banner of Sylius Rich Editor plugin](docs/images/banner.jpg)](https://monsieurbiz.com/agence-web-experte-sylius)

<h1 align="center">Rich Editor</h1>

[![Rich Editor Plugin license](https://img.shields.io/github/license/monsieurbiz/SyliusRichEditorPlugin?public)](https://github.com/monsieurbiz/SyliusRichEditorPlugin/blob/master/LICENSE.txt) [![Tests](https://github.com/monsieurbiz/SyliusRichEditorPlugin/actions/workflows/tests.yaml/badge.svg)](https://github.com/monsieurbiz/SyliusRichEditorPlugin/actions/workflows/tests.yaml) [![Security](https://github.com/monsieurbiz/SyliusRichEditorPlugin/actions/workflows/security.yaml/badge.svg)](https://github.com/monsieurbiz/SyliusRichEditorPlugin/actions/workflows/security.yaml) [![Flex Recipe](https://github.com/monsieurbiz/SyliusRichEditorPlugin/actions/workflows/recipe.yaml/badge.svg)](https://github.com/monsieurbiz/SyliusRichEditorPlugin/actions/workflows/recipe.yaml)

This plugin adds a rich editor on the fields you want. Now you can manage your content very easily!

![Example of rich editor field](docs/images/screenshots/demo.gif)

## Installation

```bash
composer require monsieurbiz/sylius-rich-editor-plugin
```

Change your `config/bundles.php` file to add the line for the plugin : 

```php
<?php

return [
    //..
    MonsieurBiz\SyliusRichEditorPlugin\MonsieurBizSyliusRichEditorPlugin::class => ['all' => true],
];
```

Then create the config file in `config/packages/monsieurbiz_sylius_rich_editor_plugin.yaml` :

```yaml
imports:
    - { resource: "@MonsieurBizSyliusRichEditorPlugin/Resources/config/config.yaml" }
```

Finally import the routes in `config/routes/monsieurbiz_sylius_rich_editor_plugin.yaml` : 

```yaml
monsieurbiz_richeditor_admin:
    resource: "@MonsieurBizSyliusRichEditorPlugin/Resources/config/routing/admin.yaml"
    prefix: /%sylius_admin.path_name%
```

And install the assets
```yaml
bin/console asset:install
```

## Use the Rich Editor

### Update your form type

To make a field use the rich editor, you must use the `RichEditorType` type for it.  
We have an example of implementation in the [file for the test application](/dist/src/Form/Extension/ProductTypeExtension.php).

If your field has some data already, like some previous text before installing this plugin, 
then we will convert it for you as an HTML Element which contains… HTML.

![Example of a rich editor field](docs/images/screenshots/form_field.png)

This way you will be able to use our plugin right away without risking any data lost!

To make it clear: to have rich editor for original admin fields, like _Product description_, _Taxon desription_ etc. you have to update each form type by an extension as the example shows above.

### Call twig render

To display the content of the rich editor field you must call the twig filter:

```twig
{{ content | monsieurbiz_richeditor_render_field }}
```

You can see an example in the [file for the test application](/dist/templates/bundles/SyliusShopBundle/Product/Show/Tabs/Details/_description.html.twig)

### Custom rendering of elements

You can also render your elements with some custom DOM around that. To do so, you have access to a twig filter that 
gives you the elements list :

```twig
{% set elements = monsieurbiz_richeditor_get_elements(content) %}
```

And then you can either render them all : 

```twig
{{ elements|monsieurbiz_richeditor_render_elements }}
```

Or one by one :
```twig
{% for element in elements %}
    {{ element|monsieurbiz_richeditor_render_element }}
{% endfor %}
```

### Filter the elements

If you want to filter the elements which are available for your field, you can use the `tags` option when you build your form.  
As example:

```php
$builder->add('description', RichEditorType::class, [
    'required' => false,
    'label' => 'sylius.form.product.description',
    'tags' => ['-default', 'product'],
]);
```

In that example, all Ui Elements with the tag `default` will be excluded, then the Ui Elements with the tag `product` will be included.  
Don't worry, you can add this filter afterwards, we won't remove the already present Ui Elements of your field. But we
won't allow to add more if they don't have one of the allowed tags!

#### Order matters

The order of the tags matters! The first tag will be less important then the second.  
As example, if you have 3 Ui Elements with the following tags:

- element1: tag1, tag2, tag3
- element2: tag1, tag2
- element3: tag2, tag3

And you set the tags of your field to `-tag1, tag2, -tag3`, then the only available Ui Element will be: `element2`.  
They all have the tag2 to include them, but the element1 and element3 have the tag3 which is excluding them after being both included.

#### Example of setting tags to an Ui Element using yaml

```yaml
monsieurbiz_sylius_richeditor:
    ui_elements:
        app.my_element:
            # … 
            tags: ['product']
```

### Deactivate an available element

Here is what really happens when deactivating an Ui Element:
- it's not displayed anymore in frontend
- it's still editable in backend for old contents but you can't add a new one
- if the element has an alias, the alias is treated the same way

Define the overload of a proposed UiElement in your configuration folder, let's say in `config/packages/monsieurbiz_sylius_richeditor_plugin.yaml` as example.

```yaml
monsieurbiz_sylius_richeditor:
    ui_elements:
        monsieurbiz.youtube:
            enabled: false
```

## Available elements

### Row Element (Layout)

It contains another rich editor field, each element will be displayed one below the other (as rows…).

### Column Element (Layout)

It contains another rich editor field, each element will be displayed side by side (as columns…).

By using this element in the `Row` element, you will be able to build some layout like the screenshot below.  
You can distinguish the `Row` element and the `Column` element by their dotted borders.

![The Columns and Rows elements](docs/images/screenshots/columns_and_rows.jpg)

### HTML Element

![The HTML element](docs/images/screenshots/html.png)

### Text element

![The text element](docs/images/screenshots/text.png)

### Quote element

![The quote element](docs/images/screenshots/quote.png)

### Image element

![The image element](docs/images/screenshots/image.png)

### Video element

![The video element](docs/images/screenshots/video.png)

### Button element

![The button element](docs/images/screenshots/button.png)

### Title element

![The title element](docs/images/screenshots/title.png)

### Separator element

![The separator element](docs/images/screenshots/separator.png)

### Youtube element

![The Youtube element](docs/images/screenshots/youtube.png)

### Image collection element

![The Image collection element](docs/images/screenshots/image_collection.png)

## Example of a rich product description

### Admin form with preview

![Admin full form](docs/images/screenshots/full_back.png)

### Front display

![Front full display](docs/images/screenshots/full_front.png)

## Create your own elements

In this example, we will add a Google Maps element.

### Define your UiElement

Define your UiElement in your configuration folder, let's say in `config/packages/monsieurbiz_sylius_richeditor_plugin.yaml` as example.

```yaml
monsieurbiz_sylius_richeditor:
    ui_elements:
        app.google_maps:
            title: 'app.ui_element.google_maps.title'
            description: 'app.ui_element.google_maps.description'
            icon: map pin
            classes:
                form: App\Form\Type\UiElement\GoogleMapsType
                #ui_element: App\UiElement\MyUiElement
            templates:
                admin_render: '/Admin/UiElement/google_maps.html.twig'
                front_render: '/Shop/UiElement/google_maps.html.twig'
            tags: []
```

You can use your own Ui Element object if needed. Be sure to implement the
`\MonsieurBiz\SyliusRichEditorPlugin\UiElement\UiElementInterface` interface.  
A trait is there for you 🤗 as well. This is very useful when you need to do some custom work in your templates, it's like
having a helper around. The Ui Element is then available via the `ui_element` variable in your templates.

### Create the Form Type we use in admin to fill your UiElement

```php
<?php

declare(strict_types=1);

namespace App\Form\Type\UiElement;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Validator\Constraints as Assert;

class GoogleMapsType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('link', TextType::class, [
                'label' => 'app.ui_element.google_maps.link',
                'required' => true,
                'constraints' => [
                    new Assert\NotBlank(),
                ],
            ])
        ;
    }
}
```

### Add your translations of course

Here is an example of possible translation for the GMap element : 

```yaml
app:
    ui_element:
        google_maps:
            title: 'Google Maps Element'
            short_description: 'Include a Google Maps'
            description: 'An element with a Google Maps link'
            link: 'Link'
```

### Create the templates to render it in front and in admin

You have to create a template for the front and also for the admin's preview.

Here is an example of a simple template for this our which can be used in front and admin:

```twig
<iframe id="gmap_canvas" src="{{ element.link }}" scrolling="no" marginheight="0" marginwidth="0" width="600" height="500" frameborder="0"></iframe>
```


### The result !

#### The element is in the UI Elements list

![The Google Maps element](docs/images/screenshots/gmap_element.png)

#### You now have a form to edit it (your form!)

![The Google Maps form](docs/images/screenshots/gmap_form.png)

#### And we use your templates to render your UiElement

In admin : 

![The GMap display](docs/images/screenshots/gmap_render_admin.png)

In front : 

![The GMap display](docs/images/screenshots/gmap_render.png)

## File in fixtures management

In some cases you will want to add UI elements to your content fixtures which are Rich Editor fields. If you need files in your UI elements, you can use a dedicated fixture. It is used as follows.   

```yaml
sylius_fixtures:
    suites:
        my_project:
            fixtures:
                my_files:
                    name: monsieurbiz_rich_editor_file
                    options:
                        files:
                            -   source_path: 'src/Resources/fixtures/bar1.png'
                                target_path: 'image/foo/bar1.png'
                            -   source_path: 'src/Resources/fixtures/file.pdf'
                                target_path: 'baz/file.pdf'
```

The example below will copy files to `public/media/image/foo/bar1.png` and `public/media/foo/file.pdf`.

Now you can use files in your content fixtures:

```yaml
description: |
    [{
        "code": "app.my_ui_element",
        "data": {
          "title": "My title",
          "image": "/media/image/foo/bar1.png",
          "file": "/media/baz/file.pdf"
        }
    }]
```

## Wireframes

From version 2.4.2 you can add a wireframe to your UiElement. 
It will be displayed in the admin form to help the user to understand what the UiElement is about.
The file need to be svg with a `.twig` extension.
You can add the files in the folder : `templates/MonsieurBizSyliusRichEditorPlugin/Wireframe/*.svg.twig`
In the YAML declaration of a UI Element, you can add the wireframe key with the name of the file without the extension.
```yaml
    monsieurbiz.title:
        alias: title
        title: 'monsieurbiz_richeditor_plugin.ui_element.monsieurbiz.title.title'
        description: 'monsieurbiz_richeditor_plugin.ui_element.monsieurbiz.title.description'
        icon: heading
        wireframe: title
        tags: [ default ]
        classes:
            form: MonsieurBiz\SyliusRichEditorPlugin\Form\Type\UiElement\TitleType
        templates:
            admin_render: '@MonsieurBizSyliusRichEditorPlugin/Admin/UiElement/title.html.twig'
            front_render: '@MonsieurBizSyliusRichEditorPlugin/Shop/UiElement/title.html.twig'
```



## Contributing

You can open an issue or a Pull Request if you want! 😘  
Thank you!
