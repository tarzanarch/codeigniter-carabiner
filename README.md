# Carabiner

Carabiner is a library for managing JavaScript and CSS assets. It's different from others that currently exist (for CodeIgniter, anyway) in that it acts differently depending on whether it is in a *production* or *development* environment. In a production environment, it will combine, minify, and cache assets. (As files are changed, new cache files will be generated.) In a development environment, it will simply include references to the original assets.

## Requirements

Carabiner requires the [JSMin](http://codeigniter.com/forums/viewthread/103039/) and [CSSMin](http://codeigniter.com/forums/viewthread/103269/) libraries that I previously ported. They're both included in this release. You don't need to load them, unless you'll be using them elsewhere. Carabiner will load them automatically as needed. *(Note: the only reason they're included as separate libraries is that it allows you to use them independently of Carabiner. If desired, you could probably include them in the carabiner.php file itself. You'd have to edit the Carabiner functions, but it could work.)*


## GZIP, Packer, etc.

Carabiner does not implement GZIP encoding, because I think that the web server should handle that. I think GZIPing is important, I just prefer not to do it PHP. If you need GZIP in an Asset Library, [AssetLibPro](http://code.google.com/p/assetlib-pro/) does it. I've also chosen not to implement any kind of JavaScript obfuscation (like [packer](http://dean.edwards.name/packer/)), primarily because of the client-side decompression overhead. More about this idea from [John Resig](http://ejohn.org/blog/library-loading-speed/). However, that's not to say you can't do it. You can easily provide a production version of a script that is packed. However, note that combining a packed script with minified scripts could cause problems. In that case, you can flag it to be not combined. (See usage below) Also, that's not to say that I won't port a [PHP Version](http://joliclic.free.fr/php/javascript-packer/en/) and include it eventually. 


## Inspiration/License

Carabiner is inspired by [Minify](http://code.google.com/p/minify/), [PHP Combine](http://rakaz.nl/extra/code/combine/) by Niels Leenheer and [AssetLibPro](http://code.google.com/p/assetlib-pro/) by Vincent Esche, among other things. Carabiner is released under a [BSD License](http://www.opensource.org/licenses/bsd-license.php).


## Usage

1. Load the library as normal: `$this->load->library('carabiner');`

Configuration can happen in either a config file, or by passing an array of values to the config() method. Config options passed to the config() method will override options in the config file.

See the included config file for more info.

To configure Carabiner using the config() method, do this:

    $carabiner_config = array(
        'script_dir' => 'assets/scripts/', 
        'style_dir'  => 'assets/styles/',
        'cache_dir'  => 'assets/cache/',
        'base_uri'   => $base,
        'combine'    => TRUE,
        'dev'        => FALSE
    );

    $this->carabiner->config($carabiner_config);


There are 8 relevant configuration options. The first 3 are required for Carabiner to function, the last 5 are not.

**script_dir**  
STRING Path to the script directory. Relative to the CI front controller (index.php)
  
**style_dir**  
STRING Path to the style directory. Relative to the CI front controller (index.php)
  
**cache_dir**  
STRING Path to the cache directory. Must be writable. Relative to the CI front controller (index.php)

**base_uri**  
STRING Base uri of the site, like 'http://www.example.com/'. Defaults to base_url config setting from main CodeIgniter config.

**dev**  
BOOLEAN Flags whether your in a development environment or not. See above for what this means. Defaults to FALSE.
  
**combine**  
BOOLEAN Flags whether to combine files. Defaults to TRUE.
  
**minify_js**  
BOOLEAN Flags whether to minify javascript. Defaults to TRUE.

**minify_css**  
BOOLEAN Flags whether to minify CSS. Defaults to TRUE.



Add assets like so:

    // add a js file
    $this->carabiner->js('scripts.js');
      
    // add a css file
    $this->carabiner->css('reset.css');
      
    // add a css file with a mediatype
    $this->carabiner->css('admin/print.css','print');


To set a (prebuilt) production version of an asset:

    //JS: pass a second string to the method with a path to the production version
    $this->carabiner->js('wymeditor/wymeditor.js', 'wymeditor/wymeditor.pack.js' );

    // CSS: add a css file with prebuilt production version
    $this->carabiner->css('framework/type.css', 'screen', 'framework/type.pack.css');


And to prevent a file from being combined:

    // JS: pass a boolean FALSE as the third attribute of the method
    $this->carabiner->js('wymeditor/wymeditor.js', 'wymeditor.pack.js', FALSE );

    // CSS: pass a boolean FALSE as the fourth attribute of the method
    $this->carabiner->css('framework/type.css', 'screen', 'framework/type.pack.css', FALSE);


You can also pass arrays (and arrays of arrays) to these methods. Like so:

    // a single array
    $this->carabiner->css( array('mobile.css', 'handheld', 'mobile.prod.css') );

    // an array of arrays
    $js_assets = array(
        array('dev/jquery.js', 'prod/jquery.js'),
        array('dev/jquery.ext.js', 'prod/jquery.ext.js')
    )

    $this->carabiner->js( $js_assets );

Carabiner is smart enough to recognize URLs and treat them differently:

    // this now works as expected
    $this->carabiner->js('http://ajax.googleapis.com/ajax/libs/jquery/1.3.2/jquery.js');

Carabiner supports grouping of assets. CSS and JS can be assigned to the same group. To define a group, do this:

    // Define JS
    $js = array(
        array('prototype.js'),
        array('scriptaculous.js')
    );

    // create group
    $this->carabiner->group('prototaculous', array('js'=>$js) );

    // an IE only group
    $css = array('iefix.css');
    $js = array('iefix.js');
    $this->carabiner->group('iefix', array('js'=>$js, 'css'=>$css) );

    // you can even assign an asset to a group individually 
    // by passing the group name to the last parameter of the css/js functions
    $this->carabiner->css('spec.css', 'screen', 'spec-min.css', TRUE, FALSE, 'spec');

For the sake of clarity, assets that aren't assigned a group are put in a group called 'main', and using the display function mentioned below (using the first 3 examples) will output this 'main' group only. New groups must be displayed independently as described below.

To output your assets, including appropriate markup:

    // display css
    $this->carabiner->display('css');
      
    //display js
    $this->carabiner->display('js');

    // display both
    $this->carabiner->display(); // OR $this->carabiner->display('both');

    // display a group
    $this->carabiner->display('groupname');


Finally, since Carabiner won't delete old cached files, you'll need to clear them out manually. To do so programatically:

    // clear css cache
    $this->carabiner->empty_cache('css');
        
    //clear js cache
    $this->carabiner->empty_cache('js');
        
    // clear both
    $this->carabiner->empty_cache();
    // clear both
    $this->carabiner->empty_cache(); // OR $this->carabiner->empty_cache('both');

    // you can also pass a string as the second parameter
    // that defines a time before which cached files will be removed
    $this->carabiner->empty_cache('both', 'yesterday');