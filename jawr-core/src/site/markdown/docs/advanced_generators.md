Jawr Advanced Generators
------------------------

Please make sure to read the [Jawr Generators documentation](./generators.html) before this page.  
In the [Jawr Generators documentation](./generators.html), we have seen
how to implement a basic generator.  
Here we will see advanced features about the generators.

### Initialization of a generator

In the Jawr configuration file, we only define the class name of the
generator.  
Jawr provides some interfaces to initialize the generator.

![Initializing Generator class diagram](../images/generator/jawr\_initializingGenerator.png)
In the class diagram above, we can see that there are 3 main interfaces
for generator initialization.

-   **net.jawr.web.resource.bundle.generator.ConfigurationAwareResourceGenerator**  
    This interface provides a method to set the Jawr configuration in
    the generator.
   
                public void setConfig(JawrConfig config);

   
-   **net.jawr.web.resource.bundle.generator.TypeAwareResourceGenerator**  
    This interface provides a method to set the type of resource, which
    the generator will handle. This interface is useful if you are using
    one generator class to generate different type of resources (JS
    and CSS).

   For example : For a resource generator which is able to handle Javascript files and CSS files.  
   If the type is "js" then the debugModePath will be "/jawr\_generator.js"
    else if the type is "css" the debugModePath will be "/jawr\_generator.css"


                public void setResourceType(String resourceType);


-   **net.jawr.web.resource.bundle.generator.PostInitializationAwareResourceGenerator**  
    This interface provides a method, which is called at the end of the
    generator initialization.

                public void afterPropertiesSet();

### Bundle processing aware generator

Jawr allows you to create generators, which can listen to bundle processing event (start and end of the process).  
For this, the generator should implements the following interface 
-   **net.jawr.web.resource.bundle.lifecycle.BundlingProcessLifeCycleListener**  
    This interface provides two methods, which are called during the bundling process.

               /**
	 			  * This method is called before the bundling process 
	 			 */
				public void beforeBundlingProcess();
	
	
				/**
				 * This method is called after the bundling process 
				 */
				public void afterBundlingProcess();

    
### Locale aware generator

Jawr allows you to create generators, which can create resources
depending on the user locale.  
There is one locale aware generator, which is already provided In Jawr,
and which handles i18n message resources :

-   **i18n messages script generator** generates javascript code with
    locale specific variants which are loaded depending on the user's
    specific language. Full documentation is
    [here](./messages_gen.html).

So Jawr provides a mechanism to define a generator which can generate
resources with locale specific variants.
You will find below the class diagram for the locale aware resource
generator.

![Locale Resource Generator class diagram](../images/generator/jawr\_variantResourceGenerator.png)

-   **net.jawr.web.resource.bundle.generator.LocaleAwareResourceGenerator**  
    This interface provides a method, which is called to know what are
    the available locale variants for a path.

                public List getAvailableLocales(String mapping);


   For example, if you have 3 differents locale variant for a CSS
    stylesheet like:  
    general.css, general\_fr.css, general\_en.css

   Your generator must return a list of String corresponding to the
    available locale variants, in our case : ("", "fr", "en")


### Variant Resource generator

Since the version 3.3, Jawr is able to generate variant bundle contents.
Like for locale aware bundles, Jawr is able to generate different bundle
depending on request header, attributes, parameters, cookies...
So Jawr is for example able to generate bundle depending on the browser.

![Variant Bundle generator class diagram](../images/variants/variantBundleGenerator.jpg)
-   **net.jawr.web.resource.bundle.generator.variant.VariantResourceGenerator**  
    This interface provides a method, which is called to know what are
    the available variants for a path.


                /**
                 * Returns the map of available variant for a resource.
                 * The key of the map is the type of variant (for ex: locale, skin...)
                 * The values associated are the list of variant for the type. 
                 * @param resource the resource name
                 * @return the map of available variant for a resource 
                 */
                Map getAvailableVariants(String resource);


   This method defines the map of variants available for a
    specific resource. For the returned map of variants, the key is the
    variant type, and the value is the VariantSet.

   For more information about variant bundle, please take a look at the
    [documentation about variant bundle](./variant_bundle.html).

### CSS Resource generator

As for every resource, Jawr allows you to create a generator for the CSS
files.  
One of the particularity of the CSS resources, is that they have
references to images.  

