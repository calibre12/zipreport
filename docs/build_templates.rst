.. _build_templates:

Building Templates
==================

Template design considerations
------------------------------

Typical report template design will require external resources, such as Javascript libraries and web fonts. These resources
can be added locally or referenced externally from eg. CDNs, as any regular web page. However, keep in mind that referencing
external resources may pose a security risk, and requires internet access when generating PDFs.

Having these resources locally will reduce the PDF rendering time and minimize or eliminate the need to have internet access,
but will increase the packaged report size and the effort required to maintain reports. Some assets may also have licensing
limitations that prevent usage and redistribution in a packaged format.

The recommended approach depends on the use case. Reports where PDF rendering time is not critical can reference resources
externally as needed; Reports where rendering time is critical (such as generated in response to a user interaction) should
have these resources referenced locally, as much as possible.


ZipReport Jinja considerations
______________________________

ZipReport allows most Jinja features, including adding your own filters. However, you can only import templates within
your report folder path (including subfolders). Adding extensions to Jinja is also supported, when using the JinjaRender
class directly. It works by instantiating :class:`zipreport.template.JinjaRender<zipreport.template.jinjarender.JinjaRender>` with
the required options:

.. code-block:: python

    from zipreport.report import ReportFileLoader
    from zipreport.template import JinjaRender

    # load report from file
    zpt = ReportFileLoader.load("simple.zpt")

    # extensions for jinja usage
    jinja_options = {
        "extensions": ['jinja2.ext.i18n'],
    }

    # initialize JinjaRender() with the desired options
    renderer = JinjaRender(zpt, jinja_options)


Waiting for Javascript execution before rendering
-------------------------------------------------

Some reports may contain complex Javascript logic for page composition, and timing the finalization of these operations
are a challenge. ZipReport provides a more reliable alternative to relying on waiting a predefined amount of time before triggering the render -
a Javascript event notification system that explicitly notifies the Electron application that client-side rendering is
finalized and PDF generation can be done.

The notification mechanism works by dispatching an event named 'zpt-view-ready'. As an example, this can be used to signal
paged.js end of operations:

.. code-block:: html

    <!DOCTYPE html PUBLIC>
    <html lang="en" lang="en">
      <head>
        <script src="https://s3.amazonaws.com/pagedmedia/pagedjs/dist/paged.js"></script>
        <script>
            // custom paged.js handler
            class handlers extends Paged.Handler {
                constructor(chunker, polisher, caller) {
                    super(chunker, polisher, caller);
                }

                afterPreview(pages) {
                    // dispatch event signaling readiness for PDF generation
                    document.dispatchEvent(new Event('zpt-view-ready'))
                }
            }

            // Register handler on paged.js library
            Paged.registerHandlers(handlers);
        </script>
      </head>
    <body>


Generating images dynamically
-----------------------------

ZipReport provides several filters that allow the generation and inclusion of images generated in runtime, without requiring
client-side composition. This is quite useful to add data-driven graphics to the reports, both for PDF and MIME generation.


Bundled Jinja tags for images
_____________________________



Using placeholders for previewing purposes
__________________________________________


Page numbers, headers and footers
---------------------------------

Generating table of contents
----------------------------

