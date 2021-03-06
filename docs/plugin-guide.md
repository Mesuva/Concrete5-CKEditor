# Plugin Guide

In this guide we will outline how to create a plugin which is available to be used with the community CKEditor package.

## Creating a CKEditor Plugin

Since CKEditor already has a guide on how to create a plugin we'll just point you at 
[their documentation](http://docs.ckeditor.com/#!/guide/plugin_sdk_sample_1). However, you should not place your plugin
within the CKEditor vender folder in this package we'll show you how to set that up in the next step here.

## Creating a package for your plugin

In our example we will create a package called "Example CKEditor Plugin". So to start we'll need the following:

### /packages/base_cke_plugin/assets/base_cke_plugin/plugin.js

```js
CKEDITOR.plugins.add( 'base_cke_plugin', {
    init: function( editor ) {
        alert("THIS IS AN EXAMPLE");
        // Plugin logic goes here...
    }
});
```

### /packages/base_cke_plugin/assets/base_cke_plugin/register.js

The register.js file is in charge of telling CKEditor where our `ckeditor_plugin` resides. We need this because
CKEditor loads all of the plugin assets on it's own, it just needs to be told what plugins to load. This file allows us
to associate a plugin key to a specific path. Notice this is not something you normally need for most standard CKEditor
plugins, this is something we're adding so that we can get a CKEditor plugin, to work with the concrete5 asset manager.

```js
if (typeof CKEDITOR !== 'undefined') {
    CKEDITOR.plugins.addExternal(
        'concrete5inline', 
        CCM_REL + '/packages/base_cke_plugin/assets/base_cke_plugin/'
    );
}
```

### /packages/base_cke_plugin/controller.php

Finally we set up our package controller. In the on_start we register our 

```php
<?php
namespace Concrete\Package\BaseCkePlugin;

use Concrete\Core\Editor\Plugin;
use Core;

class Controller extends Package
{

    protected $pkgHandle = 'base_cke_plugin';
    protected $appVersionRequired = '5.7.5';
    protected $pkgVersion = '0.9.0';


    public function getPackageName()
    {
        return t('Base CKEditor Plugin');
    }

    public function getPackageDescription()
    {
        return t('A Basic CKEditor Example Plugin');
    }
    
    public function on_start()
    {
        $this->registerPlugin();
    }
    
    protected function registerPlugin()
    {        
        $assetList = \AssetList::getInstance();
        //register our register.js asset
        $assetList->register(
            'javascript',
            'editor/ckeditor/base_cke_plugin',
            'assets/base_cke_plugin/register.js',
            array(),
            $this->pkgHandle
        );

        //add our register.js asset to a group
        $assetList->registerGroup(
            'editor/ckeditor/base_cke_plugin',
            array(
                array('javascript', 'editor/ckeditor/base_cke_plugin')
            )
        );
        
        //associate our register.js group to the plugin
        $plugin = new Plugin();
        $plugin->setKey('base_cke_plugin');
        $plugin->setName(t('Base CKEditor Plugin'));
        $plugin->requireAsset('base_cke_plugin'); 
        Core::make('editor')->getPluginManager()->register($plugin);
    }
} 
```