With Jawr, you are free to define a CSS Resource generator, which is
also an Image Resource generator.  If your CSS generator is able to
handle generated image resources, it must implements
**net.jawr.web.resource.bundle.generator.StreamResourceGenerator**

For example, you can have a CSS resource which is defined in the
classpath. This CSS can have references to images which are also
retrieved from the classpath.
So here you define that your CSS resource generator will also handle CSS
images.
All references to images will be handled by the CSS resource generator.

You will find below the class diagram for Css Resource Generator
[../images/generator/jawr\_cssGenerator.png] CSS Generator class
diagram

-   **net.jawr.web.resource.bundle.generator.CssResourceGenerator**  
    This interface is implemented by the resource generator, which
    generates CSS resources.

                public boolean isHandlingCssImage();


   This method must return true if the CSS image defined in the
    generated CSS resource are handled by the CSS generator.

   For example, If **myCssGen** is the prefix of the resource
    generator, all CSS image which are defined in the CSS resources,
    will be prefixed by **myCssGen**.
    If in the generated CSS resource, a CSS image is referenced as

   **background-image : url('/myImg/temp/myIcon.png')**

   Then, the image path will be interpreted as :

   **background-image : url('myGen:/myImg/temp/myIcon.png')**

   **Important note :** If your generator handle CSS resources, you are
    not forced to implement the
    **net.jawr.web.resource.bundle.generator.CssResourceGenerator** interface.
    You will need implement it only if your CSS generator is able to
    handle CSS images.

### Generator Resource mapping

There are three ways to map resources:

-   Directly by resource path: simply type the relative path to the
    resource, starting at the root of the WAR file, as in: '/js/foo.js'.
-   By directory, non-recurring: add the path to a directory and every
    resource under it will be added to the bundle, as in: '/js/'.
-   By directory, recurring: same as before but adding '\*\*' at
    the end. This will add every resource under the specified directory
    and any directory below it. For example: '/js/\*\*'.

   For example:

                jawr.js.bundle.myBundle.id=/bundles/myBundle.js
                jawr.js.bundle.myBundle.mappings=/js/one.js,/js/tabview/**


   In the above example, we have referenced the resource **/js/on.js**
    directly, and we have also put all js files available under
    **/js/tabview/** folder.

   With the resource generator, you can achieve the same behaviours. By
    default, the resource generator is only able to handle the direct
    reference to a resource path, like in the example below :


                jawr.js.bundle.myBundle.id=/bundles/myBundle.js
                jawr.js.bundle.myBundle.mappings=myGen:/js/one.js

 
   If you want to use the *directory* syntax for generated resource,
    your generator must implement the interface
    **net.jawr.web.resource.handler.reader.ResourceBrowser**, like the
    **GeneratedResourceBrowser** interface defined below.

   ![Resource Browser interface](../images/generator/jawr\_resourceBrowserGenerator.png)

   -   **net.jawr.web.resource.handler.reader.ResourceBrowser**
        This interface is implemented by the generators, which are able
        to handle hierarchical generated resource search.


                    public Set getResourceNames(String path);


   This method returns a list of resources at a specified path
        within the resources directory.


                    public boolean isDirectory(String path);

   Determines whether a given path is a directory.

### Debug mode for generated Resource deployed in CDN

Jawr allows you to reference bundled resources deployed in a content
delivery network (CDN).  
To achieve this, you need to use the properties
**jawr.url.contextpath.override** and **jawr.url.contextpath.ssl.override**.  
Even in debug mode, Jawr allows you to reference resources in a CDN
using the property
**jawr.url.contextpath.override.used.in.debug.mode**.  

Jawr helps you to create the resources and the structure of the files to
put in your CDN using the [build time processing feature](./bundle_preprocessing.html).
When using the [build time processing feature](./bundle_preprocessing.html) allows you to define the path used for storing the generated resources.  

To define the path of a generated resource in debug mode, the generator
must implement a specific interface.

-   **net.jawr.web.resource.bundle.generator.SpecificCDNDebugPathResourceGenerator**

   This interface is implemented by the resource generators which are
    able to define a specific path for the debug mode in the **build time bundle processing**.


                public String getDebugModeBuildTimeGenerationPath(String parameter);


   This methods must return the path to use when creating a generated
    resource in debug mode during the **build time bundle processing**.

