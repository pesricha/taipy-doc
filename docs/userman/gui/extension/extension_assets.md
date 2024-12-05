# Accessing the library assets

In certain scenarios, you might want to enrich your user interface by displaying a small image alongside text.
For example, when using a caption control to represent a company name along with its logo,
adding an image can enhance visual context and usability.

Traditionally, in HTML, you would use an img tag with the src attribute pointing to the image’s file path.
However, directly referencing resources in this way can expose your application to potential security vulnerabilities,
such as unauthorized access or malicious resource requests.

To mitigate these risks, Taipy introduces the ElementLibrary.get_resource() method.
This method acts as a secure gateway for resource handling,
allowing the application to validate and filter resource requests based on predefined settings.
It ensures that only authorized and properly configured files are served, protecting your application while maintaining
functionality.

## Declaring element {data-source="gui:doc/extension/example_library/example_library.py#L62"}

```py title="example_library.py"
import base64

from taipy.gui.extension import Element, ElementLibrary, ElementProperty, PropertyType


class ExampleLibrary(ElementLibrary):
    def __init__(self) -> None:
        # Initialize the set of visual elements for this extension library
        logo_path = self.get_resource("assets/logo.png")
        with open(logo_path, "rb") as f:
            logo_base64 = base64.b64encode(f.read()).decode("utf-8")

        self.elements = {
            "logo_with_text": Element(
                "text",
                {
                    "text": ElementProperty(PropertyType.string),
                    "logo_path": ElementProperty(PropertyType.string, default_value=logo_base64),
                },
            )
        }
```
