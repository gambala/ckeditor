# Enhanced CKEditor for Rails

## Disclaimer

This is [my](https://github.com/gambala) personal fork of [galetahub/ckeditor](https://github.com/galetahub/ckeditor) with some additional plugins, themes and enhancements.

For a couple of years I have searched for ideal WYSIWYG editor. And finally I chose CKEditor with a bunch of plugins and handcrafted configs.

## Features

* Ckeditor version 4.5.11 (7 Sep 2016)
* Rails 5.0.x, 4.2.x integration
* Files browser
* HTML5 file uploader

## Installation

1. Add gems into Gemfile:

```
gem 'carrierwave'
gem 'ckeditor', github: 'gambala/ckeditor'
gem 'mini_magick'
gem 'non-stupid-digest-assets'
```

2. Generate CKEditor configs, uploaders and models

```
rails generate ckeditor:install --orm=active_record --backend=carrierwave
```

3. Create initializers

```
# config/initializers/non_digest_assets.rb

NonStupidDigestAssets.whitelist += [/ckeditor\/.*/]
```

4. Mount Ckeditor::Engine in your routes (config/routes.rb):

```ruby
mount Ckeditor::Engine, at: '/ckeditor'
```

5. Include ckeditor javascripts in your `app/assets/javascripts/application.js`:

```
//= require ckeditor/init
```

6. Precompile ckeditor/config.js:

```ruby
# config/initializers/assets.rb

Rails.application.config.assets.precompile += %w(ckeditor/config.js)
```

### Form helpers

```slim
= form_for @page do |form|
  = form.cktext_area :notes, class: 'someclass', ckeditor: { language: 'uk'}
  = form.cktext_area :content, value: 'Default value', id: 'sometext'
  = cktext_area :page, :info, cols: 40, ckeditor: { uiColor: '#AADC6E', toolbar: 'mini' }
```

### Customize ckeditor

All ckeditor options can be found [here](http://docs.ckeditor.com/#!/api/CKEDITOR.config)

In order to configure the ckeditor default options, create the following files:

```
app/assets/javascripts/ckeditor/config.js

app/assets/javascripts/ckeditor/contents.css
```

#### Custom toolbars example

Adding a custom toolbar:

```javascript
# in app/assets/javascripts/ckeditor/config.js

CKEDITOR.editorConfig = function (config) {
  // ... other configuration ...

  config.toolbar_mini = [
    ["Bold",  "Italic",  "Underline",  "Strike",  "-",  "Subscript",  "Superscript"],
  ];
  config.toolbar = "simple";

  // ... rest of the original config.js  ...
}
```

When overriding the default `config.js` file, you must set all configuration options yourself as the bundled `config.js` will not be loaded. To see the default configuration, run `bundle open ckeditor`, copy `app/assets/javascripts/ckeditor/config.js` into your project and customize it to your needs.

### Deployment (only if you use ckeditor from gem vendor)

For Rails 4, add the following to `config/initializers/assets.rb`:

```ruby
Rails.application.config.assets.precompile += %w( ckeditor/* )
```

As of version 4.1.0, non-digested assets of Ckeditor will simply be copied after digested assets were compiled.
For older versions, use gem [non-stupid-digest-assets](https://rubygems.org/gems/non-stupid-digest-assets), to copy non digest assets.

To reduce the asset precompilation time, you can limit plugins and/or languages to those you need:

```ruby
# in config/initializers/ckeditor.rb

Ckeditor.setup do |config|
  config.assets_languages = ['en', 'fr']
  config.assets_plugins = ['image', 'smiley']
end
```

Note that you have to list your plugins, including all their dependencies.

### Include customized CKEDITOR_BASEPATH setting

Add your app/assets/javascripts/ckeditor/basepath.js.erb like

```erb
<%
  base_path = ''
  if ENV['PROJECT'] =~ /editor/i
    base_path << "/#{Rails.root.basename.to_s}/"
  end
  base_path << Rails.application.config.assets.prefix
  base_path << '/ckeditor/'
%>
var CKEDITOR_BASEPATH = '<%= base_path %>';
```

### AJAX

jQuery sample:

```html
<script type='text/javascript' charset='UTF-8'>
  $(document).ready(function(){
    $('form[data-remote]').bind('ajax:before', function(){
      for (instance in CKEDITOR.instances){
        CKEDITOR.instances[instance].updateElement();
      }
    });
  });
</script>
```

### Formtastic integration

```slim
= form.input :content, as: :ckeditor
= form.input :content, as: :ckeditor, input_html: { ckeditor: { height: 400 } }
```

### SimpleForm integration

```slim
= form.input :content, as: :ckeditor, input_html: { ckeditor: { toolbar: 'Full' } }
```

### Turbolink integration
Create a file app/assets/javascripts/init_ckeditor.coffee

```coffee
ready = ->
  $('.ckeditor').each ->
  CKEDITOR.replace $(this).attr('id')

$(document).ready(ready)
$(document).on('page:load', ready) } }
```
Make sure the file is loaded from your app/assets/javascripts/application.js

### CanCan integration

To use cancan with Ckeditor, add this to an initializer:

```ruby
# in config/initializers/ckeditor.rb

Ckeditor.setup do |config|
  config.authorize_with :cancan
end
```

At this point, all authorization will fail and no one will be able to access the filebrowser pages.
To grant access, add this to Ability#initialize:

```ruby
# Always performed
can :access, :ckeditor   # needed to access Ckeditor filebrowser

# Performed checks for actions:
can [:read, :create, :destroy], Ckeditor::Picture
can [:read, :create, :destroy], Ckeditor::AttachmentFile
```

### Pundit integration

Just like CanCan, you can write this code in your config/initializers/ckeditor.rb file:

```ruby
Ckeditor.setup do |config|
  config.authorize_with :pundit
end
```

Then, generate the policy files for model **Picture** and **AttachmentFile**

```
$ rails g ckeditor:pundit_policy
```
By this command, you will got two files:
> app/policies/ckeditor/picture_policy.rb
app/policies/ckeditor/attachment_file_policy.rb

By default, only the user that logged in can access the models (with actions *index* and *create*) and only the owner of the asset can **destroy** the resource.

You can customize these two policy files as you like.

## Engine configuration
 * To override the default CKEditor routes create a [config.js](https://github.com/galetahub/ckeditor/blob/master/app/assets/javascripts/ckeditor/config.js) file within the host application at `app/assets/javascripts/ckeditor/config.js`
 * To override the default parent controller
```
# in config/initializers/ckeditor.rb

Ckeditor.setup do |config|
  config.parent_controller = 'MyController'
end
```

## I18n

```yml
en:
  ckeditor:
    page_title: 'CKEditor Files Manager'
    confirm_delete: 'Delete file?'
    buttons:
      cancel: 'Cancel'
      upload: 'Upload'
      delete: 'Delete'
      next: 'Next'
```

## Tests

```bash
$> rake test CKEDITOR_BACKEND=paperclip
$> rake test CKEDITOR_BACKEND=carrierwave
$> rake test CKEDITOR_BACKEND=refile
$> rake test CKEDITOR_BACKEND=dragonfly
$> rake test CKEDITOR_ORM=mongoid

$> rake test:controllers
$> rake test:generators
$> rake test:integration
$> rake test:models
```

This project rocks and uses the MIT-LICENSE.
