Jawr illegal bundle request handler
-----------------------------------

Since the version 3.3, Jawr introduced a new configuration setting which is the **strict mode**.  
This **strict mode** refers to the way how Jawr handles the bundle
request where the bundle exist in Jawr but the identifier of the bundle
(hashcode) doesn't match the existing one.

This case can happen when you have an application which runs on 2
servers, and that you're deploying a new version of your application on
one server, while the other is still running.  
It can happen that the user retrieve the content of the page from one
server, and try to retrieve the bundle resources from the other. If the
requested bundle has different between the 2 servers, Jawr will receive
a request for a bundle which exists but where the hashcode is
incorrect.  
If you set the strict mode property in your Jawr configuration file to
true, Jawr allows you to define the handler for such request. Here are
the properties which you can use to handle the strict mode :

| **Property name** | **Type** | **Purpose** | **Default value** |
|-------------------|----------|-------------|-------------------|
| jawr.strict.mode  | Boolean  | Enable/disable strict mode for bundle request. | False |
| jawr.illegal.bundle.request.handler | String | The class name of the handler for illegal bundle request. | net.jawr.web.servlet.IllegalBundleRequestHandlerImpl |

![Illegal bundle request handler class
diagram.](../images/illegalBundleRequestHandler/illegalBundleRequestHandler.jpg)
The IllegalBundleRequestHandler defines the following methods :


            /**
             * This method can update the response header and 
             * returns true if the header has been written 
             * and false if Jawr must write the response header.
             * @param requestedPath the requested path
             * @param request the request
             * @param response the response
             * @return true if the header has been written 
             * and false if Jawr must write the response header.
             * @throws IOException if an IOException occurs
             */
            boolean writeResponseHeader(String requestedPath, HttpServletRequest request, HttpServletResponse response) throws IOException;
            
            /**
             * This method should return true if Jawr should send back the content of the bundle.
             * @param requestedPath the requested path
             * @param request the request
             * @return true if if Jawr should send back the content of the bundle.
             */
            boolean canWriteContent(String requestedPath, HttpServletRequest request);
            


-   **boolean writeResponseHeader(String requestedPath,
    HttpServletRequest request, HttpServletResponse response)**  
    This method is used to update the response header is needed. It must
    returns true if the response header has been written and false to
    let Jawr handles the response header like for correct bundle.
-   **boolean canWriteContent(String requestedPath,
    HttpServletRequest request)**  
    This method is used to determine if Jawr should send back the
    matching bundle content or not.


### Jawr default IllegalBundleRequestHandler implementation

Jawr provides a default implementation of the IllegalBundleRequestHandler, which returns a 404 if the bundle hashcode doesn't match.


            /* (non-Javadoc)
             * @see net.jawr.web.servlet.IllegalBundleRequestHandler#writeResponseHeader(java.lang.String, javax.servlet.http.HttpServletRequest, javax.servlet.http.HttpServletResponse)
             */
            public boolean writeResponseHeader(String requestedPath,
                            HttpServletRequest request, HttpServletResponse response) throws IOException {
                    
                    response.sendError(HttpServletResponse.SC_NOT_FOUND);
                    return true;
            }

            /* (non-Javadoc)
             * @see net.jawr.web.servlet.IllegalBundleRequestHandler#canWriteContent(java.lang.String, javax.servlet.http.HttpServletRequest)
             */
            public boolean canWriteContent(String requestedPath,
                            HttpServletRequest request) {
                    return false;
            }


### Custom IllegalBundleRequestHandler implementation

To create your own IllegalBundleRequestHandler, you must create a class which implements IllegalBundleRequestHandler.

Then in the Jawr configuration file you should add the following
setting:


    jawr.strict.mode=true
    jawr.illegal.bundle.request.handler=com.mycomp.jawr.MyIllegalBundleRequestHandler       


Here **com.mycomp.jawr.MyIllegalBundleRequestHandler** is the class name of your IllegalBundleRequestHandler
