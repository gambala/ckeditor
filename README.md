# Enhanced CKEditor for Rails

## Disclaimer

This is [my](https://github.com/gambala) personal fork of [galetahub/ckeditor](https://github.com/galetahub/ckeditor) with some additional plugins, themes and enhancements.

For a couple of years I have searched for ideal WYSIWYG editor. And finally I chose CKEditor with a bunch of plugins and handcrafted configs. I use it with a stable stack of other gems and versions (Rails 5, ActiveRecord, Carrierwave, Turbolinks 5). If your stack is different - you can see original [README.md](https://github.com/galetahub/ckeditor/blob/master/README.md) for proper setup guides.

## Features

* CKEditor version 4.6.1 full (08 Dec 2016)
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

2a. Change mount syntax style if you want:

```ruby
mount Ckeditor::Engine, at: '/ckeditor'
```

3. Set ckeditor assets as precompiled (in `config/initializers/assets.rb`):

```
Rails.application.config.assets.precompile += %w(
  ckeditor/config.js
  ckeditor/filebrowser/*
  ckeditor/plugins/*
  ckeditor/skins/moonocolor/*
)
```

4. Set ckeditor assets as non digest (in `config/initializers/non_digest_assets.rb`):

```
NonStupidDigestAssets.whitelist += [%r{ckeditor/.*}]
```

5. Add styles for `js-ckeditor` class, like these:

```
.js-ckeditor
  min-height: 271px
```

6. Add scripts for initializing with turbolinks and xhr requests:

```
#= require ckeditor/loader
#= require ckeditor/plugins/codesnippet/lib/highlight/highlight.pack

$(document).bind 'turbolinks:before-cache ajax:beforeSend', ->
  return unless typeof(CKEDITOR) != 'undefined'
  for name of CKEDITOR.instances
    $(CKEDITOR.instances[name].editable().$).trigger('ckeditor:before-destroy')
    CKEDITOR.instances[name].destroy() if name?

$(document).bind 'turbolinks:load ajax:success', ->
  return unless typeof(CKEDITOR) != 'undefined'
  $('.js-ckeditor').each ->
    CKEDITOR.replace($(this).attr('id'))
```

5. Add custom `ckeditor/loader.js.erb`:

```
//= require ckeditor/init

CKEDITOR.config.customConfig = '<%= javascript_path 'ckeditor/config.js' %>';
```

6. Add custom `ckeditor/config.coffee`:

[See the code](https://github.com/gambala/gambala/blob/master/app/assets/javascripts/ckeditor/config.coffee).

## Using

1. Add `js-ckeditor` class to your textarea:

```
= f.text_area :text, class: 'form-control ui-input js-ckeditor', rows: 4
```
